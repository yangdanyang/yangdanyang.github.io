---
layout:     post
title:      "常见问题总结"
subtitle:   " \"Question\""
date:       2017-06-08 
author:     "Yang"
header-img: "img/post-bg-alitrip.jpg"
catalog: true
tags:
    - Design Pattern
---

> “on road”
## 常见问题

### Spring单例Bean和Java单例模式的区别
    Spring的的单例是基于BeanFactory也就是spring容器，单例Bean在此Spring容器内是单个的，Java的
    单例是基于JVM，每个JVM内一个单例。

### 同步机制比较:ThreadLocal和线程同步机制相比有什么优势呢？
    ThreadLocal和线程同步机制都是为了解决多线程中相同变量的访问冲突问题。
    在同步机制中，通过对象的锁机制保证同一时间只有一个线程访问变量。这时该变量是多个线程共享的，使用同
    步机制要求程序慎密地分析什么时候对变量进行读写，什么时候需要锁定某个对象，什么时候释放对象锁等繁杂
    的问题，程序设计和编写难度相对较大。
    而ThreadLocal则从另一个角度来解决多线程的并发访问。ThreadLocal会为每一个线程提供一个独立的变
    量副本，从而隔离了多个线程对数据的访问冲突。因为每一个线程都拥有自己的变量副本，从而也就没有必要对
    该变量进行同步了。ThreadLocal提供了线程安全的共享对象，在编写多线程代码时，可以把不安全的变量封
    装进ThreadLocal。 
    概括起来说，对于多线程资源共享的问题，同步机制采用了“以时间换空间”的方式，而ThreadLocal采用
    了“以空间换时间”的方式。前者仅提供一份变量，让不同的线程排队访问，而后者为每一个线程都提供了一份
    变量，因此可以同时访问而互不影响。 










