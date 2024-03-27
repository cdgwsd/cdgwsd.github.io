---
title: "@ConfigurationProperties"
date: 2024-03-27 16:48:08
tags:
---
# @ConfigurationProperties

用于将配置文件中的属性映射到 Java 对象的属性上

配置文件：application.properties

```properties
myapp.name=test
myapp.version=1.0.0
```

Java 类引用配置文件

```java
@Component
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {
  private String name;
  private int version;
  // Getters and setters...
}
```

当配置文件中的属性值发生变化时，`@ConfigurationProperties` 注解会自动将新值注入到相应的Java对象中，而无需重启应用

## EnableConfigurationProperties

`@EnableConfigurationProperties` 注解用于将被 `@ConfigurationProperties` 注解标注的类作为 bean 注册进 Spring 容器中，且进行配置绑定