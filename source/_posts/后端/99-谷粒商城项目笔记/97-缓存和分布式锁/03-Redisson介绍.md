## `Redisson`实现分布式锁

### 一、简介

**`Redisson`** **是架设在** `Redis` **基础上**的一个 Java 驻内存数据网格（In-Memory Data Grid）。充分的利用了 Redis 键值数据库提供的一系列优势，**基于** **Java** **实用工具包中常用接口**，为使用者提供了一系列具有分布式特性的常用工具类。**使得原本作为协调单机多线程并发程序的工具包获得了协调分布式多机多线程并发系统的能力**，大大降低了设计和研发大规模分布式 系统的难度。同时结合各富特色的分布式服务，更进一步简化了分布式环境中程序相互之间的协作。 

官方文档：https://github.com/redisson/redisson/wiki/%E7%9B%AE%E5%BD%95

中文：[redisson使用全解——redisson官方文档+注释（上篇）_redisson官网中文-CSDN博客](https://blog.csdn.net/A_art_xiang/article/details/125525864)

### 二、如何使用

#### 1、引入依赖

```xml
<!-- https://mvnrepository.com/artifact/org.redisson/redisson
        使用redisson作为所有分布式锁，分布式对象等功能框架-->
        <dependency>
            <groupId>org.redisson</groupId>
            <artifactId>redisson</artifactId>
            <version>3.20.1</version>
        </dependency>
```

#### 2、配置

##### 单机模式：

```java
@Configuration
public class MyRedissonConfig {

    /**
     * 所有redisson的使用都是通过RedissonClient对象来操作
     * @return
     */
    @Bean(destroyMethod = "shutdown")
    public RedissonClient redisson() {
        //创建配置
        Config config = new Config();
        config.useSingleServer().setAddress("redis://192.168.88.129:6379");
        //创建RedissonClient实例
        return Redisson.create(config);
    }
}
```

##### 集群模式：

```java
Config config = new Config();
config.useClusterServers()
    .setScanInterval(2000) // 集群状态扫描间隔时间，单位是毫秒
    //可以用"redis://"来启用SSL连接
    .addNodeAddress("redis://192.168.88.129:7000", "redis://192.168.88.129:6379")
    .addNodeAddress("redis://192.168.88.128:7001");

RedissonClient redisson = Redisson.create(config);
```

##### 哨兵模式

```java
Config config = new Config();
config.useSentinelServers()
    .setMasterName("master")
    //可以用"redis://"来启用SSL连接
    .addSentinelAddress("127.0.0.1:26389", "127.0.0.1:26379")
    .addSentinelAddress("127.0.0.1:26319");

RedissonClient redisson = Redisson.create(config);
```

##### 主从模式

```java
Config config = new Config();
config.useMasterSlaveServers()
    //可以用"redis://"来启用SSL连接
    .setMasterAddress("redis://127.0.0.1:6379")
    .addSlaveAddress("redis://127.0.0.1:6389", "redis://127.0.0.1:6332", "redis://127.0.0.1:6419")
    .addSlaveAddress("redis://127.0.0.1:6399");

RedissonClient redisson = Redisson.create(config);
```

#### 3、使用

```java
RLock lock = redisson.getLock("anyLock");// 最常见的使用方法
lock.lock(); 

lock.lock(10, TimeUnit.SECONDS);// 加锁以后 10 秒钟自动解锁// 无需调用 unlock 方法手动解锁

boolean res = lock.tryLock(100, 10, TimeUnit.SECONDS);// 尝试加锁，最多等待 100 秒，上锁以后 10 秒自动解锁
if (res) { 
    try { ... } 
    finally { 
        lock.unlock(); 
    } 
}
```

##### lock和tryLock的区别

`lock() `方法是==阻塞获取锁==的方式，如果当前锁被其他线程持有，则当前线程会一直阻塞等待获取锁，直到获取到锁或者发生超时或中断等情况才会结束等待。该方法获取到锁之后可以保证线程对共享资源的访问是互斥的，适用于需要确保共享资源只能被一个线程访问的场景。Redisson 的 lock() 方法支持可重入锁和公平锁等特性，可以更好地满足多线程并发访问的需求。

`tryLock() `方法是一种==非阻塞获取锁==的方式，在尝试获取锁时不会阻塞当前线程，而是立即返回获取锁的结果，如果获取成功则返回 true，否则返回 false。Redisson 的 tryLock() 方法支持加锁时间限制、等待时间限制以及可重入等特性，可以更好地控制获取锁的过程和等待时间，避免程序出现长时间无法响应等问题。

因此，两种获取锁的方式各有优缺点，在实际应用中需要根据具体场景和业务需求来选择合适的方法，以确保程序的正确性和高效性。



##### lock()源码：

```java
private void lock(long leaseTime, TimeUnit unit, boolean interruptibly) throws InterruptedException {
 
	// 获取当前线程 ID
    long threadId = Thread.currentThread().getId();
    // 获取锁，正常获取锁则ttl为null，竞争锁时返回锁的过期时间
    Long ttl = tryAcquire(-1, leaseTime, unit, threadId);
    if (ttl == null) {
        return;
    }
		
    // 订阅锁释放事件
    // 如果当前线程通过 Redis 的 channel 订阅锁的释放事件获取得知已经被释放，则会发消息通知待等待的线程进行竞争
    RFuture<RedissonLockEntry> future = subscribe(threadId);
    if (interruptibly) {
        commandExecutor.syncSubscriptionInterrupted(future);
    } else {
        commandExecutor.syncSubscription(future);
    }
 
    try {
        while (true) {
            // 循环重试获取锁，直至重新获取锁成功才跳出循环
            // 此种做法阻塞进程，一直处于等待锁手动释放或者超时才继续线程    
            ttl = tryAcquire(-1, leaseTime, unit, threadId);
            if (ttl == null) {
                break;
            }
 
            if (ttl >= 0) {
                try {
                    future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } catch (InterruptedException e) {
                    if (interruptibly) {
                        throw e;
                    }
                    future.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                }
            } else {
                if (interruptibly) {
                    future.getNow().getLatch().acquire();
                } else {
                    future.getNow().getLatch().acquireUninterruptibly();
                }
            }
        }
    } finally {
        // 最后释放订阅事件
        unsubscribe(future, threadId);
    }
}
```

##### tryLock()源码：

```java
@Override
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
    // 转成毫秒，后面都是以毫秒为单位
    long time = unit.toMillis(waitTime);
    // 当前时间
    long current = System.currentTimeMillis();
    // 线程ID-线程标识
    long threadId = Thread.currentThread().getId();
 
    // 尝试获取锁 tryAcquire() !!!
    Long ttl = tryAcquire(waitTime, leaseTime, unit, threadId);
        
    // 如果上面尝试获取锁返回的是null，表示成功；如果返回的是时间则表示失败。
    if (ttl == null) {
        return true;
    }
 
    // 剩余等待时间 = 最大等待时间 -（用现在时间 - 获取锁前的时间）
    time -= System.currentTimeMillis() - current;
 
    // 剩余等待时间 < 0 失败
    if (time <= 0) {
        acquireFailed(waitTime, unit, threadId);
        return false;
    }
 
    // 再次获取当前时间
    current = System.currentTimeMillis();
    // 重试逻辑，但不是简单的直接重试！
    // subscribe是订阅的意思
    RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
    // 如果在剩余等待时间内，收到了释放锁那边发过来的publish，则才会再次尝试获取锁
    if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
        if (!subscribeFuture.cancel(false)) {
            subscribeFuture.onComplete((res, e) -> {
                if (e == null) {   
                    // 取消订阅
                    unsubscribe(subscribeFuture, threadId);
                }
            });
        }
        // 获取锁失败
        acquireFailed(waitTime, unit, threadId);
        return false;
    }
 
    try {
        // 又重新计算了一下，上述的等待时间
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(waitTime, unit, threadId);
            return false;
        }
 
        // 重试！
        while (true) {
            long currentTime = System.currentTimeMillis();
            ttl = tryAcquire(waitTime, leaseTime, unit, threadId);
 
            // 成功
            if (ttl == null) {
                return true;
            }
            
            // 又获取锁失败，再次计算上面的耗时
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(waitTime, unit, threadId);
                return false;
            }
 
            currentTime = System.currentTimeMillis();
            // 采用信号量的方式重试！
            if (ttl >= 0 && ttl < time) {
                subscribeFuture.getNow().getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
            } else {
                subscribeFuture.getNow().getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
            }
            
            // 重新计算时间（充足就继续循环）
            time -= System.currentTimeMillis() - currentTime;
            if (time <= 0) {
                acquireFailed(waitTime, unit, threadId);
                return false;
            }
        }
    } finally {
        unsubscribe(subscribeFuture, threadId);
    }
}
```



##### 示例1：

```java
 //1、获取1把锁 ，只要锁的名字一样，就是同一把锁
RLock lock = redissonClient.getLock("lock");
//2、加锁
//1)锁的自动续期：如果业务超长，运行期间自动给锁续上新的30s，不用担心业务时间长，锁自动过期删掉
//2)加锁的业务只要运行完成，就不会给当前锁续期，即使不手动解锁，所也会默认在30秒以后自动删除
lock.lock();//阻塞式等待。默认加的锁30秒有效时间
//lock.lock(20L, TimeUnit.SECONDS);//指定锁的有效时间20s，自动解锁时间一定要大于业务的执行时间,在锁时间到了以后，不会自动续期
        /**
         * 1）如果我们指定了时间，就会发送给redis执行脚本，进行占锁，默认超时就是我们指定的时间
         * 2）如果我们未指定超时时间，就使用默认30s 【lockWatchdogTimeout看门狗的默认时间】
         *      只要占锁成功，就会启动一个定时任务【重新设置锁的过期时间，新的过期时间【lockWatchdogTimeout看门狗的默认时间】】
         *      定时任务：this.internalLockLeaseTime【看门狗的默认时间】 / 3L  == 10s 执行一次
         */

//最佳实战
//1）、lock.lock(20L, TimeUnit.SECONDS);//省掉了整个续期的操作
try {
	System.out.println("加锁成功。。执行业务..."+Thread.currentThread().getId());
	Thread.sleep(30000);
} catch (InterruptedException e) {
	e.printStackTrace();
} finally {
//3、解锁  假设解锁代码没有运行
	System.out.println("释放锁。。"+Thread.currentThread().getId());
	lock.unlock();
}
```

##### 看门狗原理

如果负责储存这个分布式锁的 Redisson 节点宕机以后，而且这个锁正好处于锁住的状态时，这个锁会出现锁死的状态。

`为了避免这种情况的发生，Redisson内部提供了一个监控锁的看门狗，它的作用是在Redisson实例被关闭前，不断的延长锁的有效期。`

默认情况下，看门狗的检查锁的超时时间是`30秒钟`，也可以通过修改`Config.lockWatchdogTimeout`来另行指定。

如果我们未指定 lock 的超时时间，就使用 30 秒作为看门狗的默认时间。只要占锁成功，就会启动一个定时任务：每隔 10 秒重新给锁设置过期的时间，过期时间为 30 秒。

当服务器宕机后，因为锁的有效期是 30 秒，所以会在 30 秒内自动解锁。（30秒等于宕机之前的锁占用时间+后续锁占用的时间）

## `Redisson`之分布式信号量介绍

### 一、简介

`Semaphore`通常叫信号量，可以用来同时控制访问特定资源的线程数量，通过协调各个线程，保证合理的使用资源。

### 二、API介绍

```java
public interface RSemaphore extends RExpirable, RSemaphoreAsync {
 
    // 获得一个permit
    void acquire() throws InterruptedException; 
 
    //获得var1个permit
    void acquire(int var1) throws InterruptedException; 
 
    //尝试获得permit
    boolean tryAcquire(); 
 
   //尝试获得var1个permit
    boolean tryAcquire(int var1); 
 
   //尝试获得permit, 等待时间var1
    boolean tryAcquire(long var1, TimeUnit var3) throws InterruptedException;
 
   //尝试获得var1个permit, 等待时间var2
    boolean tryAcquire(int var1, long var2, TimeUnit var4) throws InterruptedException;
 
   //释放1个permit
    void release();
 
   //释放var1个permit
    void release(int var1);
 
   //信号量的permits数
    int availablePermits();
 
   //清空permits
    int drainPermits();
 
  //设置permits数
    /**
	 * 尝试设置许可数量，设置成功，返回true，否则返回false
	 */
    boolean trySetPermits(int var1);
 
//添加permits数
    void addPermits(int var1);
}
```

### 三、Semaphore的使用

#### 示例1：

```java
	/**
     * 设置信号量
     * @param num
     * @return
     */
    @GetMapping("/setSemaphore/{num}")
    public String setSemaphore(@PathVariable("num") Integer num) {
        RSemaphore park = redissonClient.getSemaphore("park");
        park.trySetPermits(num);
        return "ok";
    }
	/**
     * 车库停车
     * 3车位
     * 信号量也可以用作分布式限流；
     */
    @GetMapping("/park")
    @ResponseBody
    public String park() throws InterruptedException {
        RSemaphore park = redissonClient.getSemaphore("park");
//        park.acquire();//获取一个信号。（获取一个值）
        boolean b = park.tryAcquire();//非阻塞，尝试获取
        if(b) {
            //执行复杂业务
        } else {
            return "当前流量过大";
        }
        return "停车成功ok==>"+b;
    }

    @GetMapping("/out")
    @ResponseBody
    public String out() {
        RSemaphore park = redissonClient.getSemaphore("park");
        park.release();
        return "释放一个车位";
    }
```

### 四、源码分析

#### trySetPermits方法

```java
/**
 * 尝试设置许可数量，设置成功，返回true，否则返回false
 */
boolean trySetPermits(int permits);
```

源码：

```java
	@Override
    public boolean trySetPermits(int permits) {
        return get(trySetPermitsAsync(permits));
    }

 	@Override
    public RFuture<Boolean> trySetPermitsAsync(int permits) {
        RFuture<Boolean> future = commandExecutor.evalWriteAsync(getRawName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
			// 判断分布式信号量的key是否存在，如果不存在，才设置
                "local value = redis.call('get', KEYS[1]); " +
                        "if (value == false) then "
                    // set "semaphore" permits
                    // 使用String数据结构设置信号量的许可数
                        + "redis.call('set', KEYS[1], ARGV[1]); "                         
                    // 发布一条消息到redisson_sc:{semaphore}通道
                        + "redis.call('publish', KEYS[2], ARGV[1]); "
                     // 设置成功，返回1
                        + "return 1;"
                        + "end;"
                     // 设置失败，返回0
                        + "return 0;",
                Arrays.asList(getRawName(), getChannelName()), permits);

        if (log.isDebugEnabled()) {
            future.thenAccept(r -> {
                if (r) {
                    log.debug("permits set, permits: {}, name: {}", permits, getName());
                } else {
                    log.debug("unable to set permits, permits: {}, name: {}", permits, getName());
                }
            });
        }
        return future;
    }
```

> `lua脚本参数说明`:
>
> `KEYS[1]:` 我们指定的分布式信号量key。例如redissonClient.getSemaphore("park")中的"park")
> `KEYS[2]: `释放锁的channel名称，redisson_sc:{分布式信号量key}。在本例中，就是redisson_sc:{park}
>
> ```java
> public static String getChannelName(String name) {//name就是分布式信号量key
>     return prefixName("redisson_sc", name);
> }
> ```
>
> `ARGV[1]: `设置的许可数量

