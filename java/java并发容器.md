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

- ArrayBlockingQueue:一个由数组结构组成的有界阻塞队列
- LinkedBlockingQueue:一个由链表结构组成的有界阻塞队列
- PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列
- SynchronousQueue:一个不储存元素的阻塞队列
- DelayQueue:一个使用优先级队列实现的无界阻塞队列
- LinkedTransferQueue:一个由链表结构组成的无界阻塞队列
- LinkedBlockingDeque:一个由链表结构组成的双向阻塞队列

