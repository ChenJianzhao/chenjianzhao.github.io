---
categories: Redis
date: 2017-07-05 20:00
title: Redis 之 分布式锁(一)
---



## 背景

一般来说，在对数据进行“加锁”时，程序首先需要通过获取（acquire）锁来得到对数据进行排他性访问的能力，然后才能对数据执行一系列操作，最后还要将锁释放（release）给其他程序。对于能够被多个线程访问的 **共享内存数据结构（shared-memory data structure）** 来说，这种“先获取锁，然后执行操作，最后释放锁”的动作非常常见。

Redis 使用 `WATCH` 命令来代替对数据进行加锁，因为 WATCH 只会在数据被其他客户端抢先修改了的情况下通知执行了这个命令的客户端，而不会阻止其他客户端对数据进行修改，所以这个命令被称为 **乐观锁（optimistic locking）**。

**分布式锁**也有类似上面描述的动作，但这种锁既不是给同一个进程中的多个线程使用，也不是给同一台机器上的多个进程使用，而是由不同机器上的不同 Redis 客户端进行获取和释放的。

何时使用以及是否使用 WATCH 或者锁取决于 给定的应用程序。

下面会说明“为什么使用 WATCH 命令来监视被频繁访问的键可能引起性能问题”，还会展示构建一个锁的详细步骤，并最终在某些情况下使用锁去代替 WATCH 命令。

<!-- more -->



**Redis 自带乐观锁 Watch 的使用和高负载情况下的性能分析。（市场在重负载的情况下运行30秒的性能）**

为了展示锁对于性能拓展的必要性，我们会模拟市场在3种不同负载情况下的性能表现，这3种情况分别是：

- 1个玩家出售商品，另1个玩家购买商品
- 5个玩家出售商品，另1个玩家购买商品
- 5个玩家出售商品，另外5个玩家购买商品




## **数据结构**


程序分别使用散列（Hash）、集合（Set）和有序集合（ZSet）来表示用户信息（users）和用户包裹（inventory）的结构：

- 用户信息存储在一个**散列**里面，散列的各个键值对分别记录用户的姓名、用户拥有的钱数等属性。


- 用户包裹使用一个**集合**来表示，它记录了包裹里面每件商品的唯一编号。
- 为了将被销售的商品的全部信息都存储在市场里面，我们将**商品的ID和卖家的ID**拼接起来，并将拼接的结果用作成员存储到市场有序集合（ZSET）里面，而商品的售价则用作成员的分值。




| market: | zseet |
| ------- | ----- |
| ItemA.4 | 35    |
| ItemC.7 | 48    |
| ItemE.2 | 60    |
| ItemG.3 | 73    |



| users:17 | hash  |
| -------- | ----- |
| name     | Frank |
| funds    | 43    |



| users:27 | hash |
| -------- | ---- |
| name     | Bill |
| funds    | 125  |



| inventory:17 | set  |
| ------------ | ---- |
| ItemL        |      |
| ItemM        |      |
| ItemN        |      |



| inventory:27 | Set  |
| ------------ | ---- |
| ItemO        |      |
| ItemP        |      |
| ItemQ        |      |



## 1. 1个卖家，1个买家



### 1.1 数据结构和初始数据

```shell
// 在redis客户端，初始化单个卖家，单个买家数据结构
del market:
del inventory:37
del inventory:47
del users:37
del users:47

hmset users:37 name Seller funds 200000
hmset users:47 name Buyer funds 200000
```



### 1.2 主类

```java
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.List;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Pipeline;

public class SimpleMarketDemo {


	public static void main(String[] args) {

		SimpleDateFormat format = new SimpleDateFormat("yyyy:MM:dd-hh:mm:ss");

		initData(format);
		
		runSellerAndBuy1();
	}

	
	/**
	 * 初始化单个卖家商品数据
	 */
	public static void initData(SimpleDateFormat format) {
		
		Date initStart = new Date(System.currentTimeMillis());
		System.out.println("init start: \t" + format.format(initStart));
		
		Jedis conn = null;
		String seller = "inventory:37";
		int initCount = 300000;

		try{
			conn = new Jedis("localhost");
			Pipeline pipe = conn.pipelined();

				pipe.multi();
				for(int itemid = 0;itemid<initCount; itemid++) {
					pipe.sadd(seller, String.valueOf(itemid));
				}
				pipe.exec();
				pipe.sync();
				System.out.println(seller + " init count: \t" + conn.scard(seller));
		
			
		}catch(Exception e){
			e.printStackTrace();
		}finally{
			conn.disconnect();
		}
		
		Date initEnd = new Date(System.currentTimeMillis());
		System.out.println("init end: \t" + format.format(initEnd));
		System.out.println("init cost: \t" + (initEnd.getTime()-initStart.getTime()) + "ms");
		System.out.println("");
	}

	/**
	 * 1个卖家，1个买家
	 */
	protected static void runSellerAndBuy1() {
		new Thread(new Seller("37",true)).start();
//		try {
//			Thread.sleep(100);
//		} catch (InterruptedException e) {
//			e.printStackTrace();
//		}
		new Thread(new Buyer("47", "37",true)).start();
	}

  ......
  ......
}
```



