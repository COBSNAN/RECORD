---
title: Spring 基础
tags: java,Spring
grammar_cjkRuby: true
---

### JavaBean 和 POJO 区别
1. JavaBean一般需要一个无参构造器
2. JavaBean一般要实现Serializable 接口用于实现Bean的持久性
3. 在广义上讲，两者是类似的。

### 降低Java开发的复杂性策略
1. 基于POJO的轻量级和最小侵入性编程
2. 通过依赖注入和面向接口实现松耦合
3. 基于切面和惯例进行声明式编程
4. 通过切面和模板减少样板式代码

###  Spring 表达式语音
> SpEL 的特性

1. 使用Bean的ID引用
2. 调用方法或访问属性
3. 进行算术、逻辑或关系运算
4. 正则表达式
5. 集合操作



### 依赖注入
> 帮助应用对象彼此之间保持松散耦合

### 方式
1. 构造器注入（不是具体实现或初始化过程）



Spring通过应用上下文(Application Context)装载bean的定义并把它们组装起来，全权负责对象的创建和组装。Spring自带了多种应用上下文的实现，它们之间主要区别在于如何加载配置。
- 加载类路径下的XML文件配置，用ClassPathXmlApplicationContext 来加载,可加载一个或多个。
```
ClassPathXmlApplicationContext context = 
        new ClassPathXmlApplicationContext(
            "META-INF/spring/knight.xml");
    Knight knight = context.getBean(Knight.class);
```
- AnnotationConfigApplicationContext: 从一个或多个基于Java配置类中加载Spring应用上下文。
- AnnotationConfigWebApplicationContext: 从一个或多个基于Java的配置类中加载Spring Web应用上下文。
- FileSystemXmlApplicationContext:从文件系统下的一个或多个XML配置文件中加载上下文定义
- XmlWebApplicationContext:从Web应用下的一个或多个XML配置文件中加载上下文定义。

### Spring Bean装配三种装配机制
- XML显示配置
- Java中显示配置
- 隐式的bean发现机制和自动装配（用注解来装配）