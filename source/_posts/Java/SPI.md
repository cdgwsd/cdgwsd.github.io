---
title: SPI
tags:
  - SPI
categories:
  - Java
date: 2024-03-20 14:14:00
---

# SPI

Java SPI（Service Provider Interface）是一种服务发现机制，它允许服务提供者在运行时被发现和加载，而不是在编译时。SPI 非常适用于模块化开发，因为它可以用来加载扩展或插件，而无需对主程序代码进行修改。Java SPI 的使用在很多流行的库和框架中都有体现，例如 JDBC、Spring Boot 等

## 简介

SPI 旨在由第三方实现或扩展，它是一种用于动态加载服务的机制。Java 中 SPI 机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是**解耦**

Java SPI 有四个要素：

- **SPI 接口：**为服务提供者实现类约定的的接口或抽象类
- **SPI 实现类：**实际提供服务的实现类
- **SPI 配置：**Java SPI 机制约定的配置文件，提供查找服务实现类的逻辑。配置文件必须置于 **`META-INF/services`** 目录中，并且，<font color=red>文件名应与服务提供者接口的完全限定名保持一致。文件中的每一行都有一个实现服务类的详细信息，同样是服务提供者类的完全限定名称</font>
- **ServiceLoader：**Java SPI 的核心类，用于加载 SPI 实现类。ServiceLoader 中有各种实用方法来获取特定实现、迭代它们或重新加载服务

## 示例

### SPI 接口

首先，需要定义一个 SPI 接口，和普通接口并没有什么差别

```java
package me.zyp.spi;

public interface DataStorage {
    String search(String key);
}
```

### SPI 实现类

假设，我们需要在程序中使用两种不同的数据存储——MySQL 和 Redis。因此，我们需要两个不同的实现类去分别完成相应工作

MySQL 查询 MOCK 类

```java
package me.zyp.spi;

public class MysqlStorage implements DataStorage {
    @Override
    public String search(String key) {
        return "【Mysql】搜索" + key + "，结果：No";
    }
}
```

Redis 查询 MOCK 类

```java
package me.zyp.spi;

public class RedisStorage implements DataStorage {
    @Override
    public String search(String key) {
        return "【Redis】搜索" + key + "，结果：Yes";
    }
}
```

service 传入的是期望加载的 SPI 接口类型。到目前为止，定义接口并实现接口和普通的 Java 接口实现没有任何不同

### SPI 配置

如果想通过 Java SPI 机制来发现服务，就需要在 SPI 配置中约定好发现服务的逻辑。配置文件必须置于 `META-INF/services`目录中，并且，文件名应与服务提供者接口的完全限定名保持一致。文件中的每一行都有一个实现服务类的详细信息，同样是服务提供者类的完全限定名称。以本示例代码为例，其文件名应该为 `me.zyp.spi.DataStorage`

文件内容如下：

```
me.zyp.spi.MysqlStorage
me.zyp.spi.RedisStorage
```

### ServiceLoader

完成了上面的步骤，就可以通过 ServiceLoader 来加载服务

```java
import java.util.ServiceLoader;

public class SpiDemo {

    public static void main(String[] args) {
        ServiceLoader<DataStorage> serviceLoader = ServiceLoader.load(DataStorage.class);
        System.out.println("============ Java SPI 测试============");
        serviceLoader.forEach(loader -> System.out.println(loader.search("Yes Or No")));
    }
}
```

输出：

```
============ Java SPI 测试============
【Mysql】搜索Yes Or No，结果：No
【Redis】搜索Yes Or No，结果：Yes
```

## 原理

Java SPI 机制依赖于 ServiceLoader 类去解析、加载服务。因此，掌握了 ServiceLoader 的工作流程，就掌握了 SPI 的原理。ServiceLoader 的代码本身很精练，接下来，让我们通过走读源码的方式，逐一理解 ServiceLoader 的工作流程

### ServiceLoader 成员变量

先看一下 ServiceLoader 类的成员变量，大致有个印象，后面的源码中都会使用到

```java
public final class ServiceLoader<S> implements Iterable<S> {

    // SPI 配置文件目录
    private static final String PREFIX = "META-INF/services/";

    // 将要被加载的 SPI 服务接口
    private final Class<S> service;

    // 用于加载 SPI 服务的类加载器
    private final ClassLoader loader;

    // ServiceLoader 创建时的访问控制上下文
    private final AccessControlContext acc;

    // SPI 服务缓存，按实例化的顺序排列
    private LinkedHashMap<String,S> providers = new LinkedHashMap<>();

    // 懒查询迭代器
    private LazyIterator lookupIterator;

    // ...
}
```