### **1.3 卖家内部类**

```java
/**
 * 卖家
 * @author pc
 *
 */
static class Seller implements Runnable {

  protected int retryCounter = 0;
  int buyCount = 2000000;
  String  sellerid = "";

  SimpleDateFormat format = new SimpleDateFormat("yyyy:MM:dd-hh:mm:ss");

  public Seller(String sellerid) {
    this.sellerid = sellerid;
  }

  public void run() {

    Jedis conn = new Jedis("localhost");

    try{
      Date sellStart = new Date(System.currentTimeMillis());
      System.out.println("seller:" + sellerid + " sell start: \t" + format.format(sellStart));

      Calendar cal = Calendar.getInstance();
      cal.add(Calendar.SECOND, 30);

      // 测试30秒内卖出的次数
      for(int itemid = 0; itemid<buyCount; itemid++) {
        listItem(conn,itemid+"",sellerid,1);
        if( System.currentTimeMillis() >= cal.getTimeInMillis() )
          break;
      }

      Date sellEnd = new Date(System.currentTimeMillis());
      synchronized (Seller.class) {
        System.out.println("seller:" + sellerid + " sell end: \t" + format.format(sellEnd));
        System.out.println("seller:" + sellerid + " sell cost: \t" + (sellEnd.getTime()-sellStart.getTime()) + "ms");
        long count = conn.scard("inventory:" + sellerid);
        System.out.println("seller:" + sellerid + " sell Count: \t" +  count);
        System.out.println("seller:" + sellerid + " avg sell cost: \t" + (sellEnd.getTime()-sellStart.getTime())/count + "ms");
        System.out.println("seller:" + sellerid + " sell retryCounter: \t" +  retryCounter);
        System.out.println("");
      }
    }finally{
      conn.disconnect();
    }

  }

  /**
	 * 卖家上架商品
	 * @param conn
	 * @param itemid
	 * @param sellerid
	 * @param price
	 * @return
	 */
  public  boolean  listItem(Jedis conn, String itemid, String sellerid, int price) {
    String inventory ="inventory:" + sellerid;
    String item = itemid + "." + sellerid;

    // 设置重试超时时间为5s
    Calendar cal = Calendar.getInstance();
    cal.add(Calendar.SECOND, 5);
    Date timeout = cal.getTime();

    Pipeline pipe = conn.pipelined();

    while(System.currentTimeMillis() < timeout.getTime()){

      try{
        pipe.watch(inventory);
        pipe.sismember(inventory, itemid);
        List<Object> result = pipe.syncAndReturnAll();
        if(!(Boolean)result.get(1)) {
          retryCounter++;	// 记录重试次数
          continue;
        }else {
          pipe.multi();
          pipe.zadd("market:", price, item);
          pipe.srem(inventory, itemid);
          pipe.exec();
          pipe.syncAndReturnAll();
          return true;
        }

      }catch(Exception e) {
        // retry
        // retryCounter++;
        continue;
      }finally{
      }
    }

    return false;
  }
}
```



### 1.4 买家内部类

