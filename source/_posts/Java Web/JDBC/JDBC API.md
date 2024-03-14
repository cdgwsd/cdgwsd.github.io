---
title: JDBC API
tags:
  - JDBC
categories:
  - Java Web
  - JDBC
date: 2024-03-13 21:10:58
---
# 	JDBC API

## Driver Manager

### 驱动管理类

### getConnection(url,user,pwd) 获取数据库连接

## Connection

### 数据库连接对象

### createStatement() 创建 Statement 对象

### preparedStatement(String sql) 生成预处理对象

## Statement

### 执行静态 SQL 语句并返回执行结果

### int executeUpdate(String sql) 执行 DML 语句，返回受影响行数

### ResultSet executeQuery(String sql) 执行查询语句，返回结果集

### boolean execute(String sql) 执行任意 sql，返回执行结果

## PreparedStatement

### 执行预编译 SQL 语句

### int executeUpdate() 执行 DML 语句，返回受影响行数

### ResultSet executeQuery() 执行查询语句，返回结果集

### boolean execute() 执行任意 sql，返回执行结果

### setXxx(占位符索引，占位符值)

### SetObj(占位符索引，占位符值)

#### setObject() 方法可以自动识别 Java 对象的类型，并将其转换为对应的 SQL 类型

#### 如果传入参数值为 null，则必须显式指定参数类型或者使用 setNull() 方法

##### ps.setObject(1, null, Types.INTEGER); 

##### ps.setNull(1);

## ResultSet

### 数据库查询结果集

### next() 向下移动一行，如果没有下一行，返回 false

### previous 向上移动一行，如果没有上一行，返回 false

### Xxx getXxx(列索引|列名) 返回对应列的值

### Object getObject(列索引|列名) 返回对应列的值

