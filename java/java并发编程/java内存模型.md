---
title: java内存模型
tags: java
grammar_cjkRuby: true
---

### 并发编程的两个关键问题

- 通信：指线程之间以何种机制来交换信息。在命令式编程中，线程之间通信机制有两种：共享内存和消息传递
- 同步：指程序中用于控制不同线程间操作发生想到顺序的机制。

### 重排序

- 编译器优化重排序
- 指令级并行重排序
- 内存系统重排序

### 内存屏障类型表

| 屏障类型            | 指令示例                 | 说明                                                                                                                                                                                               |
| ------------------- | ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| LoadLoad Barriers   | Load1;LoadLoad;Load2     | 确保 Load1 数据的装载先于 Load2 及所有后续装载指令的装载                                                                                                                                           |
| StoreStore Barriers | Store1;StoreStore;Store2 | 确保Store1数据对其他处理器可见（刷新到内存）先于Store2及所有后续存储指令的存储                                                                                                                     |
| LoadStore Barriers  | Load1;LoadStore;Store2   | 确保Load1数据装载先于Store2及所有后续的存储指令刷新到内存                                                                                                                                          |
| StoreLoad Barriers  | Store1;StoreLoad;Load2   | 确保Store1数据对其他处理器变得可见（指刷新到内存）先于Load2及所有后续装载指令的装载。StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令 |

### happens-before

> 规则

- 程序顺序规则：一个线程中的每个操作，happens-before 于该线程中的任意后续操作
- 监视器锁规则：对一个锁的解锁，happens-before 于随后对这个锁的加锁。
- volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读
- 传递性：如果happens-before B，且B happens-before C，那么A happens-before C。

*happens-before关系，并不意味着前一个操作必须要在后一个操作之前执行！happens-before 仅仅要求前一个操作（执行结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。*

### 重排序
> 指编译器和处理器为了优化程序性能而对指令序列进行重新排序的一种手段

java内存模型（JMM）,顺序一致性内存模型

> 顺序一致性内存模型两大特征

- 一个线程中的所有操作必须按照程序的顺序来执行
- 不管线程是否同步，所有线程都只能看到一个单一的操作执行顺序。在顺序一致性内存模型中，每个操作都必须原子执行且立刻对所有线程可见

*在JSR-33内存模型开始（JDK5开始），仅仅只允许把一个64位long/double型变量的写操作拆分为两个32位的写操作来执行，任意的读操作在JSR-33中都必须具有原子性*

### volatile 关键字

> volatile变量特征

- 可见性：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入
- 原子性：对任意两个volatile变量的**读/写**具有原子性，volatile++这种复合操作不具有原子性

> 当写一个volatile变量时，JMM会把该线程对应的**本地内存中的共享变量值**刷新到主内存，也就是说其他非volatile 变量也实时的把值刷新到内存中了。对其他线程可见。

> volatile 重排序规则变

是否能重排序 | 第二个操作 |     |  |
----------- | --------- | --- |--
第一个操作 | 普通读/写 | volatile 读 | volatile 写
普通读/写 | -| - | -|
volatile 读 | NO | NO | NO
volatile 写|-   | NO | NO

>锁释放或获取的内存语义实现

- 利用volatile变量的写-读所具有的内存语义
- 利用CAS所附带的volatile读和volatile写的内存语义


### final域内存规则语义

>final域的重排序规则

- 在构造函数内对一个final域的写入，与随后把这个构造对象的引用赋值给一个引用变量，这两个操作之间不能冲排序
- 初次读一个包含final域的对象的引用，与随后初次读这个final域，这两个操作之间不能重排序
- 对应**引用类型**，在构造函数内对一个final引用的对象的成员域的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作不能重排序

### happens-before规则

1. 程序顺序规则：一个线程中的每个操作，happens-before于该线程中的任意后续操作
2. 监视器锁规则：对一个锁的解锁，happens-before于随后对这个锁的加锁
3. volatile变量规则：对一个volatile域的写，happens-before于任意后续对这个volatile域的读
4. 传递性：如果A happens-before B，且B happens-before C，那么A happens-before C。
5. start()规则：如果线程A执行操作ThreadB.start()（启动线程B），那么A 线程的ThreadB.start()操作happens-before 于线程B中的任意操作。
6. join()规则：如果线程A执行操作ThreadB.join()并成功返回，那么线程B中的任意操作happens-before 于线程A 从ThreadB.join()操作成功返回。

### 延迟初始化

> 双重检查volatile锁定

```
public class SafeDoubleCheckedLocking {
    
    //不加volatile是不安全的，因为一个对象的创建不是原子性的，有可能存在刚分配完对象地址，还没赋值
    private volatile static Student student;
    
    public static Student getStuInstance(){
        if (student == null){
            synchronized (SafeDoubleCheckedLocking.class){
                if (student == null){
                    student = new Student();
                }
            }
        }
        return student;
    }
}
```

> 基于类初始化的解决方案

```
public class ClassInitLocking {
    private static class InstanceHolder{
        public static Student student = new Student();
    }

    public static Student getStuInstance(){
        //这里将导致InstanceHolder类被初始化
        //在执行类的初始化期间，JVM会去获取一个锁
        return InstanceHolder.student;
    }
}
```
*对实例字段使用线程安全的延迟初始化，使用基于volatile的延迟初始化的方案；对静态字段使用线程安全的延迟初始化，使用基于类初始化的方案*


