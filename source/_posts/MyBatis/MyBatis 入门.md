---
title: MyBatis 入门
date: 2024-03-15 15:10:00
tags:
  - MyBatis
---
通过 MyBatis 完成 monster 表的增删改查操作

## 创建库表

```sql
DROP DATABASE IF EXISTS `mybatis`;
CREATE DATABASE `mybatis`;
DROP TABLE IF EXISTS `monster`;
CREATE TABLE `monster`  (
  `id` int NOT NULL AUTO_INCREMENT,
  `age` int NULL DEFAULT NULL,
  `birthday` date NULL DEFAULT NULL,
  `email` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci NULL DEFAULT NULL,
  `gender` tinyint NULL DEFAULT NULL,
  `name` varchar(255) CHARACTER SET utf8mb3 COLLATE utf8mb3_general_ci NULL DEFAULT NULL,
  `salary` double NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 7 CHARACTER SET = utf8mb3 COLLATE = utf8mb3_general_ci ROW_FORMAT = Dynamic;
```

## 创建 maven 项目

### pom 文件

```xml
<dependencies>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>8.0.28</version>
    </dependency>

    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.5.7</version>
    </dependency>

    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
    </dependency>

    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.20</version>
    </dependency>
</dependencies>

<!-- 在构建项目时需要将指定目录下的指定文件类型的资源文件复制到输出目录中（通常是 target/classes 目录），以确保资源文件在项目运行时可以被访问到 -->
<build>
    <resources>
        <resource>
            <directory>src/main/java</directory>
            <includes>
                <include>**/*.xml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <includes>
                <include>**/*.xml</include>
                <include>**/*.properties</include>
            </includes>
        </resource>
    </resources>
</build>
```

## 核心配置

### mybatis-config.xml

习惯上命名为 `mybatis-config.xml`，这个文件名仅仅只是建议，并非强制要求。将来整合 Spring 之后，这个配置文件可以省略
核心配置文件主要用于<font color=red>配置连接数据库的环境以及 MyBatis 的全局配置信息</font>
核心配置文件存放的位置是 `src/main/resources` 目录下

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql://127.0.0.1:3306/mybatis?useSSL=true&amp;useUnicode=true&amp;characterEncoding=UTF-8"/>
                <property name="username" value="root"/>
                <property name="password" value="123"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 引入mapper 映射文件 -->
    <mappers>
        <mapper resource="mapper/MonsterMapper.xml"/>
    </mappers>
</configuration>
```

### mapper 接口

MyBatis 中的 mapper 接口相当于以前的 dao。但是区别在于 mapper 仅仅是接口，我们不需要提供实现类

```java
public interface MonsterMapper {
    // 插入
    void insert(Monster monster);

    // 删除
    void delete(int id);

    // 更新
    void update(Monster monster);

    // 查找
    Monster findById(int id);
}
```

### mapper 映射文件

- 映射文件的命名规则
  - 表所对应的<font color=red>实体类的类名+Mapper.xml</font>
    - 例如：表 t_user，映射的实体类为 User，所对应的映射文件为 UserMapper.xml 
  - 因此<font color=red>一个映射文件对应一个实体类，对应一张表的操作</font>
  - MyBatis 映射文件用于编写 SQL，查询以及操作表中的数据
  - MyBatis 映射文件存放的位置是 `src/main/resources/mappers` 目录下
- MyBatis 中可以面向接口操作数据，要保证两个一致
  - mapper 接口的全类名和映射文件的命名空间（namespace）保持一致
  - mapper 接口中方法的方法名和映射文件中编写 SQL 的标签的 id 属性保持一致

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<!-- namespace：对应 mapper 接口的全限定类名-->
<mapper namespace="me.zyp.mapper.MonsterMapper">
    <!-- id：对应 mapper 接口的方法名-->
    <insert id="insert" parameterType="me.zyp.entity.Monster" useGeneratedKeys="true" keyProperty="id">
        insert into monster(age, birthday, email, gender, name, salary)
        values (#{age}, #{birthday}, #{email}, #{gender}, #{name}, #{salary})
    </insert>

    <delete id="delete" parameterType="int">
        delete from monster where id = #{id}
    </delete>

    <update id="update" parameterType="me.zyp.entity.Monster">
        update monster
        set age = #{age}, birthday = #{birthday}, email = #{email}, gender = #{gender}, name = #{name}, salary =
        #{salary}
        where id = #{id}
    </update>

    <select id="findById" resultType="me.zyp.entity.Monster">
        select * from monster where id = #{id}
    </select>
</mapper>
```

定义完 mapper 配置文件之后，需要在 `mybatis-config.xml` 文件中指定 mapper 配置文件位置

```xml
<mappers>
    <mapper resource="mapper/MonsterMapper.xml"/>
</mappers>
```

### 测试类

```java
public class MonsterMapperTest {

    //这个是 Sql 会话,通过它可以发出 sql 语句
    private SqlSession sqlSession;
    private MonsterMapper monsterMapper;

    @Before
    public void init() throws Exception {
        //通过 SqlSessionFactory 对象获取一个 SqlSession 会话
        sqlSession = MyBatisUtils.getSqlSession();
        //获取 MonsterMapper 接口对象, 该对象实现了 MonsterMapper
        monsterMapper = sqlSession.getMapper(MonsterMapper.class);
        System.out.println(monsterMapper.getClass());
    }

    @Test
    public void addMonster() {
        for (int i = 0; i < 1; i++) {
            Monster monster = new Monster();
            monster.setAge(100 + i);
            monster.setBirthday(new Date());
            monster.setEmail("tn@sohu.com");
            monster.setGender(1);
            monster.setName("松鼠精" + i);
            monster.setSalary(9234.89 + i * 10);
            monsterMapper.insert(monster);
            System.out.println("刚刚添加的对象的 id=" + monster.getId());
        }
        //增删改，需要提交事务
        if (sqlSession != null) {
            sqlSession.commit();
            sqlSession.close();
        }
        System.out.println("保存成功!");
    }

    @Test
    public void deleteMonster() {
        monsterMapper.delete(1);
        if (sqlSession != null) {
            sqlSession.commit();
            sqlSession.close();
        }
    }

    @Test
    public void updateMonster() {
        Monster monster = new Monster();
        monster.setId(2);
        monster.setAge(200);
        monsterMapper.update(monster);
        if (sqlSession != null) {
            sqlSession.commit();
            sqlSession.close();
        }
    }

    @Test
    public void findById() {
        Monster monster = monsterMapper.findById(3);
        System.out.println(monster);
    }
}
```