---
title: Netty源码阅读（整体介绍）
tags: [源码]
categories:
- 源码笔记
---

## Netty的作用

1. Netty实现对Java NIO的封装，提供了更方便使用的接口；

2. Netty利用责任链模式实现了ChannelPipeline这一概念，基于CHannelPipeline,我们可以优雅的实现网络消息的处理（可插拔，解耦）；

3. Netty的Reactor线程模型，利用无锁化提高了系统的性能；

4. Netty实现了ByteBuf用于对字节进行缓存和操作,相比JDK的ByteBuffer,他更容易用，同时还提供了Buffer池的功能，对于UnpooledDirectByteBuf和PooledByteBuf,Netty还对其内存使用进行了跟踪，发现内存泄漏时会给出报警。

## Netty对JDK NIO的封装

JDK NIO有ServerSocketChannel、SocketChannel、Selector、Selection几个核心概念。

Netty提供了一个Channel接口统一了对网络的IO操作，其底层的IO是交给Unsafe接口实现，而Channel主要负责更高层次的read、write、flush和ChannelPipeline、Eventloop等组件的交互，以及一些状态的展示；做到了职责的清晰划分，对使用者是很友好的，规避了JDK NIO中一些比较繁琐负责的概念和流程。

Channel、Unsafe继承UML图

Channel和Unsafe是分多级别是实现的

## ChannelPipeline责任链模式

