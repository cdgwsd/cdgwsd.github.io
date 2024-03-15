---
title: MyBatis 核心接口
tags:
  - MyBatis
categories:
  - MyBatis
date: 2024-03-15 15:00:00
---
## MyBatis 工作原理

![image-20240315143949635](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403151550947.png)

## 核心配置文件

### mybatis-config.xml

MyBatis 的配置文件 `mybatis-config.xml` 是用来配置 MyBatis 框架的全局设置和属性的文件。它的主要作用包括：

1. **数据库连接配置**：在 `mybatis-config.xml` 中可以配置数据库连接的相关信息，包括数据库驱动、连接地址、用户名、密码等。这些信息会被 MyBatis 用来建立与数据库的连接
2. **对象工厂和类型处理器配置**：可以配置对象工厂（ObjectFactory）和类型处理器（TypeHandler）的实现类，用于创建对象实例和处理数据库字段与 Java 类型之间的转换
3. **插件配置**：可以配置插件（Plugins），用于拦截和增强 MyBatis 的功能。插件可以用来实现日志记录、性能监控、权限验证等功能
4. **缓存配置**：可以配置缓存（Cache）的相关属性，包括缓存类型、缓存大小、缓存清理策略等。MyBatis 提供了一级缓存和二级缓存，可以根据需要配置不同的缓存策略
5. **环境配置**：可以配置不同的环境（Environments），例如开发环境、测试环境、生产环境等，每个环境可以配置不同的数据源和事务管理器
6. **映射器配置**：可以配置映射器（Mapper）的位置和加载方式，告诉 MyBatis 在哪里找到映射器接口或 XML 文件，并且如何加载它们

### XxxMapper.xml

映射文件是 MyBatis 中用于配置 SQL 映射的文件。这些文件定义了 SQL 查询、更新、删除等操作与 Java 方法的映射关系，其作用包括：

1. **定义 SQL 语句**：映射文件中可以编写各种 SQL 语句，包括查询、插入、更新、删除等操作。这些 SQL 语句通常与数据库操作相关联，用于执行数据库的增、删、改、查操作
2. **与 Java 方法的映射**：映射文件将 SQL 语句与 Java 方法进行了映射，通过在映射文件中定义的 `<select>`、`<insert>`、`<update>`、`<delete>` 等标签，将 SQL 语句与 Java 方法名进行关联
3. **参数映射**：映射文件中可以定义参数映射，指定 Java 方法参数与 SQL 语句中的参数之间的关系。这样可以在 Java 方法中直接使用参数，而不必在 SQL 语句中进行硬编码
4. **结果集映射**：映射文件中可以定义结果集映射，将数据库查询结果映射为 Java 对象或基本数据类型。通过定义 `<resultMap>` 标签，可以指定数据库字段与 Java 对象属性之间的映射关系
5. **动态 SQL**：映射文件支持动态 SQL，可以根据条件动态生成 SQL 语句，以满足不同的查询需求。通过使用 `<if>`、`<choose>`、`<foreach>` 等标签，可以实现条件判断、循环等逻辑
6. **引用其他映射文件**：映射文件支持引用其他映射文件，可以将 SQL 映射分解为多个文件，提高可维护性和复用性

## 核心接口

### SqlSessionFactoryBuilder

`SqlSessionFactoryBuilder` 类是 MyBatis 中用于构建 `SqlSessionFactory` 对象的建造者类

`SqlSessionFactoryBuilder` 类的主要作用是通过读取 MyBatis 的配置文件（如 `mybatis-config.xml`）并根据配置信息创建 `SqlSessionFactory` 实例。`SqlSessionFactory` 的创建过程需要使用 `SqlSessionFactoryBuilder` 的 `build()` 方法，这个方法接受一个输入流或者一个 `Reader` 对象作为参数，从而加载 MyBatis 的配置信息并创建相应的 `SqlSessionFactory`

使用 `SqlSessionFactoryBuilder` 类的流程通常是这样的：

1. 创建一个 `SqlSessionFactoryBuilder` 实例。
2. 调用 `build()` 方法并传入配置文件的输入流或者 `Reader` 对象。
3. 获取 `SqlSessionFactory` 实例。
4. 通过 `SqlSessionFactory` 实例创建 `SqlSession` 对象。

<font color=red>一旦创建了 SqlSessionFactory，就不再需要 `SqlSessionFactoryBuilder` 了，可以重复创建，它的最佳作用域是方法作用域</font>

### SqlSessionFactory

