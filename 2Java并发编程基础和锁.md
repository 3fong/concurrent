## Java并发编程基础

### 线程

现代操作系统运行一个程序时,会为其创建一个进程.而现代操作系统调度的最小单元是线程,也叫轻量级进程(Light Weight Process).    
在一个进程中可以创建多个线程,每个线程拥有独立的计数器,堆栈和局部变量等属性,并且能够访问共享的内存变量.

多线程出现原因:

1 更多处理器核心.单位时间内,一个处理核心只能运行一个线程,一个进程可以有多个线程,这样多核心的合理运用可以显著的减少程序处理时间    
2 更快的响应时间.通过多线程拆分数据一致性不强的业务流程,可以更快的响应请求,提升用户体验    
3 更好的编程模型.Java为多线程编程提供了良好,考究并且一致的编程模型,使开发人员能够更加专注于问题的解决.而多线程编程模型很容易应用

- 线程优先级

现代操作系统基本采用时分的形式调度运行的线程,操作系统会分出一个个时间片(执行时间),线程会分配到若干时间片,当线程的时间片用完时就会发生线程调度,并等待下次分配.    
线程分配到的时间片决定了线程使用处理器资源的多少,而线程优先级就是决定线程资源分配的参考属性.    
Java中通过线程参数priority来设置优先级,优先级的范围从1-10,值越大优先级越高.默认值为5.优先级高的线程分配时间片数量要多于优先级低的线程.    
但实际资源分配还要取决于操作系统的实现,不保证优先级越大的肯定会分配更多资源.    
实际设置线程优先级时,频繁阻塞(休眠或I/O操作)线程应设置较高优先级;偏重计算(耗CPU或偏运算)的线程设置较低优先级,确保处理器不被独占;    

- 线程状态

New：初始状态.新创建的线程，尚未执行；
Runnable：运行状态.运行中的线程，执行start()或run()方法；
Blocked：阻塞状态.运行中的线程，因为某些操作被阻塞而挂起;同步队列,具备锁竞争权限线程
Waiting：等待状态.运行中的线程，因为某些操作在等待中；等待队列->同步队列
Timed Waiting：超时等待状态.运行中的线程，因为执行sleep()方法正在计时等待；
Terminated：终止状态.线程已终止，因为run()方法执行完毕。

java线程状态查看方式:    
```
1 查看Java线程pid.命令行输入: jps    
2 查看指定pid线程状态.输入: jstack <pid>
```

线程状态流转图:    
![](https://img-blog.csdnimg.cn/235e29b1a10042c885e49a3145bc053a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5bCPYml0fg==,size_20,color_FFFFFF,t_70,g_se,x_16)

java将操作系统中运行和就绪两个状态合并称为运行状态.    
阻塞状态:线程阻塞进入synchronized关键字修饰的方法或代码块(获取锁)时的状态;但阻塞在concurrent包中Lock接口的线程状态却是等待状态,
因为java.concurrent包中Lock接口对于阻塞的实现均使用了LockSupport类中的相关方法.

Daemon线程

daemon线程(后台线程),是一种支持性线程,用于程序后台调度及支持性工作.当jvm中不存在daemon线程时,JVM就会退出.    
damon线程设置方式: Thread.setDaemon(true).必须要在线程运行前设置.同时它会在主线程退出时立即退出,内部的finally块不会执行.

#### 常用方法

构造线程: Thread.可以设置线程组,优先级,daemon线程,名称,线程id等信息,且父线程要进行空间分配,初始化.    
启动线程: start().当前线程(父线程)同步告知Java虚拟机,只要线程规划器空闲,应立即启动调用start()方法的线程    
中断: Thread.interrupt().是一个线程标识位属性,表示一个运行中的线程释放被其他线程进行了中断操作.    
线程通过检查自身是否中断来进行响应,线程通过方法isInterrupted()来判断是否被中断.    
如果该线程已经处于终结状态,即使该线程被中断过,在调用该线程isInterrupted()方法时会返回false;    
当线程处于waiting, sleeping,或其他 occupied或之前时,调用线程中断操作会抛出InterruptedException,中断异常抛出前会将中断标识位清除.    
中断操作是一种简便的线程间交互方式,最适合用来取消或停止任务(isInterrupted()条件满足后,可以安全优雅的终止线程,处理好线程结束前的资源清理,避免武断的线程停止.程序更健壮)


### 线程间通信

通信方式:    
```
1 volatile,synchronized    
2 等待通知机制    
3 管道输入/输出流    
4 Thread.join()    
5 ThreadLocal    

```

1. volatile,synchronized    

volatile: 告知程序任何对该变量的访问均需要从共享内存中获取,而对它的改变必须同步刷新回共享内存,它能保证所有线程对变量访问的可见性.

synchronized: 排他的获取对象监视器,通过监视器进入对象的同步方法,同步方法进入插入monitorenter,退出monitorexit.没有获取到监视器的线程进入BLOCKED状态,被阻塞在同步块和同步方法的入口处,线程被放入同步队列.


2. 等待通知机制    

等待通知依赖于同步机制,目的是确保等待线程从wait()方法返回时,能感知到通知线程对变量做出的修改

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimg.136.la%2F20210904%2F72fd0c4f23104c138833d5bdea889187.jpg&refer=http%3A%2F%2Fimg.136.la&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1658278772&t=92551ca647d3f03d837106d27c0cee8b)

线程运行状态流转:    
![](https://upload-images.jianshu.io/upload_images/9606149-2fcea95cc43cdfb8.png?imageMogr2/auto-orient/strip|imageView2/2/w/1168/format/webp)

a. WaitThread先获取对象锁,然后调用方法wait()方法,该方法会释放锁,线程由RUNNING->WAITING,进入等待队列    
b. NotifyThread随后获取锁,并调用notify方法,该方法将选择一个线程从等待队列进入同步队列,线程状态从WAITING->BLOCKED    
c. NotifyThread释放锁后,WaitThread竞争并再次获取到锁并从wait()方法返回继续执行 

等待通知范式:    

```
等待方:    
1 获取对象锁    
2 执行条件不满足,调用wait()方法    
3 条件满足,执行对应逻辑

通知方:    
1 获取对象锁    
2 改变条件    
3 通知所有等待在对象上的线程
```

3. 管道输入/输出流    

管道流主要用于线程间的数据传输,而传输媒介是内存.实现类:    
PipedOutputStream,PipedInputStream,PipedReader,PipedWriter

4. Thread.join()    

join(): 当前线程A等待thread线程终止之后才从join()返回.插入线程.实现原理:加锁,循环,处理逻辑,等待/通知范式的标准实现.

5. ThreadLocal  

线程变量.是一个以ThreadLocal对象为键,任意对象为值的存储结构.该结构依赖于当前线程.