```java
/**
 * 买家
 * @author pc
 *
 */
static class Buyer implements Runnable {

  protected int retryCounter = 0;
  int buyCount = 2000000;
  String buyerid = "";
  String sellerid = "";

  SimpleDateFormat format = new SimpleDateFormat("yyyy:MM:dd-hh:mm:ss");

  public  Buyer(String buyerid , String sellerid) {
    this.buyerid = buyerid;
    this.sellerid = sellerid;
  }

  public void run() {
    Jedis conn = new Jedis("localhost");

    try{
      Date buyStart = new Date(System.currentTimeMillis());
      System.out.println("buyer:" + buyerid + " buy start: \t" + format.format(buyStart));

      Calendar cal = Calendar.getInstance();
      cal.add(Calendar.SECOND, 30);

      // 测试30秒内购买的次数
      for(int itemid = 0;itemid<buyCount; itemid++) {
        purchaseItem(conn,buyerid,itemid+"",sellerid,1);
        if( System.currentTimeMillis() >= cal.getTimeInMillis() )
          break;
      }

      synchronized (Buyer.class) {
        Date buyEnd = new Date(System.currentTimeMillis());
        System.out.println("buyer:" + buyerid + " buy end: \t" + format.format(buyEnd));
        System.out.println("buyer:" + buyerid + " buy cost: \t" + (buyEnd.getTime()-buyStart.getTime()) + "ms");
        long count  = conn.scard("inventory:" + buyerid);
        System.out.println("buyer:" + buyerid + " buy Count: \t" +  count);
        System.out.println("buyer:" + buyerid + " avg buy cost: \t" + (buyEnd.getTime()-buyStart.getTime())/count + "ms");
        System.out.println("buyer:" + buyerid + " buy retryCounter: \t" +  retryCounter);
        System.out.println("");
      }
    }finally{
      conn.disconnect();
    }
  }


  /**
	 * 买家购买商品
	 * @param conn
	 * @param buyerid
	 * @param itemid
	 * @param sellerid
	 * @param lprice
	 * @return
	 */
  public boolean purchaseItem(Jedis conn, String buyerid, String itemid, String sellerid, int lprice) {
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

      try{
        pipe.watch(market, inventory, seller, buyer);
        pipe.zscore(market, item);
        pipe.hget(buyer,"funds");
        List<Object> check = pipe.syncAndReturnAll();
        Double price = (Double)check.get(1);
        int funds = Integer.parseInt((String)check.get(2));

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
      }
    }

    return false;
  }
}
```



### 1.5 执行结果

```
init start: 	2017:07:05-03:26:21
inventory:37 init count: 	300000
init end: 	2017:07:05-03:26:25
init cost: 	4055ms

seller:37 sell start: 	2017:07:05-03:26:25
buyer:47 buy start: 	2017:07:05-03:26:25

seller:37 sell end: 	2017:07:05-03:26:55
seller:37 sell cost: 	30001ms
seller:37 sell Count: 	211883
seller:37 avg sell cost: 	0ms
seller:37 sell retryCounter: 	0

buyer:47 buy end: 	2017:07:05-03:26:55
buyer:47 buy cost: 	30000ms
buyer:47 buy Count: 	44232
buyer:47 avg buy cost: 	0ms
buyer:47 buy retryCounter: 	44291
```





## 2. 5个卖家，1个卖家

### 2.1 数据结构和初始数据

```
del market:
del inventory:35
del inventory:36
del inventory:37
del inventory:38
del inventory:39
del inventory:47
del users:35
del users:36
del users:37
del users:38
del users:39
del users:47

hmset users:35 name Seller funds 2000000
hmset users:36 name Seller funds 2000000
hmset users:37 name Seller funds 2000000
hmset users:38 name Seller funds 2000000
hmset users:39 name Seller funds 2000000
hmset users:47 name Buyer funds 200000
```



### 2.2  主类

```java
	public static void main(String[] args) {

		SimpleDateFormat format = new SimpleDateFormat("yyyy:MM:dd-hh:mm:ss");

		initBatchData(format);
		
		runSellerAndBuy2();
	}
	
	/**
	 * 初始化多个卖家商品数据
	 */
	public static void initBatchData(SimpleDateFormat format) {
		
		Date initStart = new Date(System.currentTimeMillis());
		System.out.println("init start: \t" + format.format(initStart));
		
		Jedis conn = null;
		int initCount = 50000;

		try{
			conn = new Jedis("localhost");
			Pipeline pipe = conn.pipelined();

			for(int i=5; i<=9; i++){
				String seller = "inventory:3" + String.valueOf(i); 
				pipe.multi();
				for(int itemid = 0;itemid<initCount; itemid++) {
					pipe.sadd(seller, String.valueOf(itemid));
				}
				pipe.exec();
				pipe.sync();
				System.out.println(seller + " init count: \t" + conn.scard(seller));
			}
		
			
		}catch(Exception e){
			e.printStackTrace();
		}finally{
			conn.disconnect();
		}
		
		Date initEnd = new Date(System.currentTimeMillis());
		System.out.println("init end: \t" + format.format(initEnd));
		System.out.println("init cost: \t" + (initEnd.getTime()-initStart.getTime()) + "ms");
		System.out.println("");
		
	}

	/**
	 * 5个卖家，1个买家
	 */
	protected static void runSellerAndBuy2() {

		new Thread(new Seller("35"), "Seller-1").start();
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		new Thread(new Seller("36")).start();
		new Thread(new Seller("37")).start();
		new Thread(new Seller("38")).start();
		new Thread(new Seller("39")).start();

		new Thread(new Buyer("47", "35",true)).start();
	}
```

