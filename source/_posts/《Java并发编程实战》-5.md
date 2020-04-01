---
title: 《Java并发编程实战》-5
tags: [java]
categories:
- 读书笔记
---

# 《Java并发编程实战》-5

---

### 6.1.1 串行地执行renwu

### 6.1.2 显式地为任务创建线程

### 6.1.3 无限制创建线程的不足

* 线程生命周期的开销非常高。
* 资源消耗。
* 稳定性。

## 6.2 Executir框架

### 6.2.1 示例：基于Executor的Web服务器

### 6.2.2 执行策略

>每当看到`new Thread(runnable).start()`时，并且你希望获得一种更灵活的执行策略时，请考虑使用Excecutor来代替Thread。

### 6.2.3 线程池

* **newFixedThreadPool** 创建一个固定长度的线程池，每当提交一个任务时就创建一个线程，直到达到线程池的最大数量，这时线程池的规模将不再变化（如果某个线程由于发生了未预期的Exception而结束，那么线程池会补充一个新的线程）。
* **newCachedTheadPool** 创建一个可缓存的线程池，如果线程池的当前规模超过了处理需求时，那么将回收空闲的线程，而当需求增加时，则可以添加新的线程，线程的规模不存在任何限制。
* **newSingleThread** 一个单线程的Executor,它创建单个工作者线程来执行任务，如果这个线程异常结束，会创建另一个线程来代替。newSingleThreadExecutir能确保依照任务在队列中的顺序来串行执行（例如FIFO、LIFO、优先级）。
* **newScheduleThreadPool** 创建一个固定长度的线程池，而且以延迟或定时的方式执行任务，类是于Timer。

### 6.2.4 Executor的生命周期

* 运行
* 关闭
* 已终止


### 6.2.5 延迟任务与周期任务

## 6.3 找出可利用的并行性

### 6.3.1 示例：串行的页面渲染器

### 6.3.2 携带结果的任务Callable与Future

### 6.3.3 实例：使用Future实现页面渲染器

### 6.3.4 在异构任务并行化中存在的局限

### 6.3.5 CompletionService:Executor与BlockingQueue

CompletionService将Execute和BlockingQueue的功能融合在一起。ExecutorCompletionService实现了CompletionSeervice,并将计算部分委托给一个Executor。

### 6.3.6 实例：使用CompletionService实现页面渲染器

### 6.3.7 为任务设置时限

### 6.3.8 示例：旅行预订门户网站

## 小结

Executor框架将任务提交与执行策略解耦开来，同时还支持多种不同类型的执行策略。
要想将应用程序分解为不同的任务时获得最大的好处，必须定义清晰的任务边界。