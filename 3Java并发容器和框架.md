## Java并发容器和框架


### ConcurrentHashMap 原理与使用

ConcurrentHashMap 锁分段技术来提升并发访问率.通过增加容器锁来避免整体锁定容器,以增加并发度.首先将数据分成一段一段的存储,然后给每一段数据配一把锁,当一个线程占用锁访问其中一个段数据时,其他段数据依旧可以正常访问.

HashTable: 使用sychronized实现,由于是全局排他锁,当一个线程获取锁后,其他线程访问任何方法都会进入阻塞状态.    

HashMap: 并发不安全,在并发中可能导致程序死循环.put操作可能导致HashMap.Entry链表形成环形结构,进而造成死循环

hashmap死锁测试示例:    
```
// 实际测试没有生效
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


































