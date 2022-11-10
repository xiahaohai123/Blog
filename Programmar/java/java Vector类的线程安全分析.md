---
title: java Vector类的线程安全分析
date: 2021-08-17 16:14:30.625
updated: 2021-08-17 16:35:06.616
url: http://summersea.top:8090/archives/javavector类的线程安全分析
categories: 
tags: 线程安全 | Java
---

#### 参考文献
- [Vector 是线程安全的？](https://blog.csdn.net/xdonx/article/details/9465489)
- [Java_Thread_线程安全性的5种级别](https://blog.csdn.net/mikyz/article/details/69397904)

在学习的时候突然看到网上说Vector并不是线程安全的，但是按我已知的只是来看，Vector应当是线程安全的，但是不建议用。当时我并没有深究为什么不建议用Vector类，如今看到有人说该类并不是线程安全的，顿时引起了我的兴趣。
先说结论，Vector是否线程安全要看我们对线程安全的认知，如果**只论是否线程安全**，那答案是**否**，因为**Vector在复合操作上不具备原子性**，可能会导致意外发生。而按线程安全五个级别来划分，Vector属于有条件的线程安全，在复合操作时需要**我们额外上一把对象锁**才能保证线程安全。而额外上一把对象锁时，使用Vector和使用ArrayList其实并没有区别，所以不推荐使用。



## 线程安全的五大级别

- 过去在线程安全的判断模型上，我一直采用的两极对立的模型，即线程安全与非线程安全，如今发现了一个五级模型，该模型能更精确的按程度来判定线程安全级别。
- 该模型的指导意义比两极对立模型要大，能让我们深入了解线程安全问题面临的几大场景。

> 1. 不可变
不变的对象绝对是线程安全的，不需要线程同步，如String、Long、BigInteger
> 2. 无条件的线程安全
对象自身做了足够的内部同步，也不需要外部同步，如Random、ConcurrentHashMap、Concurrent集合、atomic
> 3. 有条件的线程安全
对象的部分方法可以无条件安全使用，但是有些方法需要外部同步，需要Collections.synchronized；有条件线程安全的最常见的例子是遍历由 Hashtable 或者 Vector 或者返回的迭代器。
> 4. 非线程安全(线程兼容)
对象本身不提供线程安全机制，但是通过外部同步，可以在并发环境使用， 如ArrayList、HashMap。
> 5. 线程对立
即使外部进行了同步调用，也不能保证线程安全，这种情况非常少，如如System.setOut()、System.runFinalizersOnExit()。




## 单一操作

以Vector举例，请问如下代码在多线程环境下是否会出错？
```java
vector.add(element);
```
答案是不会，因为Vector的add(E e)方法有一把大大的锁，我贴一个java源码如下：
```java
public synchronized boolean add(E e) {
    modCount++;
    add(e, elementData, elementCount);
    return true;
}
```
当多个线程并发执行add方法的时候，还是需要获取对象锁来进行增加操作，所以在该场景下Vector的add操作不会出问题。



## 复合操作

复合操作指的是调用方在一个代码区内依次调用该类的多个方法，而该类的数据应该按调用者的预期进行变化。
还是以Vector举例，如下代码在多线程环境下是否有概率出错？
```java
if(!vector.contains(element)){
    vector.add(element);
}
```

这是典型的`put-if-absent`场景，按次序执行了vector的两个方法。在该场景下这个代码是有可能出现不符合预期的行为的。
首先我们看看这份代码做了什么：
1. 判断元素是否存在
2. 如果元素不存在则新增此元素


我们来看看异常场景。假如此时有两个线程`thread1`与`thread2`，它们并发执行了该代码段且两个线程新增的`element`元素相同，然后出现了以下情况。

1. `thread1`执行了vector.contains(element)，得到结果`true`。
2. `thread1`时间片用完，轮到`thread2`执行方法。
3. `thread2`执行了vector.contains(element)，得到结果`true`且继续执行了vector.add(element)，vector此时新增了`element`。
4. `thread2`执行完方法后`thread1`继续执行方法，完成了vector.add(element)的执行。


最终我们会发现，vector里出现了两个相同的element，如果这个代码的逻辑就是`put-if-absent`，那么该结果并不符合预期。
所以可以得出结论，**Vector在复合操作的时候并不能保证线程安全，反而因为不停的上锁释放锁增加无用的开销，** 而`put-if-absent`场景只是众多场景中的一种。Vector在没有线程安全问题的环境中不如ArrayList，在有线程安全的问题中也不能完全保证线程安全，两头不讨好。

最终结论是为了减少意外，建议不要使用Vector类。

