### ConcurrentHashMap

> 线程安全的HashMap

### ConcurrentLinkedQueue
> 线程安全的队列


### Java中阻塞队列

> 一个支持两个附加操作的队列。这两个附加操作支持阻塞插入和移除

- 支持阻塞插入方法：当队列满时，队列会阻塞插入元素的线程，直到队列不满
- 支持阻塞移除方法：当队列为空时，获取元素的线程会等待队列变为非空。

> 阻塞队列常用于生产者和消费者的场景。

方法/处理方式 | 抛出异常 | 返回特殊值 | 一直阻塞 | 超时退出
-- | -- |-- | --|--
插入方法 | add(e) | offer(e) | put(e) | offer(e,time,unit)
移除方法 | remove() | poll() | take() | poll(time.unit)
检查方法 | element() | peek() | 不可用 | 不可用

> 阻塞队列

- ArrayBlockingQueue:一个由数组结构组成的有界阻塞队列
- LinkedBlockingQueue:一个由链表结构组成的有界阻塞队列
- PriorityBlockingQueue:一个支持优先级排序的无界阻塞队列
- DelayQueue:一个使用优先级队列实现的无界阻塞队列
- SynchronousQueue:一个不存储元素的阻塞队列
- LinkedTransferQueue:一个由链表结构组成的无界阻塞队列
- LinkedBlockingDeque:一个由链表结构组成的双向阻塞队列

### 原子操作类

- AtomicBoolean：原子更新布尔类型
- AtomicInteger：原子更新整型
- AtomicLong：原子更新长整型
- AtomicIntegerArray：原子更新整型数组里的元素
- AtomicLongArray：原子更新长整型数组里的元素
- AtomicReferenceArray：原子更新引用类型数组里的元素
- AtomicReference：原子更新引用类型
- AtomicReferenceFieldUpdater：原子更新引用类型里的字段
- AtomicMarkableReference：原子更新带标记位的引用类型
- AtomicIntegerFieldUpdater：原子更新整型的字段的更新器
- AtomicLongFieldUpdater：原子更新长整型字段的更新器
- AtomicStampedReference：原子更新带有版本号的引用类型

### 并发工具类

> CountDownLatch允许一个或多个线程等待其他线程完成操作
> CyclicBarrier让一组线程到达一个相同的屏障后，再同时进行
> Semaphore用来控制同时访问特定资源的线程数量

#### CountDownLatch 和 CyclicBarrier 区别

- CountDownLatch计数器只能使用一次，CyclicBarrier计数器可以使用reset()方法重置
- CyclicBarrier其他方法.getNumberWaiting方法可以获得CyclicBarrier阻塞的线程数量。isBroken()方法用来了解阻塞的线程是否被中断。
