---
title: @Bean
tags:
  - Spring
  - 注解
  - @Bean
date: 2024-03-20 17:10:00
---

# @Bean

@Bean 注解标注的方法返回的对象将做为组件被注册进容器中

- 方法的返回值类型就是组件类型
- 默认方法名就是组件名

@Bean 注解可以与 @Scope 注解一起使用来定义 Bean 的作用域

```java
@Bean
@Scope("prototype")
public Eoo eoo() {
    System.out.println("Eoo 实例化");
    Foo foo = foo();
    return new Eoo();
}
```

@Bean 通常与 @Configuraion 一起使用，但也可以放在被 @Component 标注的类中。当 @ComponentScan 注解制定了被扫描的包及其子包时，@Bean 也可以作用在这些包的普通类里