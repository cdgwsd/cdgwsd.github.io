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

## objectFactory（对象工厂）

**官方描述**：每次 MyBatis 创建结果对象的新实例时，它都会使用一个对象工厂（ObjectFactory）实例来完成实例化工作。 默认的对象工厂需要做的仅仅是实例化目标类，要么通过默认无参构造方法，要么通过存在的参数映射来调用带有参数的构造方法。 如果想覆盖对象工厂的默认行为，可以通过创建自己的对象工厂来实现。

当创建结果集时，MyBatis 会使用一个对象工厂来完成创建这个结果集实例。在默认的情况下，MyBatis 会使用其定义的对象工厂DefaultObjectFactory（org.apache.ibatis.reflection.factory.DefaultObjectFactory）来完成对应的工作。

**自定义对象工厂案例**：

**1、** 继承DefaultObjectFactory来创建自定义对象工厂；

```java
public class ExampleObjectFactory extends DefaultObjectFactory {
   
     

    // 处理默认构造方法
    @Override
    public <T> T create(Class<T> type) {
   
     
        return super.create(type);
    }

    // 处理有参构造方法
    @Override
    public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
   
     
        return super.create(type, constructorArgTypes, constructorArgs);
    }

    // 判断集合类型参数
    @Override
    public <T> boolean isCollection(Class<T> type) {
   
     
        return super.isCollection(type);
    }

    /**
     * mybatis核心配置文件中自配置<objectFactory><property></property></objectFactory>
     * 中的property标签的内容，会在加载配置文件后，设置到Properties对象中
     */
    @Override
    public void setProperties(Properties properties) {
   
     
        super.setProperties(properties);
        System.out.println(properties.getProperty("userName"));
    }
}
```

**1、** 全局配置添加对象工厂,其子标签property会在加载全局配置文件时通过setProperties方法被初始化到MyObjectFactory中，作为该类的全局参数使用；

```java
    <!--对象工厂-->
    <objectFactory type="org.pearl.mybatis.demo.handler.ExampleObjectFactory">
        <property name="userName" value="zhangsansan"/>
    </objectFactory>
```

**1、** 测试发现，获取到了ObjectFactory设置的属性；
![ ](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403181751439.png)

## plugins（插件）

Mybatis插件又称拦截器，Mybatis采用责任链模式，通过动态代理组织多个插件（拦截器），通过这些插件可以改变Mybatis的默认行为。MyBatis 允许你在已映射语句执行过程中的某一点进行拦截调用。默认情况下，MyBatis允许使用插件来拦截的方法调用包括：

- Executor (update, query, flushStatements, commit, rollback,getTransaction, close, isClosed) 拦截执行器的方法；
- ParameterHandler (getParameterObject, setParameters) 拦截参数的处理；
- ResultSetHandler (handleResultSets, handleOutputParameters) 拦截结果集的处理；
- StatementHandler (prepare, parameterize, batch, update, query) 拦截Sql语法构建的处理；

通过MyBatis 提供的强大机制，使用插件是非常简单的，只需实现 Interceptor 接口，并指定想要拦截的方法签名即可。

**对查询操作添加拦截器案例**：

**1、** 编写拦截器；

```java
@Intercepts({
   
     @Signature(
        type = Executor.class,
        method = "query",
        args = {
   
     MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class})})
public class ExamplePlugin implements Interceptor {
   
     
    private Properties properties = new Properties();

    @Override
    public Object intercept(Invocation invocation) throws Throwable {
   
     
        Object target = invocation.getTarget(); //被代理对象
        Method method = invocation.getMethod(); //代理方法
        Object[] args = invocation.getArgs(); //方法参数
        // do something ...... 方法拦截前执行代码块
        Object result = invocation.proceed();
        // do something .......方法拦截后执行代码块
        return result;
    }

    @Override
    public void setProperties(Properties properties) {
   
     
        this.properties = properties;
    }

    @Override
    public Object plugin(Object target) {
   
     
        return Plugin.wrap(target, this);
    }
}
```

**1、** 注册拦截器；

```java
    <!--插件-->
    <plugins>
        <plugin interceptor="org.pearl.mybatis.demo.plugins.ExamplePlugin"></plugin>
    </plugins>
```

