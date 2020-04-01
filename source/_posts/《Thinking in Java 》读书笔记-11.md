---
title: 《Thinking in Java 》读书笔记-11
tags: [java]
categories:
- 读书笔记
---

# 第11章 持有对象


----------


 
## 11.1 泛型和类型安全的容器

## 1.2 基本概念

- Collection
- Map
- List

## 1.3添加一组元素 

构建一个不包含元素的Collection，然后调用Collection.addAll()这种方式，运行速度快，也比较方便，是首先方式。

使用Array.asList()可以使用**显式类型参数**说明产生的List的实际的目标类型。


## 1.4 容器的打印
	
使用Arrays.toString()来产生数组的可打印表示。

## 11.5 List

- 基本的ArrayList，它长于随机访问元素，但是在List的中间插入移动元素时较慢。

- LinkedList,它通过代价较低的在List中间进行的插入和删除操作，提供了优化的顺序访问。LinkedList在随机访问方面相对比较慢，但它的特性较ArrayList更大。
 
## 11.6 迭代器

Java的iterator()要求容器返回一个Iterator。Iterator将准备好返回序列的第一个元素。

 1. 使用方法iterator()要求容器返回一个Iterator.Iterator将准备好返回序列的第一个元素。
 2. 使用next()获得序列中的下一个元素。
 3. 使用hashNext()检查序列中是否还有元素。
 4. 使用remove()将迭代器新近返回的元素删除。
 
Iterator可以移除由next()产生的最后一个元素，这意味着在调用remove()之前必须先调用next()。

## LinkedList

LinkedList具有能够直接实现栈的所有功能的方法，因此可以直接将LinkedList作为栈使用。

peek()方法提供栈顶元素，但是并不将其从栈顶移除，而pop()将移除并返回栈顶元素。

## 11.9 Set

Set不保存重复的元素。

TreeSet将元素储存在红-黑树数据结构中，有序。

HashSet使用的是散列函数，无序。

LinkedHashList因为查询速度的原因使用散列，但是也使用了链表来维护元素的插入顺序。

## 11.10 Map
	
Map可以用keySet()返回它的键的Set，用valueSet()它的值的Collection。

- HashMap

- TreeMap

- LinkedHashMap

## 11.111 Queue

  队列是一个典型的先进先出的容器。

### 11.11.1 PriorityQueue

优先队列声明下一个弹出元素是最需要的元素（具有最高的优先级）。

## Collection和Iterator

Collection是描述所有序列容器的共性的跟接口，它可能被认为是一个“附属接口”，即因为要表示其他若干个接口的共性而出现的接口。

实现Collection就意味着需要提供iterator()方法。

Collection可以生成Iterator，而List可以生成ListIterator。

## 11.13 Foreach与迭代器

foreach语法主要用于数组和实现了Iterable接口的对象。

### 11.13.1 适配器方法惯用法

由Arrays.asList()产生的List对象会使用底层数组作为其物理实现。只要你执行的操作会修改这个List，并且你不想原来的数组被修改，那么你应该在另一个容器中创建一个副本。

## 11.14 总结

1. 数组将数字与对象联系起来。它保存类型明确的对象，查询对象时，不需要对结果做类型转换。他可以是多维的，可以保存基本类型的数组。但是，数组一旦生成，其容量就不能改变。

2. Collection保存单一的元素，而Map保存相关的键值对。有了Java的泛型，你就可可以指定容器中存放的对象类型，因此你就不会将错误类型的对象放置到容器中，并且自从容器中获取元素时，不必进行类型转换。各种Collection和各种Map都可以在你向其中添加更多的元素时，紫红调整其尺寸。容器不能持有基本类型，但是自动包装机制会仔细地执行基本类型到容器中所持有的包装其类型之间的双向转换。

3. 像数组一样，List也建立数字索引与对象的关联，因此，数组和List都是排好序的容器。List能够自动扩充容量。

4. 如果要进行大量的随机访问，就使用ArrayList；如果要经常从表中插入或删除元素，则应该使用LinkedList。

5. 各种Queue以及栈的行为，由LinkedList提供支持。

6. Map是一种将对象（而非数字）与对象相关联的设计。HashMap设计用来快速访问；而TreeMap保持“键”始终处于排序状态，所以没有HashMap快。LinkedHashMap保持元素插入的顺序，但是也通过散列提供快速访问能力。

7. Set不接受重复元素。HashSet提供组块的查询速度，而TreeSet保持元素处于排序状态，LinkedHashSet以插入顺序保存元素。

8. 新程序中不应该使用过时的Vector、Hashtable和Stack。


