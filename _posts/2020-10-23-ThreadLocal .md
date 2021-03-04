---
layout: post
title:  "ThreadLocal"
date:   2020-10-23 10:13:34
author: zhangtejun
categories: Java
---
##### ThreadLocal
线程局部变量是局限于线程内部的变量，属于线程自身所有，不在多个线程间共享。Java提供ThreadLocal类来支持线程局部变量，是一种实现线程安全的方式。
但是在管理环境下（如 web 服务器）使用线程局部变量的时候要特别小心，在这种情况下，工作线程的生命周期比任何应用变量的生命周期都要长。
任何线程局部变量一旦在工作完成后没有释放，Java 应用就存在内存泄露的风险。

**ThreadLocal解决了什么问题？
* 保证线程安全，使用空间换时间，各个线程都持有一个变量的副本
* 数据存储问题，各个线程都持有一个变量的副本，每个线程可以访问自己内部的变量副本。

（1）每个Thread维护着一个ThreadLocalMap的引用

（2）ThreadLocalMap是ThreadLocal的内部类，用Entry来进行存储

（3）ThreadLocal创建的副本是存储在自己的threadLocals中的，也就是自己的ThreadLocalMap。

（4）ThreadLocalMap的键值为ThreadLocal对象，而且可以有多个threadLocal变量，因此保存在map中

（5）在进行get之前，必须先set，否则会报空指针异常，当然也可以初始化一个，但是必须重写initialValue()方法。

（6）ThreadLocal本身并不存储值，它只是作为一个key来让线程从ThreadLocalMap获取value。


##### ThreadLocal可以让我们拥有当前线程的变量，那这个作用有什么用呢 
* 管理Connection
  ThreadLocal能够实现当前线程的操作都是用同一个Connection，保证了事务！
  
* 避免一些参数传递


ThreadLocal内存泄漏的根源是：
* 由于ThreadLocalMap的生命周期跟Thread一样长，如果没有手动删除对应key就会导致内存泄漏，而不是因为弱引用。Entry的key(ThreadLocal) 被回收，value还存在。
* ThreadLocal在没有外部强引用时，发生GC时会被回收，如果创建ThreadLocal的线程一直持续运行，那么这个Entry对象中的value就有可能一直得不到回收，发生内存泄露。
* 比如线程池里面的线程，线程都是复用的，那么之前的线程实例处理完之后，出于复用的目的线程依然存活，所以，ThreadLocal设定的value值被持有，导致内存泄露。
Entry它并未实现Map接口，而且他的Entry是继承WeakReference（弱引用）的，也没有看到HashMap中的next，所以不存在链表了

那为什么ThreadLocalMap的key要设计成弱引用？

key不设置成弱引用的话就会造成和entry中value一样内存泄漏的场景。如果 key 是强引用，那么发生 GC 时 ThreadLocalMap 还持有 ThreadLocal 的强引用，
会导致 ThreadLocal 不会被回收，从而导致内存泄漏。

如果不是弱引用，而且用户已经不再持有这个ThreadLocal的引用并且没有调用<code>remove</code>方法，那么只要线程还在，<code>ThreadLocal</code>和<code>数据</code>就会一直被引用无法回收，就是内存泄漏了</li>


弱引用的定义是：
* 如果一个对象仅被一个弱引用指向，那么当下一次GC到来时，这个对象一定会被垃圾回收器回收掉。

[参考](https://zhuanlan.zhihu.com/p/139214244)

ThreadLocal对象存放在哪里么？

在Java中，栈内存归属于单个线程，每个线程都会有一个栈内存，其存储的变量只能在其所属线程中可见，即栈内存可以理解成线程的私有内存，而堆内存中的对象对所有线程可见，堆内存中的对象可以被所有线程访问。

##### InheritableThreadLocal
ThreadLocal设计之初就是为了绑定当前线程，如果希望当前线程的ThreadLocal能够被子线程使用，可以使用InheritableThreadLocal。

InheritableThreadLocal是复制的对象引用，所以主线程和子线程其实都引用的同一个对象，存在线程安全的问题，
只需要重写java.lang.InheritableThreadLocal#childValue方法，可以实现对象值的复制。

如果子线程取的值的同时，主线程进行了值的修改，子线程取到的仍是旧值。

