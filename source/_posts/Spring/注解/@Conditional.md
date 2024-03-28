---
title: Conditional
categories:
  - Spring
  - 注解
date: 2024-03-27 16:41:04
tags:
  - Conditional
  - Spring
  - 注解
---
# @Conditional

`@Conditional` 注解是 Spring 框架中的条件化配置注解，它允许根据特定的条件来决定是否创建一个或多个 bean。通过使用 @Conditional 注解，可以在配置类中根据运行时条件动态地包含或排除某些 bean 的定义

- 基于条件类的条件化配置

```java
  @Configuration
  @Conditional(MyCondition.class)
  public class MyConfig {
      // 根据条件 MyCondition 来配置 bean...
  }
```

  上述代码中，MyCondition 是一个实现了 Condition 接口的条件类，将根据条件类的 matches 方法返回 true 或 false 来决定是否应用该配置

- 基于条件注解的条件化配置：根据属性值决定是否应用配置

```java
  @Configuration
  @ConditionalOnProperty(name = "myapp.feature.enabled", havingValue = "true")
  public class FeatureConfig {
      // 根据属性条件来配置 bean...
  }
```

![](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202403271644404.png)