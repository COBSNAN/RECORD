---
title: Mybatis Configuration 配置
tags: java
grammar_cjkRuby: true
---

### Configuration 作用
- 读入配置文件，包括基础配置的XML文件和映射器的XML文件。
- 初始化基础配置，比如Mybatis的别名等，一些重要的类对象，例如，插件、映射器、ObjectFactory 和typeHandler对象。
- 提供单例，为后续创建SessionFactory 服务并提供配置的参数
- 执行一些重要的对象方法，初始化配置信息。

![enter description here][1]

### Configuration 包含的元素详解

> properties 元素是一个配置属性的元素

- property子元素

![enter description here][2]
- properties配置文件

![enter description here][3]
- 程序参数传递

```
package mybatistest;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.InputStream;
/**
 * Created by dida on 2017/8/17.
 */
public class mybatistest {
    public static void main(String[] args){
        String resource ="mybatis-config.xml";
        try{
            //SqlSessionFactoryBuilder 创建sqlSessionFactory ，然后声明周期就结束了
            InputStream inputStream = Resources.getResourceAsStream(resource);
            SqlSessionFactory sqlSessionFactory = null;
            sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        }catch (Exception e){}
    }
}
```

**如果这三种方式都配置的话，其优先级是倒着过来了，因为MyBatis加载顺序是从属性->文件->方法参数，而属性是会被覆盖掉的，所以优先级就倒过来了**

> setting 设置
> 
设置参数 | 描述 | 有效值 | 默认值
--- | --- | --- | --
cacheEnabled | 该配置影响所有映射器中配置的缓存全局开关 | true、false | true
lazyLoadingEnabled | 延迟加载的全局开关。当它开启时，所有关联对象都会延迟加载。特定关联关系中可以通过设置fetchType属性来覆盖该项的开关状态 | true、false | false
aggressiveLazyLoading | 当启用时，对任意延迟属性的调用会使带有延迟加载属性的对象完整加载。反之，每种属性将会按需加载 | true、false | true
multipleResultSetsEnabled | 是否允许单一语句返回多结果集（需要兼容驱动）|true、false | true
userColumnLabel | 使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试|true、false | true
useGeneratedKeys | 允许JDBC支持自动生成主键，需要驱动兼容。如果设置为true，则这个设置强制使用自动生成主键，尽管一些驱动不能兼容但仍可正常工作|true、false | false
autoMappingBehavior|指定MyBatis应如何自动映射列到字段或属性。NONE表示取消自动映射；PAPRIAL只会自动映射没有定义嵌套结果集映射的结果集；FULL会自动映射任意复杂的结果集（无论是否嵌套）|NONE、PAPTIAL、FULL|PARTIAL
defaultExecutorType | 配置默认的执行器。SIMPLE是普通的执行器；REUSE执行器会重用预处理语句（prepared statements）；BATCH执行器将重用语句并执行批量更新|SIMPLE、REUSE、BATCH|SIMPLE
defaultStatementTimeout|设置超时时间，它决定驱动等待数据库响应的秒数。当没有设置的时候它取的就是驱动默认的时间|Any positive integer |Not set(null)
safeRowBoundsEnabled|运行在嵌套语句中使用分页(RowBounds)|true、false | false
mapUnderscoreToCamelCase|是否开启自动驼峰命名规则(camel case)映射，即从经典数据库列名A_COLUMN 到经典Java属性名aColumn的类似映射|true、false | false
localCacheScope|MyBatis利用本地缓存机制(Local Cache)防止循环引用(circular references)和加速重复嵌套查询。默认值为SESSION,这种情况下会缓存一个会话中执行的所有查询。若设置值为STATEMENT，本地会话仅用在语句执行上，对相同SqlSession的不同调用将不会共享数据 | SESSION、STATEMENT |SESSION
jdbcTypeForNull | 当没有参数提供特定的JDBC类型时，为空值指定JDBC类型。某些驱动需要指定列的JDBC类型，多数情况直接用一般类型即可，比如NULL、VARCHAR、OTHER|JdbcType枚举，最常见的是NULL、VARCHAR和OTHER|OTHER
lazyLoadTriggerMethods | 指定对象的方法触发一次延迟加载 | 如果是一个方法列表,用逗号隔开|equals,clone,hashCode,toString
defaultScriptingLanguage | 指定动态SQL生成的默认语言 |配置类的别名或者类的全限定名 |org.apache.ibatis.scripting.xmltags.XMLDynamicLanguageDriver
callSettersOnNulls|指定当结果集中值为null的时候是否调用映射对象的setter(map对象时为put)方法，这对于有Map.keySet()依赖或null值初始化的时候是有用的。注意基本类型不能设置成null。|true、false | false
logPrefix | 指定Mybatis增加到日志名称的前缀 | 任何字符串|没有设置
logImpl | 指定Mybatis所用日志的具体实现，未指定时将自动查找 | SLF4J、LOG4J、LOG4J2、JDK_LOGGING、COMMONS_LOGGING、STDOUT_LOGGING、NO_LOGGING|没有设置
proxyFactory | 指定MyBatis创建具有延迟加载能力的对象所用到的代理工具 |CGLIB、JAVASSIST|版本3.3.0(含)以上JAVASSIST,否则CGLIB

