---
categories: Redis
date: 2017-07-06 12:00
title: Redis 之 分布式锁（未完成）
---

分布式锁
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
    
	public static String acquireLock(Jedis conn, String lockname, int acquireTimeout, int lockTimeout) {
		
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
}

```



## 3. 结果对比分析

| 30秒性能对比       | 上架商品数量 | 买入商品数量 | 购买重试次数 | 每次购买的平均等待时间 |
| ------------- | ------ | ------ | ------ | ----------- |
| 1个卖家，1个买家     | 211883 | 44232  | 44291  | 0ms         |
| 5个卖家，1个买家     | 99064  | 2012   | 30461  | 14ms        |
| 5个卖家，5个买家     | 156291 | 352    | 17784  | 85ms        |
| 1个卖家，1个买家，简单锁 | 206135 | 30306  | 0      | 1ms         |
| 5个卖家，1个买家，简单锁 |        |        |        |             |
|               |        |        |        |             |
|               |        |        |        |             |