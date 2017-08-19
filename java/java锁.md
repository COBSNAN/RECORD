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