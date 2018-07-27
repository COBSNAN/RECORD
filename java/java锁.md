---
title: java中的锁
tags: java
grammar_cjkRuby: true
---

### 分类
- 重入锁 ReentrantLock  通过getExclusiveOwnerThread 来判断当前获取锁得线程，从而实现锁得重入。有一个state字段，来标记加锁状态，重入锁这个字段的值会一直增加
- 读写锁 是一对锁，通过ReentrantReadWriteLock先来定义一个读写锁对象，然后通过readLock() 或writeLock() 方法定义读锁或写锁。其实通过把state变量按位切割使用，高16位表示读，低16位表示写。

### 锁的实现原理
> 基本思想是通过队列同步器 AQS（AbstractQueuedSynchronizer）和CAS操作来实现的。

摘自书上，一个锁的实现
```java
package com.order.thread;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

public class simplyLock implements Lock {
	//静态内部类，自定义同步器
	private static class Sync extends AbstractQueuedSynchronizer {
		//是否处于占用状态
		protected boolean isHeldExclusively() {
			return getState() == 1;
		}
		//当前状态为0的时候获取锁
		public boolean tryAcquire(int acquires) {
			if (compareAndSetState(0, 1)) {
				setExclusiveOwnerThread(Thread.currentThread());
				return true;
			}
			return false;
		}
		//释放锁，将状态设置为0
		protected boolean tryRelease(int releases) {
			if (getState() == 0) throw new IllegalMonitorStateException();
			setExclusiveOwnerThread(null);
			setState(0);//由于是获取锁了才能执行tryRelease方法，所以不用考虑并发问题
			return true;
		}
		//返回一个Condition，每个condition都包含一个condition队列
		Condition newCondition() {return new ConditionObject();}
		
	}
	//仅需要将代理到Sync上即可
	private final Sync sync = new Sync();
	@Override
	public void lock() {
		sync.acquire(1);
		
	}

	@Override
	public boolean tryLock() {
		return sync.tryAcquire(1);
	}

	@Override
	public void unlock() {
		sync.release(1);
	}

	@Override
	public Condition newCondition() {
		return sync.newCondition();
	}
	
	public boolean isLocked() {
		return sync.isHeldExclusively();
	}
	
	public boolean hasQueuedThreads() {
		return sync.hasQueuedThreads();
	}
	
	@Override
	public void lockInterruptibly() throws InterruptedException {
		sync.acquireInterruptibly(1);
	}

	@Override
	public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
		return sync.tryAcquireNanos(1, unit.toNanos(time));
	}

}

```

AQS中Node 节点里 waitStatus 状态来表示线程的等待状态，列出几种状态
- CANCELLED,值为1，由于在同步队列中等待的线程等待超时或者中断，需要从同步等待队列中取消等待，节点进入该状态将不会变化。
- SIGNAL,值为-1，后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行。
- CONDITION,值为-2，节点在等待队列中，节点的线程等待在Condition上，当其他线程对Condition调用了signal()方法后，该节点将会从等待队列中转移到同步队列中，加入对同步状态的获取中。
- PROPAGATE,值为-3，表示下一次共享式同步状态获取将会无条件地被传播下去
- INITIAL,值为0，初识状态。

> ConditionObject 也是由AQS实现的，它维护了一个等待队列。

### 队列同步器

> 实现锁的关键，二者之间的关系：锁是面向使用者的，它定义了使用者于锁交互的接口，隐藏了实现细节；同步器面向的是锁的实现者，它简化了锁的实现方式，屏蔽了同步状态管理、线程的排队、等待与唤醒等底层操作

> 同步队列

##### 节点的属性类型与名称以及描述

属性类型与名称 | 描述
--- | ---
int waitStatus | 等待状态。包含如下状态：1.CANCELLED,值为1，由于在同步队列中等待的线程等待超时或者被中断，需要同步队列中取消等待，节点进入该状态将不会变化。2.SIGNAL,值为-1，后继节点的线程处于等待状态，而当前节点的线程如果释放了同步状态或者被取消，将会通知后继节点，使后继节点的线程得以运行。3.CONDITION,值为-2，节点在等待队列中，节点线程等待在Condition上，当其他线程对Condition调用了signal()方法后，该节点将会从等待队列中转移到同步队列中，加入到对同步状态的获取中。4.PROPAGATE,值为-3,表示下一次共享式同步状态获取将会无条件地传播下去。5.INITIAL,值为0，初始状态
Node prev | 前驱节点，当节点加入同步队列时被设置（尾部添加）
Node next | 后继节点
Node nextWaiter | 等待队列中的后继节点。如果当前节点使共享的，那么这个字段僵尸一个SHARED常量，也就是说节点类型（独占和共享）和等待队列中的后继节点共用一个字段
Thread thread | 获取同步状态的线程

> 同步队列遵循FIFO，首节点是获取同步状态成功的节点，首节点的线程在释放同步状态时，将会唤醒后继节点，而后继节点将会在获取同步状态成功时将自己设置为首节点


### LockSupport工具

> LockSupport 提供的阻塞和唤醒方法

方法名称 | 描述
-- | ---
void park() | 阻塞当前线程，如果调用unpark(Thread thread) 方法或者当前线程被中断，才能从park()方法返回
void parkUntil(long nanos) | 阻塞当前线程，最长不超过nanos纳秒，返回条件在park()的基础上增加了超时返回
void parkUntil(long deadline) | 阻塞当前线程，直到deadline时间（从1970年开始到deadline时间的毫秒数）
void uppark(Thread thread) | 唤醒处于阻塞状态的线程thread

### Condition接口

> Object的监视器方法与Condition接口的对比

对比项 | Object Monitor Methods | Condition
-- | --- | --
前置条件 | 获取对象的锁 | 调用Lock.lock()获取锁，然后调用Lock.newCondition()获取Condition对象
调用方式 | 直接调用：object.wait() | 直接调用 condition.await()
等待队列个数 | 一个 | 多个
当前线程释放锁并进入等待状态 | 支持 | 支持
当前线程释放锁并进入等待状态，在等待状态中不断响应中断 | 不支持 | 支持
当前线程释放锁并进入超时等待状态 | 支持 | 支持
当前线程释放锁并进入等待状态到将来的某个时间 |不支持 | 支持
唤醒等待队列中的一个线程 | 支持 | 支持
唤醒等待队列中的全部线程 | 支持 | 支持

