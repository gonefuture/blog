---
title: 《Java并发编程实战》-14   第14章 构建自定义的同步工具
tags: [java]
categories:
- 读书笔记
---

# 第14章 构建自定义的同步工具

---

## 14.1 状态依赖性的管理

依赖状态的操作可以一直阻塞直到可以继续执行，这比使它们先失败再实现起来要更为方便且更不容易出错。内置的条件队列可以使线程一直阻塞，直到对象进入某个进程可以继续执行的状态，并且当被阻塞的线程可以执行时再唤醒它们。

### 14.1.1 实例：将前提条件的失败传递给调用者

### 14.1.2 实列：通过轮询与休眠来实现简单的阻塞

### 14.1.3 条件队列

条件队列使得一组线程(称之为****等待线程集合**)能够通过某种方式来等待特定得条件变成真。

正如每个Java对象对可以作为一个锁，每个对象同样可以作为一个条件队列，并且Object中的wait、notify和notifyAll方法就构成了内部条件队列的API。对象的内置锁与其内部条件队列是互相关联的。

Object.wait会自动释放锁，并请求操作系统挂起当前线程，从而使其他线程能够获得这个锁并修改对象的状态。

## 14.2 使用条件队列

### 14.2.1 条件谓词

>将与条件队列相关联的条件谓词以及在这些谓词上等待的操作都写入文档。

每一次wait调用都会隐式地与特定的条件谓词关联起来。当调用某个特定条件谓词的wait时，调用者必须已经持有与条件队列相关的锁，并且这个锁必须保护着构成条件谓词的状态变量。

### 14.2.2 过早唤醒

当使用条件等待时(例如Object.wait或Condition.await)

* 通常都有一个条件谓词--包括一些对象状态的测试，线程在执行前必须首先通过这些测试。
* 在调用wait之前测试条件谓词，并且从wait中返回时再次进行测试。
* 在一个循环中调用wait
* 确保使用与条件队列相关的锁来保护构成条件谓词的各个状态变量。
* 当调用wait,notify或notifyAll等方法时，一定要持有与条件队列相关的锁。
* 在检查条件谓语之后以及开始执行相应的操作之前，不要释放锁。

### 14.2.4 通知

>每当在等待一个条件时，一定要确保在条件谓词变为真时通过某种方式发出通知。

只有同时满足以下两个条件时，才能用单一的notify而不是notifyAll:

* **所有等待线程的类型都相同**。 只有一个条件谓语与条件队列相关，并且每个线程在从wait返回后将执行相同的操作。
* **单进单出**。 在条件变量上的每次通知，最多只能唤醒一个线程来执行。

### 14.2.5 示例：阀门类

### 14.2.6 子类的安全性问题

### 封装条件队列

### 14.2.8 入口协议与出口协议

## 14.3 显式的Condition对象

特别注意：在Condition对象中，与wait、notify和notifyAll方法对应的分别时await、signal和aignalAll。但是，Condition对Object进行了扩展，因而它也包含wait和notify方法，一定要确保使用正确的版本--await和signal。

## 14.4 Synchronizer剖析

AQS是一个用于构建锁和同步器的框架，许多同步器都可以通过AQS很容易并且高效地构造出来。不仅`ReentrantLock`和`Semaphore`是基于AQS构建的，还包括`CountDownLatch`、`ReentrantReadWriteLock`、`SynchronousQueue`和`FutureTask`。

## 14.5 AbstractQueuedSynchronizer

在基于AQS构建的同步器类中，最基本的操作包括各种形式的获取操作和释放操作。

一个简单的闭锁 使用AbstractQueueSynchronizer实现的二元闭锁

``` java
@ThreadSafe
public class OneShotLatch {
    private final Sync sync = new Sync();
    public void signal() {  sync.releaseShared(0) }
    public void await() throws InterruptedException {
        sync.acquireSharedInterruptibly(0);
    }
    private class Sync extends AbstractQueueSynchronizer {
        protected int tryAcquireShared(int ignored) {
            // 如果闭锁式开的(state == 1),那么这个操作操作将成功，否则将失败
            return (getState() == 1) ? 1 : -1;
        }

        protected boolean tryReleaseShared(int ignored) {
            setState(1);    // 现在打开闭锁
            retrun true;    // 现在其他的线程可以获取该闭锁
        }
    }
}
```

在OneShotLatch中，AQS状态用来表示闭锁状态--关闭(0)或者打开(1)。
acquireSharedInterruptibly方法在处理失败的方式，是把这个线程放入等待线程队列中。

## 14.6 java.util.concurrent同步器类中的AQS

### 14.6.1 ReentrantLock

Lock.newCondition将返回一个新的ConditionObject实例，这是AQS的一个内部类。

### 14.6.2 Semaphore与CountDownLatch

Semaphore将AQS的同步状态用于保存当前可用许可的数量。
CountdownLatch使用AQS的方式与Semaphore很相似:在同步状态中保存的是当前的计数值。

### 14.6.3 FutureTask

在FutureTask中，AQS同步状态被用来保存任务的状态，例如，正在运行、已经完成或已取消。

### 14.6.4 ReentrantReadWriteLock

ReentrantReadwriteLock使用了一个16位的状态来表示写入锁的计数，并且使用了另一个16位的状态来表示读取锁的技术。在读取锁的操作将使用共享的获取方法与释放方法，在写入锁上的操作将使用独占的获取方法与释放方法。

AQS在内部维护一个等待线程队列，其中记录了某个线程请求的是独占访问还是共享访问。

## 总结：

要实现一个依赖状态的类 ---- 如果没有满足依赖状态的前提，那么这个类的方法必须阻塞，那么最好的方式是基于现有的库类来构建，例如Semaphore.BlockingQueue或CountDownLatch。然而，有时候现有的库类不能提供足够的功能，在这种情况下，可以使用内置的条件队列、显示的Condition对象或者AbstractQueueSynchronizer来构建自己的同步器。


