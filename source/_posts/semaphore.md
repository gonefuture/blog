---
title: Semaphore （转载自zooooooooy）
---

是一个分离的信号量计数器
主要核心方法有两个：
*	acquire
*从信号队列里面获取一个信号量，如果没有获取到则会一直阻塞到获取到信号量为止*
*	release
*释放一个信号量回队列，这种模型相对来说比较适合池这种类型的业务需求*

acquire(int permits) 一次获取多个信号量
acquireUninterruptibly() 连续获取一个信号量
acquireUninterruptibly(int permits) 连续获得多个信号量





