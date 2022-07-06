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









