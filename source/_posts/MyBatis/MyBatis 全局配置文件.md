---
title: MyBatis 全局配置文件
tags:
  - MyBatis
categories:
  - MyBatis
date: 2024-03-18 14:20:28
---

# 全局配置文件

MyBatis 的配置文件包含了会深深影响 MyBatis 行为的设置和属性信息。 配置文档的顶层结构如下：

- configuration（配置）
  - properties （属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境）
    - environment（环境配置）
      - transactionManager（事物管理器）
      - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）

## XML 文件头

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
```

**1、** version=“1.0”：声明用的xml版本是1.0

**2、** encoding=“UTF-8”：声明用xml传输数据的时候的字符编码，假如文档里面有中文，编码方式不是UTF-8，传输过去再解码的话中文就会是乱码

**3、** !DOCTYPEconfiguration：DOCTYPE用于声明文档类型，引入DTD文档类型定义(DocumentTypeDefinition)约束，此处表示对于configuration标签下的标签引入了外部mybatis配置文件编写规范，会自动提示及校验配置书写

## configuration（配置）

配置文件的根标签，所有的配置都在此标签内

```xml
<configuration>
	<!--mybatis配置...-->
</configuration>
```

## properties（属性）

properties标签的主要作用是引入外部属性及自定义属性，然后其他配置引入属性使用

比如在外部文件配置数据库连接属性，然后在数据源配置中引入属性使用，这样就可以实现配置分离，需要改的时候，直接改外部配置文件即可

```xml
<properties resource="mysql.properties" url=""></properties>
```

**resource** 属性表示引入本地配置文件，**url** 属性表示引入网络资源配置

### 使用案例

#### (1) 引入外部文件配置属性案例

**1、**resources 目录下添加文件 mysql.properties

```java
db.url=jdbc:mysql://127.0.0.1:3306/angel_admin?serverTimezone=Asia/Shanghai
db.driver=com.mysql.cj.jdbc.Driver
db.username=root
db.password=123456
```

**2、**dataSource 数据源配置使用 `${}` 引入外部属性配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <properties resource="mysql.properties"/>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!--驱动名-->
                <property name="driver" value="${db.driver}"/>
                <!--数据库地址-->
                <property name="url" value="${db.url}"/>
                <!--用户名-->
                <property name="username" value="${db.username}"/>
                <!--密码-->
                <property name="password" value="${db.password}"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 添加mapper XML所在文件夹-->
    <mappers>
        <package name="org.pearl.mybatis.demo.dao"/>
    </mappers>
</configuration>
```

#### (2) 配置自定义属性案例

**1、**添加多个property标签，配置数据库连接信息

```java
<properties >
    <property name="db.driver" value="com.mysql.cj.jdbc.Driver"/>
    <property name="db.url" value="jdbc:mysql://127.0.0.1:3306/angel_admin?serverTimezone=Asia/Shanghai"/>
    <property name="db.username" value="root"/>
    <property name="db.password" value="123456"/>
</properties>
```

**2、**dataSource 数据源配置使用 `${}` 引入自定义属性

```xml
...
<dataSource type="POOLED">
    <!--驱动名-->
    <property name="driver" value="${db.driver}"/>
    <!--数据库地址-->
    <property name="url" value="${db.url}"/>
    <!--用户名-->
    <property name="username" value="${db.username}"/>
    <!--密码-->
    <property name="password" value="${db.password}"/>
</dataSource>
...
```

#### (3) 多个同名属性加载顺序

如果一个属性在不只一个地方进行了配置，比如在 resource 及 property 标签中都配置了 db.driver，那么，MyBatis 将按照下面的顺序来加载：

**1、** 首先读取在 properties 元素体内指定的属性

**2、** 然后根据 properties 元素中的 resource 属性读取类路径下属性文件，或根据 url 属性指定的路径读取属性文件，并覆盖之前读取过的同名属性

**3、** 最后读取作为代码方法参数传递的属性，并覆盖之前读取过的同名属性

因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的则是 properties 元素中指定的属性。

#### (4) 占位符指定默认值

从MyBatis 3.4.2 开始，可以为占位符指定一个默认值

**案例演示**：

**1、**这个特性默认是关闭的要启用这个特性，需要添加一个特定的属性来开启这个特性；

```java
<properties >
    <!-- 启用默认值特性 -->
    <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> 
</properties>
```

**2、**使用 `:` 设置属性的默认值；

```java
<dataSource type="POOLED">
    <!--驱动名-->
    <property name="driver" value="${db.driver:com.mysql.cj.jdbc.Driver}"/>
    <!--数据库地址-->
    <property name="url" value="${db.url:jdbc:mysql://127.0.0.1:3306/angel_admin?serverTimezone=Asia/Shanghai}"/>
    <!--用户名-->
    <property name="username" value="${db.username:root}"/>
    <!--密码-->
    <property name="password" value="${db.password:123456}"/>
</dataSource>
```

