---
title: @Import
tags:
  - Spring 
  - 注解
  - @Import
date: 2024-03-20 17:13:00Import
---

# @Import

`@Import` 注解用于在 @Componnet 直接或间接标注的类中导入其他配置类或者注册额外的 Bean 定义 。通过 @Import 注解，可以将一个或多个其他配置类引入到当前的配置类中，以便组织和管理Bean的配置信息

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

@Import 注解可导入以下类型：

1. **配置类 (`@Configuration` 类)**: 典型情况下，用于导入其他的配置类，以便在当前配置类中引入其他配置的 Bean
2. **ImportSelector 实现类**: ImportSelector 是一个接口，可以用于根据条件动态地选择要导入的配置类
3. **ImportBeanDefinitionRegistrar 实现类**: ImportBeanDefinitionRegistrar 是一个接口，用于在运行时动态地注册 BeanDefinition
4. **普通组件类**: 从 Spring 4.2 版本开始，`@Import` 注解也可以导入普通的组件类，这些类不需要标记为 `@Configuration`，它们将被作为 Bean 注册到当前配置类中

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

注意，这里只能注入一个bean，所以只能实现一个方法进行注入，如果两个都是实现，前面的一个方法生效

