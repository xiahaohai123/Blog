# java 中 map 存储 null 键值对记录

# 前言

做项目时遇到需要向 map 存入空键值对的情况，但是有些 map 的实现类是不允许存入 null 作为键值对的，在此记录一下。

## 多线程场景

多线程场景一般不允许键值对中存在 null，例如 `ConcurrentHashMap` 或 `HashTable`。

将 null 作为键值对会引发二义性问题。

主要是值为 null 时引发的二义性，当我们调用 get 方法时，返回值为 null 有以下两种可能:

1. 值没有在集合中，所以返回的结果就是 null。
2. 键值对关系存在于集合中，而该键值对中的值为 null，所以返回的结果是 null。

这种二义性在多线程场景下会出现很多让人误解的现象，而且很难排查。

模拟一下如下场景:

1. 线程 T1 打算对一个 ConcurrentHashMap 进行 `get("key")`
2. 线程 T2 在 T1 执行操作前，执行了 `remove("key")`。
3. 线程 T1 执行了 `get("key")` 后就可以认定这是因为 Map 里面没有对应的键值对关系，而不是因为 `key` 对应的值是 null。

消除这种场景上的二义性可以防止很多意外的问题发生。

## 单线程场景

单线程场景一般都是允许将 null 作为键值对的，因为单线程不存在多线程上这些场景的问题。

典型的实现类就是 `HashMap`。

## 参考文献

1. [【并发编程】为什么Hashtable和ConcurrentHashMap 是不允许键或值为 null 的，HashMap 的键值则都可以为 null？](https://blog.csdn.net/cy973071263/article/details/126354336)