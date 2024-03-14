---
categories:
  - Java
---
# I/O 流

![image.png](https://cdgwsd.oss-cn-guangzhou.aliyuncs.com/img/202312271502333.png)

下面通过 `3W` 理论介绍 Java `I/O` 流：

- What

  ```markdown
  Java I/O 流是 Java 编程语言中用于处理输入和输出数据的机制。I/O 流主要用于在程序与外部世界之间传输数据。在 Java 中，I/O 流分为输入流和输出流，用于读取和写入数据。I/O 流的工作方式类似于水流的概念，数据可以像水流一样从一个地方传输到另一个地方
  ```

- Why

  ```markdown
  在实际应用中，我们经常需要读取文件、网络通信、处理用户输入等操作，这些都需要使用 I/O 流来进行数据的读写。I/O 流的使用可以帮助程序实现更灵活、更复杂的功能，使得程序可以与用户、文件系统、网络等进行有效的通信
  ```

- How

  ```markdown
  在 Java 中，实现输入和输出的主要方式是通过使用输入流与输出流。根据类型不同又分为两大类：字节输入/输出流和字符输入/输出流
  ```

## 流类型

- 按流向分
  - 输入流
  - 输出流
- 按数据类型分
  - 字节流
  - 字符流
- 按功能分
  - 节点流：直接操作数据源或目的地
  - 处理流
    - 通过包装节点流，提供对数据的额外处理能力
    - 不能直接操作数据源或目的地，而是通过包装或封装节点流提供附加功能

### I/O 流体系

| 分类 |  | 字节输入流 | 字节输出流 | 字符输入流 | 字符输出流 |
| :--: | ---- | :--------: | :----------: | :----------: | :----------: |
|      |      |            |            |            |            |
|      |      |            |            |            |            |
|      |      |            |            |            |            |

### 流关闭

无论是哪种类型的流，在操作完毕后都需要调用 `close()` 方法关闭流并释放相应的系统资源

| 方法名                          | 方法说明                                                     |
| ------------------------------- | ------------------------------------------------------------ |
| void close() throws IOException | 流操作完毕后，必须释放系统资源，调用close方法，一般放在finally块中保证一定被执行! |

## 字节流

在计算机中，一切数据的存储都是以二进制形式进行存储的，都可以使用字节进行读取和写入

### 输入流

`InputStream` 类是字节输入流的基类

```java
// Closeable 接口中定义 close() 方法
public abstract class InputStream implements Closeable {}
```

字节输入流常用方法：

```java
// 从输入流中读取一个字节的数据并返回，读到文件末尾时返回-1
public abstract int read() throws IOException;

// 将数据读取到字节数组中，返回读取到的有效字节数，读取到末尾时返回-1
public int read(byte b[]) throws IOException {}

// 从数据源中最多读取len个字节，并从偏移量off开始写入到字节数组中，读取到末尾时返回-1
public int read(byte b[], int off, int len) throws IOException {}
```

<font color=red>返回值之所以是 int 类型而不是 byte 类型，是因为如果将 byte 类型与读取到的数据完全对应，无法判断什么时候读取到末尾</font>

### 输出流

`OutputStream` 类是字节输出流的基类

```java
// Closeable 接口中定义 close() 方法
// Flushable 接口中定义 flush() 方法
public abstract class OutputStream implements Closeable, Flushable {}
```

字节输出流常用方法：

```java
// 将 int 类型数据写入到输出流中
public abstract void write(int b) throws IOException;

// 将字节数组写入到输出流中
public void write(byte b[]) throws IOException {}

// 将字节数组从偏移量off开始写入len长度数据到输出流中
public void write(byte b[], int off, int len) throws IOException {}

// 刷新输出流并强制缓冲的字节被写出
public void flush() throws IOException {}
```

## 字符流

字符流用于以字符为单位进行数据输入和输出。在Java中，字符流主要通过 `Reader` 及其子类实现输入，通过`Writer` 及其子类实现输出。字符流与字节流的主要区别在于处理的数据单位不同，字符流适用于处理文本数据

### 输入流

`Reader` 类是字符输入流的基类

```java
public abstract class Reader implements Readable, Closeable {}
```

字符输入流常用方法

```java
// 从输入流中读取一个字符,读到文件末尾时返回-1
public int read() throws IOException {}

// 	从输入流中读取字符到char数组中,读到文件末尾时返回-1
public int read(char cbuf[]) throws IOException {}
```

### 输出流

`Writer` 类是字符输出流的基类

```java
public abstract class Writer implements Appendable, Closeable, Flushable{}
```

字符输出流常用方法

```java
// 写入单个字符到输出流中
public void write(int c) throws IOException {}

// 写入字符数组到输出流中
public void write(char cbuf[]) throws IOException {}

// 直接写入字符串到输出流中
public void write(String str) throws IOException {}

// 写入字符串的一部分，偏移量off开始，长度为len
public void write(String str, int off, int len) throws IOException {}

// 追加字符到输出流中
public Writer append(CharSequence csq) throws IOException {}
```

## 缓冲流

前面说了节点流，都是直接使用操作系统底层方法读取硬盘中的数据，缓冲流是处理流的一种实现，增强了节点流的性能，为了提高效率，缓冲流类在初始化对象的时候，内部有一个**缓冲数组**，一次性从底层流中读取数据到数组中，程序中执行 `read()` 或者 `read(byte[])` 的时候，就直接从内存数组中读取数据

分类：

- 字节缓冲流
  - BufferedInputStream
  - BufferedOutputStream
- 字符缓冲流
  - BufferedReader
  - BufferedWriter