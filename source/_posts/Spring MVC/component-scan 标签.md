---
title: component-scan 标签
categories:
  - Spring MVC
date: 2024-03-27 16:03:17
tags:
---
# component-scan 标签

```xml
<context:component-scan base-package="me.zyp.furn">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

## 作用

1. 扫描指定包（由 base-package 属性指定）及其子包下的所有类，将被 @Component、@Respository、@Service、@Controller、@RestController、@ControllerAdvice 和 @Configuration 注解标注的类注册为 Spring 的组件
2. 自动激活作用一中所注册类的 @Required、@Autowired、@PostConstruct、@PreDestroy、@Resource、@PersistenceContext 和 @PersistenceUnit 注解

## 子标签

\<context:component-scan\> 还有两个具有过滤作用的子标签

- \<context:include-filter\>
- \<context:exclude-filter\>

### include-filter

指定 component-scan 在扫描组件时<font color=red>除了默认注解外</font>还将扫描哪些注解

```xml
<context:component-scan base-package="me.zyp.furn">
    <context:include-filter type="annotation" expression="me.zyp.stereotype.Component"/>
</context:component-scan>
```

上述代码会将 `me.zyp.furn` 包及其子包下被 @Component 注解标注的类注册为 Spring 组件

### exclude-filter

指定 component-scan 在扫描组件时排除哪些类型

```xml
<context:component-scan base-package="me.zyp.furn">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

上述代码不会将 `me.zyp.furn` 包及其子包下被 @Controller 注解标注的类注册为 Spring 组件

### 过滤规则

这两个标签具有相同的属性：type、expression，通过这两个属性指定过滤规则

#### type

指定过滤器的类型，取值如下

1. **annotation**：通过注解过滤类，例如，排除所有包含 @Controller 注解的类

   ```xml
   <context:exclude-filter type="annotation" expression="me.zyp.stereotype.Component"/>
   ```

2. **assignable**：通过类或接口过滤类，例如，排除所有继承自`com.example.ExcludedClass`的类

   ```xml
   <context:exclude-filter type="assignable" expression="com.example.ExcludedClass"/>
   ```

3. **regex**：通过正则表达式过滤类，例如，排除所有以 Controller 结尾的类

   ```xml
   <context:exclude-filter type="regex" expression=".*Conteoller"/>
   ```

4. **custom**：自定义过滤器类，该类必须实现 [TypeFilter 接口](#TypeFilter%20接口)

   ```xml
   <context:exclude-filter type="custom" expression="com.example.CustomTypeFilter"/>
   ```

#### expression

过滤表达式，类型根据 type 属性确定

## 属性

### base-package

指定需要扫描的包

### use-default-filters

指定是否使用默认的过滤器（通常情况下 Spring 将使用默认过滤器来确定需要将哪些注解标注的类注册进 Spring 容器，即是否自动检测被 `@Component` 、`@Repository` 、`@Service` 或 `@Controller` 注解标注的类）

use-default-filters 属性默认值为 true，当设置为 false 时需要明确配置过滤器以确定哪些类被包含或排除

### annotation-config

是否启用自动注入

## TypeFilter 接口

`TypeFilter` 接口是 Spring 框架中的一个接口，用于定义组件扫描时的过滤规则。通过实现 `TypeFilter` 接口，你可以自定义哪些类应该被包含或排除在组件扫描的范围之外

```java
public interface TypeFilter {
    // MetadataReader 中定义了 Class 类元数据 ClassMetadata，包含类名、是否接口、是否注解等类基础信息
	boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException;
}
```

`match()` 方法用于确定给定的类是否匹配过滤条件

下面是一个简单的自定义 `TypeFilter`：

```java
import org.springframework.core.type.classreading.MetadataReader;
import org.springframework.core.type.classreading.MetadataReaderFactory;
import org.springframework.core.type.filter.TypeFilter;

import java.io.IOException;

public class CustomTypeFilter implements TypeFilter {
    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        // 这里可以编写自定义的过滤逻辑，返回true表示匹配成功，false表示不匹配
        String className = metadataReader.getClassMetadata().getClassName();
        // 在这个示例中，假设我们只想包含名称中包含"Service"的类
        return className.contains("Service");
    }
}
```
