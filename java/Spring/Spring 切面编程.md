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

### Spring切点书写方式

> 注解形式的创建切面
****

> Spring切点支持的AspectJ切点指示器
![切点指示器][2]
> Spring 新引入指示器

bean() 指示器：在切点表达式中使用bean的ID来标识bean。bean()使用bean ID或bean名称作为参数来限制切点只匹配特定的bean。
```
//execution中 第一个*表示返回任意类型，..*表示匹配test包下的所有方法名，(..)表示匹配任意方法参数
execution( * com.cobs.test..*(..)) || within(com.hello.service.*)
```
*这个切点表达式表示 匹配test包下的所有方法或service包下的所有方法*

在例子中我们使用**||**【or】来连接指示器，相应的还可以使用**&&【and】** 和**！【not】**来进行连接指示器。

### 创建切面

```
package com.spring.aop;  
  
import org.aspectj.lang.ProceedingJoinPoint;  
import org.aspectj.lang.annotation.After;  
import org.aspectj.lang.annotation.AfterReturning;  
import org.aspectj.lang.annotation.AfterThrowing;  
import org.aspectj.lang.annotation.Around;  
import org.aspectj.lang.annotation.Aspect;  
import org.aspectj.lang.annotation.Before;  
import org.aspectj.lang.annotation.Pointcut;  
  
@Aspect  
public class TestAnnotationAspect {  
  
    @Pointcut("execution(* com.spring.service.*.*(..))")  
    private void pointCutMethod() {  
    }  
  
    //声明前置通知  
    @Before("pointCutMethod()")  
    public void doBefore() {  
        System.out.println("前置通知");  
    }  
  
    //声明后置通知  
    @AfterReturning(pointcut = "pointCutMethod()", returning = "result")  
    public void doAfterReturning(String result) {  
        System.out.println("后置通知");  
        System.out.println("---" + result + "---");  
    }  
  
    //声明例外通知  
    @AfterThrowing(pointcut = "pointCutMethod()", throwing = "e")  
    public void doAfterThrowing(Exception e) {  
        System.out.println("例外通知");  
        System.out.println(e.getMessage());  
    }  
  
    //声明最终通知  
    @After("pointCutMethod()")  
    public void doAfter() {  
        System.out.println("最终通知");  
    }  
  
    //声明环绕通知，注意环绕通知的写法
    @Around("pointCutMethod()")  
    public Object doAround(ProceedingJoinPoint pjp) throws Throwable {  
        System.out.println("进入方法---环绕通知");  
        Object o = pjp.proceed();  
        System.out.println("退出方法---环绕通知");  
        return o;  
    }  
} 
```

### 启动切面自动代理功能
- JavaConfig 配置方式，可以在配置类的类级别上通过使用EnableAspectJAutoProxy注解启用自动代理功能
	```
	@Configuration
	@EnableAspectJAutoProxy
	@ComponentScan
	public class ExaminationConfig {
		//声明通知相关类bean
		@Bean
		public TestAnnotationAspect getTestAnnotationAspect() {
		 return new TestAnnotationAspect();
		}
	}
	```
- XML配置方式，需要使用Spring aop 命名空间中的<aop:aspectj-autoproxy>元素来启动代理功能
	```
	<?xml version="1.0" encoding="UTF-8"?>  
	<beans xmlns="http://www.springframework.org/schema/beans"  
		   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
		   xmlns:aop="http://www.springframework.org/schema/aop"  
		   xsi:schemaLocation="  
	http://www.springframework.org/schema/beans   
	http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
	http://www.springframework.org/schema/aop  
	http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">  
	<aop:aspectj-autoproxy/>  
		<bean  class = "com.spring.aop.TestAnnotationAspect"></bean>  
	</beans>
	```

  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/QQ%E6%88%AA%E5%9B%BE20170606082814.png "QQ截图20170606082814"
  [2]: https://www.github.com/COBSNAN/ImageHub/raw/master/er.png "er"