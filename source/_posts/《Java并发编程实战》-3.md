---
title: 《Java并发编程实战》-3
tags: [java]
categories:
- 读书笔记
---

# 对象的组合

## 4.1 设计线程安全的类

在设计线程安全类的过程中，需要包含一下三个基本要素：

* 找出构成对象状态的所有变量。
* 找出约束状态变量的不变性条件。
* 建立对象状态的并发访问管理策略。

### 4.1.1 收集同步需求

>如果不了解对象的不变性与厚颜条件，那么就不能确保线程安全性。要满足在状态变量的有效值或状态转换上的各种约束条件，就需要结组于原子性和封装性。

### 4.1.2 依赖状态的操作

### 4.13 状态的所有权

### 4.2 实例封闭

将数据封闭在对象内部，可以将数据的访问限制在对象的方法上，从而更加容易确保在访问数据时总能持有正确的锁。

封闭机制更加易于构造线程安全的类，因为当封闭类的状态时，在分析类的线程安全性时就无需检查整个程序。

### 4.2.1 Java监视器模式

通过一个私有锁来保护状态

```java
public class PrivateLock {
    private final Object myLock = new Object();
    @GuardedBy("myLock") Widget widget;

    void someMethod() {
        synchronized(myLock) {
            // 访问或修改Widget的状态
        }
    }
}
```

### 4.2.2 实例:车辆追踪

## 4.3 线程安全性的委托

### 4.3.1 示例:基于委托的车辆跟踪器

### 4.3.2 独立的状态变量

### 4.3.3 当委托失效时

如果一个类时由多个独立且线程安全的状态变量组成，并且在所有的操作中都不包含无效状态转换，那么可以将线程安全性安全性委托给底层的状态变量。

如果一个状态变量是线程安全的，并且没有任何不变性条件来约束它的值，在变量的操作上也不存在任何不允许的状态转换，那么就可以说安全地发布这个变量。

## 4.4 在现有的线程安全类中添加功能

扩展Vector并增加一个"若没有则添加"方法

```java
@TheadSafe
public class BetterVector<E> extends Vector<E> {
    public synchronized boolean putIfAbsent(E x) {
        boolean absent = !contains(x);
        if(absent) {
            add(x)
        }
        return absent;
    }
}
```

"扩展"方法比直接将代码添加到类中更加脆弱，因为现在的同步策略实现被分布到多个单独维护的源代码文件中。

### 4.4.1 客户端加锁机制

非现象安全的"若没有则添加"（不要这么做）

```java
@NotThreadSafe
public class ListHelper<E> {
    public List<E> list = Colletions.synchronizedList(new ArrayList<E>());

    pblic synchronized boolean putIfAbsent(E x) {
        boolean absent = list.contains(x);
        if(absent)
            list.add(x);
        return absent;
    }
}
```

ListHelper只是带来了同步的假象，尽管所有的链表操作都被声明为synchronized,但是却**使用了不同的锁**。(list本身的锁和ListHelper对象的锁)

通过客户端加锁来实现“若没有则添加”（线程安全）

```java
@ThreadSafe
public class ListHelper<E> {
    public List<E> list = Colletions.synchronizedList(new ArrayList<E>());

    pblic boolean putIfAbsent(E x) {
        synchronized(list) {
        boolean absent = list.contains(x);
        if(absent)
            list.add(x);
        return absent;
        }
    }
}
```

### 4.4.2 组合

通过组合实现“若没有则添加”

```java
@ThreadSafe
public class ImporvedList<T> implements List<T> {
    private final List<T> list;

    public ImprovedList(List<T> list) { this.list = list;}

    public synchronized boolean putIfAbsent(T x) {
        boolean contains = list.contains(x);
        if (contains)
            list.add(x);
        return !contains;
    }

    public synchronized void clear() {
        lsit.clear();
    }

    // ... 按照类似的方法委托List的其他方法
}
```

事实上，我们使用了Java监视器模式来封装现有的List,并且只要在类中拥有指向底层List的**唯一**外部引用，就能确保线程安全性。

## 4.5 将同步策略文档化

>在文档中说明客户端代码需要了解的线程安全性保证，以及代码人员需要了解的同步策略。
