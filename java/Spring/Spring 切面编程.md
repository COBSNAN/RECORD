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

> Spring切点支持的AspectJ切点指示器

![切点指示器][2]
> Spring 新引入指示器

bean() 指示器：在切点表达式中使用bean的ID来标识bean。bean()使用bean ID或bean名称作为参数来限制切点只匹配特定的bean。
```
//execution中 第一个*表示返回任意类型，..*表示匹配test包下的所有方法名，(..)表示匹配任意方法参数
execution( * com.cobs.test..*(..)) || within(com.hello.service.*)
```
*这个切点表达式表示 匹配test包下的所有方法或service包下的所有方法*

在例子中我们使用 ||【or】来连接指示器，相应的还可以使用 &&【and】 和 !【not】来进行连接指示器。

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
	
### 在通知中使用参数
	
> Java 配置方式

- 切点
```
package aoptest;

import java.util.Arrays;
import java.util.List;

public class HotData {
	
	List<String> hotDatas = Arrays.asList("gao kao", "kkw", "yang mi");
	
	public void getDataByOrder(int order){
		//这里没做越界判断，仅仅是打印热点数据
		System.out.println(hotDatas.get(order-1));
	}
}
```
- 切面
```
package aoptest;

import java.util.HashMap;
import java.util.Map;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class AspectConfiguration {
	private Map<Integer, Integer> readCounts = new HashMap<Integer, Integer>();
	
	@Pointcut("execution(* aoptest.HotData.getDataByOrder(int)) "+
	"&& args(dataOrder)")
	public void readCountCut(int dataOrder){}
	
	@Before("readCountCut(dataOrder)")
	public void readCount(int dataOrder){
		int readTimes = getReadTimes(dataOrder);
		readCounts.put(dataOrder, readTimes+1);
	}
	
	public int getReadTimes(int dataOrder) {
		return readCounts.containsKey(dataOrder)
			? readCounts.get(dataOrder) :0;
	}
}

```
- 启动切面代理功能和配置类
```
package aoptest;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@EnableAspectJAutoProxy
public class AspectStart {
	
	@Bean
	public HotData setHotData(){
		HotData hd = new HotData();
		return hd;
	}
	
	@Bean
	public AspectConfiguration setAspectConfiguration(){
		return new AspectConfiguration();
	}
}

```
- 测试类
```
package aoptest;

import static org.junit.Assert.assertEquals;

import org.junit.Rule;
import org.junit.Test;
import org.junit.contrib.java.lang.system.StandardOutputStreamLog;
import org.junit.contrib.java.lang.system.SystemOutRule;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;

@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=AspectStart.class)
public class AopTest {
	//获取系统输出
	@Rule
    public final SystemOutRule systemOutRule = new SystemOutRule().enableLog();

   /* @Test
    public void writesTextToSystemOut() {
        System.out.print("hello world");
        assertEquals("hello world", systemOutRule.getLog());
    }*/
	
	@Autowired
	private HotData hotData;
	
	@Autowired
	private AspectConfiguration aConfiguration;
	
	@Test
	public void testAop (){
		hotData.getDataByOrder(1);
		hotData.getDataByOrder(2);
		hotData.getDataByOrder(1);
		hotData.getDataByOrder(3);
		hotData.getDataByOrder(2);
		hotData.getDataByOrder(1);
		assertEquals(3,aConfiguration.getReadTimes(1));
		assertEquals(2,aConfiguration.getReadTimes(2));
		assertEquals(1,aConfiguration.getReadTimes(3));
		
	
	}
}

```
- pom.xml 文件
```
<dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjweaver</artifactId>
       <version>1.7.4</version>
    </dependency>

    <dependency>
       <groupId>org.aspectj</groupId>
       <artifactId>aspectjrt</artifactId>
       <version>1.7.4</version>
    </dependency>
    
    
    <dependency>
	    <groupId>junit</groupId>
	    <artifactId>junit</artifactId>
	    <version>4.12</version>
	</dependency>
	
	<dependency>
	  <groupId>com.github.stefanbirkner</groupId>
	  <artifactId>system-rules</artifactId>
	  <version>1.16.0</version>
	  <scope>test</scope>
	</dependency>
```
*pom文件还包含spring的一些包，没有列出来，这是除spring包的其他包* junit 和 system-rules 注意版本兼容！
> xml配置方式

