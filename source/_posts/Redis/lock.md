---
categories: Redis
date: 2017-07-06 12:00
title: Redis 之 分布式锁（二）
---



根据上一节对 WATCH 高负载情况下的测试结果显示，WATCH、MULTI 和 EXEC 组成的事务并不具有可拓展性，因为程序在尝试完成一个事务的时候，可能会因为事务执行失败而反复地进行重试。保证数据的正确是一件非常重要的事情，但使用 WATCH 命令的做法并不完美。

为了解决这个问题，并以可拓展的方式来处理市场交易，我们将使用锁来保证市场在任一时刻只能上架或者销售一件商品。

<!-- more -->

虽然很多Redis用户都对锁（lock）、加锁（locking）及锁超时（lock timeout）有所了解，但大部分Redis实现的锁只是基本上正确，它们发生故障的时间和方式通常难以预料。下面列出一些导致锁出现不正确行为的原因，以及锁在不正确运行时的症状。

- 持有锁的进程因为操作时间过长而导致锁被自动释放，但进程本身并不知晓这一点，甚至还可能会错误地释放掉其他进程持有的锁。
- 一个持有锁并打算执行长时间操作的进程已经崩溃，但其他想要获取锁的进程不知道哪个进程持有锁，也无法检测出持有锁的进程已经崩溃，只能拜拜浪费时间等待锁被释放。
- 在一个进程持有锁过期只有，其他多个进程同时尝试去获取锁，而且都获得了锁。
- 上面提到的第一种情况和第三种情况同时出现，导致有多个进程获得了锁，而每个进程都会以为自己是唯一一个获得锁的进程。




## 1.锁的获取

对数据进行排他性访问，程序首先要做的就是获取锁。`SETNX` 命令天生就适合用来实现所得获取功能，这个命令只会**在键不存在的情况下为键设置值**，而锁要做的就是将一个随机生成的128UUID设置为键的值，并使用这个值来防止锁被其他进程获取得。

如果程序尝试获取锁失的时候失败，那么它将不断进行重试，知道成功地取得锁或者超过给定的时限位置：



**代码清单 acquireLock() 函数**

```java

package org.demo.redisDemo.lock;

import java.util.Calendar;
import java.util.List;
import java.util.UUID;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

/**
 * Created by cjz on 2017/7/4.
 */
public class LockUtil {


	public static String acquireLock(Jedis conn, String lockname, int acquireTimeout) {

        // 标志本客户端唯一锁标志
        String identifier = UUID.randomUUID().toString();

        String lockkey = "lock:" + lockname;

        Calendar cal = Calendar.getInstance();
        cal.add(Calendar.SECOND, acquireTimeout);

        while( System.currentTimeMillis() <  cal.getTime().getTime()) {

            if(conn.setnx(lockkey, identifier) == 1) {
                return identifier;
            }

            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        return null;
    }
  
  	......
    ......
}
```



## 2.锁的应用

在实现了锁之后，我们就可以使用锁来代替针对市场的 WATCH 操作了，代码展示了使用锁重新实现商品购买的操作：程序首先对市场进行加锁，接着检查商品的价格，并在确保买家有足够的钱来购买商品之后，对钱和商品进行相应的转移。当操作完成之后，程序就会释放锁。



**代码清单  purchaseItemWithLock() 函数**

```java
public boolean purchaseItemWithLock(Jedis conn, String buyerid, String itemid, String sellerid, int lprice) {
		String inventory ="inventory:" + buyerid;
		String market = "market:";
		String item = itemid + "." + sellerid;
		String buyer = "users:" + buyerid;
		String seller = "users:" + sellerid;
		
		// 设置重试超时时间为5s
		Calendar cal = Calendar.getInstance();
		cal.add(Calendar.SECOND, 5);
		Date timeout = cal.getTime();
		
		Pipeline pipe = conn.pipelined();
		
		while(System.currentTimeMillis() < timeout.getTime()){
			
			/*
			 * 获得锁
			 */
			String lock = LockUtil.acquireLock(conn, market, 3, 5);
			if(lock==null)
				continue;
			
			try{
				
//				pipe.watch(market, buyer);
				pipe.zscore(market, item);
				pipe.hget(buyer,"funds");
				List<Object> check = pipe.syncAndReturnAll();
				Double price = (Double)check.get(0);
				int funds = Integer.parseInt((String)check.get(1));
				
				// 商品还未放入市场
				if( price == null || 
						price != null && (lprice!=price || price>funds)) {
					continue;
				}else {
					pipe.multi();
					pipe.hincrBy(seller, "funds", price.longValue());
					pipe.hincrBy(buyer, "funds", -price.longValue());
					pipe.sadd(inventory,itemid);
					pipe.zrem(market,item);
					pipe.exec();

					List<Object> result= pipe.syncAndReturnAll();
					// Watch 的键发生变化，同步返回值为null
					if(result.get(5)==null) {
						retryCounter++;
						continue;
					}
					else {
						return true;
					}
				}
				
			}catch(Exception e) {
				// retry
				retryCounter++;
			}finally{
				LockUtil.releaseLock(conn, market, lock);
			}
		}
		
		return false;
	}
```

