# IoC

Spring 的 IoC（Inversion of Control，控制反转）是一种设计原则，它将应用程序中对象的创建和管理交由 Spring 容器负责，而不是由开发者手动管理，通过这种方式实现了对象的解耦和更灵活的组件化开发

在 Spring 中 ApplicationContext 是 IoC 容器的一种实现，用于管理和组织应用程序中的 Bean 对象

## ApplicationContext

`ApplicationContext` 是 Spring 框架的核心接口之一，它代表着一个 IoC 容器，负责管理和组织应用程序中的 Bean 对象，提供了丰富的功能，包括依赖注入、Bean 的生命周期管理以及对各种应用程序服务的访问

ApplicationContext 被设计为接口类型，有以下好处：

- 灵活性：使用接口类型设计允许存在不同的 `ApplicationContext` 实现，以适用不同的使用场景和需求
  - `ClassPathXmlApplicationContext`：基于 XML 配置的非 Web 应用程序
  - `XmlWebApplicationContext`：Web 应用程序
- 可扩展
- 替代性
- 标准化：约定一种标准的创建和配置 Spring IoC 容器的方法

## API

- IoC 常用 API

```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml")

//通过这种⽅式获得对象，就不需要强制类型转换
Person person = ctx.getBean("person", Person.class);
System.out.println("person = " + person);

//当前Spring的配置⽂件中 只能有⼀个 bean class是Person类型
Person person = ctx.getBean(Person.class);
System.out.println("person = " + person);

//获取的是Spring⼯⼚配置⽂件中所有bean标签的id值 person person1
String[] beanDefinitionNames = ctx.getBeanDefinitionNames();
for (String beanDefinitionName : beanDefinitionNames) {
     System.out.println("beanDefinitionName = " + beanDefinitionName);
}

//根据类型获得Spring配置⽂件中对应的id值
String[] beanNamesForType = ctx.getBeanNamesForType(Person.class);
for (String id : beanNamesForType) {
     System.out.println("id = " + id);
}

//⽤于判断是否存在指定id值的bean
if (ctx.containsBeanDefinition("a")) {
     System.out.println("true = " + true);
}else{
     System.out.println("false = " + false);
}

//⽤于判断是否存在指定id值的bean
if (ctx.containsBean("person")) {
     System.out.println("true = " + true);
}else{
     System.out.println("false = " + false);
}
```

- 配置文件细节

```markdown
1. 只配置class属性 <bean class="me.zyp.basic.Person"/>
	a) 上述这种配置 有没有 id 值？有：me.zyp.basic.Person#0
	b) 应⽤场景：如果这个 bean 只需要使⽤⼀次，那么就可以省略 id 值；如果这个 bean 会使⽤多次，或者被其他 bean 引⽤则需要设置 id 值
2. name属性
	作⽤：⽤于在 Spring 的配置⽂件中，为 bean 对象定义别名（⼩名）
	相同：
 		1. ctx.getBean("id|name") --> object
 		2. <bean id="" class=""/> 等效 <bean name="" class=""/>
	区别：
 		1. 别名可以定义多个,但是 id 属性只能有⼀个值
		2. containsBeanDefinition 方法只能判断 id 属性，不能判断 name 属性；containsBean 方法即可以判断 id 属性，也可以判断 name 属性
```

## 注入

Spring IoC 注入是指 Spring 容器负责将依赖关系和配置信息自动注入到应用程序的组件中（即为组件的成员变量赋值），以此实现对象之间的解耦和灵活性

注入方式：

- Setter 方法注入
- 构造注入

### Setter 注入

通过为目标类提供 setter 方法，让 Spring 容器可以通过这些 setter 方法来注入依赖对象

```xml
<!-- 通过 setter 方法注入属性 -->
<bean id="user1" class="me.zyp.spring5.User">
	<!-- 通过 property 标签进行属性注入 -->
	<property name="name" value="张三"></property>
</bean>
```

value 属性可使用标签替代，上面代码等同于：

```xml
<!-- 通过 setter 方法注入属性 -->
<bean id="user1" class="me.zyp.spring5.User">
	<!-- 通过 property 标签进行属性注入 -->
	<property name="name">
    	<value>张三</value>
	</property>
</bean>
```

