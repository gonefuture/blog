---
title: 《Java并发编程实战》-11
tags: [java]
categories:
- 读书笔记
---

# 第11章 性能与可伸缩性

---

## 11.1 对性能的思考

### 11.1.1 性能与可伸缩性

>可伸缩性指的是:当增加计算资源时(例如CPU、内存、存储容量或I/O带宽)，程序的吞吐量或处理能力能相应地增加。

### 11.1.2 评估各种性能权衡因素

>避免不成熟的优化。首先使程序正确，然后提高运行速度--如果它还运行得不够快。

以测试为基准，不要猜测。

## 11.2 Amdahl定律

Amdahl定律描述的是：在增加计算资源的情况下，程序在理论上能够实现最高加速比，这个值取决于程序中可并行组件和串行组件所占的比重。

>在所有并发程序中都包含一些串行部分。如果你认为在你的程序中不存在串行部分，那么可任意在仔细检查一遍。

### 11.2.1 实列：在各种框架中隐藏的串行部分

### 11.2.2 Amdahl定律的应用

## 11.3 线程引入的开销

### 11.3.1 上下文切换

切换线程上下文需要一定的开销，而在线程调度过程中需要访问由操作系统和JVM共享的数据结构。应用程序、操作系统以及JVM都使用一组相同的CPU。

### 11.3.2 内存同步

同步操作的性能开销包括多个方面。在`synchronized`和`volatile`提供的可见性保证中可能会使用一些特殊指令，即内存栅栏(Memory Barrier)。内存栅栏可以刷新缓存，使缓存无效，刷新硬件的写缓冲，以及停止执行管道。

>不要过度担心非竞争同步带来的开销。这个基本的机制已经非常快了，并且JVM还能进行额外的优化以及进一步降低或消除开销。因此，我们应该将优化重点放在那些发生锁竞争的地方。

### 阻塞

当线程无法获取某个锁或者由于某个条件等待或在I/O操作上阻塞时，需要被挂起，在这个过程中将包含两次额外的上下文切换，以及所有必要的操作系统操作和缓存操作：被阻塞的线程在其执行时间片还未用完之前就被就被交换换出去，而在随后当要获取的锁或者其他资源可用时，又再次被切换回来。（由于锁竞争而导致阻塞时，线程在持有锁时将存在一定的开销：当它释放锁时，必须告诉操作系统恢复运行阻塞的线程）。

## 11.4 减少锁的竞争

>在并发程序中，对可伸缩性的最主要威胁就是独占方式的资源锁。

由3种方式可以降低锁的竞争程度：

* 减少锁的持有时间。
* 降低锁的请求频率。
* 使用带有协调机制的独占锁，这些机制允许更高的并发性。

### 11.4.1 缩小锁的范围（"快进快出"）

降低发生竞争可能性的一种有效方式就是尽可能缩短锁的持有时间。例如，而可以将一些与锁无关的代码移出同步代码块，尤其是那些开销较大的操作，以及可能被阻塞的操作，例如I/O操作。

### 11.4.2 减小锁的粒度

通过锁分段和锁分解等降低线程请求锁的频率（从而减小竞争的可能性）的技术可以减小锁的持有时间，在这些技术中将采用多个互相独立的锁来保护独立的状态变量，从而改变这些变量之前由单个锁保护的情况。

### 11.4.3 锁分段

锁分段的一个劣势在于:与采用单个锁来实现独占访问相比，要获取多个锁来实现独占访问将更加困难并且开销更高。

### 11.4.4 避免热点域

### 11.4.5 一些替代独占锁的方法

放弃使用独占锁来管理共享状态:

* 使用并发容器
* 读-写锁
* 不可变对象
* 原子对象

### 11.4.6 监测CPU的利用率

如果CPU没有得到充分利用，那么需要找到其中的原因。通常有一下几种原因:

* 负载不充足。
* I/O密集。
* 外部限制。
* 锁竞争。

## 11.4.7 向对象池说“不”

>通常，对象分配操作的开销比同步的开销更低。

## 11.5 实例：比较Map的性能

## 11.6 减少上下文切换的开销

## 小结

由于使用线程常常是为了充分利用多个处理器的计算能力，因此并发程序性能的讨论中，童话参观更多地将侧重点放在吞吐量和可伸缩性上，而不是服务时间。Amdahl定律告诉我们，程序的可伸缩性取决于在所有代码中必须被串行执行的代码比例。因为Java程序中串行执行的主要来源是独占凡是的资源锁，因此通常可以通过以下方式来提升可伸缩性：减少锁的持有时间，降低锁的粒度，意见采用非独占锁或非阻塞锁来代替独占锁。