上面的插件将会拦截在 Executor 实例中所有的 “query” 方法调用， 这里的 Executor 是负责执行底层映射语句的内部对象。

## environments（环境配置）

在MyBatis 中，运行环境主要的作用是配置数据库信息，它可以配置多个数据库，一般而言只需要配置其中的一个就可以了。

它下面又分为两个可配置的元素：事务管理器（transactionManager）、数据源（dataSource）。

在实际的工作中，大部分情况下会采用 Spring 对数据源和数据库的事务进行管理。

MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。例如，开发、测试和生产环境需要有不同的配置；或者想在具有相同 Schema 的多个生产数据库中使用相同的 SQL 映射。还有许多类似的使用场景。

**不过要记住：尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**

所以，如果你想连接两个数据库，就需要创建两个 SqlSessionFactory 实例，每个数据库对应一个。而如果是三个数据库，就需要三个实例，依此类推，记起来很简单：

- ***每个数据库对应一个 SqlSessionFactory 实例***

**多环境切换案例演示**：

**1、** 添加配置文件，配置多个环境；

```java
    <!--多环境配置-->
    <!--default默认使用的环境ID，此处表示默认使用开发环境配置-->
    <environments default="development">
        <!--开发环境配置-->
        <!--id：指定当前环境的唯一标识-->
        <environment id="development">
            <!--事务管理器的配置（比如：type="JDBC"）-->
            <transactionManager type="JDBC"/>
            <!--数据源的配置（比如：type="POOLED"）-->
            <dataSource type="POOLED">
                <!--驱动名-->
                <property name="driver" value="${db.driver:com.mysql.cj.jdbc.Driver}"/>
                <!--数据库地址-->
                <property name="url"
                          value="${db.url:jdbc:mysql://127.0.0.1:3306/angel_admin?serverTimezone=Asia/Shanghai}"/>
                <!--用户名-->
                <property name="username" value="${db.username:root}"/>
                <!--密码-->
                <property name="password" value="${db.password:123456}"/>
            </dataSource>
        </environment>
        <!--测试环境配置-->
        <environment id="test">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${db.driver:com.mysql.cj.jdbc.Driver}"/>
                <property name="url"
                          value="${db.url:jdbc:mysql://192.168.17.1:3306/angel_admin?serverTimezone=Asia/Shanghai}"/>
                <property name="username" value="${db.username:root}"/>
                <property name="password" value="${db.password:123456}"/>
            </dataSource>
        </environment>
```

**1、** 根据不同的环境创建SqlSessionFactory，执行SQL；

```java
public class Test002 {
   
     
    public static void main(String[] args) throws IOException {
   
     
        // 开发环境
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream,"development");
        SqlSession sqlSession = sqlSessionFactory.openSession();
        UserMapper mapper = sqlSession.getMapper(UserMapper.class);
        User user = mapper.selectOneById(1L);
        System.out.println(user);
        // 测试环境
        String resourceTest = "mybatis-config.xml";
        InputStream inputStreamTest = Resources.getResourceAsStream(resourceTest);
        SqlSessionFactory sqlSessionFactoryTest = new SqlSessionFactoryBuilder().build(inputStreamTest,"test");
        SqlSession sqlSessionTest = sqlSessionFactoryTest.openSession();
        UserMapper mapperTest = sqlSessionTest.getMapper(UserMapper.class);
        User userTest = mapperTest.selectOneById(1L);
        System.out.println(userTest);
    }
}
```

### transactionManager（事务管理器）

在MyBatis 中有两种类型的事务管理器（也就是 type="[JDBC|MANAGED]"）：

- JDBC – 这个配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
- MANAGED – 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。 默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 closeConnection 属性设置为 false 来阻止默认的关闭行为。

如果你正在使用 Spring + MyBatis，则没有必要配置事务管理器，因为 Spring 模块会使用自带的管理器来覆盖前面的配置。

这两种事务管理器类型都不需要设置任何属性。它们其实是类型别名，换句话说，你可以用 TransactionFactory 接口实现类的全限定名或类型别名代替它们。

```java
public interface TransactionFactory {
   
     
  default void setProperties(Properties props) {
   
      // 从 3.5.2 开始，该方法为默认方法
    // 空实现
  }
  Transaction newTransaction(Connection conn);
  Transaction newTransaction(DataSource dataSource, TransactionIsolationLevel level, boolean autoCommit);
}
```