### ServiceLoader 工作流程

#### ServiceLoader 静态方法

应用程序加载 Java SPI 服务，都是先调用 `ServiceLoader.load` 方法

`ServiceLoader.load` 方法的作用是

1. 指定类加载器 ClassLoader 和访问控制上下文
2. 重新加载 SPI 服务
   1. 清空缓存中所有已实例化的 SPI 服务
   2. 根据 ClassLoader 和 SPI 类型，创建懒加载迭代器

相关源码如下

```java
public static <S> ServiceLoader<S> load(Class<S> service) {
    // 获取当前线程类加载器
    ClassLoader cl = Thread.currentThread().getContextClassLoader();
    return ServiceLoader.load(service, cl);
}

public static <S> ServiceLoader<S> load(Class<S> service,ClassLoader loader){
        return new ServiceLoader<>(service, loader);
}

// 初始化 ServiceLoader
private ServiceLoader(Class<S> svc, ClassLoader cl) {
    // 服务接口不能为 NULL
    service = Objects.requireNonNull(svc, "Service interface cannot be null");
    // 指定类加载 ClassLoader 和访问控制上下文
    loader = (cl == null) ? ClassLoader.getSystemClassLoader() : cl;
    acc = (System.getSecurityManager() != null) ? AccessController.getContext() : null;
    reload();
}

// 重加载
public void reload() {
    // LinkedHashMap<String,S> providers = new LinkedHashMap<>();
    // 清空缓存的接口实现类
    providers.clear();
    // LazyIterator lookupIterator
    // 初始化懒加载迭代器
    lookupIterator = new LazyIterator(service, loader);
}

private LazyIterator(Class<S> service, ClassLoader loader) {
    this.service = service;
    this.loader = loader;
}
```

#### 遍历 SPI 实例

ServiceLoader 的类定义，明确了 ServiceLoader 类实现了 Iterable<T> 接口，所以，它是可以迭代遍历的。实际上，ServiceLoader 类维护了一个缓存 providers（ LinkedHashMap 对象），缓存 providers 中保存了已经被成功加载的 SPI 实例，这个 Map 的 key 是 SPI 接口实现类的全限定名，value 是该实现类的一个实例对象

当应用程序调用 ServiceLoader 的 iterator 方法时，ServiceLoader 会先判断缓存 providers 中是否有数据：如果有，则直接返回缓存 providers 的迭代器；如果没有，则返回懒加载迭代器的迭代器

```java
public Iterator<S> iterator() {
    return new Iterator<S>() {

        // 缓存 SPI providers
        Iterator<Map.Entry<String,S>> knownProviders = providers.entrySet().iterator();

        // 缓存中有就从缓存取
        public boolean hasNext() {
            if (knownProviders.hasNext())
                return true;
            return lookupIterator.hasNext();
        }

        public S next() {
            if (knownProviders.hasNext())
                return knownProviders.next().getValue();
            return lookupIterator.next();
        }

        public void remove() {
            throw new UnsupportedOperationException();
        }
    };
}
```

#### 懒加载迭代器

上面的源码中提到了，lookupIterator 是 LazyIterator 实例，而 LazyIterator 用于懒加载 SPI 实例。那么， LazyIterator 是如何工作的呢？

LazyIterator 类实现 Iterator 接口，用户遍历。主要方法如下：

```java
// SPI 接口
Class<S> service;
// 类加载器
ClassLoader loader;
Enumeration<URL> configs = null;
Iterator<String> pending = null;
String nextName = null;

private boolean hasNextService() {
    if (nextName != null) {
        return true;
    }
    if (configs == null) {
        try {
            // 获取 SPI 接口全限定类名，并加载资源文件
            String fullName = PREFIX + service.getName();
            if (loader == null)
                configs = ClassLoader.getSystemResources(fullName);
            else
                configs = loader.getResources(fullName);
        }
    }
    // 解析资源文件，获取 SPI 接口实现类的全限定类名
    while ((pending == null) || !pending.hasNext()) {
        if (!configs.hasMoreElements()) {
            return false;
        }
        pending = parse(service, configs.nextElement());
    }
    nextName = pending.next();
    return true;
}

private S nextService() {
    String cn = nextName;
    nextName = null;
    Class<?> c = null;
    // 实例化实现类
    c = Class.forName(cn, false, loader);
    // 强制转换并放入 SPI 服务缓存
    S p = service.cast(c.newInstance());
    providers.put(cn, p);
    return p;
}
```

