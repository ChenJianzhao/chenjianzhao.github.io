---
categories: Redis
date: 2017-07-09 20:00
title: 任务队列
---

在处理Web客户端发送的命令请求是，某些操作的执行时间可能会比我们预期的更长一些。通过将待执行任务的相关信息放入队列里面，并在之后对队列进行处理，用户可以推迟执行那些需要一段时间才能完成的操作，这种将工作交给任务处理器来执行的做法被称为**任务队列（task queue）**。

<!-- more -->



## 1.先进先出队列

这一节将介绍两种不同类型的任务队列，第一种队列会根据任务被插入队列的顺序来尽快地执行任务，而第二种队列则具有安排在未来某个特定时间执行的能力。

Redis的列表结构允许用户通过 `RPUSH` 和 `LPUSH` 以及 `RPOP` 和 `LPOP` ，从列表的两端推入和弹出元素。这次邮件队列将使用 RPUSH 命令来将待发送的邮件推入列表的右端，并且因为工作的进程除了发送邮件之外不需要执行其他工作，所以它将使用阻塞版本的弹出命令 `BLPOP` 从队列中弹出待发送的邮件，而命令的最大阻塞时限为30秒。



**邮件入队**

**代码清单 sendSolEmailViaQueue() 函数**

```java
	/**
	 * 待发送邮件入队
	 * @param conn
	 * @param seller
	 * @param item
	 * @param price
	 * @param buyer
	 */
	public static void sendSolEmailViaQueue(Jedis conn, String seller, String item, Double price, String buyer) {
		
		String callback = "sendEmail";

		Map<String,Object> data = new HashMap<String,Object>();
		data.put("callback", callback);

		Map<String,Object> args = new HashMap<String,Object>();
		args.put("sellerid", seller);
		args.put("itemid", item);
		args.put("price", price);
		args.put("buyerid", buyer);
		args.put("time", System.currentTimeMillis());
		data.put("args", args);

		String jsonObj = JSONObject.valueToString(data);
		conn.rpush(queueKey, jsonObj);

	}
```



**邮件出队**

**代码清单 processSoldEmailQueue() 函数**

```java
/**
	 * 从队列中取出邮件发送
	 * @param conn
	 */
	public static void processSoldEmailQueue(Jedis conn) {
		
		String queueKey = "queue:email";
		
		while(true) {
			List<String > packed = conn.blpop(3,queueKey);
			
			if(packed.size() == 0) {
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                continue;
            }
			else {
				JSONObject jsonObj = new JSONObject(packed.get(1));
				Map<String,Object> email = jsonObj.toMap();

				// 发送邮件
				sendEmail(email);
			}
		}
	}

	public static void  sendEmail(Map<String,Object> email) {
		System.out.println(email);
	}
```



## 2. 多个可执行任务

- 在一些情况下，为每种任务单独使用一个队列的做法并不少见，但是在另外一些情况下，如果一个队列能够处理多种不同类型的任务，那么事情就会方便很多。
- 如下代码展示的工作进程会监视用户提供的多个队列，并从多个已知的已注册回调函数里面，选出一个函数来处理Json编码的函数调用。
- `BLPOP` 命令和 `BRPOP` 命令都允许用户给定多个列表作为弹出操作的执行对象：其中前者将弹出第一个非空队列的第一个元素，后者则会弹出第一个非空队列的最后一个元素。

```java
	/**
	 * 任务优先级队列，根据优先级取出队列中任务执行相应的回调函数
	 * @param conn
	 * @param queues 传入多个队列，实现优先级
	 * @param callbacks	提供的毁掉函数列表
	 * @throws NoSuchMethodException
	 * @throws InvocationTargetException
	 * @throws IllegalAccessException
	 */
	public static void workerWatchQueue(Jedis conn, String[] queues, String[] callbacks){

		while(true) {
		    List<String> packed = null;
			// 传入多个队列，实现优先级
			Object result = conn.blpop(3,queues);
			if(result instanceof List)
                packed = (List)result;

			if(packed!=null && packed.size() == 0) {
				try {
					Thread.sleep(100);
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
				continue;
			}

			JSONObject jsonObj = new JSONObject(packed.get(1));
			Map<String,Object> data = jsonObj.toMap();

			String callback = (String)data.get("callback");
			Object email = data.get("args");

			try {
				Method method = QueueDemo.class.getMethod(callback, new Class[]{Map.class});
				method.invoke(QueueDemo.class, email);
			} catch (NoSuchMethodException e) {
				e.printStackTrace();
			} catch (InvocationTargetException e) {
				e.printStackTrace();
			} catch (IllegalAccessException e) {
				e.printStackTrace();
			}
		}
	}
```



