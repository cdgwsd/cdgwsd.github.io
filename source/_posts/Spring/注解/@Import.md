---
title: Import
tags:
  - Spring
  - 注解
  - Import
date: '2024-03-20 17:13:00Import'
categories:
  - Spring
  - 注解
---

# @Import

`@Import` 注解用于导入一个或多个组件类

`@Import` 注解可导入以下类型：

1. **@Configuration**：典型情况下，用于导入其他的配置类，以便在当前配置类中引入其他配置的 Bean
2. **ImportSelector 实现**：ImportSelector 是一个接口，可以用于根据条件动态地选择要导入的配置类
3. **ImportBeanDefinitionRegistrar 实现**：ImportBeanDefinitionRegistrar 是一个接口，用于在运行时动态地注册 BeanDefinition
4. **普通组件类**: 从 Spring 4.2 版本开始，`@Import` 注解也可以导入普通的组件类，这些类不需要标记为 `@Configuration`，它们将被作为 Bean 注册到当前配置类中

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Import {

	/**
	 * {@link Configuration @Configuration}, {@link ImportSelector},
	 * {@link ImportBeanDefinitionRegistrar}, or regular component classes to import.
	 */
	Class<?>[] value();

}
```

## ImportSelector 接口

ImportSelector 接口源代码：

```java
public interface ImportSelector {
    String[] selectImports(AnnotationMetadata importingClassMetadata);
    
    @Nullable
	default Predicate<String> getExclusionFilter() {
		return null;
	}
}
```

- **selectImports**：返回一个包含了类全限定名的数组，这些类会注入到Spring容器当中。注意如果为null，要返回空数组，不然后续处理会报错空指针

- **getExclusionFilter**：该方法制定了一个对类全限定名的排除规则来过滤一些候选的导入类，默认不排除过滤。该接口可以不实现

### 实例

编写一个类实现 ImportSelector 接口

```java
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"me.zyp.config.Student", "me.zyp.config.Person"};
    }
}
```

在配置类中使用 @Import 导入

```java
@Configuration
@Import({MyImportSelector.class})
public class MyConfig {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MyConfig.class);
        String[] beanDefinitionNames = applicationContext.getBeanDefinitionNames();
        // 遍历Spring容器中的beanName
        for (String beanDefinitionName : beanDefinitionNames) {
            System.out.println(beanDefinitionName);
        }
    }
}
```

## ImportBeanDefinitionRegistrar 接口

ImportBeanDefinitionRegistrar 接口源代码

```java
public interface ImportBeanDefinitionRegistrar {
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
		registerBeanDefinitions(importingClassMetadata, registry);
	}
    
    default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {}
}
```

一共有两个同名重载方法，都是用于将类的 BeanDefinition 注入。

唯一的区别就是，2个参数的方法，只能手动的输入 beanName，而 3 个参数的方法，可以利用BeanNameGenerator 根据 beanDefinition 自动生成 beanName

### 实例

```java
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    //使用 BeanNameGenerator自动生成beanName
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry, BeanNameGenerator importBeanNameGenerator) {
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(Person.class);
        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        String beanName = importBeanNameGenerator.generateBeanName(beanDefinition, registry);
        registry.registerBeanDefinition(beanName, beanDefinition);
    }

    // 手动指定beanName
//    @Override
//    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
//        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(Student.class);
//        AbstractBeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
//        registry.registerBeanDefinition("student001", beanDefinition);
//    }

}
```

注意，这里只能注入一个bean，所以只能实现一个方法进行注入，如果两个都实现，前面的一个方法生效

### BeanDefinitionRegistry

BeanDefinitionRegistry 用于向 Spring 容器中注册新的 BeanDefinition。这些 BeanDefinition 包含了 Bean 的配置信息，例如类名、作用域、初始化方法、销毁方法等

### BeanDefinition

该接口定义 Spring 容器中 Bean 的配置元数据，包括：

1. **描述 Bean 的配置信息**：`BeanDefinition` 描述了一个 Bean 的配置信息，包括了 Bean 的类名、作用域、构造函数参数、属性值、初始化方法、销毁方法等。这些信息决定了 Spring 容器如何创建和管理这个 Bean。
    
2. **提供 Bean 的元数据**：通过 `BeanDefinition`，可以了解到一个 Bean 的各种元数据，如类名、构造函数、属性等，这些信息对于容器的运行和管理非常重要。
    
3. **允许延迟初始化**：`BeanDefinition` 中的配置信息可以用于延迟初始化 Bean。通过设置懒加载属性，可以让容器在需要时才去实例化 Bean，而不是在启动时就创建。
    
4. **支持不同的作用域**：`BeanDefinition` 可以指定 Bean 的作用域，如单例、原型、请求、会话等。这决定了容器如何管理 Bean 的生命周期和实例化。
    
5. **实现了扩展性**：Spring 允许通过编程方式、XML 配置或者注解来定义 Bean，这些定义最终都会被转换为 `BeanDefinition` 对象。因此，`BeanDefinition` 的存在使得 Spring 在不同的配置方式之间能够保持一致性。
    
6. **支持自定义 BeanPostProcessor 和 BeanFactoryPostProcessor**：通过 `BeanDefinition`，可以指定 BeanPostProcessor 和 BeanFactoryPostProcessor，这样可以在容器实例化 Bean 或者在 Bean 初始化前后做一些自定义操作。