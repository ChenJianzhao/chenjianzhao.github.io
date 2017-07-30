---
categories: Redis
date: 2017-07-7 12:00
title: Redis 之 信号量
---



计数信号量是一种锁，它可以让用户限制一项资源最多能够同时被多少个进程访问，通常用于限定同时使用的资源数量。

计数信号量和其他锁的区别在于，当客户端获取锁失败的时候，通常会选择等待；而当客户端获取计数信号量失败的时候，客户端通常会选择立即返回失败结果。

<!-- more -->

多进程情况下，一般都需要保证信号量的公平和数量的准确，所以以下直接给出带锁的公平信号量



# 1 公平信号量

- 为了尽可能地减少系统时间不一致带来的问题，我们需要给信号量实现添加一个计数器以及一个有序集合。其中，计数器创建出一种类似于计时器（Timer）的机制，确保最先对计数器执行自增操作的客户端能够获得信号量。


- 另外，为了满足这一要求，程序会将计数器生成的值用作分值，存储到一个”信号量拥有者“的有序集合里面，然后通过检查客户端生成的标识符在有序集合中的排名来判断客户端是否取得了信号量。
- 通过从“系统时间”有序集合里面移除过期元素的方式来清理过期信号量。
- 另外，公平信号量实现还会通过 `ZINTERSTORE` 命令以及该命令的 `WEIGHTS` （权重）参数，将信号量的超时时间传递给新的信号量拥有者有序集合。



## 1.1 获取信号量

- 程序首先通过对超时有序集合里面移除过期元素的方式来移除超时的信号量，
- 接着对超时有序集合和信号量拥有者有序集合执行交集计算，并将计算结果保存到信号量拥有者的有序集合里面，覆盖有序集合中原有的数据。
- 之后，程序会对计数器执行自增操作，并向计数器生成的值添加到信号量拥有者有序集合里面；
- 与此同时，系统还会将当前的系统时间添加到超时有序集合里面。
- 在完成以上操作之后，程序会检查当前客户端添加的标识符在信号量拥有者有序集合中的排名是否足够靠前，如果是的话就表示客户端成功取得 了信号量。相反地，如果客户端未能取得信号量，那么程序将从信号量拥有者有序集合以及超时有序集合里面移除与该客户端有关的元素。



**代码清单 acquireFairSemaphore() 函数**

```java
	/**
	 * 获取公平信号量
	 * @param conn
	 * @param semname
	 * @param limit
	 * @param timeout
	 * @return
	 */
	public static String acquireFairSemaphore(Jedis conn, String semname, Integer limit, Integer timeout) {
		
		if( timeout == null ) 
			timeout = 10;
		if(limit==null)
			limit = 5;
			
		String identifier = UUID.randomUUID().toString();
		String czset = semname + ":owner";
		String ctr = semname + ":counter";
		
		long now = System.currentTimeMillis();
		Pipeline pipe = conn.pipelined();
		pipe.multi();
		
		// 移除过期信号量
		Calendar cal = Calendar.getInstance();
		cal.add(Calendar.SECOND, -timeout);
		pipe.zremrangeByScore(semname, 0, (double)cal.getTimeInMillis());
		ZParams param = new ZParams();
		param.weightsByDouble(1,0); 	// 信号量拥有者 权重1，过期集合 0
		pipe.zinterstore(czset,param,czset,semname); // 同步更新信号量拥有者 
		
		// 计数器自增
		pipe.incr(ctr);
		
		pipe.exec();
		List<Object> counterResult = pipe.syncAndReturnAll();
		long counter = (Long)((List)counterResult.get(4)).get(2);

		pipe.multi();
		// 尝试获取信号量
		pipe.zadd(semname, System.currentTimeMillis(), identifier);
		pipe.zadd(czset, counter, identifier);
		
		// 检查排名来判断客户端是否取得信号量
		pipe.zrank(czset, identifier);
		pipe.exec();
		
		List<Object> result = pipe.syncAndReturnAll();
		long rank = (Long)((List)result.get(4)).get(2);
		
		if(rank<limit){
			return identifier;
		}else{
			pipe.multi();
			pipe.zrem(semname, identifier);
			pipe.zrem(czset, identifier);
			pipe.exec();
			pipe.sync();
			return null;
		}
	}

	/**
	 * 带锁的信号量请求
	 * @param conn
	 * @param semname
	 * @param limit
	 * @param timeout
	 * @return
	 */
	public static String acquireFairSemaphoreWithLock(Jedis conn, String semname, Integer limit, Integer timeout) {
		
		String identifier = LockUtil.acquireLock(conn, semname, 3, 5);
		if( identifier!=null ) {
			try{
				return acquireFairSemaphore(conn, semname, limit, timeout);
			}finally {
				LockUtil.releaseLock(conn, semname, identifier);
			}
		}
		return null;
	}
```



## 1.2 释放信号量	

公平信号量的的释放很简单，释放需要同时从信号量拥有者有序集合以及超时有序集合里面删除当前客户端的标识符。

**代码清单 releaseFairSemaphore() 函数**

```java
	/**
	 * 释放信号量
	 * @param conn
	 * @param semname
	 * @param identifier
	 * @return
	 */
	public static boolean releaseFairSemaphore(Jedis conn, String semname, String identifier) {
		Pipeline pipe = conn.pipelined();
		pipe.multi();
		pipe.zrem(semname, identifier);
		pipe.zrem(semname + ":owner", identifier);
		pipe.exec();
		List<Object> result = pipe.syncAndReturnAll();
		if(result.get(3)!=null)
			return true;
		else return false;
	}
```



## 1.3 刷新信号量

- 前面介绍的信号量实现默认只能设置10秒的超时时间，它主要用于实现超时限制特性并掩盖自身包含的潜在缺陷，但是短短的10秒时间对于 长时间的流API的使用者来说是远远不够的，因此我们需要想办法对信号量进行刷新，防止其过期。
- 因为公平信号量区分开了超时有序集合和信号量拥有者有序集合，所以程序只需要对超时有序集合进行更新，就可以立即刷新信号量的超时时间了。

**代码清单 refreshFairSemaphore() 函数**

```java
	/**
	 * 刷新信号量
	 * @param conn
	 * @param semname
	 * @param identifier
	 * @return
	 */
	public static boolean refreshFairSemaphore(Jedis conn, String semname, String identifier) {
		// 添加成功，客户端已失去信号量
		if( conn.zadd(semname, (double)System.currentTimeMillis(), identifier) == 1) {
			releaseFairSemaphore(conn, semname, identifier);
			return false;
		}else{
			// 添加失败，更新了 键的分值
			return true;
		}
	}
```

只要客户端持有的信号量没有因为过期而被删除，refreshFairSemaphore() 函数就可以对信号量的超时时间进行刷新。另一方面，如果客户端持有的信号量已经因为超时而被删除，那么函数将释放信号量，并将信号量已经丢失的信号告诉调用者。在长时间使用信号量的时候，我们必须以足够频繁的频率对信号量进行刷新，防止它因为过期而丢失。