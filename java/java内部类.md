---
title: java内部类
tags: java
grammar_cjkRuby: true
---

### 定义
内部类是指在一个外部类的内部再定义一个类。编译成功，就会成为完全不同的两类。

### 内部类有四种情况

- 成员内部类：成员内部类，就是作为外部类的成员，可以直接使用外部类的所有成员和方法，即使是private的。同时外部类要访问内部类的所有成员变量/方法，则需要通过内部类的对象来获取。实例化方式：new Out().new In();
- 局部内部类：是指内部类定义在方法和作用域内。
- 静态内部类：其实它和成员内部类很相似，只是只能访问外部类的静态成员变量，具有局限性。实例化方式：new Out.In();其实你会发现就是把内部类看成一个属性来对待。
- 匿名内部类：匿名内部类不能加访问修饰符。其实我们最常见的执行一个线程就是用的内部类的方式。
	```
	new Thread(){
			@Override
			public void run(){
				System.out.println(name);
			}
		}.start();
	```
 
 **例子**
 
> 成员内部类

 ```
 package innerclass;

public class innerOut {
	private int num = 12;
    
    class innerIn {
        private int num = 13;
        public void print() {
            int num = 14;
            System.out.println("局部变量：" + num);
            System.out.println("内部类变量：" + this.num);
            System.out.println("外部类变量：" + innerOut.this.num);
        }
    }
}
 ```
> 静态内部类

```
package innerclass;

public class Out {
	private static int age = 12;
    
    static class In {
        public void print() {
            System.out.println(age);
        }
    }
}
```
> 匿名内部类

```
package innerclass;

public class anonymityClass {
	//注意是final类型的
	public void startThread(final String name){
		new Thread(){
			@Override
			public void run(){
				System.out.println(name);
			}
		}.start();
	}
}
```

> 局部内部类

```
package innerclass;

public class methodInnerClass {
	private int num = 12;
	 
    public void Print(final int x) {
        class In {
            public void inPrint() {
                System.out.println(x);
                System.out.println(num);
            }
        }
        new In().inPrint();
    }
}
```
### 内部类的特点总结
> 在方法间定义的非静态内部类： 

- 外围类和内部类可互相访问自己的私有成员。
- 内部类中不能定义静态成员变量。

> 在方法间定义的静态内部类：

- 只能访问外部类的静态成员。

> 在方法中定义的局部内部类：

- 该内部类没有任何的访问控制权限
- 外围类看不见方法中的局部内部类的，但是局部内部类可以访问外围类的任何成员。
- 方法体中可以访问局部内部类，但是访问语句必须在定义局部内部类之后。
- 局部内部类只能访问方法体中的常量，即用final修饰的成员。

> 在方法中定义的匿名内部类：

- 没有构造器，取而代之的是将构造器参数传递给超类构造器。
