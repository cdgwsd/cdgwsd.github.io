---
categories:
  - Spring
---
# Spring

## 前置

### 概述

> Spring 是一个轻量级的开源 Java 框架，提供了全面的基础设施和功能，用于简化企业级应用程序的开发和管理

Spring 的轻量级主要体现在以下方面
- 非侵入性
  Spring 框架不会强制应用程序继承特定的类或实现特定的接口。开发者可以选择性地利用 Spring 的功能，而不会对应用程序的整体架构造成过多的限制

- 模块化设计
  Spring 框架被设计为一系列相互协作的模块，每个模块都有特定的责任。这种模块化设计使得开发者可以根据实际需求选择使用框架的哪些部分，而不必引入整个框架
  - controller：SpringMVC
  - service：AOP
  - dao：jdbcTemplate

- 灵活配置
  开发者可以根据需要采用多种配置方式，如 XML、注解和 Java 配置
### 设计模式

> 设计模式是在软件设计中反复出现的问题的可重用解决方案，它提供了一套经过验证的最佳实践，能够帮助开发者更有效地解决特定类型的问题

Spring 容器涉及到了多种设计模式，其中主要的设计模式有：
- 单例模式
  Spring 中的 IoC 容器默认使用单例模式管理 Bean

- [[工厂模式]]
  Spring 使用工厂模式创建和管理 Bean。IoC 容器负责实例化、配置和组装 Bean，开发者只需配置 Bean 的元数据

- 代理模式

- 观察者模式

- 模板模式

- 策略模式

- 装饰者模式

- 建造者模式
##  IoC

> Spring 的 IoC（Inversion of Control，控制反转）是一种设计原则，它将应用程序中对象的创建和管理交由 Spring 容器负责，而不是由开发者手动管理，通过这种方式实现了对象的解耦和更灵活的组件化开发

在 Spring 中 ApplicationContext 是 IoC 容器的一种实现，用于管理和组织应用程序中的 Bean 对象
### ApplicationContext

`ApplicationContext` 是 Spring 框架的核心接口之一，它代表着一个 IoC 容器，负责管理和组织应用程序中的 Bean 对象，提供了丰富的功能，包括依赖注入、Bean 的生命周期管理以及对各种应用程序服务的访问
ApplicationContext 被设计为接口类型，有以下好处：
- 灵活性：使用接口类型设计允许存在不同的 `ApplicationContext` 实现，以适用不同的使用场景和需求
  - `ClassPathXmlApplicationContext`：基于 XML 配置的非 Web 应用程序
  - `XmlWebApplicationContext`：Web 应用程序

- 可扩展

- 替代性

- 标准化：约定一种标准的创建和配置 Spring IoC 容器的方法
### API

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
1. 只配置class属性 <bean class="com.baizhiedu.basic.Person"/>
	a) 上述这种配置 有没有 id 值？有：com.baizhiedu.basic.Person#0
	b) 应⽤场景：如果这个 bean 只需要使⽤⼀次，那么就可以省略 id 值；如果这个 bean 会使⽤多次，或者被其他 bean 引⽤则需要设置 id 值
2. name属性
	作⽤：⽤于在 Spring 的配置⽂件中，为 bean 对象定义别名（⼩名）
	相同：
 		1. ctx.getBean("id|name") --> object
 		2. <bean id="" class=""/> 等效 <bean name="" class=""/>
	区别：
 		1. 别名可以定义多个,但是 id 属性只能有⼀个值
		2. XML 的 id 属性的值，命名要求：必须以字母开头，跟字⺟、数字、下划线、连字符。不能以特殊字符开头 /person；name属性的值，命名没有要求。XML发展到了今天：ID 属性已不存在限制
		3. containsBeanDefinition 方法只能判断 id 属性，不能判断 name 属性；containsBean 方法即可以判断 id 属性，也可以判断 name 属性
```

### 注入

Spring IoC 注入是指 Spring 容器负责将依赖关系和配置信息自动注入到应用程序的组件中（即为组件的成员变量赋值），以此实现对象之间的解耦和灵活性

