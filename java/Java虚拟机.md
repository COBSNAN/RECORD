---
title: Java虚拟机
tags: java
grammar_cjkRuby: true
---

### Java虚拟机示意图

![示意图][1]

- 方法区（持久代）
	- 存放class信息和类的静态方法和属性、常量。大小由-XX:PermSize来调节

- 堆
	- 用于存放类的实例化对象信息的 堆大小由-Xmx 和 -Xms来调节
	- 分为{Old + New={Eden,from,to}}
- 栈
	- 存放方法执行时的局部变量、执行顺序等 栈大小由Xss来调节
- 程序计数器
	- 它的作用是当前线程所执行的字节码的行号指示器。
- 本地方法栈
	- JVM采用本地方法栈来支持native方法的执行，此区域用于储存每个native方法调用的状态。

例子  -XX:MaxPermSize=512m -Xms256M -Xmx512M -XX:PermSize=256m

> 介绍几个常用的配置选项

配置参数 | 功能
---- | ----
-Xms | 初始化堆大小。如：-Xms256m
-Xmx | 最大堆大小。如：-Xmx512m
-Xmn | 新生代大小。通常为Xmx 的1/3 或1/4.新生代 = Eden + 2个Survivor空间。实际可用空间为=Eden+ 1个Survivor，即 90%
-Xss | JDK1.5+ 每个线程堆栈大小为1M 就足以。
-XX:NewRatio | 新生代与老年代的比例，如-XX:NewRatio=2，则新生代占整个对空间的1/3
-XX:SurvivorRatio | 新生代中Eden与Survivor的比值。默认值为8.即Eden占新生代空间的8/10.
-XX:PermSize | 永久代的初始大小
-XX:MaxPermSize | 永久代的最大值
-XX:+PrintGCDetails | 打印GC信息
-HeapDumpOnOutOfMemoryError | 让虚拟机在发生内存溢出时Dump出当前的内存堆储快照，以便分析


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1490710913631.jpg "1490710913631"