**3、**当配置的属性名也存在 `:` 时（如：`db:username`），此时会有冲突，需要设置自定义的分隔符；

```java
<properties >
    <!--添加自定义默认分隔符-->
    <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="?:"/>
</properties>
```

## settings（设置）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项设置的含义、默认值等。

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| :------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :---------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true /false                                                  | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。 | true /false                                                  | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 lazyLoadTriggerMethods)。 | true/false                                                   | false （在 3.4.1 及之前的版本中默认为 true）          |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true / false                                                 | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true /false                                                  | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true / false                                                 | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。         | NONE: 不做任何反应WARNING: 输出警告日志（‘org.apache.ibatis.session.AutoMappingUnknownColumnBehavior’ 的日志等级必须设置为 WARN）FAILING: 映射失败 (抛出 SqlSessionException)NONE, WARNING, FAILING | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY /SCROLL_SENSITIVE /SCROLL_INSENSITIVE/DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true/false False                                             |                                                       |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true/false                                                   | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true /false                                                  | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION/STATEMENT                                            | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。 | OTHER                                                        |                                                       |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 TypeHandler 。                          | （新增于 3.4.5） 一个类型别名或全限定类名。                  | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true/false                                                   | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 null。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true/ false                                                  | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J / LOG4J/ LOG4J2 /JDK_LOGGING/COMMONS_LOGGING /STDOUT_LOGGING/NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB/JAVASSIST                                              | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 -parameters 选项。（新增于 3.4.1） | true/ false                                                  | true                                                  |
| configurationFactory             | 指定一个提供 Configuration 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为static Configuration getConfiguration() 的方法。 | （新增于 3.2.3） 一个类型别名或完全限定类名。                | 未设置                                                |
| shrinkWhitespacesInSql           | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5) | true/false                                                   | false                                                 |
| defaultSqlProviderType           | 指定保存提供程序方法的sql提供程序类（自3.5.6起）。当省略这些属性时，此类将应用于sql提供程序批注（例如@SelectProvider）上的type（或value）属性. | 一个类型别名或完全限定类名                                   | 未设置                                                |

一个配置完整的 settings 元素的示例如下：

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

### **演示案例**

在数据库字段命名规范中，通常使用下划线 “_” 来连接两个单词，比如：user_type。但是在 Java 开发中，实体字段通常采用驼峰命名法，因此会在 mapper 文件的 SQL 语句中使用 “AS” 设置别名来匹配实体

Mybatis 在 settings 配置项中有一个 `mapUnderscoreToCamelCase` 参数，设置为 `True` 即可开启自动驼峰命名规则映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射，默认为 `False`。

**1、**settings 标签下添加配置

```java
<!--驼峰命名 自动将数据库字段下划线转为驼峰-->
<setting name="mapUnderscoreToCamelCase" value="true"/>
```

**2、**SQL 去掉 as

```xml
<select id="selectOneById" resultType="user">
    select * from base_user where user_id ={id}
</select>
```

## typeAliases（类型别名）

在之前 mapper XML 中设置 SQL 语句的返回类型 resultType 时，写的是全限定类名，比较长，所以 Mybatis 提供了类型别名设置，为 Java 类型设置一个短的名字，可以方便我们引用某个类。类很多的情况下，也可以批量设置别名这个包下的每一个类

```xml
<mapper namespace="org.pearl.mybatis.demo.dao.UserMapper">
    <select id="selectOneById" resultType="org.pearl.mybatis.demo.pojo.entity.User">
    select * from base_user where user_id ={id}
  </select>
</mapper>
```

设置了别名后，resultType就可以直接写别名了，简洁性提升了不少。

```xml
<select id="selectOneById" resultType="user">
    select * from base_user where user_id ={id}
</select>
```

### (1) 使用 typeAliases 标签

typeAlias 可以为某个类设置一个别名

package 可以指定一个包名，MyBatis 会在包名下面搜索需要的 Java Bean 并全部设置别名

```java
<typeAliases>
    <!--单个类起别名-->
    <typeAlias type="org.pearl.mybatis.demo.pojo.entity.User" alias="user"/>
    <!--为某个包起别名 默认别名为类名小写-->
    <package name="org.pearl.mybatis.demo.pojo.entity"/>
</typeAliases>
```

### (2) 使用 @Alias 注解

使用`@Alias`注解标注在类上，为这个类起别名。

```java
@Data
@Alias("user")
public class User implements Serializable {}
```

<font color=red>MyBatis 中别名大小写不敏感</font>

