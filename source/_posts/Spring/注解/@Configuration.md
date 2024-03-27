---
title: @Configuration
tags:
  - Spring 
  - 注解
  - @Configuration
date: 2024-03-20 16:57:00
---

# @Configuration

`@Configuration` 注解是 Spring 框架中用于标识一个类为配置类的注解。在 Spring 中，<font color=red>配置类是用来定义 Bean 的地方，通常用于替代传统的 XML 配置文件</font>

被 `@Configuration` 注解的类内部包含有一个或多个被 `@Bean` 注解的方法，用于构建 bean 定义，初始化Spring 容器

被 `@Configuration` 标注的类需要满足以下条件

- 不可以是 final 类型
- 不可以是匿名类
- 嵌套的 @Configuration 必须是静态类

`@Configuration` 注解源码如下：

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Configuration {
    @AliasFor(annotation = Component.class)
	String value() default "";
    
    boolean proxyBeanMethods() default true;
}
```

<font color=red>`@Configuration` 被 `@Componnet` 进行标注，因此 `@Configuration` 注解具有 `@Componnet` 相同的功能，即将被标注的类做为组件注册进容器中</font>

## proxyBeanMethods 属性

该属性用于指定类中被 @Bean 标注的方法是否应该被代理，即当直接调用 @Bean 方法时，是否返回共享的 Bean 实例

- true：代理 @Bean 方法，默认值
- false：不代理

接下来通过实例进行说明：

- 配置类

  ```java
  @Configuration
  public class ConfigurationConfig {
      @Bean
      public Eoo eoo() {
          System.out.println("Eoo 实例化");
          Foo foo = foo();
          return new Eoo();
      }
  
      @Bean
      public Foo foo() {
          System.out.println("Foo 实例化");
          Foo foo = new Foo();
          return foo;
      }
  }
  ```

- 测试方法

  ```java
  @Test
  public void testConfigurationConfig(){
      AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(ConfigurationConfig.class);
      Eoo eoo1 = ioc.getBean("eoo", Eoo.class);
      Eoo eoo2 = ioc.getBean("eoo", Eoo.class);
      System.out.println(eoo1 == eoo2);
      ConfigurationConfig configurationConfig = ioc.getBean("configurationConfig", ConfigurationConfig.class);
      Eoo eoo3 = configurationConfig.eoo();
      System.out.println(eoo3 == eoo1);
  }
  ```

- proxyBeanMethods = true

  ```
  Eoo 实例化
  Foo 实例化
  true
  true
  ```

- proxyBeanMethods = false

  ```xml
  Eoo 实例化
  Foo 实例化
  Foo 实例化
  true
  Eoo 实例化
  Foo 实例化
  false
  ```

  每次直接调用 @Bean 方法时都会产生一个新的对象

