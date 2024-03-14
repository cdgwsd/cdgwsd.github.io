---
title: JDBC 批处理
tags:
  - JDBC
categories:
  - Java Web
  - JDBC
date: 2024-03-13 21:10:53
---
使用 JDBC 操作数据库的时候，经常会执行一些批量操作。

例如，一次性给会员增加可用优惠券若干，我们可以执行以下 SQL 代码：

```sql
INSERT INTO coupons (user_id, type, expires) VALUES (123, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (234, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (345, 'DISCOUNT', '2030-12-31');
INSERT INTO coupons (user_id, type, expires) VALUES (456, 'DISCOUNT', '2030-12-31');
```

实际上执行 JDBC 时，因为只有占位符参数不同，所以 SQL 实际上是一样的：

```sql
for (var params : paramsList) {
    PreparedStatement ps = conn.preparedStatement("INSERT INTO coupons (user_id, type, expires) VALUES (?,?,?)");
    ps.setLong(params.get(0));
    ps.setString(params.get(1));
    ps.setString(params.get(2));
    ps.executeUpdate();
}
```

类似的还有，给每个员工薪水增加 10%～30%：

```sql
UPDATE employees SET salary = salary * ? WHERE id = ?
```

通过一个循环来执行每个 `PreparedStatement` 虽然可行，但是性能很低。SQL 数据库对 SQL 语句相同，但只有参数不同的若干语句可以作为 <font color=red>batch</font> 执行，即批量执行，这种操作有特别优化，速度远远快于循环执行每个 SQL

在 JDBC 代码中，我们可以利用 SQL 数据库的这一特性，<font color=red>把同一个 SQL 但参数不同的若干次操作合并为一个 batch 执行</font>。我们以批量插入为例，示例代码如下：

```java
try (PreparedStatement ps = conn.prepareStatement("INSERT INTO students (name, gender, grade, score) VALUES (?, ?, ?, ?)")) {
    // 对同一个PreparedStatement反复设置参数并调用addBatch():
    for (Student s : students) {
        ps.setString(1, s.name);
        ps.setBoolean(2, s.gender);
        ps.setInt(3, s.grade);
        ps.setInt(4, s.score);
        ps.addBatch(); // 添加到batch
    }
    // 执行batch:
    int[] ns = ps.executeBatch();
    for (int n : ns) {
        System.out.println(n + " inserted."); // batch中每个SQL执行的结果数量
    }
}
```

执行 batch 和执行一个SQL不同点在于，需要对同一个 `PreparedStatement` 反复设置参数并调用 `addBatch()`，这样就相当于给一个 SQL 加上了多组参数，相当于变成了“多行” SQL。

第二个不同点是调用的不是 `executeUpdate()`，而是 `executeBatch()`，因为我们设置了多组参数，相应地，返回结果也是多个 `int` 值，因此返回类型是 `int[]`，循环 `int[]` 数组即可获取每组参数执行后影响的结果数量

## MySQL

JDBC 连接 MySQL 数据库时，如果需要使用批处理功能，需要在连接 URL 中设置 `rewriteBatchedStatements = true`

> 当设置 `rewriteBatchedStatements=true` 时，MySQL 将尝试将一组相似的 SQL 语句重写为单个 SQL 语句。这样做可以减少通信开销，并且在数据库端可以更有效地执行批处理操作，从而提高整体性能。