---
categories: Redis
date: 2017-07-7 12:00
title: 信号量
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

