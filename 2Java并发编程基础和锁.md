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


等待超时: 调用方法要么线程返回结果,要么等待一定时间后超时返回: wait(time)

#### 线程池

预创建线程,统一管理优点:    
1 消除频繁创建和消亡线程的系统资源开销    
2 面对过量任务的提交能够平缓的劣化.

原理:    
线程池的本质是使用一个线程安全的工作队列连接工作者线程和客户端线程,客户端线程将任务放入工作队列(并notify()工作者线程消费)后便返回.工作者线程不断从工作队列上取出消息并执行.    
当工作队列为空时,工作者线程则等待工作线程,当客户端线程提交消息后,通知任一工作者现象,工作者线程进行执行

线程池拆分了请求方和执行方的逻辑关联,请求方只需要向工作队列放入请求,即完成请求;执行方只需要不断的从工作队列获取请求,即可完成业务.

线程池请求时序图:    
![线程池请求时序图](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fupload-images.jianshu.io%2Fupload_images%2F15002197-ee38d88a54913f42.png&refer=http%3A%2F%2Fupload-images.jianshu.io&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1658451614&t=f74a6ef06f8672a4702e736f2cd55337)


## Java中的锁

### Lock接口

锁时用来控制多个线程访问共享资源的方式.一般来说,锁最常用的场景是防止多个线程同时访问共享资源,但读写锁,可以实现多个线程同时访问.

sychronized锁: 便捷的隐式获取锁方式.但是它将锁的获取和释放固化;简化了锁的使用,但是扩展性不好,具体体现:    
    1 线程sychronized获取锁,如果遇到阻塞,只能一直等待;而Lock可以使用tryLock(long time, TimeUnit unit)) 或者 能够响应中断来中断阻塞    
    2 sychronized不区分读写锁,Lock可以区分读写锁,锁运用更灵活    
    3 我们可以通过Lock得知线程有没有成功获取到锁 (解决方案：ReentrantLock) ，但这个是synchronized无法办到的
Lock接口: 显示的获取和释放锁;扩展性好,可以灵活的按需配置锁资源,使用上比sychronized繁琐

