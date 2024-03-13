---
title: JDBC 事物
date: 2024-03-13 21:10:55
tags:
  - JDBC
  - 事务
---
# JDBC 事物

数据库事务（Transaction）是由若干个 SQL 语句构成的一个操作序列。数据库系统保证在一个事务中的所有 SQL 要么全部执行成功，要么全部不执行，即数据库事务具有 ACID 特性：

- Atomicity：原子性
- Consistency：一致性
- Isolation：隔离性
- Durability：持久性

要在 JDBC 中执行事务，本质上就是如何把多条 SQL 包裹在一个数据库事务中执行。JDBC 事物具有以下特点

- JDBC 程序中当一个 Connection 对象创建时，默认情况下是自动提交事务：每次执行一个 SQL 语句时，如果执行成功，就会向数据库自动提交，而不能回滚
- JDBC 程序中为了让多个 SQL 语句作为一个整体执行，需要使用事务
- 调用 Connection 的 `setAutocommit(false)` 可以取消自动提交事务
- 在所有的 SQL 语句都成功执行后，调用 Connection 的 `commit()` 方法提交事务
- 在其中某个操作失败或出现异常时，调用 Connection 的 `rolback()` 方法回滚事务

## 实例

```java
@Test
public void testTransAction() throws Exception {
    Class.forName("com.mysql.jdbc.Driver");
    Connection connection = null;
    PreparedStatement prepareStatement = null;
    try {
        connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false&characterEncoding=UTF-8", "root", "12345678");
        connection.setAutoCommit(false);
        prepareStatement = connection.prepareStatement("update t_user set name = ? where id = ?");
        prepareStatement.setString(1, "lisi");
        prepareStatement.setInt(2, 1);
        int id = prepareStatement.executeUpdate();
        connection.commit();
        System.out.println(id);
        connection.setAutoCommit(true);
    } catch (Exception e) {
        connection.rollback();
        System.out.println(e.getMessage());
    } finally {
        if (prepareStatement != null) {
            prepareStatement.close();
        }
        if (connection != null) {
            connection.close();
        }
    }
}
```

