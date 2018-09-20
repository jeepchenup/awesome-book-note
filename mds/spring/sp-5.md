# Spring 如何保证 Controller 并发的安全性？

> 前段时间就在看《Spring IN ACTION》这本书，然而看到一半，觉得得需要深入了解 Java 线程才行。于是花了个把月终于现在兜兜转转又回来研究 Spring 了。

### 1. 为什么会出现非线程不安全的现象？

对于这个问题，我们首先要知道在 Spring 中，其创建的 bean 对象默认情况下都是单例的。所以相对的 Controller、Service、DAO这些层级中的 bean 如果不显示指定其他作用范围，其创建的 bean 对象都是单例的。

非线程不安全的本质就是在多线程下，对线程对共享参数或者共享变量的修改导致结果不一致的现象。

### 2. 如何解决？

其实本文问题的关键点是如何解决资源竞争的问题。

当然如果 Controller 中不存在共享参数或者只是对共享参数进行读取是不会出现非线程安全的问题。

如果 Controller 中存在共享参数，可以有以下几个方法来避免：

1.  将 Controller 的 bean 范围设置成 `Prototype`。
    
    Spring Bean 的作用范围：
    
    1.  Singleton（默认），每次注入都是同一个对象。
    1.  Prototype，每次注入都是新创建的对象。
    1.  Session，在 Web 应用中，为每个会话创建一个 bean 实例。
    1.  Request，在 Web 应用中，为每次请求创建一个 bean 实例。

    了解过 bean 的作用范围，答案就已经出来了，就是设置 Controller 的作用范围为 `Prototype` 就可以了。

2.  ThreadLocal 变量来隔绝共享参数的出现。不熟悉 ThreadLocal 可以来一下这篇文章 -> [ThreadLocal](../../mds/concurrency/c-6.md)。

##  [BACK](../../mds/summary.md)