```
<settings>
	<setting name="cacheEnabled" value="true" />
	<setting name="lazyLoadingEnabled" value="true" />
</settings>
```

> 别名设置

- 系统别名
![系统别名][4]
![系统别名][5]
- 自定义别名
```
<typeAliases>
	<!-- 直接定义别名-->
	<typeAlias alias="user" type="com.study.entity.User"/>
	<!-- 通过自动扫描包自定义别名,包下的类需要使用@Alias注解-->
	<package name="com.study.entity"/>
</typeAliases>
```
包下的类
```
@Alias("user")
public class User{}
```
> typeHandler类型处理器

其注册是在org.apache.ibatis.TypeHandlerRegistry中类通过register来进行注册的，下面列出一些系统默认注册的typeHandler。

![typeHandler类型处理器][6]
![typeHandler类型处理器][7]

简单的说下自定义typeHandler
- 首先需要注册自定义typeHandler
```
<typeHandlers>
	<typeHandler jdbcType="VARCHAR" javaType="string" handler="com.study.testhandler.MytestTypeHandler"/>
	<package name="com.study.testhandler"/>
</typeHandlers>
```
- 编写自定义typeHandler，需要实现TypeHandler 或继承BaseTypeHandler 。还需要通过@MappedTypes({String.class})和@MappedJdbcTypes(JdbcType.VARCHAR) 来在类上进行注解
- 对所需要的属性配置typeHandler
	- 在resultMap 定义转换的时候，通过typeHandler 定义
	- 在使用的时候比如查询的时候字节定义typeHandler

> ObjectFactory

编写自定义的ObjectFactory，需要实现ObjectFactory接口。
> environments配置环境

![environments][8]
	- environments 中的属性default，标明在缺省的情况下，我们将启用那个数据源配置
	- environment元素时配置一个数据源的开始，id是数据源标志
		- transactionManager配置数据库事务
			- JDBC,采用JDBC方式管理事务
			- MANAGED 采用容器方式管理事务，在JNDI数据源中常用
			- 自定义
		- property元素配置数据源各类属性
		- dataSource配置数据源连接信息，type属性是对数据库连接方式的配置
			- UNPOOLED 非连接池数据库
			- POOLED 连接池数据库
			- JNDI JNDI数据源
			- 自定义数据源


> 映射器配置

- 用文件路径引入映射器
![enter description here][9]
- 用包名引入映射器
![enter description here][10]
 
-  用类注册引入映射器
![enter description here][11]
- 用userMapper.xml 引入映射器
![enter description here][12]
 


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503444647407.jpg
  [2]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503445687812.jpg
  [3]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503445724983.jpg
  [4]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503489606114.jpg
  [5]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503489639452.jpg
  [6]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503531878858.jpg
  [7]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503531910485.jpg
  [8]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503533376017.jpg
  [9]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503533984111.jpg
  [10]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503533992376.jpg
  [11]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503534000995.jpg
  [12]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503534007891.jpg