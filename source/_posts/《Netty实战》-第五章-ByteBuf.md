---
title: 《Netty实战》-第五章-ByteBuf
tags: 
- Netty
- Java
- NIO
categories:
- 读书笔记
---

# 《Netty实战》-第五章-ByteBuf

---

## 5.1 ByteBuf的API

Netty的数据处理API通过两个组件暴露---abstract class ByteBuf和interface ByteBufHolder。
下面是一些ByteBuf API的优点：

* 他可以被用户自定义的缓冲区类型扩展
* 通过内置的复合缓冲区类型实现了透明的零拷贝
* 容量可以按需增长（类似于JDK的StringBuilder）
* 在读和写这两种模式之间切换不需要调用ByteBuffer的flip()方式
* 读和写使用不同了的索引
* 支持方法的链式调用
* 支持引用计数
* 支持池化
 
其他类可用于管理ByteBuf实例的分配，以及执行各种针对数据容器本身和它所持有的数据的操作。

---

## 5.2 ByteBuf类 —— Netty的数据容器

### 5.2.1 它是如何工作的

ByteBuf维护两个不同的索引：一个用于读取，一个用于写入。

名称以read或者write开头的ByteBuf方法，将会推进其对应的索引，而名称以set或者get开头的操作则不会。后面的这些方法将一个参数传入的一个相对索引上执行操作。

### 5.2.2 ByteBuf的使用模式

1. 堆缓冲区
    最常用的ByteBuf模式是将数据储存在JVM的堆空间中。这种模式被成为支撑数组。
2. 直接缓冲区
    直接缓冲区是另一种ButeBuf模式，在垃圾回收之外。
3.复合缓冲区
    复合缓冲区为多个ByteBuf提供了一个聚合视图，

## 5.3 字节级操作

### 5.3.1 随机访问索引

BuyteBuf的索引是从零开始的：第一个字节的索引是0，最后一个字节的索引总是capacity()-1

### 5.3.2 顺序访问索引

ByteBuf同时具有读索引和写索引，但是JDK的ByteBuffer却只有一个索引，这也是为甚恶魔必须调用filp()方法来在读模式和写模式之间进行切换的原因。

### 5.3.3 可丢弃字节

### 5.3.4 可读字节

ByteBuf的可读字节分段储存例如实际数据。新分配的、包管理的或者复制的缓冲区的默认的readerIndex的值为0。

### 5.3.5 可写字节

可写字节分段是指一个拥有未定义内容的、写入就绪的内存区域。新分配的缓冲区的writerIndex的默认值为0。

### 5.3.6 索引管理

### 5.3.7 查找管理

调用clear()比调用discardReadBytes()轻量得多，因为它将只是重置索引而不会复制任何得内存。

### 5.3.7 查找操作

indexOf()方法可以用来确定指定值得索引的方法。

### 5.3.8 派生缓冲区

派生缓冲区为ByteBuf提供了专门的方式来呈现其内容的视图。这类视图是通过一下方法被创建的：

* duplicate()
* slice()
* slice(int, int)
* Unpoooled.unmodifiableBuffer { ... }
* order(ByteOrder)
* readSlice(int)

每个这些方法都将返回一个新的ByteBuf实例，它具有自己的读索引、写索引和标记索引。

**BYteBuf复制** 如果需要一个现有缓冲区的真实副本，请使用copy()或者copy(int, int)方法。不同于派生缓冲区，由于这个调用所返回的ByteBuf拥有独立的数据副本。

### 5.3.9 读/写操作

netty有两种类别的读/写操作。

* get()和set()操作，从给定的索引开始，并且保持索引不变。
* read()和write()操作，从给定的索引开始，并且会根据已经访问过的字节数对索引进行掉整。

### 5.3.10 更多的操作

## ByteBufHolder接口

ByteBufHolder有几种用于访问底层数据和引用计算的方法。

## 5.5 ByteBuf分配

### 5.5.1 按需分配：ByteBufAllocator接口

 为了降低分配和释放内存的开销，Netty通过interface ByteBufAllocator实现了(ByteBuf的)池化，它可以用来分配我们所描述过的任意类型的ByteBuf实例。使用池化是应用程序的决定，其并不会以任何方式改变ByteBuf API(的语义)。

 Netty提供了两种ByteBufAllocator的实现：**PooledByteBufAllocator**和**UnpooledByteBufAllocator**。前者池化了ByteBuf的实例以提高性能并最大限度地减少内存碎片.此实现使用了一种为jemalloc的已被大量现代操作系统所采用的高效方法来分配内存。后者的实现不池化ByteBuf实例，并且在每次被它调用时都会返回一个新的实例。

### 5.5.2 Unpooled缓冲区

Unpooled的工具类提供了静态的辅助方法来创建未池化的ByteBuf实例。

### 5.5.3 ByteBufUtil类

ByteBufUtil.hexdump()方法以十六进制的表示形式打印bytebuf的内容。

## 5.6 引用计算

引用计算是一种通过在某个对象所持有的资源不再被其他对象引用时释放该对象所持有的资源来优化内存使用和性能的技术。ByteBuf和BytebufHolder引入了引用计数计数，它们都实现了interface ReferenceCounted。

## 5.7 小结

ByteBuf的要点：

* 使用不同的读索引和写索引来控制数据访问。
* 使用内存的不同方式——基于字节数组和直接缓冲区
* 通过CompositeByteBuf生成多个ByteBuf的聚合视图
* 数据访问方法——搜索、切片以及复制
* 读、写、获取和设置API
* ByteBufAllocator池化和引用计算



