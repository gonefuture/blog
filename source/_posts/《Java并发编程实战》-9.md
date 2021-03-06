---
title: 《Java并发编程实战》-9
tags: [java]
categories:
- 读书笔记
---

# 第10章 避免活跃性危险

---

## 10.1 死锁

哲学家问题

### 10.1.1 锁顺序死锁

>如果所有线程以固定的顺序来获得锁，那么在线程中就不会出现锁顺序死锁问题。

### 10.1.2 动态的顺序死锁

通过锁顺序来避免死锁

### 10.1.3 在协助对象之间发生的死锁

如果在持有锁时调用某个外部方法，那么将出现活跃性问题。在这个外部方法中可能会获取其他锁(这可能会产生死锁)，或者阻塞时间过长，导致其他线程无法及时获得当前被持有的锁。

### 10.1.4 开放调用

如果在调用某个方法时不需要持有锁，那么这种调用被称为开放调用(Open Call)。

>在程序中应尽量使用开发调用。与那些在持有锁时调用外部方法的程序相比，更易于对依赖于开放调用的程序进行死锁分析。

### 10.1.5 资源死锁

当多个线程早相同的资源集合上等待时，也会发生死锁。

## 10.2 死锁的避免与诊断

### 10.2.1 支持定时的锁

显式使用Lock类中定时tryLock功能代替内置锁机制可以检测死锁和从死锁中恢复过来。

### 10.2.2 通过线程转储信息来分析死锁

虽然防止死锁的主要责任在于你自己，但JVM仍然通过线程转储(Thread Dump)来帮助识别死锁的发生。线程转储包括各个运行中的线程的栈追踪信息，这类似发生异常时的栈追踪信息。

## 10.3 其他活跃性危险

活跃性危险：

* 死锁
* 饥饿
* 丢失信号
* 活锁

### 10.3.1 饥饿

当线程由于无法访问它所需要的资源而无法继续执行时，就发生了“饥饿”,引发饥饿的最常见资源就是CPU时钟周期。

>要避免使用线程优先级，因为这会增加平台依赖性，并可能导致活跃性问题。在大多数并发应用程序中，都可以使用默认的线程优先级。

### 10.3.2 糟糕的响应性

### 10.3.3 活锁

活锁是另一种形式的活跃性问题，该问题尽管不会阻塞线程，但也不能继续执行，因为线程将不断重复地执行相同的操作，并且总会失败。活锁通常发生在处理事务消息的应用程序中：如果蹦年成功处理某个消息，那么消息处理机制将回滚整个事务，并且将它重新发到队列的开头。

## 总结

活跃性故障时一个非常严重的问题，因为当出现活跃性故障时，除了终止应用程序之外没有其他任何机制可以邦帮助从这些故障中恢复过来。最常见的活跃性故障就是所顺序死锁。在设计时应该避免产生锁顺序死锁：确保线程在获取多个锁时采用一致的顺序。最好的解决方法是在程序中始终使用开放调用。这种大大减少需要同时持有多个锁的地方，也更容易发现这些地方。
