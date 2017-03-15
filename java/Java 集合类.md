---
title: Java 集合类
tags: java
grammar_cjkRuby: true
---
![集合类][1]

从图中可知
Collection是 List，Set，Quene 的根接口，Map 是另一个接口
Iterator用于遍历集合中元素的接口

下面是各集合的特性

|  |  |是否有序  | 是否允许元素重复 | 
|---------- | --------|---------|----------|
Liist || 否 | 是 
Set | HashSet | 否 | 否 
|| TreeSet | 是（二叉排序树）|否
Map | HashMap | 否 | key唯一，value可重复
|| TreeMap | 是（二叉排序树）|同上
[集合类特点]

## 集合类方法
#### 遍历Map方法
1. 方法一
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {  
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());  
}
```
2. 方法二
```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();  
Iterator<Map.Entry<Integer, Integer>> entries = map.entrySet().iterator();  
while (entries.hasNext()) {  
    Map.Entry<Integer, Integer> entry = entries.next();  
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());  
} 
```
#### LinkList 方法
- peek() 或 element() 获取但不移除此列表的头
- poll() 或 remove() 获取并移除此列表的头
- offer(4) 将指定元素添加到此列表的末尾


## OTHER

一般集合的加载因子为0.75
ArrayList 初始化容量为10
HashMap 初始化容量为16 即使你初始化的时候指定一个值去初始化容量 其值也不一定是你指定的那个值，其一定是2的幂次方,代码如下
```java
static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
 此时22.56 算是写完一篇	:smile:


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1489586143029.jpg "1489586143029"