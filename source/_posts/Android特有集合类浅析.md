---
title: Android特有集合类浅析
date: 2019-04-21 10:02:19
updated: 2019-04-21 10:02:25
categories:
- 数据结构
tags:
- Android
- 数据结构
- 集合
---

# Android特有集合类浅析

* ArrayMap
* ArraySet
* SparseArray
* SparseIntArray
* SparseBooleanArray
* SparseLongArray

## 设计这些Android特有的集合类的意义

为什么要设计这些Android特有的集合类呢?

主要目的： 优化内存占用。在移动设备端内存资源很珍贵，Java原本的集合类为实现快速查询带来了很大内存的浪费。

## ArrayMap

### 设计目的
    
代替HashMap<Object,Object>，且key唯一。

#### HashMap实现方式

我们先来简单回忆下HashMap，HashMap是用散列表（数组+链表）实现：

![HashMap](/assert/img/androidmap/hashmap.jpg)

#### HashMap碰撞冲突解决方式

HashMap用**链地址法**解决hashCode冲突，这就是链表的设计目的，碰撞冲突就用头插法在对应key的链表插入value。

顺便提一下解决碰撞冲突的常用四个方法：

1. 链地址法
2. 再Hash法
3. 开放地址法(线性探测,二次探测,伪随机探测)
4. 建立公共溢出区

#### HashMap扩容机制

HasMap默认初始容量16，加载因子0.75。
HasMap扩容机制：元素个数超过数组容量 *负载因子，扩大为2倍。

#### HashMap总结

HashMap的存取速度都是相对比较快的，在一般情况下都能实现**O(1)**的速度。但是代价是什么呢？
但是从初始容量和负载因子都可以看出，这种快速的读是通消耗更多内存来换取的(**空间换时间**)。

### 设计原理

对比上面回忆的HashMap特征来看ArrayMap。

#### ArrayMap实现方式

ArrayMap是用双数组实现：

一个int[]数组，用于保存每个item的**hashCode**. 
一个Object[]数组，保存key/value键值对。容量是上一个数组的**两倍**。

![ArrayMap](/assert/img/androidmap/arraymap.jpg)

#### ArrayMap解决碰撞冲突方式

ArrayMap会对key**从小到大排序**，使用**二分法查询**key对应在数组中的下标。
先对存储hashCode的int[]数组使用**二分查找法**得到相应的index，通过index换算，可以得到Object[]数组的**key下标（index*2）**和**value下标（index*2+1）**。

那么如果出现hashCode冲突，该怎么办呢？
ArrayMap采用自己的**开放地址法**来解决碰撞冲突。

以put一个键值对key/value为例子：

![ArrayMap进行put](/assert/img/androidmap/arraymap_put.jpg)

hashCode碰撞时会发生这样的情景， 存储hashCode的int[]数组**二分查找**后得到的index不为负数（因为如果该hashCode存储过数据，index必然大于等于0，也就是能在int[]数组中查找到值），且同时根据index换算后得到的key下标和value下标在Object[]数组也已存有数据。
ArrayMap会以当前换算好的key下标为**中心点**，**向后和向前遍历**，查询对比是否有**相同key值**。
相同则说明重复put了；向后和向前遍历都没有找到相同key值的，最后将在向后遍历的最后一个下标后插入当前key/value。

#### ArrayMap扩容机制

ArrayMap默认容量为0。

ArrayMap扩容机制：
1. 如果容量大于等于8，则扩容一半
2. 否则容量大于等于4，则扩容到8
3. 否则扩容到4
4. 扩容时只需要数组拷贝工作，不需要重建哈希表

为什么4是扩容默认的size， 因为4是相对效率较高的大小。

#### ArrayMap总结

从存储方式和扩容机制可以看出，ArrayMap将内存的使用率提高了很多，但是读取的复杂度却是**O(lgN)**（因为二分法查找）。在数据量不是很大（**千级以内**）的时候，使用ArrapMap可以优化内存，并且存取速度几乎是不受什么影响的。

尚未提及的一点是，ArrayMap在删除元素时，如果集合剩余元素少于一定阈值，还有**收缩（shrunk）**功能。（等以后研究一下后补上）

可以看出 ArrayMap 和 HashMap 大体上是时间和空间的权衡罢了。

## ArraySet

### 设计目的

代替HashSet,只保存value，value唯一

### 设计原理

和ArrayMap类似，不做展开。

## SparseArray

### 设计目的

代替HashMap<Integer,Object>，SparseArray只能在key为int的时候才能使用，注意是int而不是Integer。
这样做就是为了进步一优化ArrayMap，避免装箱拆箱的操作，从而节省内存和提高效率。

### 设计原理

原理和Arraymap基本一致，也是基于双数组和二分查找，这里就不展开。

## SparseIntArray

### 设计目的

代替HashMap<Integer,Integer>，进一步优化SparseArray，避免value为Integer的时候的装箱拆箱操作。

## SparseBooleanArray

### 设计目的

代替HashMap<Integer,Boolean>，进一步优化SparseArray，避免value为Boolean的时候的装箱拆箱操作。

## SparseLongArray

### 设计目的

代替HashMap<Integer,Long>，进一步优化SparseArray，避免value为Long的时候的装箱拆箱操作。

