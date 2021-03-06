---
title: Spring表达式语言（SpEL）
tags: java,Spring
grammar_cjkRuby: true
---

### 特性
- 使用bean的ID来引用bean
- 调用方法和访问对象的属性
- 对值进行算术、关系和逻辑运算
- 正则表达式匹配
- 集合操作

### 属性占位符和SpEL 区别
- 书写方式不一样，属性占位符用${...}表示，而SpEL用#{...}来表示。
- 属性占位符一般需要加载一个配置文件，再通过${...}来获取其中的值，SpEL不需要
	- 配置文件有两种形式
		- java配置方式，通过@PropertySource 来配置一个配置文件路径，其中的内容会构造一个Environment类型的bean
		```
		import org.springframework.beans.factory.annotation.Autowired;
		import org.springframework.context.annotation.Bean;
		import org.springframework.context.annotation.Configuration;
		import org.springframework.context.annotation.PropertySource;
		import org.springframework.core.env.Environment;
		@Configuration
		@PropertySource("classpath:/com/soundsystem/app.properties")
		public class EnvironmentConfig {
		  @Autowired
		  Environment env;
		  @Bean
		  public BlankDisc blankDisc() {
			return new BlankDisc(
				env.getProperty("disc.title"),
				env.getProperty("disc.artist"));
		  }
		}
		```
		- XML配置方式
		```
		<context:property-placeholder location="com/soundsystem/app.properties" />
	    <bean class="com.soundsystem.BlankDisc" c:_0 = "${disc.title}" c:_1 = "${disc.artist}"/>
		```
> 介绍Environment接口
1. boolean containsProperty(String key); //是否包含某个key
2. String getProperty(String key);//从配置文件找key的value值
3. String getProperty(String key, String defaultValue);//先从配置文件中找，找不到返回defaultValue值
4. T getProperty(String key, Class<T> targetType); //只是把取出的非空值转化成我们传入的(targetType)类型。

### SpEL的使用
1. 表示字面值:#{3.14159}，#{'hello'}
2. 引用bean、属性和方法: #{testBean} testBean是bean的ID，#{testBean.testField}，#{testBean.testMethod().toUpperCase()}
	当方法返回值后，我们不确定值是否是null，可以通过 ==?.== 运算符来避免，它会先判断返回值是否为空，再调用后面toUpperCase方法
3. 在表达式中使用类型：通过 ==T()== 运算符来访问类作用域内的方法和常量。eg. #{T(java.lang.Math).random()}
4. SpEL运算符
    ![enter description here][1]
	#{test.first?:'default'} 中，如果test.first的值是不是null，如果是null，那么计算结果用"default"。同城称为Elvis运算符
5. 正则表达式:支持通过**maches**运算符来进行模式匹配,返回一个Boolean类型的值。eg.#{admin.number matches '\d+'} 来检查是否是纯数字字符串。
6. 计算集合:通过**[ ]**来从集合或数组中按照索引获取元素。eg.#{shop.book[2].name}
	 它还有一些对集合进行过滤的查询运算符，下面看一下
	 
| 查询运算符 | 作用 |
| ----- | ---- |
|`.?[ ]`  | 对集合进行过滤，得到集合的一个子集。[ ]里面试过滤条件 |
| `.^[ ] `| 获取满足条件的第一个匹配项 |
| `.$[ ]` | 获取满足条件的最后一个匹配项 |
| `.![ ]` | 投影运算符，它会从集合的每个成员中特点的属性放到另外一个集合中。|
	
	

  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1495153440008.jpg