就是把java配置去掉注解，不要配置类，下面看下xml配置方式。
```
<?xml version="1.0" encoding="UTF-8"?>  
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
       xmlns:aop="http://www.springframework.org/schema/aop"  
       xsi:schemaLocation="  
http://www.springframework.org/schema/beans   
http://www.springframework.org/schema/beans/spring-beans-4.0.xsd  
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">  
      
    <aop:config>  
        <!-- 定义切面，命名为 aspectTest-->  
        <aop:aspect id = "aspectTest" ref = "aspectConfiguration" >  
        <!-- 定义公共的切入点表达式 -->  
        <aop:pointcut expression="execution(* aoptest.HotData.getDataByOrder(int)) and args(dataOrder)" id="readCountCut"/>  
        
            <aop:before pointcut-ref="readCountCut" method="readCount"/>  
        </aop:aspect>    
    </aop:config>  
    <bean id = "hotData" class = "aoptest.HotData"></bean>  
    <bean id = "aspectConfiguration" class = "aoptest.AspectConfiguration"></bean>  
</beans>
```
** 大家发现，不管是xml和java配置，都定义了hotData bean，我们发现它只用于测试类，如果我们不用测试类，是不是可以去掉**
> 答案是否定的，aop监听的package下，必须用ioc容器管理，才会有监听的效果。HotData hotData = new HotData();
是不会有aop的监听事件触发

### 通过切面引入新的功能方法

> java 配置方式

- 新方法配置类
```
package aopinject;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.DeclareParents;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class MyAspect {
	@DeclareParents(value="aopinject.UserService", 
			defaultImpl=aopinject.BasicVerifier.class)
	public Verifier verifer;
}
```
- java配置类
```
package aopinject;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.EnableAspectJAutoProxy;

@Configuration
@ComponentScan
@EnableAspectJAutoProxy
public class JavaConfig {
	
}
```
- 切入功能方法类

```
//接口
package aopinject;
public interface Verifier {
	public boolean validate(User user);
}

//实现类
package aopinject;
public class BasicVerifier implements Verifier {
	public boolean validate(User user) {
		if(user.getUsername().equals("jack") && user.getPassword().equals("1234")) {
			return true;
		}
		return false;
	}
}
```
- 切点
```
package aopinject;

import org.springframework.stereotype.Component;
@Component("service")
public class UserService {
	public void serve(User user) {
		System.out.println(user.toString());
	}
}
```
- 测试类
```
package aopinject;


import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.test.context.ContextConfiguration;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(classes=JavaConfig.class)
public class AopInjectTest {
	
	@Autowired
	private UserService userService;
	
	@Test
	public void testAopInject (){
		User user1 = new User();
		user1.setUsername("jack");
		user1.setPassword("1234");
		
		User user2 = new User();
		user2.setUsername("haha");
		user2.setPassword("123456");
		
		Verifier v = (Verifier) userService;
		
		if(v.validate(user1)) {
			System.out.println("user1验证成功");
			userService.serve(user1);
		}
		if(v.validate(user2)) {
			System.out.println("user2验证成功");
			userService.serve(user2);
		}
	
	}
}

```
**结果**
![执行结果][3]

> xml配置方式

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"  
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
       xmlns:context="http://www.springframework.org/schema/context" 
       xmlns:aop="http://www.springframework.org/schema/aop"  
       xsi:schemaLocation="  
http://www.springframework.org/schema/beans   
http://www.springframework.org/schema/beans/spring-beans-4.0.xsd
http://www.springframework.org/schema/context   
http://www.springframework.org/schema/context/spring-context-4.0.xsd 
http://www.springframework.org/schema/aop  
http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">  
      
    <context:component-scan base-package="aopinject"/>
	<aop:aspectj-autoproxy/>
	<aop:config>   
		<aop:aspect>
			<aop:declare-parents types-matching="aopinject.UserService"
				implement-interface="aopinject.Verifier"
				default-impl="aopinject.BasicVerifier" />
		</aop:aspect>
	</aop:config>
	<!-- delegate-ref="" 这个替换default-impl ，指定的是BasicVerifier的bean Id-->
</beans>
```
> @DeclareParents注解

- value属性指定了哪种类型的bean要引入该接口
- defaultImpl属性指定了为引入功能提供实现的类
- @DeclareParents注解所标注的属性指明了要引入了接口

  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/QQ%E6%88%AA%E5%9B%BE20170606082814.png "QQ截图20170606082814"
  [2]: https://www.github.com/COBSNAN/ImageHub/raw/master/er.png "er"
  [3]: https://www.github.com/COBSNAN/ImageHub/raw/master/1498003041643.jpg