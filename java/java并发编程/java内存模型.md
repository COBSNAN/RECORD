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

| 屏障类型          | 指令示例             | 说明                                                     |
| ----------------- | -------------------- | -------------------------------------------------------- |
| LoadLoad Barriers | Load1;LoadLoad;Load2 | 确保 Load1 数据的装载先于 Load2 及所有后续装载指令的装载 |
| StoreStore Barriers | Store1;StoreStore;Store2 |确保Store1数据对其他处理器可见（刷新到内存）先于Store2及所有后续存储指令的存储 |
| LoadStore Barriers | Load1;LoadStore;Store2 | 确保Load1数据装载先于Store2及所有后续的存储指令刷新到内存 | 
| StoreLoad Barriers | Store1;StoreLoad;Load2 | 确保Store1数据对其他处理器变得可见（指刷新到内存）先于Load2及所有后续装载指令的装载。StoreLoad Barriers 会使该屏障之前的所有内存访问指令（存储和装载指令）完成之后，才执行该屏障之后的内存访问指令