<font color=red>针对 Bean 属性的类型不同，value 标签还需替换为其他标签</font>，Spring 可注入数据类型如下

- 八种数据类型及字符串

  ```xml
  <!-- 注入 JDK 内置类型使用 value 标签 -->
  <value>张三</value>
  ```

- 数组

  ```xml
  <!-- 注入数组类型使用 arr 标签 -->
  <property name="array">
  	<array>
        <value>1</value>
        <value>2</value>
      </array>
  </property>
  ```

- List 列表

  ```xml
  <!-- 注入 List 类型使用 list 标签 -->
  <property name="list">
      <list>
          <value>张三</value>
          <!-- 对象类型 -->
          <ref bean="user"></ref>
      </list>
  </property>
  ```

- Set 集合

  ```xml
  <!-- 注入 Set 类型使用 set 标签 -->
  <property name="set">
      <set>
          <value>zhangsan</value>
          <!-- 对象类型 -->
          <ref bean="user"></ref>
      </set>
  </property>
  ```

- Map 集合

  ```xml
  <!-- 注入 Map 类型使用 map 标签 -->
  <!-- 每一个元素使用 entry 标签表示，key 表示 map 的键，值根据类型不同使用不同标签 -->
  <property name="map">
      <map>
          <entry key="name" value="张三"></entry>
          <!-- 对象类型 -->
          <!-- key-ref、value-ref:bean的id -->
          <entry key-ref="user" value-red="user1"></entry>
      </map>
  </property>
  ```

- Properties

  ```xml
  <!-- 注入 Properties 类型使用 property 标签 -->
  <!-- 每一个元素使用 props 标签表示，Properties 的每个元素的键值都为字符串类型 -->
  <bean id="user" class="me.zyp.spring5.User">
    <property name="props">
  	  <props>
  		  <prop key="username">root<prop>
  		  <prop key="password">1234<prop>
  	  </props>
    </property>
  </bean>
  ```

- 自定义 Bean

  ```xml
  <!-- 注入自定义 Bean 有两种方式：1.内部 Bean，2.ref 属性引用其他 Bean -->
  <bean id="userService" class="xxxx.UserServiceImpl">
      <property name="userDAO">
          <bean class="xxx.UserDAOImpl"/>
      </property>
  </bean>
  <!-- 内部 Bean 注入方式创建的 Bean 无法被其他 Bean 引用，为其他 Bean 再次注入时需要再次创建 -->
  <bean id="userDAO" class="xxx.UserDAOImpl"/>
  <bean id="userService" class="xxx.UserServiceImpl">
      <property name="userDAO">
          <ref bean="userDAO"/>
      </property>
  </bean>
  ```

使用 `p 命名空间` 可以直接为 Bean 的属性赋值，避免了繁琐的 property 标签的使用，简化 Spring 配置

```xml
<bean id="person" class="xxx.Person" p:name="suns"/>

<bean id="userService" class="xxx.UserServiceImpl" p:userDAO-ref="userDAO"/>
```

### 构造注入

构造注入是通过构造函数接收参数来实现依赖注入，Spring 容器在创建 Bean 时通过提供的构造参数值来实例化对象，建立对象之间的关联关系

```xml
<!-- 通过有参构造注入属性-->
<bean id="user2" class="me.zyp.spring5.User">
  <constructor-arg name="name" value="张三"></constructor-arg>
</bean>
```

可以通过构造方法重载，完成不同属性注入

- 参数个数不同时

  ```markdown
  通过控制 <constructor-arg> 标签的数量进⾏区分
  ```

- 参数个数相同时

  ```markdown
  通过在标签引⼊ type 属性 进⾏类型的区分 <constructor-arg type="">
  ```

## 复杂对象创建

针对简单对象（可以通过 new 关键字创建的对象）来讲，通过简单配置即可以进行对象的实例化与属性注入。但对于复杂对象（无法通过 new 关键字创建的对象，如 Connection、SqlSessionFactory 等）则通常使用 `FactoryBean`、静态工厂与实例工厂进行实例化与属性注入

### FactoryBean

`FactoryBean` 是 Spring 框架提供的一种接口，允许定制化对象的创建过程，通过实现 `FactoryBean` 接口的 `getObject` 方法来返回所需的对象实例

FactoryBean 开发步骤：

