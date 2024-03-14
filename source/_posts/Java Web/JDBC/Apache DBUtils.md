---
title: Apache DBUtils
tags:
  - JDBC
categories:
  - Java Web
  - JDBC
date: 2024-03-13 21:10:25
---
# Apache DBUtils

学习 `Apache DBUtils` 之前我们先回顾一下传统的 JDBC 有什么缺点：

1. 返回的结果集 ResultSet 与 Connection 是关联的，当调用 Connection 的 close 方法关闭连接后（放回连接池），ResultSet 对象就不能用了。如果在关闭连接后仍调用ResultSet，会报异常，如下图所示 : 

   ![img](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403122023050.png)

2. 即使获取完 ResultSet 的数据之后再关闭连接，ResultSet 也仅仅使用了一次，<font color=red>不利于数据的管理</font>

3. ResultSet 获取结果只能通过 `getXxx(int|String)` 或 `getObject(int|String)` 方法，不符合日常代码习惯

而 `Apache DBUtils` 就是为了解决上述问题出现爱你的

> Apache DBUtils 通过创造一个 Java 类用于对应一张表，该类中所有的属性对应表中的所有字段，即该类的每个对象都表示了表中的一条记录。查询到表中有几条记录，就创建几个该类的实例，不同实例的属性可以自行设置。这样一来，我们只需要将该类的对象存放在 ArrayList 集合中，就实现了数据的“迁移”，结果集中的数据也得以复用
>

![image-20240312202952102](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403122029560.png)

## API

dbutils 中常用类与接口如下：

- **QueryRunner** 类 : 该类封装了SQL的执行，并且是线程安全的；可以实现增删查改，并且支持批处理
- **ResultSetHandler** 接口 : 该接口用于处理 `java.sql.ResultSet`，将数据按照要求转换为另一种格式。常见实现类如下
  - **ArrayHandler** : 将结果集中的第一行数据转换成对象数组
  - **ArrayListHandler** : 将结果集中的每一行数据转换成对象数组，再存入 List 中
  - **BeanHandler** : 将结果集中的第一行数据封装到一个对应的 JavaBean 实例中（适用于返回单条记录的情况）
  - **BeanListHandler** : 将结果集中的每一行数据都封装到对应的 JavaBean 实例中，再存放到 List 集合中
  - **ColumnListHandler** : 将结果集中某一列的数据存放到 List 中
  - **KeyedHandler(name)** : 将结果集中的每行数据都封装到 Map 里，然后将所有的map再单独存放到一个 map 中，其 key 为指定的key
  - **MapHandler** : 将结果集中的第一行数据封装到一个 Map 里，key 是列名，value 就是对应的值
  - **MapListHandler** : 将结果集中的每一行数据都封装到 Map 里，再存入 List
  - **ScalarHandler** : 将结果集中的一列映射为一个 Object 对象，适用于返回单行单列的情况

## 实例

表结构

![image-20240312205221034](C:\Users\ZYP\AppData\Roaming\Typora\typora-user-images\image-20240312205221034.png)

实体类

```java
@Getter
@Setter
@Data
@ToString
public class User {
    private int id;
    private String name;
    private int age;
    private double salary;
    private int sex;
}
```

测试

```java
@Test
public void testDBUtils() throws Exception{
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("druid.properties"));
    DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
    try (Connection connection = dataSource.getConnection()) {
        String sql = "select * from t_user where id = ?";
        QueryRunner queryRunner = new QueryRunner();
        List<User> userList = (List<User>) queryRunner.query(connection, sql, new BeanListHandler(User.class), 1);
        for (int i = 0; i < userList.size(); i++) {
            // User(id=1, name=lisi, age=1, salary=0.0, sex=0)
            System.out.println(userList.get(i));
        }
    }
}
```

## 源码解析

```java
private <T> T query(Connection conn, boolean closeConn, String sql, ResultSetHandler<T> rsh, Object... params) {
    PreparedStatement stmt = null;
    ResultSet rs = null;
    T result = null;

    try {
        // 调用 conn.preparedStatement() 方法创建 prepareStatement
        stmt = this.prepareStatement(conn, sql);
        // 设置参数
        this.fillStatement(stmt, params);
        // 获取结果集
        rs = this.wrap(stmt.executeQuery());
        // 根据 Handler 类型针对结果集做处理
        result = rsh.handle(rs);
    } catch (SQLException e) {
        this.rethrow(e, sql, params);
    } finally {
        // 关闭连接
        try {
            close(rs);
        } finally {
            close(stmt);
            if (closeConn) {
                close(conn);
            }
        }
    }
	// 返回结果集
    return result;
}
```

BeanListHandler 处理结果集

```java
public <T> List<T> toBeanList(ResultSet rs, Class<? extends T> type) throws SQLException {
    List<T> results = new ArrayList<T>();

    if (!rs.next()) {
        return results;
    }
	// 获取类信息
    PropertyDescriptor[] props = this.propertyDescriptors(type);
    // 获取结果集元信息
    ResultSetMetaData rsmd = rs.getMetaData();
    int[] columnToProperty = this.mapColumnsToProperties(rsmd, props);
    do {     
        // 实例化对象
        results.add(this.createBean(rs, type, props, columnToProperty));
    } while (rs.next());
    return results;
}
```

## DML

```java
@Test
public void testDBUtilsDML() throws Exception{
    Properties properties = new Properties();
    properties.load(this.getClass().getClassLoader().getResourceAsStream("druid.properties"));
    DataSource dataSource = DruidDataSourceFactory.createDataSource(properties);
    try (Connection connection = dataSource.getConnection()) {
        String sql = "insert into t_user(name) values(?)";
        QueryRunner queryRunner = new QueryRunner();
        // 返回自增列
        Long id = queryRunner.insert(connection, sql, new ScalarHandler<>(), "zhansan");
        System.out.println(id);
        User user = new User();
        user.setAge(20);
        user.setName("zhangsan");
        user.setId(Math.toIntExact(id));
        sql = "update t_user set name = ?,age = ? where id = ?";
        // 返回受影响行数
        int update = queryRunner.update(connection, sql, user.getName(), user.getAge(), user.getId());
        System.out.println(update);
        sql = "delete from t_user where id = ?";
        // 返回受影响行数
        update = queryRunner.update(connection, sql, id);
        System.out.println(update);
    }
}
```

