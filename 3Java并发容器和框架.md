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




