程序中用到的锁是用来锁住市场数据的，它之所以会包围着购买操作的代码，是因为程序在操作市场数据期间必须一致持有锁。



## 3.锁的释放

代码清单 releaseLock() 函数展示了锁释放的操作：函数首先使用 WATCH 命令监视代表锁的键，接着检查键目前的值是否和加锁时设置的值相同，并在确认值没有变化之后删除该键（这个检查还可以防止程序错误地释放同一个锁多次）。

**代码清单 releaseLock() 函数**

```java
public static boolean releaseLock(Jedis conn, String lockname, String identifier) {
    	
    	Pipeline pipe = conn.pipelined();
    	String lockkey = "lock:" + lockname;
    	
    	while(true) {
    		try{
    			pipe.watch(lockkey);
    			pipe.get(lockkey);
    			List<Object> result = pipe.syncAndReturnAll();
    			if( result.get(1) !=null && result.get(1).equals(identifier)) {
    				pipe.multi();
    				pipe.del(lockkey);
    				pipe.exec();
    				List<Object> delResl = pipe.syncAndReturnAll();
    				if(delResl.get(2) == null)
    					continue;
    				else return true;
    			}else
    				return false;
    		}catch(Exception e) {
    			e.printStackTrace();
    		}
    	}
    }
```



## 4.带有超时限制特性的锁

前面提到过，目前的锁实现在持有者崩溃的时候不会自动释放锁，者将导致锁一直处于已被获取的状态。为了解决这个问题，我们将为锁加上超时的功能。

为了给锁机上超时的显示限制特性，程序将在获得锁之后，调用 `EXPIRE` 命令来为锁上设置过期时间，使得 Redis 可以自动删除超时的锁。为了确保在客户端已经崩溃的情况下任然能够自动被释放，客户端会在尝试获取锁失败之后，检查锁的超时时间，并为未设置超时时间的锁设置超时时间。因此锁总会带有超时时间，并最终因为超时而自动被释放，使得其他客户端您可以继续尝试获取已被释放的锁。



**代码清单 acquireLockWithTimeout() 函数**

```java
public static String acquireLockWithTimeout(Jedis conn, String lockname, int acquireTimeout, int lockTimeout) {
		
		// 标志本客户端唯一锁标志
        String identifier = UUID.randomUUID().toString();

        String lockkey = "lock:" + lockname;

        Calendar acquireCal = Calendar.getInstance();
        acquireCal.add(Calendar.SECOND, acquireTimeout);
        
        Calendar lockCal = Calendar.getInstance();
        lockCal.add(Calendar.SECOND, lockTimeout);

        while( System.currentTimeMillis() < acquireCal.getTime().getTime()) {

            if(conn.setnx(lockkey, identifier) == 1) {
            	// 设置过期时间
            	conn.expire(lockkey, lockTimeout);
                return identifier;
            }else if ( conn.ttl(lockkey) == null ){
            	conn.expire(lockkey, lockTimeout);
            }

            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }

        return null;
	}
```





## 5.结果对比分析（数据与预期不符，未完成）

| 30秒性能对比       | 上架商品数量 | 买入商品数量 | 购买重试次数 | 每次购买的平均等待时间 |
| ------------- | ------ | ------ | ------ | ----------- |
| 1个卖家，1个买家     | 211883 | 44232  | 44291  | 0ms         |
| 5个卖家，1个买家     | 99064  | 2012   | 30461  | 14ms        |
| 5个卖家，5个买家     | 156291 | 352    | 17784  | 85ms        |
| 1个卖家，1个买家，简单锁 | 206135 | 30306  | 0      | 1ms         |
| 5个卖家，1个买家，简单锁 |        |        |        |             |
| 5个卖家，5个买家，简单锁 |        |        |        |             |



**注：数据和 《Redis实战》中给出的测试数据有出入**