1. 实现 FactoryBean 接口

   ![image.png](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202311212057140.png)
   
   ```java
   public class MyFactoryBean implements FactoryBean {
       @Override
       public Object getObject() throws Exception {
           User user = new User();
           return user;
       }
   
       @Override
       public Class<?> getObjectType() {
           return User.class;
       }
   
       @Override
       public boolean isSingleton() {
           return true;
       }
   }
   ```

2. 配置文件配置

  ```xml
    <!-- 在容器中通过 id="factoryBean" 获取到的对象为 MyFactoryBean 类的 getObject 方法返回的 User 类型对象，而非 MyFactoryBean 类型对象 -->
    <bean id="factoryBean" class="me.zyp.spring5.MyFactoryBean"></bean>
  ```

<font color = red>如果想要获取 FactoryBean 类型的对象，只需要在 id 前加 & 符号即可</font>

```java
public void testBean(){
      ApplicationContext applicationContext = new ClassPathXmlApplicationContext("bean2.xml");
      Object factoryBean = applicationContext.getBean("&factoryBean");
}
```

### 实例工厂

实例工厂是**将对象的创建过程封装到另外一个对象实例的方法**里

- 实例工厂类

  ```java
  public class SimpleObjectFactory {
      public SimpleObject createSimpleObject() {
          return new SimpleObject();
      }
  }
  ```

- 配置文件

  ```xml
  <bean id="simpleObjectFactory" class="com.example.SimpleObjectFactory" />
  
  <!-- factory-bean：实例工厂Bean，factory-method：实例工厂用于创建对象的方法 -->
  <bean id="simpleObject" factory-bean="simpleObjectFactory" factory-method="createSimpleObject" />
  ```

### 静态工厂

静态工厂是**将对象创建的过程封装到静态方法**中

- 静态工厂类

  ```java
  public class SimpleObjectFactory {
      public static SimpleObject createSimpleObject() {
          return new SimpleObject();
      }
  }
  ```

- 配置文件

  ```xml
  <bean id="simpleObject" id="simpleObjectFactory" class="com.example.SimpleObjectFactory" factory-method="createSimpleObject" />
  ```

### 总结

- **FactoryBean：** Spring 提供的接口，允许自定义对象的创建和配置，通过实现 `FactoryBean` 接口的 `getObject` 方法来返回所需的 Bean 实例
- **静态工厂：** 使用类的静态方法创建 Bean 实例，通过类名直接调用，适用于简单对象的创建
- **实例工厂：** 创建 Bean 实例的类的实例方法，通常需要先实例化工厂类，再调用其实例方法，适用于一些需要状态管理或创建过程较为复杂的场景

## 作用域

Spring 作用域定义了 Bean 实例的生命周期范围，影响 Bean 在容器中的存在时间和共享性

Bean 的作用域有以下几种取值：

- singleton：单例
- prototype：多例

使用 bean 标签的 **scope** 属性来指定 Bean 的作用域

```xml
<bean id="exampleBean" class="com.example.ExampleBean" scope="singleton"></bean>
```

## 生命周期

Spring Bean 的生命周期包括实例化、初始化、使用和销毁四个阶段，由 Spring 容器管理，允许开发者通过回调方法和配置进行定制

### 实例化

实例化是指在 Spring 容器中创建 Bean 对象的过程

根据 Bean 作用域不同，创建时机也不同

- singleton
  - Spring 工厂创建的同时，对象也被创建
  - 可通过设置 `lazy-init=true` 使对象等到被获取时再创建
- prototype
  - 在获取对象时，对象被创建

### 初始化

初始化是指在 Spring 容器创建 Bean 实例后，通过调⽤对象的初始化⽅法，完成对应的初始化操作

配置 Bean 初始化方法有两种方式

1. 实现 `InitializingBean` 接口

   ```java
   public class ExampleBean implements InitializingBean {
   
       @Override
       public void afterPropertiesSet() throws Exception {
           // 在这里可以执行Bean初始化时的逻辑
           System.out.println("Bean正在进行初始化...");
       }
   }
   ```

   当该 Bean 被 Spring 容器创建后，`afterPropertiesSet` 方法将被调用，从而执行自定义的初始化逻辑

