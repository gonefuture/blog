---
title: 《Java并发编程实战》-15   第16章 Java内存模型
tags: [java]
categories:
- 读书笔记
---

# 第16章 Java内存模型

---

## 16.1 什么内存模型，为什么需要它

JMM规定了JVM必须遵循一组最小保证，这组保证规定了对变量的写入操作在何时将对于其他线程可见。

## 16.1.1 平台的内存模型

在共享内存的多处理器体系架构中，每个处理器都拥有自己的缓存，并且定期地与主内存进行协调。

Java程序不需要指定内存栅栏的位置，而只需通过正确地使用同步来找出何时访问共享状态。

## 16.1.2 重排序

各种使操作延迟或者看似乱序执行的不同原因，都可以归为重排序。

## Java内存模型简介

Java内存模型是通过各种操作来定义的，包括对变量的读/写操作，监视器的加锁和释放操作，以及线程的启动和合并操作。JMM为程序中所有操作定义了一个偏序关系，称之为Happens-Before关系，那么JVM可以对它们任意地重排序。如果两个操作之间缺乏Happens-Before关系，那么JVM可以对它们任意地重排序。

Happens-Before的规则包括：

* **程序顺序规则**。 如果程序中操作A在操作B之前，那么在线程中A操作将在B操作之前执行。
* **监视器锁定规则**。 在监视器锁上的解锁操作必须在同一个监视器锁上的加锁操作之前执行。
* **volatile变量规则**。 对volatile变量的写入操作必须在对该变量的读操作之前执行。
* **线程启动规则**。 子线程上对Thread.Start的调用必须在该线程中执行任何操作之前执行。
* **线程结束规则**。 线程中的任何操作都必须在其他线程测到该线程已经结束之前执行，或者从Thread.join中成功返回，或者在调用Thread.isAlive时返回false。
* **中断规则**。 当一个线程在另一个线程上调用interrupt时，必须在被中断线程检测到interrup调用之前执行（通过抛出InterruptedException，或者调用isInterrupted和interrupted）。
* **终结器规则**。 对象的构造函数必须在启动该对象的终结器之前执行完成。
* **传递性** 如果操作A在操作B之前执行，并且操作B在操作C之前执行，那么操作A必须在操作C之前执行。

## 16.1.4 借助同步

由于Happens-Before，因此有时候可以“借助(Piggyback)”现有同步机制的可见性属性。这需要将Happens-Before的程序顺序规则与其他某个顺序规则（通常是监视器锁规则或者volatile变量规则）结合起来，从而对某个未被锁保护的变量的访问操作进行排序。

在类库中提供的其他Happens-Before排序包括：

* 将一个元素放入一个线程安全容器的操作将在另一个线程从该容器中获得这个元素的操作之前执行。
* 在CountDownLatch上的倒数操作将在线程从闭锁的await方法中返回之前执行。
* 释放Semaphore许可的操作将在从该Semaphore上获得一个许可之前执行。
* Future表示的任务的所有操作将在从Future.get中返回之前执行。
* 向Executor提交一个Runnable或Callable的操作将在任务开始之前执行。
* 一个线程到达CyclicBarrier或Exchanger的操作将在其他到达该栅栏或交换点的线程被释放之前执行。如果CyclicBarrier使用一个栅栏操作，那么到达栅栏的操作将在栅栏操作之前执行，而栅栏操作又会在线程从栅栏中释放之前执行。

## 16.2 发布

### 16.2.1 不安全的发布

 >除了不可变对象以外，使用被另一个线程初始化的对象通常都是被安全的，除非对象的发布操作是在使用该对象的线程开始使用之前执行。

### 16.2.2 安全的发布

事实上，Happens-Before比安全发布提供了提供了更加可见性与顺序保证。

### 16.2.3 安全初始化模式

程序安全的延迟初始化

``` java
@ThreadSafe
public calss SafeLazyInitialization {
    private static Resource resource;
    public synchronized static Resource getInstance() {
        if (resource == null )
            resource = new Resource();
        return resource;
    }
}

```

由于JVM将在初始化期间获得一个锁，并且每个线程都至少获取一次这个锁已确保这个类已经加载，因此在静态初始化期间，内存写入操作将自动对所有线程可见。

``` java
@ThreadSafe
public calss ResourceFactory {
    private static Resource = new Resource();
    public static Resource getResource{
        return ResourceHolder.resource;
    }
}
```

### 16.2.4 双重检查加锁

DDL（双重检查加锁）已经被广泛地废弃了。

## 16.3 初始化过程中的安全性

>初始化安全性将确保，对于被正确构造的对象，所有线程都能看到由构造函数为对象给各个final域设置的正确性，而不管采用何种方式来发布对象。而且，对于可以通过被正确构造对象中某个final域到达的任意变量（例如某个final数组中的元素，或者由一个final域引用的HashMap的内容）将同样对于其他线程是可见的。

``` java
@ThreadSafe
public class SafeStates {
    private final Map<String, String> states;

    public SafeStates() {
        states = new HashMap<String, String>();
        states.put("a",1)
        ...
        states.put("b",2)
        states.put("c",3)
    }
    public String getAbbreviation(String s) {
        return states.get(s);
    }
}

```

>初始化安全性只能保证通过final域可达的值从构造过程完成时开始的可见性。对于通过非final域可达的值，或者在构成过程完成后可能改变的值，必须采用同步来确保可见性。

## 小结

Java内存模型说明了某个线程的内存操作在哪些情况下对于其他线程是可见的。其中包括确保这些操作是按照一种Happens-Before的偏序关系进行排序，而这种关系是基于内存操作和同步操作级别来定义的。
