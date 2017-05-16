---
title: Spring Profile
tags: Spring
grammar_cjkRuby: true
---

### profile 
 > 解决不同环境使用不同的配置

#### 源码
在Spring4中的源码是基于@Conditional和Condition实现的。
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {
	String[] value();
}
```

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

*example*
```
<?xml version="1.0" encoding="UTF-8"?>
<web-app version="2.5"
	xmlns="http://java.sun.com/xml/ns/javaee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javee
		http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	....
	<!-- Web应用的上下文参数配置-->
	<context-param>
		<param-name>spring.profiles.default</param-name>
		<param-value>dev</param-value>
	</context-param>
	....
	<!-- DispatcherServlet的初始化参数配置-->
	<servlet>
		<servlet-name>appServlet</servlet-name>
		<servlet-class>
			org.springframework.web.servlet.DispatcherServlet
		</servlet-class>
		<init-param>
			<param-name>spring.profiles.default</param-name>
			<param-value>dev</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>appServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
</web-app>
```

> 测试类使用profile进行测试
```
@RunWith(SpringJUnit4ClassRunner.class)
  @ContextConfiguration(classes=DataSourceConfig.class)
  @ActiveProfiles("dev")
  public static class DevDataSourceTest {
    @Autowired
    private DataSource dataSource;
    
    @Test
    public void shouldBeEmbeddedDatasource() {
      assertNotNull(dataSource);
      JdbcTemplate jdbc = new JdbcTemplate(dataSource);
      List<String> results = jdbc.query("select id, name from Things", new RowMapper<String>() {
        @Override
        public String mapRow(ResultSet rs, int rowNum) throws SQLException {
          return rs.getLong("id") + ":" + rs.getString("name");
        }
      });
      
      assertEquals(1, results.size());
      assertEquals("1:A", results.get(0));
    }
  }
  ```
  ### 条件化bean
  
> 使用@Conditional注解来判断是否满足规定条件，是否创建对应的bean

#### Conditional注解源码
```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface Conditional {

	/**
	 * All {@link Condition}s that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();

}
```
从源码可以看出，Conditional注解可运用到类和方法上，对注解不太清楚的童鞋可以看看我以前的文章。Conditional注解成员参数是一个绑定类型为Condition的泛型数组，所以我们可以传多个条件类，来进行多条件判断。
#### Condition接口源码
```
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```
其只有一个matches的抽象方法，所以我们在设置条件类的时候，一定实现Condition接口并重写抽象方法matches。

*example*

**java配置**
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MagicConfig {
  @Bean
  @Conditional(MagicExistsCondition.class)
  public MagicBean magicBean() {
    return new MagicBean();
  }
  
}
```
**MagicExistsCondition实现类**

```
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.env.Environment;
import org.springframework.core.type.AnnotatedTypeMetadata;

public class MagicExistsCondition implements Condition {
  @Override
  public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
    Environment env = context.getEnvironment();
    return env.containsProperty("magic");
  }
}
```
> ConditionContext接口 

方法名 | 作用
------------------ | ----------
getRegistry() | 返回BeanDefinitionRegistry检查bean定义
getBeanFactory() | 返回ConfigurableListableBeanFactory检查bean是否存在，探查bean的属性
getEnvironment() | 返回Environmen，在后可以检查某个环境变量是否存在或值是多少。
getResourceLoader() | 读取并探查所返回的ResourceLoader所加载的资源。
getClassLoader() | 返回ClassLoader加载并检查类是否存在。

> AnnotatedTypeMetadata接口

*查询注解相关的内容*
```
public interface AnnotatedTypeMetadata {
	boolean isAnnotated(String annotationName);
	Map<String, Object> getAnnotationAttributes(String annotationName);
	Map<String, Object> getAnnotationAttributes(String annotationName, boolean classValuesAsString);
	MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationName);
	MultiValueMap<String, Object> getAllAnnotationAttributes(String annotationName, boolean classValuesAsString);
}
```

### 自动装配歧义性--> 注解解决

1. 用@Primary 来表示当有歧义时，首选这个bean。这个注解可以用到bean上，亦可以用到注入方法属性上。如果用xml方式配置可以添加个属性（primary="true"）
2. 用@Qualifier 来限定注入bean的类型。@Qualifier("xxxx") 也是既可以用到bean上，也可以用到注入的方法属性上。

**Java不允许在同一个条目上重复出现相同类型的多个注解**

*在java中也有一个注解类似于@Autowired 是@Resource ，以后会写两者的区别*






