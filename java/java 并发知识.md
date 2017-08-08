---
title: java 并发知识
tags: java
grammar_cjkRuby: true
---

### volatile 使用优化

追加字节到64字节，这样占用一个高速缓存行，这样在处理的时候会将整个缓存行锁定而不会影响其他处理器的访问。

### 锁

> 锁信息储存在java对象头的 Mark Word 中。

#### 锁的种类
- 偏向锁
- 轻量级锁
- 重量级锁

以上锁的级别是逐渐升级的，锁可以升级但不能降级。
> 锁的优缺点对比

锁 | 优点 | 缺点 | 适用场景
-- | -- | -- | --
偏向锁 | 加锁和解锁不需要额外的消耗 | 如果线程间存在锁竞争，会有锁撤销的消耗 |适用于只有一个线程访问的同步块场景
轻量级锁 | 竞争的线程不会阻塞 |如果始终得不到锁竞争的线程，使用自旋会消耗CPU | 追求响应时间、同步块执行速度非常快
重量级锁 | 线程竞争不使用自旋，不会消耗CPU | 线程阻塞，响应时间缓慢 | 追求吞吐量、同步块执行速度较长

### 处理器如何实现原子操作
- 使用总线锁保证原子性
- 使用缓存锁保证原子性

### java如何实现原子操作

通过锁和循环CAS的方式来实现原子操作

### 三种重排序

- 编译器优化重排序
- 指令级并行的重排序
- 内存系统的重排序

发生重排序，是在单线程操作其执行结果不会发生变化。有数据依赖关系的不会发生重排序，有三种类型

名称 | 代码示例 | 说明
--- | -- | --
写后读 | a = 1;b = a; | 写一个变量后，再读这个位置
写后写 | a = 1;a = 2; | 写一个变量之后，再写这个变量
读后写 | a = b;b = 1; | 读一个变量之后，再写这个变量

### volatile 重排序规则
- 在每一个volatile写操作的前面插入一个StoreStore屏障
- 在每一个volatile写操作的后面插入一个StoreLoad屏障
- 在每一个volatile读操作的后面插入一个LoadLoad屏障
- 在每一个volatile读操作的后面插入一个LoadStore屏障

### final域的重排序规则
- 在构造函数内对一个final域的写入，与随后把这个被构造对象的引用赋值给一个引用变量，这两个操作之间不能重排序。
- 初次读一个包含final域的对象的引用，与随后初次读这个final域。这两个操作之间不能重排序。

> concurrent包的基础实现都是由volatile变量 和CAS 为基础实现的。

### 两种线程安全实例化

#### 双重检查锁定
```java
	public class SafeDoubleCheckedLocking{
		private volatile static Instance instance;
		
		public static Instance getInstance() {
			if (instance == null) {
				synchronized (SafeDoubleCheckedLocking.class) {
					if (instance == null) {
						instance = new Instance();
					}
				}
			}
			return instance;
		}
	}
```
#### 基于类初始化的解决方案
```java
	public class InstanceFactory{
		private static class InstanceHolder {
			public static Instance instance = new Instance();
		}
		
		public static Instance getInstance(){
			return InstanceHolder.instance;
		}
	}

```