#### acquire()方法

```java
 @Override
    public void acquire() throws InterruptedException {
        acquire(1);//默认1个
    }

    @Override
    public void acquire(int permits) throws InterruptedException {
        if (tryAcquire(permits)) {
            return;
        }
	// 对于没有获取锁的那些线程，订阅redisson_sc:{分布式信号量key}通道的消息
        CompletableFuture<RedissonLockEntry> future = subscribe();
        semaphorePubSub.timeout(future);
        RedissonLockEntry entry = commandExecutor.getInterrupted(future);
        try {
            while (true) {
                 // 不断循环尝试获取许可
                if (tryAcquire(permits)) {
                    return;
                }
                entry.getLatch().acquire();
            }
        } finally {
            // 取消订阅
            unsubscribe(entry);
        }
//        get(acquireAsync(permits));
    }
```

#### tryAcquire方法

```java
	@Override
    public boolean tryAcquire() {
        return tryAcquire(1);
    }

    @Override
    public boolean tryAcquire(int permits) {
        return get(tryAcquireAsync(permits));
    }
@Override
    public RFuture<Boolean> tryAcquireAsync(int permits) {
        if (permits < 0) {
            throw new IllegalArgumentException("Permits amount can't be negative");
        }
        if (permits == 0) {
            return new CompletableFutureWrapper<>(true);
        }

        return commandExecutor.evalWriteAsync(getRawName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
                  // 获取当前剩余的许可数量
                  "local value = redis.call('get', KEYS[1]); " +
                 // 许可不为空，并且许可数量 大于等于 当前线程申请的许可数量   
                  "if (value ~= false and tonumber(value) >= tonumber(ARGV[1])) then " +
                      // 通过decrby减少剩余可用许可 
                      "local val = redis.call('decrby', KEYS[1], ARGV[1]); " +
						//成功获取，返回1
                      "return 1; " +
                  "end; " +
                  // 获取失败，返回0    
                  "return 0;",
                  Collections.<Object>singletonList(getRawName()), permits);
    }
```

