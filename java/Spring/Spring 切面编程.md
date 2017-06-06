---
title: Spring 切面编程
tags: java,Spring,切面
grammar_cjkRuby: true
---

### AOP 术语

- 通知(Advice):定义了切入的方法和何时执行方法
	![通知的5中方式][1]
- 连接点(Join point):可切入的时刻点，比如调用方法时，抛出异常时，修改字段等
- 切点(Poincut):所要切入连接点的一个范围
- 切面(Aspect):通知和切点的结合，就是一个总体含义
- 引入(Introduction):向现有的类添加新方法或属性
- 织入(Weaving):把切面应用到目标对象并创建新的代理对象的过程。织入的生命周期：
	- 编译期：切面在目标类编译时被织入。AspectJ 的织入编译器就是这种方式。
	- 类加载期：切面在目标类加载到JVM时被织入，需要特殊的类加载器，可以在目标类被引入应用之前增强该目标类的字节码。AspectJ5的加载时织入就支持这种方式。
	- 运行期：切面在应用运行的某个时刻被织入。在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。这也是Spring AOP的织入方式。


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/QQ%E6%88%AA%E5%9B%BE20170606082814.png "QQ截图20170606082814"