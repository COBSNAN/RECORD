---Spring Profile
title: 2017-5-12未命名文件 
tags: Spring
grammar_cjkRuby: true
---

### profile 
 > 解决不同环境使用不同的配置

- java配置文件中可以通过@Profile("xxx") 来进行配置（当然注解配置既可以在类上，也可在方法上。）
- XML中配置profile
  -  xml文件全部配置可以在beans根节点里配置profile属性
  ```
  <beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
  xmlns:jee="http://www.springframework.org/schema/jee" xmlns:p="http://www.springframework.org/schema/p"
  xsi:schemaLocation="
    http://www.springframework.org/schema/jee
    http://www.springframework.org/schema/jee/spring-jee.xsd
    http://www.springframework.org/schema/jdbc
    http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd"
	profile="dev">
  <!-- 配置数据源c3p0 -->
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
         <property name="user" value="${uname}"></property>
         <property name="password" value="${upass}"></property>
         <property name="jdbcUrl" value="${url}"></property>
         <property name="driverClass" value="${driver_class}"></property>

         <!-- 初始化池子大小 -->
         <property name="initialPoolSize" value="${initPoolSize}"></property>

         <!-- 池子最大数 -->
         <property name="maxPoolSize" value="${maxPoolSize}"></property>
     </bean>
  </beans>
  ```

   - 一个xml文件指定多个profile
  ```
  <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
   xmlns:jee="http://www.springframework.org/schema/jee" xmlns:p="http://www.springframework.org/schema/p"
   xsi:schemaLocation="
     http://www.springframework.org/schema/jee
     http://www.springframework.org/schema/jee/spring-jee.xsd
     http://www.springframework.org/schema/jdbc
     http://www.springframework.org/schema/jdbc/spring-jdbc.xsd
     http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans.xsd">

   <beans profile="dev">
     <bean id="devtest" class="xxx.xxx" />
   </beans>
  
   <beans profile="prod">
      <bean id="prodtest" class="xxx.xxx" />
   </beans>
   </beans>
   ```
### 激活profile
> 激活profile依赖两个独立的属性: spring.profiles.active 和 spring.profiles.default

##### 多种方式设置这两个属性
- 作为DispatcherServlet的初始化参数。
- 作为Web应用的上下文参数。
- 作为JNDI条目
- 作为环境变量
- 作为JVM的系统属性
- 在集成测试类上，使用@ActiveProfiles注解设置。