MyBatis 已经为许多常见的 Java 类型内建了相应的类型别名。我们在起别名的时候千万不要占用已有的别名

| 别名                      | 映射的类型   |
| ------------------------- | ------------ |
| _byte                     | byte         |
| _char (since 3.5.10)      | char         |
| _character (since 3.5.10) | char         |
| _long                     | long         |
| _short                    | short        |
| _int                      | int          |
| _integer                  | int          |
| _double                   | double       |
| _float                    | float        |
| _boolean                  | boolean      |
| string                    | String       |
| byte                      | Byte         |
| char (since 3.5.10)       | Character    |
| character (since 3.5.10)  | Character    |
| long                      | Long         |
| short                     | Short        |
| int                       | Integer      |
| integer                   | Integer      |
| double                    | Double       |
| float                     | Float        |
| boolean                   | Boolean      |
| date                      | Date         |
| decimal                   | BigDecimal   |
| bigdecimal                | BigDecimal   |
| biginteger                | BigInteger   |
| object                    | Object       |
| date[]                    | Date[]       |
| decimal[]                 | BigDecimal[] |
| bigdecimal[]              | BigDecimal[] |
| biginteger[]              | BigInteger[] |
| object[]                  | Object[]     |
| map                       | Map          |
| hashmap                   | HashMap      |
| list                      | List         |
| arraylist                 | ArrayList    |
| collection                | Collection   |
| iterator                  | Iterator     |

## typeHandlers（类型处理器）

MyBatis 在设置预处理语句（PreparedStatement）中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值以合适的方式转换成 Java 类型。比如实体类的某个字段是 String，在数据库中则会是 VARCHAR，他们之间进行交会映射时，都需要转换为自己的类型进行处理。

从3.4.5 开始，MyBatis 默认支持 JSR-310（日期和时间 API） 。MyBatis3.4以前的版本需要我们手动注册这些处理器，以后的版本都是自动注册。

**下表描述了一些默认的类型处理器**：

| 类型处理器                 | Java 类型                     | JDBC 类型                                                    |
| :------------------------- | :---------------------------- | :----------------------------------------------------------- |
| BooleanTypeHandler         | java.lang.Boolean, boolean    | 数据库兼容的 BOOLEAN                                         |
| ByteTypeHandler            | java.lang.Byte, byte          | 数据库兼容的 NUMERIC 或 BYTE                                 |
| ShortTypeHandler           | java.lang.Short, short        | 数据库兼容的 NUMERIC 或 SMALLINT                             |
| IntegerTypeHandler         | java.lang.Integer, int        | 数据库兼容的 NUMERIC 或 INTEGER                              |
| LongTypeHandler            | java.lang.Long, long          | 数据库兼容的 NUMERIC 或 BIGINT                               |
| FloatTypeHandler           | java.lang.Float, float        | 数据库兼容的 NUMERIC 或 FLOAT                                |
| DoubleTypeHandler          | java.lang.Double, double      | 数据库兼容的 NUMERIC 或 DOUBLE                               |
| BigDecimalTypeHandler      | java.math.BigDecimal          | 数据库兼容的 NUMERIC 或 DECIMAL                              |
| StringTypeHandler          | java.lang.String              | CHAR, VARCHAR                                                |
| ClobReaderTypeHandler      | java.io.Reader                | -                                                            |
| ClobTypeHandler            | java.lang.String              | CLOB, LONGVARCHAR                                            |
| NStringTypeHandler         | java.lang.String              | NVARCHAR, NCHAR                                              |
| NClobTypeHandler           | java.lang.String              | NCLOB                                                        |
| BlobInputStreamTypeHandler | java.io.InputStream           | -                                                            |
| ByteArrayTypeHandler       | byte[]                        | 数据库兼容的字节流类型                                       |
| BlobTypeHandler            | byte[]                        | BLOB, LONGVARBINARY                                          |
| DateTypeHandler            | java.util.Date                | TIMESTAMP                                                    |
| DateOnlyTypeHandler        | java.util.Date                | DATE                                                         |
| TimeOnlyTypeHandler        | java.util.Date                | TIME                                                         |
| SqlTimestampTypeHandler    | java.sql.Timestamp            | TIMESTAMP                                                    |
| SqlDateTypeHandler         | java.sql.Date                 | DATE                                                         |
| SqlTimeTypeHandler         | java.sql.Time                 | TIME                                                         |
| ObjectTypeHandler          | Any                           | OTHER 或未指定类型                                           |
| EnumTypeHandler            | Enumeration Type              | VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值） |
| EnumOrdinalTypeHandler     | Enumeration Type              | 任何兼容的 NUMERIC 或 DOUBLE 类型，用来存储枚举的序数值（而不是名称）。 |
| SqlxmlTypeHandler          | java.lang.String              | SQLXML                                                       |
| InstantTypeHandler         | java.time.Instant             | TIMESTAMP                                                    |
| LocalDateTimeTypeHandler   | java.time.LocalDateTime       | TIMESTAMP                                                    |
| LocalDateTypeHandler       | java.time.LocalDate           | DATE                                                         |
| LocalTimeTypeHandler       | java.time.LocalTime           | TIME                                                         |
| OffsetDateTimeTypeHandler  | java.time.OffsetDateTime      | TIMESTAMP                                                    |
| OffsetTimeTypeHandler      | java.time.OffsetTime          | TIME                                                         |
| ZonedDateTimeTypeHandler   | java.time.ZonedDateTime       | TIMESTAMP                                                    |
| YearTypeHandler            | java.time.Year                | INTEGER                                                      |
| MonthTypeHandler           | java.time.Month               | INTEGER                                                      |
| YearMonthTypeHandler       | java.time.YearMonth           | VARCHAR 或 LONGVARCHAR                                       |
| JapaneseDateTypeHandler    | java.time.chrono.JapaneseDate | DATE                                                         |

