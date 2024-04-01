---
title: ThreadLocal
categories:
  - Java
  - 多线程
date: 2024-04-01 17:04:32
tags:
---

# ThreadLocal

## 简介

在 Java 并发编程中，ThreadLocal 是一个非常有用的工具，它提供了一种<font color=red>线程局部变量</font>的概念。简单来说，ThreadLocal 使得每个使用该变量的线程都拥有该变量的独立副本。这意味着每个线程都可以修改自己的副本，而不会影响其他线程中的副本。ThreadLocal 在处理用户会话信息、事务管理等场景中非常有用，可以避免复杂的线程同步问题。

### 使用场景

- **数据库连接管理**：在Web应用中，利用 ThreadLocal 保存数据库连接可以确保在一个线程中的所有数据库操作都使用同一个数据库连接。
- **用户会话管理**：在多用户应用中，利用 ThreadLocal 存储会话信息可以确保用户的请求在处理过程中始终访问到正确的用户数据。

### 优点

- **线程隔离**：ThreadLocal 提供了线程间数据隔离的能力，减少了线程同步的需要，从而提高了应用的性能。
- **减少同步开销**：由于每个线程访问自己的副本，减少了线程之间的竞争，避免了同步带来的性能开销。

### 注意事项

- **内存泄漏风险**：如果 ThreadLocal 变量持有大型对象或连接等资源，而且没有及时清理，可能会导致内存泄漏。

### 最佳实践

- **及时清理**：使用完ThreadLocal变量后，应该调用 `ThreadLocal.remove()` 方法清理资源，避免内存泄漏。
- **尽量避免存储大型对象**：避免在 ThreadLocal 中存储生命周期长、体积大的对象。

## 原理

ThreadLocal 通过为每个线程提供一个独立的值存储空间来实现其功能。它使用 Thread 内部的ThreadLocalMap，这是一个特殊的Map，**键是 ThreadLocal 实例，值是线程特定的对象**。当线程首次通过ThreadLocal 的 `get()` 或 `set()` 访问变量时，ThreadLocal 会在当前线程的 ThreadLocalMap 中查找对应的值，从而确保每个线程都只能访问到自己的变量副本。

`ThreadLocalMap` 类

```java
// ...
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;
    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
// ....
```

`get()` 方法

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

`set()` 方法

```java
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

`createMap(Thread t, T firstValue)` 方法

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
void createMap(Thread t, T firstValue) {
    t.threadLocals = new ThreadLocalMap(this, firstValue);
}
```

`getMap(Thread t)` 方法

```java
ThreadLocal.ThreadLocalMap threadLocals = null;
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

## 实例

<font color=red>每个 `ThreadLocal` 实例只能为每个线程存储一个值。如果你需要在一个线程中存储多个值，你可以创建多个 `ThreadLocal` 实例，每个实例负责存储一个独立的值</font>

假设你在一个 Web 应用中需要为每个线程独立地存储用户信息和事务状态，你可以为每种类型的信息创建一个 `ThreadLocal` 实例

```java
java
public class ThreadContext {
    private static final ThreadLocal<String> currentUser = new ThreadLocal<>();
    private static final ThreadLocal<String> currentTransaction = new ThreadLocal<>();

    public static void setCurrentUser(String user) {
        currentUser.set(user);
    }

    public static String getCurrentUser() {
        return currentUser.get();
    }

    public static void setCurrentTransaction(String transaction) {
        currentTransaction.set(transaction);
    }

    public static String getCurrentTransaction() {
        return currentTransaction.get();
    }

    public static void clear() {
        currentUser.remove();
        currentTransaction.remove();
    }
}
```

## ThreadLocal与其他并发解决方案的比较

ThreadLocal提供的是线程间的数据隔离，而`synchronized`、`volatile`、`ReentrantLock`等提供的是线程间的同步和通信机制。在需要线程隔离而非线程间通信的场景下，使用ThreadLocal更加合适。
