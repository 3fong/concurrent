## Java并发容器和框架


### ConcurrentHashMap 原理与使用

ConcurrentHashMap 锁分段技术来提升并发访问率.通过增加容器锁来避免整体锁定容器,以增加并发度.首先将数据分成一段一段的存储,然后给每一段数据配一把锁,当一个线程占用锁访问其中一个段数据时,其他段数据依旧可以正常访问.

HashTable: 使用sychronized实现,由于是全局排他锁,当一个线程获取锁后,其他线程访问任何方法都会进入阻塞状态.    

HashMap: 并发不安全,jdk7在并发中可能导致程序死循环和数据覆盖.put方法resize()中transfer()扩容操作可能导致HashMap.Entry链表形成环形结构,进而造成死循环.
jdk8修复了死循环问题,但是数据覆盖依旧存在.只要是共享变量写操作的地方都可能出现数据覆盖

hashmap死锁测试示例:    
```
// jdk7测试可以出现死循环问题和数据覆盖;jdk8测试依旧会出现数据覆盖问题
public static void main(String[] args) throws InterruptedException {
        Map<String,String> map = new HashMap<>(2);
        Thread t = new Thread(new Runnable() {
            @Override
            public void run() {
                for (int i = 0; i < 30000; i++) {
                    new Thread(new Runnable() {
                        @Override
                        public void run() {
                            map.put(UUID.randomUUID().toString(),"");
                        }
                    },"ftf"+i).start();
                }
            }
        },"ftf");
        t.start();
        t.join();
    }
```

- ConcurrentHashMap 的结构

