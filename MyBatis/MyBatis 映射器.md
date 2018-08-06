---
title: MyBatis 映射器
tags: java
grammar_cjkRuby: true
---

### 映射器内部组成
- MappedStatement，它保存映射器的一个节点（select|insert|delete|update）。包括许多我们配置的SQL、SQL的id、缓存信息、resultMap、parameterType、resultType、languageDriver等重要配置内容
- SqlSource，提供BoundSql对象的地方。是MappedStatement的一个属性。
- BoundSql，建立SQL和参数的地方、有3个常用的属性：SQL、parameterObject、parameterMappings

### SqlSession 下的四大对象
- Executor代表执行器，由它来调度StatementHandler、ParameterHandler、ResultHandler来执行对应的SQL
- StatementHandler的作用是使用数据库的Statement(PreparedStatement)执行操作，核心
- ParameterHandler用于SQL对参数的处理
- ResultHandler是进行最后数据集(ResultSet)的封装返回处理

> 执行器：有三种执行器，从setting元素的属性defaultExecutorType配置

- SIMPLE：简单执行器，默认
- REUSE：一种执行器重用预处理语句
- BATCH：执行器重用语句和批量更新，针对批量专用的执行器


![SqlSession运行总结][1]


  [1]: https://www.github.com/COBSNAN/ImageHub/raw/master/1503964478080.jpg