### SPI 和类加载器

通过上面的分析，我们已经大致了解 Java SPI 的工作原理，<font color=red>即通过 ClassLoader 加载 SPI 配置文件，解析 SPI 服务，然后通过反射，实例化 SPI 服务实例</font>。我们不妨思考一下，为什么加载 SPI 服务时，需要指定类加载器 ClassLoader 呢？

学习过 JVM 的读者，想必都了解过类加载器的**双亲委派模型**（Parents Delegation Model）。双亲委派模型要求除了顶层的 `BootstrapClassLoader` 外，其余的类加载器都应有自己的父类加载器。这里类加载器之间的父子关系一般通过组合（Composition）关系来实现，而不是通过继承（Inheritance）的关系实现

双亲委派机制约定了：**一个类加载器首先将类加载请求传送到父类加载器，只有当父类加载器无法完成类加载请求时才尝试加载**

**双亲委派的好处：**使得 Java 类伴随着它的类加载器，天然具备一种带有优先级的层次关系，从而使得类加载得到统一，不会出现重复加载的问题：

1. 系统类防止内存中出现多份同样的字节码
2. 保证 Java 程序安全稳定运行

例如：java.lang.Object 存放在 rt.jar 中，如果编写另外一个 java.lang.Object 的类并放到 classpath 中，程序可以编译通过。因为双亲委派模型的存在，所以在 rt.jar 中的 Object 比在 classpath 中的 Object 优先级更高，因为 rt.jar 中的 Object 使用的是启动类加载器，而 classpath 中的 Object 使用的是应用程序类加载器。正因为 rt.jar 中的 Object 优先级更高，因为程序中所有的 Object 都是这个 Object。

**双亲委派的限制：**子类加载器可以使用父类加载器已经加载的类，而父类加载器无法使用子类加载器已经加载的。——这就导致了双亲委派模型并不能解决所有的类加载器问题。Java SPI 就面临着这样的问题：

- SPI 的接口是 Java 核心库的一部分，是由 BootstrapClassLoader 加载的；
- 而 SPI 实现的 Java 类一般是由 AppClassLoader 来加载的。BootstrapClassLoader 是无法找到 SPI 的实现类的，因为它只加载 Java 的核心库。它也不能代理给 AppClassLoader，因为它是最顶层的类加载器。这也解释了本节开始的问题——为什么加载 SPI 服务时，需要指定类加载器 ClassLoader 呢？因为如果不指定 ClassLoader，则无法获取 SPI 服务。

如果不做任何的设置，Java 应用的线程的上下文类加载器默认就是 AppClassLoader。在核心类库使用 SPI 接口时，传递的类加载器使用线程上下文类加载器，就可以成功的加载到 SPI 实现的类。线程上下文类加载器在很多 SPI 的实现中都会用到。

通常可以通过 `Thread.currentThread().getClassLoader()` 和 `Thread.currentThread().getContextClassLoader()` 获取线程上下文类加载器。

## 优缺点

### 优点

1. **松耦合**：SPI 机制实现了服务接口和服务实现的解耦，服务提供者可以在不修改代码的情况下替换服务的实现
2. **扩展性**：通过 SPI 机制，系统可以动态地加载和使用服务的实现类，从而实现系统的功能扩展和模块化
3. **灵活性**：SPI 机制使得系统的各个模块可以按需加载和卸载，从而实现更灵活的系统架构和组件管理
4. **可插拔性**：SPI 机制使得系统的服务实现可以通过简单的配置文件来注册和替换，从而实现了服务的可插拔性

### 缺点

1. **发现成本**：SPI 机制需要额外的配置文件来描述服务接口和实现类之间的关系，这增加了一定的开发和维护成本
2. **动态性限制**：SPI 机制虽然提供了动态加载服务实现的功能，但加载过程是在启动时进行的，因此不够灵活，无法在运行时动态加载服务实现
3. **可扩展性限制**：SPI 机制的实现类需要遵循一定的命名约定和配置规范，这限制了系统的可扩展性和灵活性
4. **类加载器问题**：在一些复杂的应用场景中，如在 Web 容器中使用 SPI 机制，可能会涉及到类加载器的问题，需要特别注意类加载器的隔离和冲突
5. **不能按需加载**：需要遍历所有的实现，并实例化，然后在循环中才能找到我们需要的实现。如果不想用某些实现类，或者某些类实例化很耗时，它也被载入并实例化了，这就造成了浪费