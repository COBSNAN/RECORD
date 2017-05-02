---
title: 泛型 generic
tags: java
grammar_cjkRuby: true
---
### 格式
```
<T extends Comparable> 
```
- T应该是绑定类型的子类型，T和绑定类型可以使类，也可以是接口。
- 一个类型变量或通配符可以有多个限定，T extends Ccomparable & Serializable 
- 限定类型用“&” 分隔。而逗号用来分隔类型变量。
- 在Java 的继承中，可以根据需要拥有多个接口超类型，但限定中至多有一个类型。如果用一个类作为限定，它必须是限定列表中的第一个。

### 类型擦除
- 擦除类型变量，并替换为限定类型(无限定的变量用Object)
- 在验证阶段进行类型擦除
> **为了提高效率，应该将标签(tagging)接口(即没有方法的接口)放在边界列表的末尾**