2. 配置 `inti-method` 属性

   ```xml
   <bean id="exampleBean" class="com.example.ExampleBean" init-method="init"></bean>
   ```

   `ExampleBean` 类的 `init` 方法将在该 Bean 被创建后进行调用，执行自定义的初始化逻辑

### 销毁

销毁是指在 Spring 容器关闭或销毁时，对 Bean 进行资源释放和清理的过程

配置 Bean 销毁方法有两种方式

1. 实现 `DisposableBean` 接口

   ```java
   public class ExampleBean implements DisposableBean {
   
       @Override
       public void destroy() throws Exception {
           // 在这里可以执行Bean销毁时的清理逻辑
           System.out.println("Bean正在进行销毁...");
       }
   }
   ```

   当该 Bean 被 Spring 容器销毁时，`destroy` 方法将被调用，从而执行自定义的清理逻辑

2. 配置 `destroy-method` 属性

   ```xml
   <bean id="exampleBean" class="com.example.ExampleBean" destroy-method="destroy"></bean>
   ```

    `ExampleBean` 类的 `destroy` 方法将在该 Bean 被销毁后进行调用，执行自定义的销毁逻辑

## 配置文件参数化

在之前使用 Spring 集成数据源配置时，常常会将数据源配置信息直接硬编码在配置文件中

```xml
<!--连接池-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://localhost:3306/suns?useSSL=false"></property>
    <property name="username" value="root"></property>
    <property name="password" value="123456"></property>
</bean>
```

随着项目增加，各种配置信息将混杂在一起，后续维护、修改会很麻烦。为了解决以上问题，可以将这些经常需要修改的数据维护到单独的配置文件中。

Spring 引入了 `context:property-placeholder` 用于解决这一问题

### 使用

- 定义配置文件
	
	```properties
	password=xxxx
	username=xxxx
	url=xxxx
	driver=xxxx
	```
	
- 在 Spring 配置中引入 `context` 命名空间
	
	```xml
	xmlns:context="http://www.springframework.org/schema/context"
	```
	
- 使用 `context:property-placeholder` 引入外部配置文件
	
	```xml
	<context:property-placeholder location="classpath:jdbc.properties"/>
	```
	
- 使用 `${}` 占位符引用属性值
	
	```xml
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="${driver}"></property>
    <property name="url" value="${url}"></property>
    <property name="username" value="${username}"></property>
    <property name="password" value="${password}"></property>
	</bean>
	```

## 类型转换器

在 Spring 中类型转换器用于将一种数据类型转换为另一种数据类型。在下面的示例中 Spring 将自动将 String 类型转换为 int 类型并注入给 age 属性

```java
private int age;
```

```xml
<property name="age" value="18"></property>
```

Spring 类型转换器基于 `Converter` 接口

```java
public interface Converter<S, T> {
    // 将类型 S 转换为 类型 T
	T convert(S source);
}
```

Spring 为我们提供了一批内置类型转换器用于处理常用的数据类型转换。除此之外我们也可以通过实现 `Converter` 接口定义自定义类型转换器

### 自定义类型转换器

定义自定义类型转换器只需要实现 `Converter` 接口，并 `convert` 方法中实现转换逻辑即可

下面代码将实现 yyyy-MM-dd 格式日期字符串转 Date 类型

```java
public class DateConverter implements Converter<String, Date> {
    @Override
    public Date convert(String source) {
        Date date = null;
        try {
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            date = sdf.parse(source);
        } catch (ParseException e) {
            e.printStackTrace();
        }
        return date;
    }
}
```


## 后置处理器

通过后置处理器可以对 Spring 创建的对象进行再加工

- 创建后置处理器类
	
	```java
	public class MyBeanPostProcessor implements BeanPostProcessor {  
	    @Override  
	    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {  
	        System.out.println("========postProcessBeforeInitialization========");  
	        System.out.println(bean);  
	        System.out.println(beanName);  
	        return bean;  
	    }  
	
	    @Override  
	    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {  
	        System.out.println("========postProcessAfterInitialization========");  
	        System.out.println(bean);  
	        System.out.println(beanName);  
	        return bean;  
	    }  
	}
	```
	
- 注册后置处理器
	
	```xml
	<bean class="me.zyp.entity.MyBeanPostProcessor" id="beanPostProcessor"></bean>
	```

<font color="red">配置的后置处理器将对所有注册的 Bean 生效</font>