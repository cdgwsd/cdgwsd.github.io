---
title: JDBC 连接池
tags:
  - JDBC
  - 数据库连接池
categories:
  - Java Web
  - JDBC
date: 2024-03-13 21:10:32
---
# JDBC 连接池

在执行 JDBC 的增删改查操作时，如果每一次操作都要建立一次数据库连接，操作，关闭连接，那么创建和销毁JDBC 连接的开销就太大。为了避免频繁地创建和销毁 JDBC 连接，我们可以通过连接池（Connection Pool）<font color=red>复用</font>已经创建好的连接

## 原理

数据库连接池是一种管理和维护数据库连接的技术，其原理可以简单描述如下：

1. **连接池初始化：** 在应用程序启动时，数据库连接池会初始化一定数量的数据库连接，并将这些连接保存在一个连接池中
2. **连接请求处理：** 当应用程序需要与数据库交互时，它会从连接池中请求一个数据库连接
3. **连接复用：** 如果连接池中有空闲连接可用，那么连接池会返回一个空闲连接给应用程序使用。如果连接池中没有空闲连接，且此时连接数已达最大连接数则请求进入等待队列，否则连接池会创建一个新的连接并返回给应用程序使用
4. **连接释放：** 当应用程序使用完数据库连接后，它会将连接<font color=red>释放回连接池，而不是关闭连接</font>。连接池会将释放的连接标记为空闲状态，以备后续的重用
5. **连接池管理：** 连接池会负责管理连接的状态和生命周期。它会监控连接的空闲时间，如果连接空闲时间过长，则可能会关闭连接并释放资源。此外，连接池还可以根据需求动态调整连接池中连接的数量，以适应当前的数据库负载和应用程序的需求

![img](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403121130817.png)

## 连接池种类

JDBC 连接池有一个标准的接口 `javax.sql.DataSource`，这个类位于 Java 标准库中，但仅仅是接口。要使用JDBC 连接池，我们必须选择一个 JDBC 连接池的实现。常用的 JDBC 连接池有：

- C3P0：速度较慢，稳定性可以
- HikariCP：速度快，稳定
- Druid：速度快，稳定
- BoneCP：速度快

### C3P0

#### 通过类进行配置

```java
@Test
public void testC3P0ByClass() throws Exception {
    ComboPooledDataSource dataSource = new ComboPooledDataSource();
    // 设置数据源连接信息
    dataSource.setDriverClass("com.mysql.jdbc.Driver");
    dataSource.setJdbcUrl("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false&characterEncoding=UTF-8");
    dataSource.setUser("root");
    dataSource.setPassword("12345678");
    // 设置连接池信息
    // 最大连接数
    dataSource.setMaxPoolSize(50);
    // 初始连接数
    dataSource.setInitialPoolSize(10);
    try (Connection connection = dataSource.getConnection()) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("select * from t_user where id = ?")) {
            prepareStatement.setInt(1, 1);
            try (ResultSet resultSet = prepareStatement.executeQuery()) {
                while (resultSet.next()) {
                    System.out.println(resultSet.getInt("id"));
                }
            }
        }
    }
```

#### 配置文件

```xml
<c3p0-config>
    <!-- 数据源名称代表连接池 -->
    <named-config name="mybatis_study">
        <!-- 驱动类 -->
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <!-- url-->
        <property name="jdbcUrl">jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false&amp;characterEncoding=UTF-8</property>
        <!-- 用户名 -->
        <property name="user">root</property>
        <!-- 密码 -->
        <property name="password">12345678</property>
        <!-- 每次增长的连接数-->
        <property name="acquireIncrement">5</property>
        <!-- 初始的连接数 -->
        <property name="initialPoolSize">10</property>
        <!-- 最小连接数 -->
        <property name="minPoolSize">5</property>
        <!-- 最大连接数 -->
        <property name="maxPoolSize">50</property>

        <!-- 可连接的最多的命令对象数 -->
        <property name="maxStatements">5</property>

        <!-- 每个连接对象可连接的最多的命令对象数 -->
        <property name="maxStatementsPerConnection">2</property>
    </named-config>
</c3p0-config>
```

测试代码

```java
@Test
public void testC3P0ByXML() throws Exception {
    ComboPooledDataSource dataSource = new ComboPooledDataSource("mybatis_study");
    try (Connection connection = dataSource.getConnection()) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("select * from t_user where id = ?")) {
            prepareStatement.setInt(1, 1);
            try (ResultSet resultSet = prepareStatement.executeQuery()) {
                while (resultSet.next()) {
                    System.out.println(resultSet.getInt("id"));
                }
            }
        }
    }
}
```

### DRUID

配置文件

```properties
# 数据库驱动
driverClassName=com.mysql.jdbc.Driver
# 数据库连接URL
url=jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false
# 数据库用户名
username=root
# 数据库密码
password=12345678

# 初始化连接数
initialSize=5
# 最小空闲连接数
minIdle=5
# 最大活跃连接数
maxActive=20
# 获取连接时最大等待时间（单位：毫秒）
maxWait=60000

# 获取连接时是否检测连接的有效性
testWhileIdle=true
# 定期检测连接的有效性，单位：毫秒
timeBetweenEvictionRunsMillis=60000
# 连接在池中最小生存时间，单位：毫秒
minEvictableIdleTimeMillis=300000
```

测试代码

```java
@Test
public void testDruid() throws Exception{
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("druid.properties"));
    DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
    try (Connection connection = dataSource.getConnection()) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("select * from t_user where id = ?")) {
            prepareStatement.setInt(1, 1);
            try (ResultSet resultSet = prepareStatement.executeQuery()) {
                while (resultSet.next()) {
                    System.out.println(resultSet.getInt("id"));
                }
            }
        }
    }
}
```