#### release方法

```java
@Override
public void release() {
    release(1);
}

@Override
public void release(int permits) {
    get(releaseAsync(permits));
}

@Override
public RFuture<Void> releaseAsync(int permits) {
    if (permits < 0) {
        throw new IllegalArgumentException("Permits amount can't be negative");
    }
    if (permits == 0) {
        return new CompletableFutureWrapper<>((Void) null);
    }

    RFuture<Void> future = commandExecutor.evalWriteAsync(getRawName(), StringCodec.INSTANCE, RedisCommands.EVAL_VOID,
		// 通过incrby增加许可数量
            "local value = redis.call('incrby', KEYS[1], ARGV[1]); " +
        // 发布一条消息到redisson_sc:{semaphore}中
                    "redis.call('publish', KEYS[2], value); ",
            Arrays.asList(getRawName(), getChannelName()), permits);
    if (log.isDebugEnabled()) {
        future.thenAccept(o -> {
            log.debug("released, permits: {}, name: {}", permits, getName());
        });
    }
    return future;
}
```

### 五、应用场景

#### 限制并发访问量

在某些场景下，我们可能需要限制系统的并发访问量，防止资源被过度消耗。例如，我们希望限制某个接口每秒钟只能被100个请求访问。下面是一个使用Redisson信号量实现的示例代码：