在事务管理器实例化后，所有在 XML 中配置的属性将会被传递给 setProperties() 方法。你的实现还需要创建一个 Transaction 接口的实现类，这个接口也很简单，使用这两个接口，你可以完全自定义 MyBatis 对事务的处理。

```java
public interface Transaction {
   
     
  Connection getConnection() throws SQLException;
  void commit() throws SQLException;
  void rollback() throws SQLException;
  void close() throws SQLException;
  Integer getTimeout() throws SQLException;
}
```

### dataSource（数据源）

dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。

- 大多数 MyBatis 应用程序会按示例中的例子来配置数据源。虽然数据源配置是可选的，但如果要启用延迟加载特性，就必须配置数据源。

有三种内建的数据源类型（也就是 type="[UNPOOLED|POOLED|JNDI]"）：

***UNPOOLED***– 这个数据源的实现会每次请求时打开和关闭连接。虽然有点慢，但对那些数据库连接可用性要求不高的简单应用程序来说，是一个很好的选择。 性能表现则依赖于使用的数据库，对某些数据库来说，使用连接池并不重要，这个配置就很适合这种情形。UNPOOLED 类型的数据源仅仅需要配置以下 5 种属性：

- driver – 这是 JDBC 驱动的 Java 类全限定名（并不是 JDBC 驱动中可能包含的数据源类）。
- url – 这是数据库的 JDBC URL 地址。
- username – 登录数据库的用户名。
- password – 登录数据库的密码。
- defaultTransactionIsolationLevel – 默认的连接事务隔离级别。
- defaultNetworkTimeout – 等待数据库操作完成的默认网络超时时间（单位：毫秒）。查看 java.sql.Connection#setNetworkTimeout() 的 API 文档以获取更多信息。

作为可选项，你也可以传递属性给数据库驱动。只需在属性名加上“driver.”前缀即可，例如：

- driver.encoding=UTF8

这将通过 DriverManager.getConnection(url, driverProperties) 方法传递值为 UTF8 的 encoding 属性给数据库驱动。

***POOLED***– 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。

除了上述提到 UNPOOLED 下的属性外，还有更多属性用来配置 POOLED 的数据源：

- poolMaximumActiveConnections – 在任意时间可存在的活动（正在使用）连接数量，默认值：10
- poolMaximumIdleConnections – 任意时间可能存在的空闲连接数。
- poolMaximumCheckoutTime – 在被强制返回之前，池中连接被检出（checked out）时间，默认值：20000 毫秒（即 20 秒）
  poolTimeToWait – 这是一个底层设置，如果获取连接花费了相当长的时间，连接池会打印状态日志并重新尝试获取一个连接（避免在误配置的情况下一直失败且不打印日志），默认值：20000 毫秒（即 20 秒）。
- poolMaximumLocalBadConnectionTolerance – 这是一个关于坏连接容忍度的底层设置， 作用于每一个尝试从缓存池获取连接的线程。 如果这个线程获取到的是一个坏的连接，那么这个数据源允许这个线程尝试重新获取一个新的连接，但是这个重新尝试的次数不应该超过
- poolMaximumIdleConnections 与 poolMaximumLocalBadConnectionTolerance 之和。 默认值：3（新增于 3.4.5）
  poolPingQuery – 发送到数据库的侦测查询，用来检验连接是否正常工作并准备接受请求。默认是“NO PING QUERY SET”，这会导致多数数据库驱动出错时返回恰当的错误消息。
- poolPingEnabled – 是否启用侦测查询。若开启，需要设置 poolPingQuery 属性为一个可执行的 SQL 语句（最好是一个速度非常快的 SQL 语句），默认值：false。
- poolPingConnectionsNotUsedFor – 配置 poolPingQuery 的频率。可以被设置为和数据库连接超时时间一样，来避免不必要的侦测，默认值：0（即所有连接每一时刻都被侦测 — 当然仅当 - poolPingEnabled 为 true 时适用）。

***JNDI*** – 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。这种数据源配置只需要两个属性：

