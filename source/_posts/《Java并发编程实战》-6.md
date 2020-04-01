---
title: 《Java并发编程实战》-6
tags: [java]
categories:
- 读书笔记
---

# 取消与关闭

---

行为良好的软件能够完善地处理失败,关闭和取消等过程。

## 7.1任务取消

取消某个操作的原因:

* 用户请求取消
* 有时间限制的操作
* 应用程序时间
* 错误
* 关闭

### 7.1.1 中断

在Java的API或语言规范中，并没有将中断与任何取消语言关联起来，但实际是激昂，如果在取消之外的其他操作中使用中断，那么都是不合适的，并且很难支撑起更大的应用。

``` java
public class Thread {
    // 中断目标线程
    public void  interupt() { ... }
    // 返回目标线程的中断状态
    public boolean isInterrupted() { ...}
    // 静态， 将清楚当前线程的中断状态，并返回它之前的值，这也是清楚中断状态的唯一方法
    public static boolean interrupted() { ... }
}
```

非阻塞状态下中断时，它的中断状态将被设置，中断操作将变得“有粘性” -- 如果不触发InterruptedEXception，那么中断状态将一直保持，直到明确地清除中断状态。

>调用interrupt并不意味着立刻停止目标线程正在进行的工作，而只是传递了请求中断的消息。中断操作并不会真正地中断一个正在运行的线程，而只是发出中断请求，然后由线程在下一个合适的时刻中断自己（这些时刻也被称为取消点）。

通常，中断是实现取消的最合理方式。

通过中断来取消。

``` java
class PrimeProducer extends Thread {
    private final BlockingQueue<BigInteger> queue;

    PrimeProducer(BlockingQueue<BigInteger> queue) {
        this.queue = queue;
    }

    public void run() {
        BigInteger p = BigInteger.ONE;
        while (!Thread.currentThread().isInterrupted()) {
            queue.put( p = p.nextProbablePrime());
        } catch (InterruptedException consumed) {
            /* 允许线程推出 */
        }
    }

    public void cancel() { interrupt(); }

}
```

### 7.1.2 响应中断

两种实用策略可以用于处理InterruptedException

* 传递异常 (可能在执行某个特定于任务的清除操作之后)，从而使你的方法也可以成为可中断的阻塞方法。
* 恢复中断状态，从而使调用栈中的上层代码能够对其进行处理。

>只有实现了线程中断策略的代码才可以屏蔽中断请求，在常规的任务和库代码中都不应该屏蔽中断请求。

不可取消的任务在推出前恢复中断

``` java
public Task getNextTask(BlockingQueue<Task> queue) {
    boolean interrupted = false;
    try {
        try {
            return queue.take();
        } catch (InterruptedException e) {
            interrupted = ture;
        }
    } finally {
        if (interrupted) 
            Thread.currentThread().interrupt();
    }
}
```

### 7.1.4 示例：计时器运行


### 7.1.5 通过Future来实现取消

>当Future.get()抛出InterruptedException或TimeoutException时，如果你知道不再需要结果，那么就可以调用Future.cancel来取消任务。

### 7.1.6 处理不可中断的阻塞

* Java.io包中的同步Socket I/O。
* java.io包的同步I/O。
* Selector的异步I/O。
* 获取某个锁。

### 7.1.7 采用newTaskFor来封装非标准的取消

## 7.2 停止基于线程的服务

>对于持有线程的服务，只要服务的存在时间大于创建线程的方法的存在时间，那么就应该提供生命周期的方法。

### 7.2.1 示例：日志服务

### 7.2.2 关闭ExecutorService

### 7.2.3 “毒丸”对象

### 7.2.4 示例：只执行一次的服务

### 7.2.5 shutdownNow的局限性

## 7.3 处理非正常的线程终止

### 未捕获异常的处理

将异常写入日志的UncaughtExceptionHandler

``` java
public class UEHLogger implements THread.UncaughtExceptionHandler {
    public void uncaughtException(Thread t, Throwable e) {
        Logger logger = Logger.getAnonymousLogger();
        logger.log(Level.SEVERE, 
        "Thread terminated with exception: " + t.getName(), e);
    }
}
```

>在运行时间较长的应用程序中，通常会为所有线程的未捕获异常指定同一个异常处理器，并且该处理器至少会将异常信息记录到日志中。

## 7.4 JVM关闭

### 7.4.1 关闭钩子

### 7.4.2 守护线程

>此外，守护线程通常不能用来代替应用程序管理中各个服务的生命周期。

### 7.4.3 终结器

> 避免使用终结器。

## 小结

在任务、线程、服务以及应用程序等模块中的生命周期结束问题，可能会增加它们在设计和实现时的负责性。Java并没有提供某种抢占式的机制来取消操作或者终结线程。相反，它提供了一种协作式的中断机制来实现取消操作，但这要依赖于如何构建取消操作的协议，以及能否始终遵循这些协议。通过使用FutureTask和Executor框架，可以帮助我们构建可取消的任务和服务。



