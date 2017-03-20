---
title: java 基础知识点
tags: java
grammar_cjkRuby: true
---

> Java中 private、protected、public和default的区别

作用域 | 当前类 | 同一包类 | 子孙类 |其他包
------|-----|-----|-----|-------
public | Y | Y | Y | Y
protected | Y | Y | Y | N
default | Y | Y | N | N
private | Y | N | N | N

类似于倒三角形的样子
**注意** 
- 上面说的是类的方法属性作用域，default是类属性的默认修饰符。
- 接口的属性和方法只有public 和default 修饰词。而接口的默认属性是默认是public static final ，方法是public abstract。
- 类的修饰符只有public和默认不选（表示同包可访问）

> override 和 overload

- override 重写，表示子类重写了父类的方法
	- 方法名、参数、返回值相同
	- 子类方法不能缩小父类的访问权限
	- 子类方法不能抛出比父类方法更多的异常，但可以不抛出异常
	- 方法被定义成final，static 不能被重写

- overlord 重载 一个类定义多个同名的方法
	- 参数类型，个数不对应
	- 不能通过访问权限、返回类型、抛出的异常进行重载

**注意**：static方法 官网推荐是直接用类名调用。


>　接口 interface

在Java8 中接口也可以定义默认方法与静态方法
```java?linenums
import java.util.function.Supplier;

public class testInterfaceMethodes {
	private interface Defaulable {
	    // Interfaces now allow default methods, the implementer may or 
	    // may not implement (override) them.
	    default String notRequired() { 
	        return "Default implementation"; 
	    }        
	}
	private static class DefaultableImpl implements Defaulable {
	}
	private static class OverridableImpl implements Defaulable {
	    @Override
	    public String notRequired() {
	        return "Overridden implementation";
	    }
	}
	private interface DefaulableFactory {
	    // Interfaces now allow static methods
	    public static Defaulable create( Supplier< Defaulable > supplier ) {
	        return supplier.get();
	    }
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		Defaulable defaulable = DefaulableFactory.create( DefaultableImpl::new );
	    System.out.println( defaulable.notRequired() );
	         
	    defaulable = DefaulableFactory.create( OverridableImpl::new );
	    System.out.println( defaulable.notRequired() );

	}

}
```
==>执行结果 
Default implementation
Overridden implementation
	







