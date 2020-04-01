---
title: 《Java并发编程实战》-4
tags: [java]
categories:
- 读书笔记
---

# 《Java并发编程实战》-4

## 5.1 同步容器类

Collections.synchronizedXxx等工厂方法创建的同步实现线程安全的方式是：将它们的状态封装起来，并对每个公有方法都进行同步，使得每次只有一个线程能访问容器的状态。

### 5.5.1 同步容器的问题

在使用客户端加锁的Vector上的复合操作

```java
public static Object getList(Vector list) {
    synchronized (list) {
        int lastIndex = list.size() - 1;
        return list.get(lastIndex);
    }
}

public static void deleteLast(Vector list) {
    synchronized (this) {
        int lastIndex = list.size() - 1;
        list.remove(lastIndex);
    }
}
```

### 5.1.2 迭代器与ConcurrentModificationException

当容器类发现容器在迭代过程中被修改时，就会抛出一个"ConcurrentModificationException
"，这就是“快速失败”。

### 5.1.3 隐藏迭代器

正如封装对象的状态有助于维持不变性条件，封装对象的同步机制同样有助确保实施同步策略。

## 5.2 并发容器

通过并发容器来代替同步容器，可以极大地提高伸缩性并降低风险。

并发容器 | 同步容器 | 普通容器 |
-|-|-|
BlockingQueue | ConcurrentLinkedQueue | PriorityQueue |
ConcurrentHashMap, SkipListMap  | hashtable,  Collections.synchroniedList(TreeSet set)| hashmap,SortedMap, SortedSet |
CopyOnWriteArrayList | Vector, Collections.synchroniedList(List list) | ArrayList, LinkedList |
CopyOnWriteArraySet, ConcurrentSkipListListSet|  Collections.synchroniedSet(Set set) | LinkedHashSet, HashSet, TreeSet |

### 5.2.1 ConcurrentHashMap

ConcurrentHashMap使用更细粒度的加锁机制---**分段锁**来实现更大程度的共享。

ConcurrentHashMap具有**弱一致性**(Weakly Consistent)，而非"及时失败"。

只有当应用程序需要加锁Map以进行独占访问时，才应该放弃使用ConcurrentHashMap。

### 5.2.2 额外的原子Map操作

```java
public interface ConcurrentMap<K,V> extends Map<K,V> {
    // 仅当K没有相应的映射值时才插入
    V putIfAbsent(K key, V value);

    // 仅当K被映射到oldValue时才替换为newValue
    boolean remove(K key, V value);

    // 仅当K被映射到oldValue时才替换为newValue
    boolean replace(K key, V oldValue, V newValue);

    // 仅当K被映射到某个值时才替换为newValue
    boolean replace(K key, V newValue);
}
```

### 5.2.3 CopyOnWriteArrayList

CopyOnWriteArrayLsit用于替代同步List，某些情况下它提供了更好的并发行性能，并且在迭代期间不需要对容器进行加锁或复制（类似地，CopyOnWriteArraySet的作用是替代同步Set）。

每当修改容器时都会复制底层数组，这需要一定的开销。

仅当迭代操作远远多于修改操作时，才应该使用"写入时负责"容器。

## 5.3 阻塞队列和生产者-消费者模式

在构建高可靠的应用程序时，有界队列是一种强大的资源管理工具:它们能抑制并防止产生过多的工作项，使应用程序在负荷过载的情况下变得更加健壮。

BlockingQueue的方法:

* put 如果队列满则阻塞
* take 如果队列为空则阻塞
* offer 如果队列满则返回一个失败状态，一般是false
* poll 如果队列空则返回一个表示空的数据,一般是null

BlockingQueue的多种实现:

* **LinkedBlockingQueue** 基于链表，FIFO
* **ArrayBlockingQueue** 基于数组, FIFO
* **PriorityBlockingQueue** 按优先级排序的队列
* **SynchronousQueue** 它不会为队列中元素维护储存空间，它维护一组线程，这些线程在等待着吧元素加入或者移出队列。

### 5.3.1 示例:桌面搜索

### 5.3.2 串行线程封闭

对象池利用了串行线程封闭，将一个对象“借给”一个请求线程。

还可以通过ConcurrentMap的原子方法remove或者AtomicReference的原子方法compareAndSet来转笔可变对象的所有的所有权（但必须确保只有一个线程能接受被转移的对象）。

### 双端队列与工作密取

Deque是一个双端队列，实现了在队头和队尾的高效插入和移除。具体实现包括ArrayList和LinkedBlockingDeque。

双端队列适用于工作密取模式。

## 5.4 阻塞方法与中断方法

* 传递
* 恢复中断

