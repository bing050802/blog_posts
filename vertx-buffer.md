---
layout: post
comments: true
title: vertx中的buffer
date: 2017-11-17 23:08:31
tags:
- vert.x
categories:
---

### Buffer

在 Vert.x 内部，大部分数据被重新组织（shuffle，表意为洗牌）成 Buffer 格式。

一个 Buffer 是可以读取或写入的0个或多个字节序列，并且根据需要可以自动扩容、将任意字节写入 Buffer。您也可以将 Buffer 想象成字节数组（译者注：类似于 JDK 中的 ByteBuffer）。

<!-- more -->

### 创建 Buffer

可以使用静态方法 `Buffer.buffer` 来创建 `Buffer`。

`Buffer` 可以从字符串或字节数组初始化，或者直接创建空的`Buffer`。

这儿有一些创建 Buffer 的例子。

创建一个空的 Buffer：

```java
Buffer buff = Buffer.buffer();
```

从字符串创建一个 Buffer，这个 Buffer 中的字符会以 UTF-8 格式编码：
```
Buffer buff = Buffer.buffer("some string");
```
从字符串创建一个 Buffer，这个字符串可以用指定的编码方式编码，例如：
```
Buffer buff = Buffer.buffer("some string", "UTF-16");
```

从字节数组 byte[] 创建 Buffer：

```java
byte[] bytes = new byte[] {1, 3, 5};
Buffer buff = Buffer.buffer(bytes);
```

创建一个指定初始大小的 Buffer。若您知道您的 Buffer 会写入一定量的数据，您可以创建 Buffer 并指定它的大小。这使得这个 Buffer 初始化时分配了更多的内存，比数据写入时重新调整大小的效率更高。注意以这种方式创建的`Buffer` 是空的。它不会创建一个填满了 0 的Buffer。代码如下：

```java
Buffer buff = Buffer.buffer(10000);
```

### 向Buffer写入数据

向 Buffer 写入数据的方式有两种：`追加和随机写入`。任何一种情况下 Buffer 都会自动进行扩容，所以不可能在使用 Buffer 时遇到 IndexOutOfBoundsException。

### 追加到Buffer

您可以使用`appendXXX`方法追加数据到 Buffer。Buffer 类提供了追加各种不同类型数据的追加写入方法。

因为`appendXXX`方法的返回值就是 Buffer 自身，所以它可以链式地调用:

```java
Buffer buff = Buffer.buffer();

buff.appendInt(123).appendString("hello\n");

socket.write(buff);
```

### 随机访问写Buffer

您还可以指定一个索引值，通过 setXXX 方法写入数据到 Buffer，它也存在各种不同数据类型的方法。所有的 set 方法都会将索引值作为第一个参数 —— 这表示 Buffer 中开始写入数据的位置。Buffer 始终根据需要进行自动扩容。

```java
Buffer buff = Buffer.buffer();

buff.setInt(1000, 123);
buff.setString(0, "hello");
```

### 从Buffer中读取

可使用`getXXX`方法从 Buffer 中读取数据，它存在各种不同数据类型的方法，这些方法的第一个参数是从哪里获取数据的索引（获取位置）。

```java
Buffer buff = Buffer.buffer();
for (int i = 0; i < buff.length(); i += 4) {
  System.out.println("int value at " + i + " is " + buff.getInt(i));
}
```

### 使用无符号数

可使用`getUnsignedXXX`、`appendUnsignedXXX`和`setUnsignedXXX` 方法将无符号数从 Buffer 中读取或追加/设置到 Buffer 里。这对以优化网络协议和最小化带宽消耗为目的实现的编解码器是很有用的。

下边例子中，值 200 被设置到了仅占用一个字节的特定位置：

```java
Buffer buff = Buffer.buffer(128);
int pos = 15;
buff.setUnsignedByte(pos, (short) 200);
System.out.println(buff.getUnsignedByte(pos));
```

控制台中显示 200。

### Buffer长度

可使用 `length` 方法获取Buffer长度，Buffer的长度值是Buffer中包含的字节的最大索引 + 1。

### 拷贝Buffer

可使用 `copy` 方法创建一个Buffer的副本。

### 裁剪Buffer

裁剪得到的Buffer是基于原始Buffer的一个新的Buffer。它不会拷贝实际的数据。使用 `slice` 方法裁剪一个Buffer。

### Buffer 重用

将Buffer写入到一个Socket或其他类似位置后，Buffer就不可被重用了。





