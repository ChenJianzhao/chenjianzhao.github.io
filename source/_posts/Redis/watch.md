市场在重负载的情况下运行30秒的性能

又 Redis乐观锁的性能测试

## 1. 1个卖家，1个买家

### 1.1 数据结构和初始数据

```shell
// 在redis客户端，初始化单个卖家，单个买家数据结构
del market:
del inventory:37
del inventory:47
del users:37
del users:47

hmset users:37 name Seller funds 2000000
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

		Date initStart = new Date(System.currentTimeMillis());
		System.out.println("init start: \t" + format.format(initStart));
		initData();
		Date initEnd = new Date(System.currentTimeMillis());
		System.out.println("init end: \t" + format.format(initEnd));
		System.out.println("init cost: \t" + (initEnd.getTime()-initStart.getTime()) + "ms");
		System.out.println("");

		// 一个卖家
		new Thread(new Seller("37")).start();
      	 // 一个买家
		new Thread(new Buyer("47", "37")).start();
	}
  
 	 /**
	 * 初始化单个卖家商品数据
	 */
	public static void initData() {
		
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
						retryCounter++;
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
//					retryCounter++;
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





## 2. 多个卖家，一个卖家

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

		Date initStart = new Date(System.currentTimeMillis());
		System.out.println("init start: \t" + format.format(initStart));
		initBatchData();
		Date initEnd = new Date(System.currentTimeMillis());
		System.out.println("init end: \t" + format.format(initEnd));
		System.out.println("init cost: \t" + (initEnd.getTime()-initStart.getTime()) + "ms");
		System.out.println("");
		
      	
		new Thread(new Seller("35"), "Seller-1").start();
     	 // 让第一个卖家线程跑一阵，因为买家线程只购买第一个买家的商品
		try {
			Thread.sleep(100);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		new Thread(new Seller("36")).start();
		new Thread(new Seller("37")).start();
		new Thread(new Seller("38")).start();
		new Thread(new Seller("39")).start();

		new Thread(new Buyer("47", "37")).start();
	}

	/**
	 * 初始化多个卖家商品数据
	 */
	public static void initBatchData() {
		
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



## 3. 结果对比分析

| 30秒性能对比   | 上架商品数量 | 买入商品数量 | 购买重试次数 | 每次购买的平均等待时间 |
| --------- | ------ | ------ | ------ | ----------- |
| 1个卖家，1个买家 | 211883 | 44232  | 44291  | 0ms         |
| 5个卖家，1个买家 | 99064  | 2012   | 30461  | 14ms        |



注：数据和 《Redis实战》中给出的测试数据有出入