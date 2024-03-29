---
title: JDBC CRUD
tags:
  - JDBC
categories:
  - Java Web
  - JDBC
date: 2024-03-13 21:11:01
---
## **查询**

```java
@Test
public void testInsert() throws Exception {
    Class.forName("com.mysql.jdbc.Driver");
    try (Connection connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false", "root", "12345678")) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("insert into t_user(name,age,salary,sex) values(?,?,?,?)", 1)) {
            prepareStatement.setString(1, "李四");
            prepareStatement.setInt(2, 27);
            prepareStatement.setDouble(3, 2.75);
            prepareStatement.setInt(4, 1);
            int i = prepareStatement.executeUpdate();
            System.out.println(i);
            try (ResultSet generatedKeys = prepareStatement.getGeneratedKeys()) {
                while (generatedKeys.next()) {
                    System.out.println(generatedKeys.getInt(1));
                }
            }
        }
    }
}
```
## 新增

```java
@Test
public void testInsert() throws Exception {
    Class.forName("com.mysql.jdbc.Driver");
    try (Connection connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false", "root", "12345678")) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("insert into t_user(name,age,salary,sex) values(?,?,?,?)", 1)) {
            prepareStatement.setString(1, "李四");
            prepareStatement.setInt(2, 27);
            prepareStatement.setDouble(3, 2.75);
            prepareStatement.setInt(4, 1);
            int i = prepareStatement.executeUpdate();
            System.out.println(i);
            try (ResultSet generatedKeys = prepareStatement.getGeneratedKeys()) {
                while (generatedKeys.next()) {
                    System.out.println(generatedKeys.getInt(1));
                }
            }
        }
    }
}
```
### 获取自增主键

如果数据库的表设置了自增主键，那么在执行 `INSERT` 语句时，并不需要指定主键，数据库会自动分配主键。对于使用自增主键的程序，有个额外的步骤，就是如何获取插入后的自增主键的值

JDBC 中 可以在创建 `PreparedStatement` 的时候，指定一个 `RETURN_GENERATED_KEYS` 标志位，表示 JDBC 驱动必须返回插入的自增主键，然后使用 `getGeneratedKeys()` 方法获取自增主键

```java
// int RETURN_GENERATED_KEYS = 1; 返回自增主键
// int NO_GENERATED_KEYS = 2; 不返回
PreparedStatement prepareStatement(String sql, int autoGeneratedKeys)
```

如果自增列有多个，可以通过数组进行指定

```java
PreparedStatement prepareStatement(String sql, int columnIndexes[]);
    
PreparedStatement prepareStatement(String sql, String columnNames[]);
```

示例：`monster` 表中的 `id` 字段自增

```java
@Test  
public void testConn() throws Exception {  
    // 注册驱动  
    Class.forName("com.mysql.jdbc.Driver");  
    // 获取连接  
    try (Connection connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis?useSSL=true&useUnicode=true&characterEncoding=UTF-8", "root", "Zyp,1234")) {  
        // 执行数据库操作  
        String sql = "insert into monster(name) values (?)";  
        try (PreparedStatement preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS)) {  
            preparedStatement.setString(1, "cdgwsd");  
            int i = preparedStatement.executeUpdate();  
            System.out.println("插入成功：" + i);  
            try(ResultSet generatedKeys = preparedStatement.getGeneratedKeys()){  
                while (generatedKeys.next()){  
                    System.out.println("id："+generatedKeys.getObject(1));  
                }  
            }  
        }  
    }  
}
```
## 删除

```java
@Test
public void testDel() throws Exception {
    Class.forName("com.mysql.jdbc.Driver");
    try (Connection connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false", "root", "12345678")) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("delete from t_user where id = ?")) {
            prepareStatement.setInt(1, 4);
            int i = prepareStatement.executeUpdate();
            System.out.println(i);
        }
    }
}
```
## 更新

```java
@Test
public void testUpdate() throws Exception {
    Class.forName("com.mysql.jdbc.Driver");
    try (Connection connection = DriverManager.getConnection("jdbc:mysql://120.24.90.60:3306/mybatis_study?useSSL=false", "root", "12345678")) {
        try (PreparedStatement prepareStatement = connection.prepareStatement("update t_user set name = ? where id = ?")) {
            prepareStatement.setString(1, "张三");
            prepareStatement.setInt(2, 1);
            int i = prepareStatement.executeUpdate();
            System.out.println(i);
        }
    }
}
```