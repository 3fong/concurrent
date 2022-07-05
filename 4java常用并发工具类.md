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













