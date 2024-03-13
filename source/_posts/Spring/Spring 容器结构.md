# BeanFactory

Spring 容器最重要的作用就是负责实例化应用程序中的对象（Bean）。包括创建对象、初始化对象以及维护对象的生命周期

Spring 提供了两种主要容器

1. **BeanFactory** 容器：
   - **介绍：** `BeanFactory` 是 Spring 框架最基本的容器接口。它提供了最简单的容器实现，负责管理 Spring 应用程序中的 Bean
   - **特点：** 延迟初始化，即在需要使用 Bean 时才进行初始化
   - **主要实现类：** `XmlBeanFactory` 是 `BeanFactory` 接口的一个实现，它从 XML 文件中加载 Bean 定义
2. **ApplicationContext 容器：**
   - **介绍：** `ApplicationContext` 是 `BeanFactory` 的子接口，提供了更多的企业级功能。它是一个更高级别的容器，除了提供 Bean 的管理之外，还提供了事件机制、AOP 等功能。
   - **特点：** 提前初始化，即在容器启动时就进行初始化
   - **主要实现类：**
     - `ClassPathXmlApplicationContext`：从类路径加载 XML 配置文件创建容器
     - `FileSystemXmlApplicationContext`：从文件系统路径加载 XML 配置文件创建容器
     - `AnnotationConfigApplicationContext`：基于 Java 配置类（使用注解）创建容器
     - 其他如 `GenericWebApplicationContext` 等，用于特定环境或特定配置

# BeanFactory 属性

![image-20231113172930753](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202311131729272.png)

BeanFactory 中有几个重要的属性

- `beanDefinitionMap`：用于存储 bean 定义的映射
- `beanDefinitionNames`：用于存储所有 bean 的名称
- `singletonObjects`：用于存储单例 bean 的实例
- `earlySingletonObjects`：用于存储尚未完全初始化的单例 bean 的实例
- `registeredSingletons`：用于存储已注册的单例 bean 的名称
- `beanPostProcessors`：用于存储 bean 后置处理器

## BeanDefinitionMap

```java
/**
* key：Bean 的名称
* Value：包含了 Bean 各种配置信息，如：Bean 类名、作用域、构造函数参数、属性值等的 BeanDefinition 对象
*/
private final Map<String, BeanDefinition> beanDefinitionMap
```



![image-20231113174626481](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202311131746624.png)

## BeanDefinitionNames

![image-20231113202800504](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202311132028270.png)

## SingletonObjects

用于缓存 Spring 容器中已经创建并初始化的单例（singleton）bean 的实例。在 Spring 中，单例 bean 是指在容器启动时创建的，且整个应用中只存在一个实例的 bean。

![image-20231113174746687](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202311131747563.png)