### 2.3 执行结果

```
init start: 	2017:07:05-03:05:23
inventory:35 init count: 	50000
inventory:36 init count: 	50000
inventory:37 init count: 	50000
inventory:38 init count: 	50000
inventory:39 init count: 	50000
init end: 	2017:07:05-03:05:25
init cost: 	2305ms

seller:35 sell start: 	2017:07:05-03:05:25
seller:36 sell start: 	2017:07:05-03:05:25
seller:38 sell start: 	2017:07:05-03:05:25
buy start: 	2017:07:05-03:05:25
seller:37 sell start: 	2017:07:05-03:05:25
seller:39 sell start: 	2017:07:05-03:05:25

seller:35 sell end: 	2017:07:05-03:05:55
seller:35 sell cost: 	30000ms
seller:35 sell Count: 	21153
seller:35 avg sell cost: 	1ms
seller:35 sell retryCounter: 	0

seller:36 sell end: 	2017:07:05-03:05:55
seller:36 sell cost: 	30002ms
seller:36 sell Count: 	17563
seller:36 avg sell cost: 	1ms
seller:36 sell retryCounter: 	0

seller:38 sell end: 	2017:07:05-03:05:55
seller:38 sell cost: 	30001ms
seller:38 sell Count: 	17593
seller:38 avg sell cost: 	1ms
seller:38 sell retryCounter: 	0

47 buy end: 	2017:07:05-03:05:55
47 buy cost: 	30011ms
47 buy Count: 	2012
47 avg buy cost: 	14ms
47 buy retryCounter: 	30461

seller:37 sell end: 	2017:07:05-03:05:55
seller:37 sell cost: 	30016ms
seller:37 sell Count: 	21392
seller:37 avg sell cost: 	1ms
seller:37 sell retryCounter: 	0

seller:39 sell end: 	2017:07:05-03:05:55
seller:39 sell cost: 	30000ms
seller:39 sell Count: 	21363
seller:39 avg sell cost: 	1ms
seller:39 sell retryCounter: 	0
```



## 3. 5个卖家，5个买家

### 3.1 数据结构和初始数据

```
## 5个卖家，5个买家

del market:
del inventory:35
del inventory:36
del inventory:37
del inventory:38
del inventory:39
del inventory:45
del inventory:46
del inventory:47
del inventory:48
del inventory:49
del users:35
del users:36
del users:37
del users:38
del users:39
del users:45
del users:46
del users:47
del users:48
del users:49

hmset users:35 name Seller funds 2000000
hmset users:36 name Seller funds 2000000
hmset users:37 name Seller funds 2000000
hmset users:38 name Seller funds 2000000
hmset users:39 name Seller funds 2000000
hmset users:45 name Buyer funds 200000
hmset users:46 name Buyer funds 200000
hmset users:47 name Buyer funds 200000
hmset users:48 name Buyer funds 200000
hmset users:49 name Buyer funds 200000

```



### 3.2 主类

```java
public static void main(String[] args) {

		SimpleDateFormat format = new SimpleDateFormat("yyyy:MM:dd-hh:mm:ss");

		initBatchData(format);
		
		runSellerAndBuy3();
	}


	/**
	 * 5个卖家，5个买家(全部购买第一个买家的商品)
	 */
	protected static void runSellerAndBuy3() {
		
		new Thread(new Seller("35"), "Seller-1").start();
		new Thread(new Seller("36")).start();
		new Thread(new Seller("37")).start();
		new Thread(new Seller("38")).start();
		new Thread(new Seller("39")).start();

		new Thread(new Buyer("45", "35")).start();
		new Thread(new Buyer("46", "35")).start();
		new Thread(new Buyer("47", "35")).start();
		new Thread(new Buyer("48", "35")).start();
		new Thread(new Buyer("49", "35")).start();
	}
```



