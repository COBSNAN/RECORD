---
title: java并发容器
tags: java
grammar_cjkRuby: true
---

###  ConcurrentHashMap
> ConcurrentHashMap使用分段锁技术是其相较于HashTable效率提高

ConcurrentHashMap初始化方法是通过initialCapacity、loadFactor和concurrencyLevel等几个参数来初始化segment数组。段偏移量segmentShift、段掩码segmentMask和每个segment里的HashEntry数组来实现。

jdk1.8中的代码
```
 private void writeObject(java.io.ObjectOutputStream s)
        throws java.io.IOException {
        // For serialization compatibility
        // Emulate segment calculation from previous version of this class
        int sshift = 0;
        int ssize = 1;
        while (ssize < DEFAULT_CONCURRENCY_LEVEL) {
            ++sshift;
            ssize <<= 1;
        }
        int segmentShift = 32 - sshift;
        int segmentMask = ssize - 1;
        @SuppressWarnings("unchecked")
        Segment<K,V>[] segments = (Segment<K,V>[])
            new Segment<?,?>[DEFAULT_CONCURRENCY_LEVEL];
        for (int i = 0; i < segments.length; ++i)
            segments[i] = new Segment<K,V>(LOAD_FACTOR);
        s.putFields().put("segments", segments);
        s.putFields().put("segmentShift", segmentShift);
        s.putFields().put("segmentMask", segmentMask);
        s.writeFields();

        Node<K,V>[] t;
        if ((t = table) != null) {
            Traverser<K,V> it = new Traverser<K,V>(t, t.length, 0, t.length);
            for (Node<K,V> p; (p = it.advance()) != null; ) {
                s.writeObject(p.key);
                s.writeObject(p.val);
            }
        }
        s.writeObject(null);
        s.writeObject(null);
        segments = null; // throw away
    }
```
### ConcurrentLinkedQuene
> 一个基于链接节点的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部。

HOPS 值，目的是不用每次都使用CAS更新，表示tail和尾节点的距离最大不草果常量HOPS.

### 阻塞队列
方法/处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出
-- | -- | -- | -- | --
插入方法 | add(e) | offer(e) | offer(e,time,unit)
移除方法 | remove() | poll() | take() | poll(time,unit)
检查方法 | element() | peek() | 不可用 | 不可用、

- ArrayBlockingQueue:一个由数组结构组成的有界阻塞队列。先进先出的原则对元素进行排序
- LinkedBlockingQueue:一个由链表结构组成的有界阻塞队列。默认最大长度Integer.MAX_VALUE
- PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列.元素默认排序采用自然顺序升序排列。可以元素自定义compareTo()方法或构造参数Comparator来对元素进行排序
- SynchronousQueue:一个不储存元素的阻塞队列。每一个put操作必须等待一个take操作。
- DelayQueue:一个使用优先级队列实现的无界阻塞队列。只有在延迟期满时才能从队列中提取元素。
- LinkedTransferQueue:一个由链表结构组成的无界阻塞队列。多了tryTransfer和transfer方法。transfer方法当有消费者正在等待接受元素时，transfer方法会把生产者立刻给消费者，并等待**该元素被消费了才返回**。tryTransfer方法是当没有等待消费者，直接返回false，否则返回true。
- LinkedBlockingDeque:一个由链表结构组成的双向阻塞队列，就是双向队列，可以两头插入删除

### 工具类 CountDownLatch、CyclicBarrier和Semaphore

- CountDownLatch：允许一个或多个线程等待其他线程完成操作，其不能复用
- CyclicBarrier：让一组线程到达一个屏障点，然后一起执行，比如说排队进旅游景点，隔50个人进一波。可以再次使用。
- Semaphore：信号量，比如小孩滑滑梯，一个滑梯的顶部只能容纳几个人，只有等一个小孩滑下来，才能有另一个小孩爬上去。

关于区别一个比较好的网址 ：http://www.cnblogs.com/dolphin0520/p/3920397.html  