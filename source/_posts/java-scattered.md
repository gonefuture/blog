---
title: JAVA scatter （转载自zooooooooy）
---

查看系统的线程上下文切换次数

使用vmstat 3
![vmstat](/images/TIM截图20180704163235.png)

其中cs就是context switch的次数，数值越高代表上线文切换的越频繁，耗时越多。每秒的切换次数可以和进程的总线程数进行比对，如果大于线程总数代表是满负荷运转

_ _ _

java 线程状态之间的跃迁
![thread change](/images/TIM截图20180704193005.png)

线程的各个状态及说明如下：

| 状态名称 | 说明 |
|:--------|--------|
|    NEW    |     初始状态   |
|    RUNNABLE |  运行状态      |
|    BLOCKED    | 阻塞状态       |
|    WAITING    |等待状态|
|    TIME_WAITING    |  超时等待状态      |
|    TERMINATED    |终止状态        |

_ _ _

synchronized 关键字的使用场景

|锁的方式|锁的对象|伪代码|
|:--------|--------|--------|
|实例方法|类的实例对象|public synchronized void method|
|静态方法|类对象|public synchronized static void method|
|实例对象|类的实例对象|synchronized (this)|
|class对象|类对象|synchronized (A.class)|
|普通对象|普通对象|synchronized (o)|

_ _ _

mark 下 通过ReentrantLock中线程等待唤醒的状态跃迁
![await|singal](/images/TIM截图20180720114120.png)

_ _ _

java queue 方法

| 方法名称 | 说明 | 是否阻塞 | 是否抛出异常 |
|:--------|--------|--------|--------|
|add|插入|否|是|
|offer|插入|否|是|
|remove|删除|否|否|
|poll|删除|否|否|
|element|获取|否|是|
|peek|获取|否|是|

blocking queue

| 方法名称 | 说明 | 是否阻塞 | 是否抛出异常 |
|:--------|--------|--------|--------|
|put|插入|是|否|
|offer *支持超时* |插入|是|否|
|take|删除|是|否|
|poll *支持超时* |删除|是|否|

