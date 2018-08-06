---
title: MyBatis 插件
tags: java
grammar_cjkRuby: true
---

### 实现接口
```
package org.apache.ibatis.plugin;

import java.util.Properties;

public interface Interceptor {

  Object intercept(Invocation invocation) throws Throwable;

  Object plugin(Object target);

  void setProperties(Properties properties);

}
```

- intercept方法：直接覆盖你所拦截对象原有的方法
- plugin方法：target是被拦截对象，它的作用是给被拦截对象生成一个代理对象，并返回它。一般通过Plugin的静态方法wrap来生成代理对象
- setProperties 方法：允许在plugin元素中配置所需参数，方法在插件初始化的时候调用并存入配置中

### 插件初始化
```
package org.apache.ibatis.builder.xml
public class XMLConfigBuilder extends BaseBuilder {
	private void pluginElement(XNode parent) throws Exception {
		if (parent != null) {
		  for (XNode child : parent.getChildren()) {
			String interceptor = child.getStringAttribute("interceptor");
			Properties properties = child.getChildrenAsProperties();
			Interceptor interceptorInstance = (Interceptor) resolveClass(interceptor).newInstance();
			interceptorInstance.setProperties(properties);
			configuration.addInterceptor(interceptorInstance);
		  }
		}
	  }
}
 ```
 
 ### MetaObject工具
 - MetaObject forObject(Object object, ObjectFactory objectFactory, ObjectWrapperFactory objectWrapperFactory)方法用于包装对象，现在用SystemMetaObject.forObject(Object object)进行代替
 - Object getValue(String name)方法用于获取对象属性值，支持OGNL
 - void setValue(String name, Object value)方法用于修改对象属性值
 

### 确定拦截的签名
1. 确定拦截的对象
	- Executor是执行SQL的全过程，包括组装参数，组装结果集返回和执行SQL过程，都可以拦截。
	- StatementHandler是执行SQL的过程，我们可以重写执行SQL的过程
	- ParameterHandler主要是拦截执行SQL的参数组装
	- ResultSetHandler用于拦截执行结果的组装
2.  拦截方法和参数

```
package org.apache.ibatis.executor.statement;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

import org.apache.ibatis.executor.parameter.ParameterHandler;
import org.apache.ibatis.mapping.BoundSql;
import org.apache.ibatis.session.ResultHandler;

public interface StatementHandler {

  Statement prepare(Connection connection)
      throws SQLException;

  void parameterize(Statement statement)
      throws SQLException;

  void batch(Statement statement)
      throws SQLException;

  int update(Statement statement)
      throws SQLException;

  <E> List<E> query(Statement statement, ResultHandler resultHandler)
      throws SQLException;

  BoundSql getBoundSql();

  ParameterHandler getParameterHandler();

}
```

比如说拦截StatementHandler的prepare方法，插件签名定义
```
import com.mysql.jdbc.Connection;
@Intercepts({
	@Signature(type = StatementHandler.class,
			method = "prepare",
			args = {Connection.class})
})
public class test implements Interceptor{
	// ...
}
```

### 配置和运行
```
<plugins>
	<plugin interceptor="plugin.test">
		<property name="dbType" value= "mysql" />
	</plugin>
</plugins>
```


### 一个插件实例
```
package com.time.test;

import java.util.Properties;

import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.plugin.Interceptor;
import org.apache.ibatis.plugin.Intercepts;
import org.apache.ibatis.plugin.Invocation;
import org.apache.ibatis.plugin.Plugin;
import org.apache.ibatis.plugin.Signature;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;

import com.mysql.jdbc.Connection;
@Intercepts({
	@Signature(type = StatementHandler.class,
			method = "prepare",
			args = {Connection.class})
})
public class test implements Interceptor{
	private String dbType;
	private static final String LMT_TABLE_NAME = "limit_table_name";
	
	@Override  
    public Object intercept(Invocation invocation) throws Throwable {  

            StatementHandler statementHandler = (StatementHandler) invocation.getTarget();  
            MetaObject metaStatementHandler = SystemMetaObject.forObject(statementHandler);  
            // 分离代理对象链(由于目标类可能被多个拦截器拦截，从而形成多次代理，通过下面的两次循环  
            // 可以分离出最原始的的目标类)  
            while (metaStatementHandler.hasGetter("h")) {  
                Object object = metaStatementHandler.getValue("h");  
                metaStatementHandler = SystemMetaObject.forObject(object);  
            }  
            // 分离最后一个代理对象的目标类  
            while (metaStatementHandler.hasGetter("target")) {  
                Object object = metaStatementHandler.getValue("target");  
                metaStatementHandler = SystemMetaObject.forObject(object);  
            }  
            String sql = (String) metaStatementHandler.getValue("delegate.boundSql.sql");  
              
            //重写限制sql  
            String limitSql="";
            if("mysql".equals(this.dbType) && sql.indexOf(LMT_TABLE_NAME)==-1){
            	sql = sql.trim();
            	limitSql = "select * from ("+sql+") "+ LMT_TABLE_NAME + " limit 100";
            }
            metaStatementHandler.setValue("delegate.boundSql.sql", limitSql);  
            
            // 将执行权交给下一个拦截器  
            return invocation.proceed();  
    }
  
    @Override  
    public Object plugin(Object target) {  
        return Plugin.wrap(target, this);  
    }

	@Override
	public void setProperties(Properties properties) {
		// TODO Auto-generated method stub
		this.dbType = (String) properties.getProperty("dbType","mysql");
		
	} 
}

```