ConcurrentHashMap 分段锁结构:    
![](https://upload-images.jianshu.io/upload_images/17755742-0aeb208cbf2192f9.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/502/format/webp)

ConcurrentHashMap由一组Segment组成,每个segment是一种可重入锁(ReentrantLock);segment的结构与hashmap类似,是一种数组+链表结构.一个segment由一组HashEntry组成,每个HashEntry是一个链表结构元素.每个Segment守护着一个HashEntry数组里的元素.当对HashEntry元素修改时,必须先获取它对应的Segment锁.

ConcurrentHashMap CAS+sychronized 锁结构(jdk1.8):    
![](https://upload-images.jianshu.io/upload_images/17755742-84349c0ca1005c43.png?imageMogr2/auto-orient/strip|imageView2/2/w/446/format/webp)

由CAS+sychronized 来保证并发安全,数组+链表+红黑树来实现数据存取

- ConcurrentHashMap 的初始化

ConcurrentHashMap 的初始化即是Segment的创建数量,segment的数量按2的倍数递增,数量最大是2^16次方=65536

segmentShift:参与散列运算的位数      
segmentMask:散列运算掩码   

两个参数用于散列hash值运算,减少hash冲突

- ConcurrentHashMap 操作

get(key): 

通过volatile 声明共享变量,保持线程间可见性.比锁的性能高,且实现简单.但是当读到的值为null时,会加锁重读.定位HashEntry的散列算法与与定位Segment的散列算法不一样,以避免两次hash后的值一样.

定位segment: (hash >>> segmentShift) & segmentMask    
定位HashEntry: hash & (tab.length - 1)

put(key,value):(*) 

需要加锁保证共享变量安全写入.put首先定位到segment,然后在segment里进行插入操作.插入步骤:   
1 判断segment里 HashEntry数组是否扩容:先判断HashEntry数组是否超过容量后再扩容;而hashmap是先添加元素再扩容,存在空扩容的问题     
2 对指定segment里HashEntry数组扩容后插入新元素:2倍扩容后,原数组元素再hash后插入新数组里(分散).不会对整个容器扩容

size:

jdk1.7中 ConcurrentHashMap 先尝试两次遍历求和segment.modCount不相同,才会锁住所有segment;modCount共享变量只会在写操作(加锁)才会更新,以实现线程间共享变化.

### ConcurrentlinkedQueue

一个安全的队列有两种实现方式: 阻塞算法实现的阻塞队列;非阻塞算法实现的非阻塞队列.    
ConcurrentlinkedQueue 是一种非阻塞队列,基于链接节点的无界线程安全队列,采用先进先出的规则对节点进行排序.通过"wait-free"(CAS算法)实现.

ConcurrentlinkedQueue 结构:    
![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fimage93.360doc.com%2FDownloadImg%2F2016%2F01%2F2121%2F64842160_1.jpg&refer=http%3A%2F%2Fimage93.360doc.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1659056894&t=653b7ad50f0d4850860f2dd5954cb4cb)

ConcurrentlinkedQueue 由head节点和tail节点组成,每个节点(Node)由节点元素(item)和指向下一个节点(next)的引用组成,节点与节点间就是通过这个next关联起来,组成一张链表结构的队列.默认情况下head节点存储的元素为空,tail节点等于head节点.

- 入队

入队就是将入队节点添加到队列的尾部.两个步骤:    
1 将入队节点设置成当前队列尾节点的下一个节点;    
2 更新tail节点.如果tail节点的next节点不为空,则将入队节点设置成tail节点;如果tail节点的next节点为空,则将入队节点设置成tail的next节点.->tail节点不总是尾节点


JDK1.8 入队实现方法:    
```
 public boolean offer(E e) {
        final Node<E> newNode = new Node<E>(Objects.requireNonNull(e));
        // p 尾节点实时位置临时变量;t是当前时点尾节点变量
        for (Node<E> t = tail, p = t;;) {
            Node<E> q = p.next;
            // 当尾节点的next为空时,将新节点链入p.next=newNode
            if (q == null) {
                // p is last node
                if (NEXT.compareAndSet(p, null, newNode)) {
                    // Successful CAS is the linearization point
                    // for e to become an element of this queue,
                    // and for newNode to become "live".
                    if (p != t) // hop two nodes at a time; failure is OK
                        TAIL.weakCompareAndSet(this, t, newNode);
                    return true;
                }
                // Lost CAS race to another thread; re-read next
            }
            // p.next==p==null时
            else if (p == q)
                // p已经离链(被删除).We have fallen off list.  If tail is unchanged, it
                // will also be off-list, in which case we need to
                // jump to head, from which all live nodes are always
                // reachable.  Else the new tail is a better bet.
                // 将临时变量t与赋值为tail后的t比较是否相等.即实时获取tail数据与历史t数据是否相等.相等则t已经离链,跳转到head,从头开始遍历链表;不相等则是其他线程修改了tail,直接返回赋值为tail后t
                p = (t != (t = tail)) ? t : head;
            else
                // Check for tail updates after two hops.
                // 返回最新的尾节点.其他线程修改了tail数据且tail被删除则返回新的tail节点,否则返回p.next节点
                p = (p != t && t != (t = tail)) ? t : q;
        }
    }
```


- 出队

```
    public E poll() {
        restartFromHead: for (;;) {
            for (Node<E> h = head, p = h, q;; p = q) {
                final E item;
                if ((item = p.item) != null && p.casItem(item, null)) {
                    // Successful CAS is the linearization point
                    // for item to be removed from this queue.
                    if (p != h) // hop two nodes at a time
                        updateHead(h, ((q = p.next) != null) ? q : p);
                    return item;
                }
                else if ((q = p.next) == null) {
                    updateHead(h, p);
                    return null;
                }
                else if (p == q)
                    continue restartFromHead;
            }
        }
    }
```

#### Java阻塞队列

阻塞队列(BlockingQueue): 一个支持阻塞插入和阻塞移除的队列.    
阻塞插入: 当队列满时,队列会阻塞插入元素的线程,直到队列不满    
阻塞移除: 当队列为空时,队列阻塞元素移除,直到队列非空

常用场景: 生产者,消费者

阻塞队列插入和移除操作的4种处理方式:    
![](https://img-blog.csdnimg.cn/img_convert/9a4c36ef5895dd57370ea566e7c81956.png)

阻塞队列
```
ArrayBlockingQueue: 数组构成的有界队列FIFO     
LinkedBlockingQueue: 链表构成的有界队列FIFO
PriorityBlockingQueue: 支持优先级排序的无界阻塞队列    
DelayQueue:支持延迟获取元素的无界阻塞队列    
SynchronousQueue: 不存储元素的阻塞队列,必须等待放入元素被取走,才能继续放入元素.支持公平访问.吞吐高于ArrayBlockingQueue,LinkedBlockingQueue    
LinkedTransferQueue: 链表构成的无界阻塞队列    
LinkedBlockingDeque: 链表构成的双向阻塞队列
```

- DelayQueue

队列元素必须实现Delayed接口,在创建时可指定延迟获取元素时间,延迟期满才会获取元素.使用PriorityQueue实现

适用场景:    
1 缓存系统.延迟时间是有效期满时间    
2 定时任务.延迟期满即任务执行.TimerQueue是使用DelayQueue实现的

使用方式(可参考ScheduledThreadPoolExecutor.ScheduledFutureTask)    
1 创建队列,初始化基本数据.延迟时间time,队列顺序sequenceNumber    
2 实现getDelay方法,定义返回当前元素还需的延迟时间,单位纳秒    
3 实现compareTo()指定元素顺序

#### 阻塞队列实现原理

队列只是一个中间状态存储,如何有效的通知线程间队列状态的感知,是队列效率的核心.阻塞队列采取的方式是: 通知模式.    
通知模式: 生产者添加队列元素,队列满时会阻塞生产者添加,当消费者消费了一个队列中的元素后,会通知生产者当前队列可用.

使用Condtion.await()->LockSupport.park() -> unsafe.park

unsafe.park退出场景:    
1 unpark执行    
2 线程被中断    
3 延时到期    
4 触发异常,异常现象没有任何原因










