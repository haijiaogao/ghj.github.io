---
title: Difference between ConcurrentHashMap and HashMap
tags: Java
categories: Java
date: 2020-11-06 10:54:17
---



## ConcorrentHashMap和HashMap的区别

1. ConcorrentHashMap 是线程安全(写入数据的时候加锁，读取不加锁)的，而HashMap是非线程安全的（写入和读取都没有加同步）。
2. ConcorrentHashMap 和 HashTable 类似，两者都是线程安全，而且不允许null key和null value，如果遇到会抛出NullPointerException异常，HashMap是可以写入空的key和Value的
但是HashTable的缺点是，它的写入和读取操作加锁（整表锁住），所以多线程的共享的情况下，效率会比较低。实际上应用比较少，但是因为历史问题（java legency 用到），所以予以保留，ConcorrentHashMap相当于是其升级版本。
ConcorrentHashMap优势：
    1) 读取不加锁。
    2) 写入的时候加锁但是不是锁住整表，只是锁住变化的部分。
3. ConcorrentHashMap在遍历的时候删除，不会抛出ConcorrentModifierException异常，HashMap会抛出异常。

## HashMap 多线程同步的实现方式

Java的Collections中提供了Collections.synchronizedMap()方法，可以对不安全的Map，进行包装，同理，也可以通过其他的Collections.synchronizedAPI().对非线程安全的容器进行同步包装，实现同步。


## HashMap&TreeMap&LinkedHashMap&HashTable对比

![HashMap&TreeMap&LinkedHashMap&HashTable对比](images/sDoih.png)


