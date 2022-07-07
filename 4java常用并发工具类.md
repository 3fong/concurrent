## java原子操作

原子操作:实现类似sychronized关键字的内存语义,保证共享变量的原子性,可见性,有序性.

java从1.5开始提供java.util.concurrent.atomic包提供原子操作类实现简单高效,线程安全的操作方法.    
atomic包中共有13个类,可划分为4中类型:原子更新基本类型,原子更新数组,原子更新引用,原子更新属性.基本都使用Unsafe实现的包装类

- 原子更新基本类型
  
AtomicBoolean
AtomicInteger
AtomicLong


- 原子更新数组

AtomicIntegerArray
AtomicLongArray
AtomicReferenceArray


- 原子更新引用

AtomicReference
AtomicMarkableReference
AtomicStampedReference

- 原子更新属性

DoubleAccumulator
DoubleAdder
LongAccumulator
LongAdder

## 并发工具类

CountDownLatch:允许一个或多个线程等待其他线程完成操作    

阻塞当前线程直到latch计数器=0
```
CountDownLatch latch = new CountDownLatch(1);
latch.countDown();
latch.await();
```

CyclicBarrier: 当阻塞所有线程后才放行线程执行.类似于信号枪.

```
CyclicBarrier b = new CyclicBarrier(2);//2是拦截线程的数量
b.await();//阻塞执行线程,直到阻塞2个线程时才会继续执行拦截线程
```

- CountDownLatch与 CyclicBarrier 区别

CyclicBarrier可处理更复杂的业务场景: CountDownLatch计数器只能用一次.CyclicBarrier的计数器可以使用reset()方法重置.

Semaphore: 控制并发线程数

应用场景: 用于流量控制,特别是公共资源有限的应用场景.如数据库连接 

Semaphore控制10个线程并发
```
Semaphore s = new Semaphore(10);//10并发数
s.acquire();
...
s.release();
```


Exchanger: 线程间交换数据

Exchanger提供一个同步点,用于建立线程间数据交换的时点.先到达同步点的线程会等待另一线程.

```
Exchanger ex = new Exchanger<String>();
String data = x.exchange(data);//交换数据,交换双方方法一样,都会获取到对方的数据
```
当一个线程到达 exchange 调用点时，如果其他线程此前已经调用了此方法，则其他线程会被调度唤醒并与之进行对象交换，然后各自返回；如果其他线程还没到达交换点，则当前线程会被挂起，直至其他线程到达才会完成交换并正常返回，或者当前线程被中断或超时返回。

应用场景: 数据校对;遗传算法.

## 线程池

价值:    
1 降低资源消耗(整体).重复利用线程,减少线程创建和销毁的成本    
2 提高响应速度(初始化之后).任务到达后不需要创建线程,可以直接使用    
3 提高线程的可管理性.统一分配,调优,监控线程资源,避免无限制的创建线程    

### 线程池实现原理

![](https://img-blog.csdnimg.cn/img_convert/334efaa114cdc53142b70404a30643f2.png)

ThreadPoolExecutor.execute()执行流程:    
![](http://ddrvcn.oss-cn-hangzhou.aliyuncs.com/2019/3/aemAbi.png)

1 当前运行的线程少于corePoolSize,则创建新线程来执行任务(需要获取全局锁)    
2 >=corePoolSize,则将运行线程放入BlockingQueue    
3 BlockingQueue满时,创建新的线程来处理任务(需要获取全局锁)    
4 创建线程>manximumPoolSize,任务被拒绝,调用拒绝策略

线程池中线程执行任务场景:     
1 execute()创建线程后,让该线程执行当前任务     
2 这个线程执行完1的任务后,反复从BlockingQueue获取任务来执行

### 线程池的使用

```
new ThreadPoolExecutor(corePoolSize,maximumPoolSize,keepAliveTime,milliseconds,runnableTaskQueue,handler);
```

- corePoolSize: 核心线程数量   

当线程池中线程数量小于corePoolSize时,每调用一次execute(),都会创建一个线程,直到线程池中线程数量=corePoolSize.一次创建并启动所有基本线程:prestartAllCoreThreads()

- maximumPoolSize: 最大线程数

线程池允许创建的最大线程数.队列满后,线程数小于最大线程数,线程池会创建新的线程执行任务.无界队列中此参数没有作用

- keepAliveTime: 线程活动保持时间.

线程池工作线程空闲时,保持存活的时间.

- milliseconds: 线程活动保持时间单位.

- runnableTaskQueue: 任务队列,保存等待执行的任务的阻塞队列.

ArrayBlockingQueue: 基于数组结构的有界阻塞队列,此队列按FIFO原则进行元素排序    
LinkedBlockingQueue: 基于链表结构的阻塞队列.FIFO.吞吐量比ArrayBlockingQueue高(采用双锁队列,ArrayBlockingQueue只是用一把锁).Executor.newFixedThreadPool使用了该队列.    
SynchronousQueue: 同步阻塞队列.不存储元素,没插入一个元素,必须有其他线程取走,否则阻塞插入操作.吞吐量高于LinkedBlockingQueue(???).Executor.newCachedThreadPool使用了该队列.    
PriorityBlockingQueue: 具有优先级的无限阻塞队列

- handler: 饱和策略.RejectedExecutionHandler

当线程池饱和时需要采取的新任务处理策略.    
1 AbortPolicy(默认):无法处理时抛出异常    
2 CallerRunsPolicy: 使用调用者所在线程运行任务    
3 DiscardOldestPolicy: 覆盖最近任务    
4 DiscardPolicy: 丢弃当前任务 

- ThreadFactory: 设置创建线程的工厂    

#### 向线程池提交任务