- initial_context – 这个属性用来在 InitialContext 中寻找上下文（即，initialContext.lookup(initial_context)）。这是个可选属性，如果忽略，那么将会直接从 InitialContext 中寻找 data_source 属性。
- data_source – 这是引用数据源实例位置的上下文路径。提供了 initial_context 配置时会在其返回的上下文中进行查找，没有提供时则直接在 InitialContext 中查找。
- 

和其他数据源配置类似，可以通过添加前缀“env.”直接把属性传递给 InitialContext。比如：

- env.encoding=UTF8

这就会在 InitialContext 实例化时往它的构造方法传递值为 UTF8 的 encoding 属性。

可以通过实现接口 org.apache.ibatis.datasource.DataSourceFactory 来使用第三方数据源实现：

```java
public interface DataSourceFactory {
   
     
  void setProperties(Properties props);
  DataSource getDataSource();
}
```

org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory 可被用作父类来构建新的数据源适配器，比如下面这段插入 C3P0 数据源所必需的代码：

```java
import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;
import com.mchange.v2.c3p0.ComboPooledDataSource;

public class C3P0DataSourceFactory extends UnpooledDataSourceFactory {
   
     

  public C3P0DataSourceFactory() {
   
     
    this.dataSource = new ComboPooledDataSource();
  }
}
```

## databaseIdProvider（数据库厂商标识）

数据库种类很多，虽然大多都是基于SQL标准，但是每个数据库都有自己的方言，或者函数。

Mybatis也做了多数据库支持，只需要告诉框架用的是什么数据库，MyBatis 可以根据不同的数据库厂商执行不同的语句。

**适配Mysql及Oracle数据库案例**：

**1、** 添加配置；

```java
    <!--数据库厂商标识-->
    <!--DB_VENDOR: 使用MyBatis提供的VendorDatabaseIdProvider解析数据库厂商标识。也可以实现DatabaseIdProvider接口来自定义-->
    <databaseIdProvider type="DB_VENDOR">
        <!--添加两个数据库厂商别名-->
        <!--name：数据库厂商标识-->
        <!--value：为标识起一个别名，方便SQL语句使用databaseId属性引用-->
        <property name="Oracle" value="oracle"/>
        <property name="MySQL" value="mysql"/>
    </databaseIdProvider>
```

**1、** xml中指定databaseId为响应的数据库；

```xml
<mapper namespace="org.pearl.mybatis.demo.dao.UserMapper">
    <select id="selectOneById" resultType="org.pearl.mybatis.demo.pojo.entity.User" databaseId="mysql">
    select * from base_user where user_id ={
   
     id}
  </select>
</mapper>
```

**1、** 测试，查看当前的数据库厂商；

```java
public class Test001 {
   
     
    public static void main(String[] args) throws IOException {
   
     
        // 根据xml配置文件（全局配置文件）创建一个SqlSessionFactory对象
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        String databaseId = sqlSessionFactory.getConfiguration().getDatabaseId();
        System.out.println(databaseId+"数据库");
        }
}
```

![ ](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403181750869.png)

**匹配规则**:

- 如果没有配置databaseIdProvider标签，那么databaseId=null
- 如果配置了databaseIdProvider标签，使用标签配置的name去匹配数据库信息，匹配上设置databaseId=配置指定的值，否则依旧为null
- 如果databaseId不为null，他只会找到配置databaseId的sql语句
- MyBatis 会加载不带 databaseId 属性和带有匹配当前数据库databaseId 属性的所有语句。如果同时找到带有 databaseId 和不带databaseId 的相同语句，则后者会被舍弃。

### mappers（映射器）

既然MyBatis 的行为已经由上述元素配置完了，我们现在就要来定义 SQL 映射语句了。 但首先，我们需要告诉 MyBatis 到哪里去找到这些语句。 在自动查找资源方面，Java 并没有提供一个很好的解决方案，所以最好的办法是直接告诉 MyBatis 到哪里去找映射文件。 你可以使用相对于类路径的资源引用，或完全限定资源定位符（包括 file:/// 形式的 URL），或类名和包名等。例如：

```xml
<!-- 使用相对于类路径的资源引用 -->
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>
<!-- 使用完全限定资源定位符（URL） -->
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>
<!-- 使用映射器接口实现类的完全限定类名 -->
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>
<!-- 将包内的映射器接口实现全部注册为映射器 -->
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```
