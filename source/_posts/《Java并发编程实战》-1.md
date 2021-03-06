---
title: 《Java并发编程实战》-1
tags: [java]
categories:
- 读书笔记
---

# 第2章 线程安全性

## 2.1 什么是线程安全性

> 当多个线程访问某个类时，不管运行时环境采用何种调度方式或者这些线程架构如何交替执行，并且在主调用代码中不需要任何额外的同步或协同，这个类都表示出正确的行为，那么就称这个类是线程安全。

> 在线程安全类中封装了必要的同步机制，因此客户端无需进一步采用同步措施。

>无状态对象一定是线程安全的。

## 2.2 原子性

### 2.2.1 竞态条件

### 2.2.2 延迟初始化中的竞态条件

延迟初始化中的竞态条件（不要这样做）

```java
@NotThreadSafe
public class LazyInitRace {
	private ExpensiveObject instance = null;
	public ExpensiveObject getInstance() {
	if( instance == null)
		instance = new ExpensiveObject();
	return instance;
}
```

### 2.2.3 复合操作

>假定有两个操作A和B，如果从执行A的线程来看，当另一个线程执行B时，要么将B全部执行完，要么完成不执行B，那么A和B对彼此来说时原子的。原子操作是指，对于访问同一个状态的所有操作(包括该操作本身)来说，这个操作是一个以原子方式执行的操作。

>在实际情况中，应尽可能使用现有的线程安全对象（例如AcomicLong）来管理类的状态。与非线程安全的对象相比，判断线程安全对象的可能状态及其状态转换情况要更为容易，从而也更容易维护和验证线程安全性。

## 2.3 加锁机制

> 要保持状态的一致性，就需要在单个原子操作中更新所有相关的状态变量。

### 2.3.1 内置锁

### 2.3.2 重入

## 2.4 用锁来保护状态

### 对于可能被多个线程同时访问的可变状态变量，在访问它的时都需要持有同一个锁，在这种情况下，我们称状态变量是由这个锁保护的。

### 每个共享的和可变的变量都应该只由一个锁来保护，从而使维护人员知道是哪一个锁。

### 对于每个包含多个变量的不变性条件，其中涉及的所有变量都需要由同一个锁来保护。

 ```java
 if (!vector.contains(element) )
	 vector.add(element)
 ```
 
 >即使Vector时同步的，因为上米娜的复合操作不是原子的，所以上面的代码也是非线程安全的。



## 2.5 活跃性与性能

> 通常，在简单性与性能之间存在着互相约制因素。当实现某个同步策略时，一定不要盲目地为了性能而牺牲简单性（这可能破坏安全性）。

> 当执行时间较长的计算或者可能无法完成的操作时(例如，网络I/O或控制台I/O)，一定不要持有锁。