[sychronized与lock的区别](https://www.cnblogs.com/myseries/p/10784076.html)

```
Lock lock = ...;
lock.lock();
try{
    //处理任务
}catch(Exception ex){

}finally{
    lock.unlock();   //释放锁
}
```

- 问题

为什么lock.lock()获取锁操作不能再try代码块中?    
如果未获得锁就释放的话,会被抛锁异常

Lock比sychronized多出的特性    
![](https://www.icode9.com/i/ll/?i=20201214103843929.png)

Lock的API
![](https://www.icode9.com/i/ll/?i=20201214103823924.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM0Mjc1Mjc3,size_16,color_FFFFFF,t_70)

### 队列同步器 AbstractQueuedSynchronizer

![AbstractQueuedSynchronizer结构](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57a18c88fb7c4d60b4d8a50b78a84e70~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

队列同步器: 用来构建锁或其他同步组件的基础框架,它使用了一个int成员变量表示同步状态,通过内置的FIFO队列来完成资源获取线程的排队工作.

同步器的主要使用方式是继承,子类通过继承同步器并实现它的抽象方法来管理同步状态,同步器提供了3个方法(getState(),setState(int newState),compareAndSetState(int expect,int update))来安全的更改同步状态.

子类推荐被定义为自定义同步组件的静态内部类,同步器自身没有实现任何同步接口,仅定义了若干同步状态获取和释放的方法.同步器即可以独占式地获取同步状态,也可以支持共享式地获取同步状态,这样就可以方便实现不同类型的同步组件(ReentrantLock,ReentrantReadWriteLock,CountDownLatch)

同步器是实现锁的关键,在锁的实现中聚合同步器,利用同步器实现锁的语义.即: 锁是面向使用者的,定义使用者与锁交互的接口(如允许两个线程并行访问),隐藏了实现细节;同步器面向的是锁的实现者,简化了锁的实现方式,屏蔽了同步状态管理,线程的排队,等待与唤醒等底层操作.

- 队列同步器的接口与示例

同步器的设计是基于模板方法模式(框架模式),使用者需要继承同步器并重写指定的方法,随后将同步器组合在自定义同步组件的实现中,并调用同步器提供的模板方法,而模板方法调用使用自定义实现方法

同步器实现的方法:    
![同步器实现的方法](https://pic2.zhimg.com/80/v2-6d829ab3928c190156587548334d9ff1_720w.jpg)

同步器模板方法:    
![同步器模板方法](https://pic2.zhimg.com/80/v2-f954622a9a6740437476340bd522fead_720w.jpg)

方法分类: 独占式同步状态操作,共享式同步状态操作,同步队列线程查询

#### 队列同步器的实现分析

队列同步器的实现主要包括: 同步队列,独占式同步状态获取与释放,共享式同步状态获取与释放,超时获取同步状态等数据结构和模板方法

- 同步队列

同步器依赖内部的同步队列(FIFO双向队列-有序)来完成同步状态管理,当前线程获取同步状态失败时,同步器会将当前线程以及等待状态等信息构造成为一个节点并加入同步队列,同时阻塞当前线程,当同步状态释放时,会把首节点中的线程唤醒,使其再次尝试获取同步状态

队列节点: 保存获取同步状态失败线程引用,等待状态及前驱和后继节点,节点的属性类型与名称及描述:    

![同步队列节点的属性类型与名称及描述]()

```
static final class Node {

    // 用于标记一个节点在共享模式下等待
    static final Node SHARED = new Node();

    // 用于标记一个节点在独占模式下等待
    static final Node EXCLUSIVE = null;

    // 等待状态：取消。由于再同步队列中等待的线程等待超时或被中断，需要从同步队列中取消等待，节点进入该状态将不会变化。
    static final int CANCELLED = 1;

    // 等待状态：通知。后继节点的线程处于等待状态，当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点。
    static final int SIGNAL = -1;

    // 等待状态：条件等待。存放在条件队列，调用signal()方法或signalAll()方法，会将该节点转移至同步队列中。
    static final int CONDITION = -2;

    // 等待状态：传播。表示下一次同步状态的获取将会无条件的被传播下去
    static final int PROPAGATE = -3;

    // 等待状态,volatile保证可见性
    volatile int waitStatus;

    // 前驱节点,当节点加入同步队列时被设置
    volatile Node prev;

    // 后继节点
    volatile Node next;

    // 节点对应的线程
    volatile Thread thread;

    // 等待队列中的后继节点
    Node nextWaiter;

    // 当前节点是否处于共享模式等待
    final boolean isShared() {
        return nextWaiter == SHARED;
    }

    // 获取前驱节点，如果为空的话抛出空指针异常
    final Node predecessor() throws NullPointerException {
        Node p = prev;
        if (p == null) {
            throw new NullPointerException();
        } else {
            return p;
        }
    }

    Node() {
    }

    Node(Thread thread, Node mode) {
        this.nextWaiter = mode;
        this.thread = thread;
    }

    Node(Thread thread, int waitStatus) {
        this.waitStatus = waitStatus;
        this.thread = thread;
    }
}
```

同步器节点维护

![新增节点](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/17a4cd059d554befac11ef6a80591c82~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

![设置首节点](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ac71e3f49e654f0a8d34c313f3f21786~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

- 独占式同步状态获取与释放

- - 锁获取流程

Lock.lock() -> sync.acquire(1) -> tryAcquire(arg) -> addWaiter(Node.EXCLUSIVE) -> acquireQueued(Node, arg) -> selfInterrupt()

获取锁.如果当前线程在获取锁时,锁未被其他线程获取则将锁状态改为1后返回方法.如果线程已经获取了锁,则将锁状态每次自增1后并返回方法.    
如果锁被其他线程获取,则当前线程不会被线程调度并休眠到获取到锁为止,并将锁状态更新为1.

```
    public void lock() {
        sync.acquire(1);
    }
```

尝试获取锁,如果失败则循环创建排他锁后放入同步队列末尾
```
    public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
    // static final class FairSync extends Sync 公平锁实现.尝试将当前线程按序设置为排他锁拥有线程
            protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            // 如果时第一次获取锁(锁初始状态为0),且要求当前节点无前驱节点
            if (c == 0) {
                if (!hasQueuedPredecessors() &&
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```
自旋(自循环)将当前节点设置为尾节点
```
    private Node addWaiter(Node mode) {
        Node node = new Node(mode);

        for (;;) {
            Node oldTail = tail;
            if (oldTail != null) {
                node.setPrevRelaxed(oldTail);
                if (compareAndSetTail(oldTail, node)) {
                    oldTail.next = node;
                    return node;
                }
            } else {
                initializeSyncQueue();
            }
        }
    }
``
循环获取当前节点的前一个节点是否是头节点,如果是并获取到锁后则更新自己为头节点.    
1 头节点是成功获取到同步状态的节点,头节点的线程释放同步状态后,会唤醒其后继节点,后继节点的线程被唤醒后,检查自己的前驱节点是否是头节点    
2 维护同步队列的FIFO原则.acquireQueued方法中节点会自旋获取同步状态的行为,不管自己是不是头节点的后继节点,所有节点都会主动自旋来检查自己的身份是否头节点的后继节点
```
    final boolean acquireQueued(final Node node, int arg) {
        boolean interrupted = false;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC 从同步队列中移除已经释放锁的原头节点
                    return interrupted;
                }
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            if (interrupted)
                selfInterrupt();
            throw t;
        }
    }
```

独占式同步状态获取流程

![独占式同步状态获取流程](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/059980f76ba242d3889501fff29b6fa4~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)


- - 独占式锁释放流程

lock.unlock() -> sync.release(1) -> tryRelease(arg) -> unparkSuccessor(h)

```
    public final boolean release(int arg) {
        if (tryRelease(arg)) {
            Node h = head;
            if (h != null && h.waitStatus != 0)
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

    protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }

    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        if (ws < 0)
            node.compareAndSetWaitStatus(ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        Node s = node.next;
        if (s == null || s.waitStatus > 0) {
            s = null;
            for (Node p = tail; p != node && p != null; p = p.prev)
                if (p.waitStatus <= 0)
                    s = p;
        }
        if (s != null)
        // 唤醒头节点的后继节点线程.告诉他可以去抢锁了
            LockSupport.unpark(s.thread);
    }

```

在获取同步状态时,同步器维护一个同步队列,获取状态失败的线程都会被加入到队列中并在队列中进行自旋;移出队列或停止自旋的条件是前驱节点为头节点且成功获取了同步状态.    
在释放同步状态时,同步器调用tryRelease释放同步状态,然后唤醒头节点的后继节点去获取锁同步状态.


- 共享式同步状态获取与释放

ReentrantReadWriteLock.lock() -> sync.acquireShared(1) -> 

```
    public final void acquireShared(int arg) {
        if (tryAcquireShared(arg) < 0)
            doAcquireShared(arg);
    }

    private void doAcquireShared(int arg) {
        final Node node = addWaiter(Node.SHARED);
        boolean interrupted = false;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head) {
                    int r = tryAcquireShared(arg);
                    if (r >= 0) {
                        setHeadAndPropagate(node, r);
                        p.next = null; // help GC
                        return;
                    }
                }
                if (shouldParkAfterFailedAcquire(p, node))
                    interrupted |= parkAndCheckInterrupt();
            }
        } catch (Throwable t) {
            cancelAcquire(node);
            throw t;
        } finally {
            if (interrupted)
                selfInterrupt();
        }
    }    

    // static final class FairSync extends Sync 

        protected int tryAcquireShared(int acquires) {
            for (;;) {
                if (hasQueuedPredecessors())
                    return -1;
                int available = getState();
                int remaining = available - acquires;
                if (remaining < 0 ||
                    compareAndSetState(available, remaining))
                    return remaining;
            }
        }
        // 通过循环和CAS保证同步状态线程安全释放
        protected final boolean tryReleaseShared(int releases) {
            for (;;) {
                int current = getState();
                int next = current + releases;
                if (next < current) // overflow
                    throw new Error("Maximum permit count exceeded");
                if (compareAndSetState(current, next))
                    return true;
            }
        }
```

- 独占式超时获取同步状态

![](https://upload-images.jianshu.io/upload_images/23716394-4816dc319b6bcf1f.png?imageMogr2/auto-orient/strip|imageView2/2/w/594/format/webp)

独占式超时获取同步状态相较于独占式获取同步状态,在未获取到同步状态时增加了指定时间的wait等待状态,该状态中要么被唤醒要么等待超时后再继续尝试获取同步状态,资源利用效率更高


### 重入锁

重入锁: 支持一个线程对资源的重复加锁.synchronized隐式支持冲进入.线程在显示调用ReentrantLock.lock()方法获取到锁后再调用lock()方法获取锁而不被阻塞.    
公平锁: 先申请获取锁的线程请求先被满足;    
非公平锁: 不看申请锁顺序,先抢到的先被满足.    
公平锁机制为了保证锁获取的顺序,实现效率没有非公平锁高.

- 实现重进入

重进入: 任意线程在获取锁后能再次获取该锁而不被锁阻塞.核心问题:    
1 线程再次获取锁.识别线程是当前锁拥有线程    
2 锁的释放.锁内部维护了锁重进入计数,获取时递增,释放时递减.锁为0表示锁最终释放成功.

- 公平与非公平获取锁的区别

公平: FIFO.具体实现上非公平锁只要cas修改锁状态成功即获取了锁.但是公平锁需要判断无前驱节点(!hasQueuedPredecessors())+cas才能获取锁.    
刚释放锁的线程再拥有锁的概率比较大.由于线程获取锁是需要当前线程释放锁后,其他线程才能感知到锁状态变化,而作为释放锁线程,它再次拥有锁就拥有最直接的状态变化感知能力,因此在
锁竞争中拥有优势.    
非公平锁执行效率比公平锁要高.由于刚释放锁的线程再拥有锁的概率大,所以隐形中线程上下文切换就会变少,执行效率就会高

### 读写锁

读写锁维护了一对锁,一个读锁,一个写锁.    
1 通过分离读锁和写锁,使得并发性比排他锁大大增加.读的场景比写的场景多,共享读可以减少程序阻塞    
2 读写锁简化读写交互场景.读时获取读锁,写时获取写锁.写锁获取后阻塞后续的读写操作,写锁释放后,后续操作可以继续.实现机制比等待通知机制更简单明了

> 读锁的价值:    
与写锁互斥,避免脏读.任何锁表面上是互斥，但本质是都是为了避免原子性问题（如果程序没有原子性问题，那只用volatile来避免可见性和有序性问题就可以了，效率更高）.    
读锁自然也是为了避免原子性问题，比如一个long型参数的写操作并不是原子性的，如果允许同时读和写，那读到的数很可能是就是写操作的中间状态，
比如刚写完前32位的中间状态。long型数都如此，而实际上一般读的都是复杂的对象，那中间状态的情况就更多了。
(实际由于线程栈的存在,如果不使用读锁,当写操作后,线程会直接读取栈数据,可能会读取到旧值)

#### 读写锁的实现分析

读写锁ReentrantReadWriteLock的实现: 读写状态的设计,写锁的获取与释放,读锁的获取与释放,锁降级

- 读写状态的设计

同步器的核心是同步状态的维护,读写锁的读写状态就是其同步器的同步状态.同步状态表示锁被一个线程重复获取的次数.读写锁的同步状态需要维护多个读线程和一个写线程的状态.
为了实现状态操作的原子性,当前比较好的方式将一个变量切分为两部分,比如使用整型变量来维护同步状态: 高16位部分表示读,低16位部分表示写.

读写锁状态划分方式:    
![读写锁状态划分方式](https://img1.baidu.com/it/u=1280169129,1093477081&fm=253&fmt=auto&app=138&f=JPEG?w=650&h=286)

当前同步状态表示一个线程获取了写锁,且重进入了两次(写状态=3);同时连续两次获取了读锁.实际读写状态的获取通过位运算进行.同步状态为S,则    
写状态= S & 0x0000FFFF(将高位抹去)    
读状态= S >>> 16(无符号补0右移16位)    
写状态 + 1 = S + 1    
读状态 + 1 = S + (1 << 16)

S不等于0时,当写状态(S & 0x0000FFFF)等于0,则读状态大于0 -> 即读锁已被获取

- 写锁的获取与释放

写锁是一个支持重进入的排他锁.    
1 当前线程获取了写锁,则增加写状态(重进入)    
2 当前线程获取写锁时,读锁已经被获取(读状态不为0)或写锁被其他线程获取,则进入等待状态

```
@ReservedStackAccess
        protected final boolean tryAcquire(int acquires) {
            /*
             * Walkthrough:
             * 1. If read count nonzero or write count nonzero
             *    and owner is a different thread, fail.
             * 2. If count would saturate, fail. (This can only
             *    happen if count is already nonzero.)
             * 3. Otherwise, this thread is eligible for lock if
             *    it is either a reentrant acquire or
             *    queue policy allows it. If so, update state
             *    and set owner.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            int w = exclusiveCount(c);
            if (c != 0) {
                // 读锁判断标准,与读锁互斥: (Note: if c != 0 and w == 0 then shared count != 0)
                if (w == 0 || current != getExclusiveOwnerThread())
                    return false;
                if (w + exclusiveCount(acquires) > MAX_COUNT)
                    throw new Error("Maximum lock count exceeded");
                // Reentrant acquire
                setState(c + acquires);
                return true;
            }
            if (writerShouldBlock() ||
                !compareAndSetState(c, c + acquires))
                return false;
            setExclusiveOwnerThread(current);
            return true;
        }
```

写锁与读锁互斥原因: 读写锁要确保写锁的操作对读锁可见,如果允许读锁在已被获取的情况下获取写锁,那么正在运行的其他读线程无法感知到当前写线程的操作.
因为写线程在释放锁时,才会将缓存数据回写主内存,这时其他线程才可见写结果.

写锁释放: 每次释放均减少写状态,当写状态为0时,表示写锁释放成功.步骤与ReentrantLock类似.

- 读锁的获取与释放

读锁是一个支持重进入的共享锁,它能被多个线程同时获取,在没有其他写线程访问(写状态为0)时,读锁总会被成功获取,只要递增读状态即可.    
如果当前线程已获取了读锁,则增加读状态;如果当前线程在获取读锁时,写锁已被其他线程获取,则进入等待状态.

```
@ReservedStackAccess
        protected final int tryAcquireShared(int unused) {
            /*
             * Walkthrough:
             * 1. If write lock held by another thread, fail.
             * 2. Otherwise, this thread is eligible for
             *    lock wrt state, so ask if it should block
             *    because of queue policy. If not, try
             *    to grant by CASing state and updating count.
             *    Note that step does not check for reentrant
             *    acquires, which is postponed to full version
             *    to avoid having to check hold count in
             *    the more typical non-reentrant case.
             * 3. If step 2 fails either because thread
             *    apparently not eligible or CAS fails or count
             *    saturated, chain to version with full retry loop.
             */
            Thread current = Thread.currentThread();
            int c = getState();
            if (exclusiveCount(c) != 0 &&
                getExclusiveOwnerThread() != current)
                return -1;
            int r = sharedCount(c);
            if (!readerShouldBlock() &&
                r < MAX_COUNT &&
                compareAndSetState(c, c + SHARED_UNIT)) {
                if (r == 0) {
                    firstReader = current;
                    firstReaderHoldCount = 1;
                } else if (firstReader == current) {
                    firstReaderHoldCount++;
                } else {
                    HoldCounter rh = cachedHoldCounter;
                    if (rh == null ||
                        rh.tid != LockSupport.getThreadId(current))
                        cachedHoldCounter = rh = readHolds.get();
                    else if (rh.count == 0)
                        readHolds.set(rh);
                    rh.count++;
                }
                return 1;
            }
            return fullTryAcquireShared(current);
        }
```

1 其他线程获取了写锁则锁获取失败    
2 当前线程获取读锁不阻塞且cas成功则获取读锁

读锁释放: 每次递减读状态

- 锁降级

锁降级: 写锁降级为读锁.当前线程拥有写锁后,再获取到读锁,随后释放写锁的过程.需要在一个原子操作中完成.    
场景: 线程需要实时通知其他线程数据变动(写锁),且要保证其他线程数据变动自己可见(读锁)

写锁与读锁互斥,但读锁与写锁不互斥: 当前线程可以同时拥有读锁和写锁.但是如果要同时拥有两个锁,必须是先拥有写锁,再拥有读锁(锁降级).因为只有这时才能保证数据的可见性和一致性.

锁降级示例:    
```
    public void lockDown(){
        readLock.lock();
        if(!update) {
            // 写锁与读锁互斥(不支持锁升级),必须要先释放读锁,才能获取写锁
            readLock.unlock();
            // 获取写锁
            writeLock.lock();
            try{
                if(!update) { 
                    // 写锁价值,更新共享变量
                    update = true;
                }
                readLock.lock(); //降级为读锁,读锁是为了获取一致的共享变量
            } finally {
                // 释放写锁,通知其他线程共享变量已更新
                writeLock.unlock();
            }
            // 锁降级成功,写锁降级为读锁.
        }
        try{
            // 共享变量读取操作
        }finally {
            readLock.unlock();
        }
    }
```

### LockSupport

LockSupport支持方法:    
![LockSupport支持方法](https://img-blog.csdnimg.cn/70712bdeb95c4b8f9c1f7624392df400.png)

上述方法不仅可以使当前线程进入等待状态，还可以设置一个对象赋给线程的Blocker对象，这个对象一般用于问题排查和系统的监控。

需要注意的是，在调用park方法并传入blocker对象后，只要该线程一直处于等待状态，都可以通过getBlocker方法获得该对象，但是当线程脱离上述状态后，就会将parkBlocker成员变量设为null。

LockSupport的park方法和unpark方法在JDK1.8中是基于sun.misc.Unsafe实现的，Unsafe直接提供了park和unpark方法，它们都是native方法，并在JVM层实现。

在上文我们对比Object和LockSupport时，我们提到了LockSupport的唤醒和等待可以乱序调用而不会时线程进入等待状态。这是因为Thread底层的Parker对象维护了一个许可，当首先调用unpark方法时，相当于给了这个线程一个许可，当再次调用park方法时，线程发现已经持有了这个许可后就不会进入等待状态了；如果线程发现没有许可，就会进入等待状态


- LockSupport.park与Object.wait区别:    

共同点    
LockSupport 中的park方法和Object中的wait方法都可以使线程进入WAIT或者TIMED_WAIT状态    
LockSupport中的unpark方法和Object中的notify可以使线程脱离WAIT、TIMED_WAIT状态    
二者都可以通过调用线程的interrupt方法脱离等待状态    
两者都是通过JVM层，也就是native代码实现的    

不同点    
Object中的wait方法在调用时当前线程必须要对该Object进行加锁，否则会抛出IllegalMonitorException。而LockSupport无需加锁，直接调用其静态方法park就可以使当前线程进入阻塞状态。    
Object中wait和notify方法必须要按顺序调用，如果因为线程调度问题导致线程A先调用notify方法而线程B后调用wait方法，那么会使线程A永远处于WAIT状态。对于LockSupport而言则没有这种限制，如果有线程A首先调用了unpark方法并传入了线程B的引用，然后线程B再调用了park方法，那么线程B是不会进入等待状态的。    
调用Object的wait方法后，可以调用该线程的interrupt方法脱离等待状态并捕获InterruptedException。而LockSupport并不能捕获InterruptedException。    

### Condition 接口

Condition 接口定义一组等待/通知的方法.类似于Object的监视器方法(wait(),wait(long timeout),notify,notifyAll()).

Object 监视器方法与Condition接口对比:    
![Object 监视器方法与Condition接口对比](https://dandelioncloud.cn/images/20210813/3a52c1db43ef4300ae04dc9ba3d9c71f.png)

Condtion 接口方法:    
![](https://dandelioncloud.cn/images/20210813/cc5f578445b84843a9633c1858d505d4.png)

Condtion 使用示例:   
```
public class BoundedQueue {
    private Object[] items;
    // 添加的下标，删除的下标和数组当前数量
    private int addIndex, removeIndex, count;
    // Condition的使用依赖于Lock锁对象
    private Lock lock = new ReentrantLock();
    private Condition notEmpty = lock.newCondition();
    private Condition notFull = lock.newCondition();

    public BoundedQueue(int size) {
        items = new Object[size];
    }

    // 添加一个元素，如果数组满，则添加线程进入等待状态，直到有"空位"
    public void add(T t) throws InterruptedException {
        lock.lock();
        try {
            while (count == items.length)
                notFull.await();
            items[addIndex] = t;
            if (++addIndex == items.length)
                addIndex = 0;
            ++count;
            notEmpty.signal();
        } finally {
            lock.unlock();
        }
    }

    // 由头部删除一个元素，如果数组空，则删除线程进入等待状态，直到有新添加元素
    @SuppressWarnings("unchecked")
    public T remove() throws InterruptedException {
        lock.lock();
        try {
            while (count == 0)
                notEmpty.await();
            Object x = items[removeIndex];
            if (++removeIndex == items.length)
                removeIndex = 0;
            --count;
            notFull.signal();
            return (T) x;
        } finally {
            lock.unlock();
        }
    }
```

- Condition 实现分析

由于Condition 都是锁相关的操作,所以它的实现通过 AbstractQueuedSynchronizer 的内部类 ConditionObject 来实现.每个 ConditionObject 都包含一个队列(等待队列),入队即等待,出队即通知.

Condition 的实现分三部分: 等待队列,等待,通知

1 等待队列

等待队列是一个FIFO队列,在队列的每个节点都包含了一个线程引用,该线程就是在 ConditionObject 上等待的线程.如果一个线程调用了Condition.await()方法,那么该线程就是释放锁,构造成节点并加入等待队列并进入等待状态.等待队列的节点与同步队列的节点使用了同一个静态内部类AbstractQueuedSynchronizer.Node.    

等待队列结构    
![等待队列结构](https://images2015.cnblogs.com/blog/731716/201702/731716-20170220185902382-488345990.png)


Object监视器模型上,一个对象拥有一个同步队列和等待队列;而Lock(同步器)拥有一个同步队列和多个等待队列:    
每调用一次lock.newCondition() 都会创建一个Condition 对象.可以创建多个   
```
    // ReentrantLock
    final ConditionObject newCondition() {
        return new ConditionObject();
    }
```
Lock(同步器)监视器模型    
![Lock(同步器)监视器模型](https://images2015.cnblogs.com/blog/731716/201702/731716-20170220190001757-480946370.png)

Condtion 的实现 ConditionObject 是同步器的内部类,它可以访问同步器的内部方法,相当于每个ConditionObject都有所属同步器的引用

2 等待

当调用 Condtion.await()方法时,会使当前线程进入等待状态并释放锁,同时线程状态变为等待状态.当从await()方法返回时,当前线程一定获取了Condtion 相关联的锁(LockSupport.park(this)).

```
    /**
        * Implements interruptible condition wait.
        * <ol>
        * <li>If current thread is interrupted, throw InterruptedException.
        * <li>Save lock state returned by {@link #getState}.
        * <li>Invoke {@link #release} with saved state as argument,
        *     throwing IllegalMonitorStateException if it fails.
        * <li>Block until signalled or interrupted.
        * <li>Reacquire by invoking specialized version of
        *     {@link #acquire} with saved state as argument.
        * <li>If interrupted while blocked in step 4, throw InterruptedException.
        * </ol>
        */
    public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        Node node = addConditionWaiter();
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        while (!isOnSyncQueue(node)) {
            // 线程进入等待状态.只有获取到锁,才能从阻塞方法中返回
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }

    private Node addConditionWaiter() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node t = lastWaiter;
        // If lastWaiter is cancelled, clean out.
        if (t != null && t.waitStatus != Node.CONDITION) {
            unlinkCancelledWaiters();
            t = lastWaiter;
        }

        // 创建节点时,会关联当前线程
        Node node = new Node(Node.CONDITION);
        // 由于线程已经获取锁,原子性由锁来保证,不需要CAS
        if (t == null)
            firstWaiter = node;
        else
            t.nextWaiter = node; 
        lastWaiter = node;
        return node;
    }
```
只有成功获取了锁的线程(同步队列的首节点)才能调用此方法.    
1 该方法中会将当前线程构造成等待节点加入等待队列    
2 然后释放同步状态(唤醒后继节点);    
3 当前线程进入等待状态.    
4 等待队列中节点被唤醒后尝试获取同步状态    
5 如果不是通过其他线程调用 Condition.signal()方法唤醒,而是对等待线程进行中断,则抛出 InterruptedException 

await()阻塞在同步器的节点流转    
![await()阻塞在同步器的节点流转](https://imgedu.lagou.com/f988245867544b0bae7e6f76ac4ec2a4.jpg)


3 通知

Condition.signal():唤醒等待队列中的首节点,在唤醒节点前,会将节点移到同步队列中.

```
    public final void signal() {
        if (!isHeldExclusively())
            throw new IllegalMonitorStateException();
        Node first = firstWaiter;
        if (first != null)
            doSignal(first);
    }
    private void doSignal(Node first) {
        do {
            if ( (firstWaiter = first.nextWaiter) == null)
                lastWaiter = null;
            first.nextWaiter = null;
        } while (!transferForSignal(first) &&
                    (first = firstWaiter) != null);
    }
        final boolean transferForSignal(Node node) {
        /*
         * If cannot change waitStatus, the node has been cancelled.
         */
        if (!node.compareAndSetWaitStatus(Node.CONDITION, 0))
            return false;

        /*
         * Splice onto queue and try to set waitStatus of predecessor to
         * indicate that thread is (probably) waiting. If cancelled or
         * attempt to set waitStatus fails, wake up to resync (in which
         * case the waitStatus can be transiently and harmlessly wrong).
         */
        Node p = enq(node);
        int ws = p.waitStatus;
        if (ws > 0 || !p.compareAndSetWaitStatus(ws, Node.SIGNAL))
            LockSupport.unpark(node.thread);
        return true;
    }
```

1 只有获取锁的线程才能进入该方法    
2 将等待队列首节点移到同步队列尾节点   
3 LockSupport唤醒等待线程
4 被唤醒线程,将从await()方法中while循环退出(isOnSyncQueue(node)=true),节点已在同步队列中    
5 调用同步器acquireQueued(node, savedState)方法竞争同步状态
6 成功获取同步状态(锁)之后,被唤醒的线程将从await()方法返回,线程已成功获取锁并进入正常的业务执行流程


Condition.signalAll(): 将等待队列中所有节点都执行signal()方法,相当于将等待队列中所有节点都放入同步队列中,并唤醒节点对应的线程