```java
public class TackRunnable implements Runnable {
    try {
            processTack(queue.take());
        } catch (InterruptedException e) {
            // 恢复被中断的状态
            Thread.currentThread().interrupt();
        }
}

```

## 5.5 同步工具类

### 5.5.1 闭锁

CoutDwonLatch，倒计时门闩,又名闭锁、倒计时计数器，是一种同步工具类，可以延迟线程大的进度直到其到达终止状态。

闭锁可以用来确保某些活动直到其他活动都完成后才能继续执行。

* 确保某个计算在其需要得所有资源都被初始化之后才继续执行。二元闭锁（包括两个状态）可以用来表示“资源R已经被初始化”，而所有需要R的操作都必须现在这个闭锁上等待。
* 确保某个服务在其依赖的所有其他服务都已经启动之后才启动。每个服务都有一个相关的二元闭锁。当启动服务S时，将首先在S依赖的其他服务的闭锁上等待，在所有依赖的服务启动后会释放锁S，这样其他依赖S的服务才能继续执行。
* 等待直到某个操作的所有参与者（例如，在多个玩家游戏中的所有玩家）都就绪再继续执行。这种情况中，当所有玩家都准备就绪时，闭锁将到达结束状态。

### 5.5.2 FutureTask

FutureTask表示的计算是通过Callable来实现的，相当于一种可生成结果的Runnable，并且可以处于一下三种状态：

* 等待运行(Wating to run)
* 正在运行(Running)
* 运行完成(Completed)

### 信号量

计数信号量用来控制同时访问某个某个特殊资源的操作数量，或者同时执行某个指定操作的数量。计算信号量还可以用来实现某种资源池，或者对容器施加边界。

### 5.5.4 栅栏

栅栏类似于闭锁，他能阻塞一组线程直到某个事件发生。栅栏与闭锁的关键区别在于，所有线程必须同时到达栅栏位置，才能继续执行。闭锁用于等待事件，而栅栏用于等待其他线程线程。

CyclicBarrier可以使一定数量的参与反复地在栅栏位置汇集，他在并行迭代算法中非常有用：这种算法通常将一个问题拆分成一系列相互独立的子问题。

另一种形式的栅栏使Exchange，他是一种两方栅栏，各方在栅栏位置上交换数据。当两分法执行不对称的操作时，Exchanger会非常有用，例如当一个线程向缓冲区写入数据，而另一个线程从缓冲区读取数据。

## 5.6 构建高效且可伸缩的结果缓存

使用ConcurrentHashMap和FutureTask来构架缓存

```java

public interface Computable<A, V> {
    V compute(A arg) throws InterruptedExcetion;
}

public class Menoizer<A, V> implements Computable<A, V> {
    private final ConcurrentMap<A, Future<V>> cache = new ConcurrentHashMap<>();
    private final Computable<A, V> c;

    public Menoizer(Computable<A, V> c) { this.c = c; }
    while (true) {
        Future<V> f = cache.get(arg);
        if (f == null) {
            Callable<V> eval = new Callable<V>() {
                public V call() throws InterruptedException {
                    return c.compute(args);
                }
            };
            FutureTask<V> ft = new FutureTask<V>(eval);
            // 复合操作（“若没有则添加”）
            f = cache.putIfAbsent(arg, ft);
            if(f == null) {
                f = ft;
                ft.run();
            }
        }
        try{
            // 若正在计算，则阻塞等待结果
            return f.get();
        } catch (CancellationException e) {
            // 出现异常则去除缓存，防止缓存污染
            cache.renove(arg,f);
        } catch (ExecutionException e) {
            throw launderThrowable(w.getCause())
        }
    }
}

```

## 第一部分小结

* 可变状态是至关重要的。
    所有的并发问题都可以归结为如何协调对并发状态的访问，可变状态越少，就越容易确保线程安全。
* 尽量将域声明为final类型，除非需要它们是可变的。
* 不可变对象一定是线程安全的。
    不可变对象能极大地降低并发编程的复杂性。它们更为简单而且安全，可以任意共享而无须使用加锁或保护性复制等机制。
* 封装有助于管理复杂性
    在编写线程安全的程序时，虽然可以将所有数据都保存在全局变量中，但为什么要这么做？将数据封装在对象中，更易于维持不变性条件：将同步机制封装在对象中，更易于遵循同步策略。
* 用锁来保护每个可变变量。
* 当保护同一个不变性条件中的所有变量时，要使用用一个锁。
* 在执行复合操作期间，要持有锁。
* 如果从多个线程中访问同一个可变变量没有同步机制，那么程序会出现问题。
* 不要故作聪明地推断出不需要使用同步。
* 在设计过程中考虑线程安全，或者在文档中明确地指出它不是线程安全的。
* 将同步策略文档化。