`SqlSessionFactory` 是 MyBatis 框架中的一个重要接口，用于创建 `SqlSession` 实例。其主要作用是：

1. **创建 `SqlSession` 实例**：`SqlSessionFactory` 接口定义了 `openSession()` 方法，用于创建一个新的 `SqlSession` 对象。每个 `SqlSession` 都是一个单独的数据库会话，它负责执行一系列的 SQL 命令
2. **获取配置信息**：`SqlSessionFactory` 接口通常会保存 MyBatis 的配置信息，包括数据库连接信息、映射器（Mapper）配置、缓存配置等。这些配置信息会在创建 `SqlSession` 实例时被使用
3. **管理资源**：`SqlSessionFactory` 接口通常会负责管理资源，如数据库连接池、缓存等。它会确保资源的正确关闭和释放，以避免资源泄露和内存溢出
4. **线程安全性**：`SqlSessionFactory` 接口通常是线程安全的，多个线程可以同时使用同一个 `SqlSessionFactory` 实例来创建 `SqlSession` 对象，而不会出现线程安全问题

<font color=red>SqlSessionFactory 一旦被创建就应该在应用的运行期间一直存在，没有任何理由丢弃它或重新创建另一个实例，它的最佳作用域是应用作用域</font>

### SqlSession

`SqlSession` 接口是 MyBatis 中用于执行 SQL 命令和管理事务的核心接口之一。它代表了与数据库的一次会话，可以通过它来执行 SQL 命令、获取映射器（Mapper）、管理事务等操作。

`SqlSession` 接口的主要作用包括：

1. **执行 SQL 命令**：通过 `SqlSession` 接口提供的方法可以执行各种 SQL 命令，包括查询（select）、插入（insert）、更新（update）、删除（delete）等操作。这些方法会将 SQL 命令发送到数据库并获取执行结果
2. **获取映射器（Mapper）**：通过 `SqlSession` 接口提供的 `getMapper()` 方法可以获取映射器（Mapper）接口的实例。映射器是 MyBatis 中用于定义 SQL 映射关系的接口，通过映射器可以执行与之关联的 SQL 命令
3. **管理事务**：通过 `SqlSession` 接口提供的事务管理方法可以手动管理事务的提交（commit）和回滚（rollback）。如果不手动管理事务，则 `SqlSession` 会自动提交或回滚事务，具体取决于 `SqlSessionFactory` 的配置
4. **关闭会话**：在使用完 `SqlSession` 后，需要手动调用 `close()` 方法来关闭会话。关闭会话可以释放资源、释放数据库连接等，避免资源泄露和内存溢出

<font color=red>每个线程都应该有它自己的 SqlSession 实例。SqlSession 的实例不是线程安全的，因此是不能被共享的，所以它的最佳的作用域是请求或方法作用域</font>

### 映射器

在 MyBatis 中，映射器（Mapper）是用于执行 SQL 语句的接口实例。每个映射器接口对应着一个或多个 SQL 语句，通过映射器实例可以方便地执行这些 SQL 语句，并将结果映射到 Java 对象中

1. **接口定义**：映射器实例通常是一个接口，其中定义了一系列的方法，每个方法对应着一个 SQL 查询、插入、更新、删除等操作。这些方法的命名和参数与对应的 SQL 语句相关联
2. **与 XML 文件的映射**：在 MyBatis 中，可以使用 XML 文件或者注解来定义映射器接口的实现。XML 文件通常与映射器接口同名，并且位于相同的包路径下，通过 XML 文件中的配置可以与数据库中的表和字段进行映射
3. **执行 SQL 操作**：通过映射器实例的方法可以执行各种 SQL 操作，包括查询（select）、插入（insert）、更新（update）、删除（delete）等。这些方法会将 SQL 语句发送到数据库并获取执行结果。
4. **参数传递**：映射器实例的方法通常会接受一个或多个参数，这些参数会作为 SQL 语句的输入参数，可以根据需要传递给 SQL 语句
5. **结果映射**：执行 SQL 操作后，映射器实例会将查询结果映射到 Java 对象中。可以通过配置文件或者注解来指定结果集的映射关系，将数据库表中的字段映射到 Java 对象的属性中
6. **事务支持**：映射器实例通常会参与到事务的管理中，可以通过配置来控制事务的提交、回滚等操作

<font color=red>然从技术层面上来讲，任何映射器实例的最大作用域与请求它们的 SqlSession 相同。但方法作用域才是映射器实例的最合适的作用域。 也就是说，映射器实例应该在调用它们的方法中被获取，使用完毕之后即可丢弃</font>
