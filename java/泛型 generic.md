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

### 桥方法

在一个类继承一个泛型类时，又想重写泛型类方法，这时候由于参数类型的不一样。导致不是重写，这是就需要桥方法，做下转接。用桥方法再去调用我们要执行的方法。
```
//泛型类中方法  Pair<T>
public void setSecond(T newValue ){

}
//泛型类类型擦除后
public void setSecond(Object newValue ){
	//.....一些操作
}

//子类中实现重写方法（桥方法）
//桥方法
public void setSecond(Object second){
	setSecond((Date) second);
}
public void setSecond(Date second){
	if (second.getDay() == 10) {
		//发工资
	}
}
```
 > java 泛型转换要点

- 虚拟机中没有泛型，只有普通的类和方法
- 所有的类型参数都用它们的限定类型替换
- 桥方法被合成来保持多态
- 为保持类型安全性，必要时插入强制类型转换

 > 知识点

1. 不能用基本类型实例化类型参数，Pair<int> 不允许
2. 运行时类型查询只适用于原始类型，也就是说，类型擦除后的类型
3. 一般不要创建参数化类型的数组
4. 泛型类不能**实例化类型变量**
5. 泛型类的静态上下文中类型变量无效，private static T singleInstance; 这是错误的
6. 不能抛出或捕获泛型类的实例