可以重写已有的类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型，用的最多的应该是枚举类型。

**案例演示**：

**1、** 编写类型处理器（会覆盖已有的处理JavaString类型的属性以及VARCHAR类型的参数和结果的类型处理器）；

```java
/**
 * Created by TD on 2021/6/9
 * 类型处理器: String《=》VARCHAR
 * 实现org.apache.ibatis.type.TypeHandler接口或者继承org.apache.ibatis.type.BaseTypeHandler
 * MappedJdbcTypes: 指定数据库的数据类型
 * BaseTypeHandler泛型： 指定JAVA数据类型
 */
@MappedJdbcTypes(JdbcType.VARCHAR)
public class ExampleTypeHandler extends BaseTypeHandler<String> {
   
     

    /**
     * javaType转换成jdbcTpe
     */
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, String parameter, JdbcType jdbcType) throws SQLException {
   
     
        ps.setString(i, parameter);
    }

    /**
     *  将从结果集根据列名称获取到的数据的jdbcType转换成javaType
     */
    @Override
    public String getNullableResult(ResultSet rs, String columnName) throws SQLException {
   
     
        return rs.getString(columnName);
    }

    /**
     * 将从结果集根据列索引获取到的数据的jdbcType转换成javaType
     */
    @Override
    public String getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
   
     
        return rs.getString(columnIndex);
    }

    /**
     *  存储过程
     */
    @Override
    public String getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
   
     
        return cs.getString(columnIndex);
    }
}
```

**2、** 在mybatis全局配置文件中注册处理器，也可通过扫描包下的处理器；

```java
    <!--类型处理器-->
    <typeHandlers>
        <typeHandler handler="org.pearl.mybatis.demo.handler.ExampleTypeHandler"/>
    </typeHandlers>
<typeHandlers>
  <package name="org.pearl.mybatis.demo.handler"/>
</typeHandlers>
```

MyBatis 不会通过检测数据库元信息来决定使用哪种类型，所以你必须在参数和结果映射中指明字段是 VARCHAR 类型， 以使其能够绑定到正确的类型处理器上。这是因为 MyBatis 直到语句被执行时才清楚数据类型。

通过类型处理器的泛型，MyBatis 可以得知该类型处理器处理的 Java 类型，不过这种行为可以通过两种方法改变：

- 在类型处理器的配置元素（typeHandler 元素）上增加一个 javaType 属性（比如：javaType=“String”）；
- 在类型处理器的类上增加一个 @MappedTypes 注解指定与其关联的 Java 类型列表。 如果在 javaType 属性中也同时指定，则注解上的配置将被忽略。

可以通过两种方式来指定关联的 JDBC 类型：

- 在类型处理器的配置元素上增加一个 jdbcType 属性（比如：jdbcType=“VARCHAR”）；
- 在类型处理器的类上增加一个 @MappedJdbcTypes 注解指定与其关联的 JDBC 类型列表。 如果在 jdbcType 属性中也同时指定，则注解上的配置将被忽略。

**处理枚举类型**
若想映射枚举类型 Enum，则需要从 EnumTypeHandler 或者 EnumOrdinalTypeHandler 中选择一个来使用。

比如说我们想存储取近似值时用到的舍入模式。默认情况下，MyBatis 会利用 EnumTypeHandler 来把 Enum 值转换成对应的名字。

```xml
<!-- mybatis-config.xml -->
<typeHandlers>
  <typeHandler handler="org.apache.ibatis.type.EnumOrdinalTypeHandler" javaType="java.math.RoundingMode"/>
</typeHandlers>
```