### 3.3 执行结果

```
init start: 	2017:07:06-10:10:03
inventory:35 init count: 	50000
inventory:36 init count: 	50000
inventory:37 init count: 	50000
inventory:38 init count: 	50000
inventory:39 init count: 	50000
init end: 	2017:07:06-10:10:05
init cost: 	1958ms

seller:35 sell start: 	2017:07:06-10:10:05
seller:37 sell start: 	2017:07:06-10:10:06
seller:39 sell start: 	2017:07:06-10:10:06
buyer:46 buy start: 	2017:07:06-10:10:06
seller:36 sell start: 	2017:07:06-10:10:06
seller:38 sell start: 	2017:07:06-10:10:06
buyer:45 buy start: 	2017:07:06-10:10:06
buyer:47 buy start: 	2017:07:06-10:10:06
buyer:49 buy start: 	2017:07:06-10:10:06
buyer:48 buy start: 	2017:07:06-10:10:06

seller:35 sell end: 	2017:07:06-10:10:35
seller:35 sell cost: 	30001ms
seller:35 sell Count: 	31550
seller:35 avg sell cost: 	0ms
seller:35 sell retryCounter: 	0

seller:37 sell end: 	2017:07:06-10:10:36
seller:37 sell cost: 	30002ms
seller:37 sell Count: 	31837
seller:37 avg sell cost: 	0ms
seller:37 sell retryCounter: 	0

seller:39 sell end: 	2017:07:06-10:10:36
seller:39 sell cost: 	30000ms
seller:39 sell Count: 	31893
seller:39 avg sell cost: 	0ms
seller:39 sell retryCounter: 	0

buyer:46 buy end: 	2017:07:06-10:10:36
buyer:46 buy cost: 	30000ms
buyer:46 buy Count: 	0
buyer:46 buy retryCounter: 	12

seller:36 sell end: 	2017:07:06-10:10:36
seller:36 sell cost: 	30009ms
seller:36 sell Count: 	30518
seller:36 avg sell cost: 	0ms
seller:36 sell retryCounter: 	0

buyer:47 buy end: 	2017:07:06-10:10:36
buyer:47 buy cost: 	30001ms
buyer:47 buy Count: 	0
buyer:47 buy retryCounter: 	10

seller:38 sell end: 	2017:07:06-10:10:36
seller:38 sell cost: 	30004ms
seller:38 sell Count: 	30493
seller:38 avg sell cost: 	0ms
seller:38 sell retryCounter: 	0

buyer:45 buy end: 	2017:07:06-10:10:36
buyer:45 buy cost: 	30003ms
buyer:45 buy Count: 	759
buyer:45 avg buy cost: 	39ms
buyer:45 buy retryCounter: 	18763

buyer:49 buy end: 	2017:07:06-10:10:36
buyer:49 buy cost: 	30002ms
buyer:49 buy Count: 	0
buyer:49 buy retryCounter: 	9

buyer:48 buy end: 	2017:07:06-10:10:36
buyer:48 buy cost: 	30000ms
buyer:48 buy Count: 	0
buyer:48 buy retryCounter: 	7
```



## 4. 结果对比分析



| 30秒性能对比   | 上架商品数量 | 买入商品数量 | 购买重试次数 | 每次购买的平均等待时间 |
| :-------- | ------ | ------ | ------ | ----------- |
| 1个卖家，1个买家 | 211883 | 44232  | 44291  | 0ms         |
| 5个卖家，1个买家 | 99064  | 2012   | 30461  | 14ms        |
| 5个卖家，5个买家 | 156291 | 352    | 17784  | 85ms        |

根据上表模拟的结果显示，WATCH、MULTI 和 EXEC 组成的事务并不具有可拓展性，因为程序在尝试完成一个事务的时候，可能会因为事务执行失败而反复地进行重试。

保证数据的正确是一件非常重要的事情，但使用 WATCH 命令的做法并不完美。我们将用锁来解决这个问题。

**注：数据和 《Redis实战》中给出的测试数据有出入**