```java
try {
    RSemaphore semaphore = redissonClient.getSemaphore("queryList:access:limit");
	boolean acquired = semaphore.tryAcquire();
	if(acquired) {
		//执行接口逻辑
		//。。。。。
	} else {
		//返回接口访问量过大的错误信息
	}
} catch(Exception e) {
    //处理异常。。。。
} finally {
   //释放许可
	semaphore.release(); 
}
```



## Redisson之读写锁

### 简介

保证一定能读到最新数据，修改期间写锁是互斥锁（排他锁）。读锁是一个共享锁

### 使用

```java
// 获取key为"rwLock"的锁对象，此时获取到的对象是 RReadWriteLock
 RReadWriteLock rwLock = redissonClient.getReadWriteLock("rwLock");

 RLock lock = rwLock.readLock();  // 获取读锁read
 // or
 RLock lock = rwLock.writeLock(); // 获取写锁write
 // 2、加锁
 lock.lock();
 try {
   // 进行具体的业务操作
   ...
 } finally {
 　　// 3、释放锁
 　　lock.unlock();
 }
```

### 特性

```
/**
 * 读写锁：
 *  改数据加写锁，读数据加读锁
 *
 *  保证一定能读到最新数据；修改期间 写锁 是排他锁（互斥锁，独享锁）。读锁是一个共享锁
 *  写锁没释放，读就必须等待。
 *  写+读：等待写锁释放
 *  写+写：阻塞方式
 *  读+写：等待读锁释放-
 *  读+读： 相当于无锁
 *
 *  只要有写锁存在，则必须等待
 */
```

### 总结

***无论是读请求先执行还是写请求先执行，只要涉及到写锁，则都会阻塞，如果是先写再读，则读锁等待，如果是先读再写，则写锁等待***