## 3. 延迟任务

**为了实现延迟执行的特性，修改现有的队列实现：**

- 把所有需要在未来执行的任务都添加到有序集合里面，并将任务的执行时间设置为分值，
- 另外再使用一个进程来查找有序集合里面是否存在可以立即被执行的任务，如果有的话，就从有序集合里面移除那个任务，并将他添加到适当的任务队列里面。

```javascript
	/**
     * 将需要延迟执行的任务添加到任务延迟有序集合
     * @param conn
     * @param queue
     * @param callback
     * @param args
     * @param delay
     * @return
     */
	public static String executeLater(Jedis conn, String queue, String callback, Map args, Long delay) {
		if (delay==null) delay = 0l;

		String delayed = "delayed:";
		String identifier = UUID.randomUUID().toString();
		Map<String, Object> data = new HashMap<String, Object>();
		data.put("identifier", identifier);
		data.put("queue", queue);
		data.put("callback", callback);
		data.put("args", args);

		JSONObject jsonObj = new JSONObject(data);
		jsonObj.toString();

		Calendar cal = Calendar.getInstance();
		cal.add(Calendar.SECOND, delay.intValue());
		if(delay>0)
			conn.zadd(delayed, cal.getTimeInMillis(), jsonObj.toString());
		else
			conn.rpush(queue, jsonObj.toString());

		return identifier;

	}
```



### 3.2 从延迟执行队列拉取任务执行

- 因为Redis没有提供直接的方法可以阻塞有序集合直到元素的分值低于当前的UNIX时间戳为止，所以我们需要自己来查找有序集合里面分值低于当前UNIX时间戳的任务。

```java
	/**
	 * 从延迟执行队列拉取任务执行
	 * @param conn
	 */
	public static void poll_queue(Jedis conn) {

		String delayed = "delayed:";
		while(true) {
			Set<Tuple> result = conn.zrangeWithScores(delayed, 0,0);
			if(result == null || result!=null && result.size()==0) {

			    // 队列中暂时没有任务，短暂休眠后继续读取
                try {
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                continue;
            }
			else{
			    // 任务还未达到执行的时间
                Tuple tuple = (Tuple)new ArrayList(result).get(0);
			    if(tuple.getScore() > System.currentTimeMillis())
			        continue;

				String item = tuple.getElement();
				JSONObject jsonObj = new JSONObject(item);
				Map<String,Object> data = (Map<String,Object>)jsonObj.toMap();
				String identifier = (String)data.get("identifier");
				String queue = (String)data.get("queue");
				String name = (String)data.get("name");
				Object args = data.get("args");

				String lock = LockUtil.acquireLock(conn, identifier, 3, 5);

				try {
					if(lock == null)
						continue;
					else {
						conn.zrem(delayed, item);
						conn.rpush(queue, item);
					}
				}finally {
					LockUtil.releaseLock(conn, identifier, lock);
				}

			}
		}
	}
```



## 4. 主类：

```java
public class QueueDemo {

  	// 优先级队列名
    protected static final String queueHigh = "queue:high";
    protected static final String queueMiddle = "queue:middle";
    protected static final String queueLow = "queue:low";

    public static String[] QUEQUS = new String[]{queueHigh, queueMiddle, queueLow};
    public static String[] CALLBACKS = new String[]{"sendEmail"};

	public static void main(String[] args) {

		Jedis conn = new Jedis("localhost");
		
		sendSolEmailViaQueue(conn,"seller","item",(double)188,"buyer");

      	// 任务执行线程
        new Thread(new QueueWorker()).start();
        // 延迟任务拉取线程
        new Thread(new PollWorker()).start();
	}

	/**
     * 任务执行线程
	 */
	static class QueueWorker implements Runnable{

        public void run() {
            Jedis conn = new Jedis("localhost");
            workerWatchQueue(conn, QueueDemo.QUEQUS ,QueueDemo.CALLBACKS);
        }
    }

    /**
     * 延迟任务拉取线程
     */
	static class PollWorker implements Runnable{

        public void run() {
            Jedis conn = new Jedis("localhost");
            poll_queue(conn);
        }
    }
    
  	......
    ......
}
```



## 5. 运行结果

在延迟时间到达之后出现结果

```
{itemid=item, sellerid=seller, price=188, time=1499583113301, buyerid=buyer}
```

