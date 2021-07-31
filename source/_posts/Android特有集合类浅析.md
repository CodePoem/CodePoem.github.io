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

- ArrayMap
- ArraySet
- SparseArray
- SparseIntArray
- SparseBooleanArray
- SparseLongArray

## 设计这些 Android 特有的集合类的意义

为什么要设计这些 Android 特有的集合类呢?

主要目的： 优化内存占用。在移动设备端内存资源很珍贵，Java 原本的集合类为实现快速查询带来了很大内存的浪费。

## ArrayMap

### 设计目的

代替 HashMap<Object,Object>，且 key 唯一。

#### HashMap 实现方式

我们先来简单回忆下 HashMap，HashMap 是用散列表（数组+链表）实现：

![HashMap](/assert/img/androidmap/hashmap.jpg)

#### HashMap 碰撞冲突解决方式

HashMap 用**链地址法**解决 hashCode 冲突，这就是链表的设计目的，碰撞冲突就用头插法在对应 key 的链表插入 value。

顺便提一下解决碰撞冲突的常用四个方法：

1. 链地址法
2. 再 Hash 法
3. 开放地址法(线性探测,二次探测,伪随机探测)
4. 建立公共溢出区

#### HashMap 扩容机制

HasMap 默认初始容量 16，加载因子 0.75。
HasMap 扩容机制：元素个数超过数组容量 \*负载因子，扩大为 2 倍。

#### HashMap 总结

HashMap 的存取速度都是相对比较快的，在一般情况下都能实现**O(1)**的速度。但是代价是什么呢？
但是从初始容量和负载因子都可以看出，这种快速的读是通消耗更多内存来换取的(**空间换时间**)。

### 设计原理

对比上面回忆的 HashMap 特征来看 ArrayMap。

#### ArrayMap 实现方式

ArrayMap 是用双数组实现：

一个 int[]数组，用于保存每个 item 的**hashCode**.
一个 Object[]数组，保存 key/value 键值对。容量是上一个数组的**两倍**。

![ArrayMap](/assert/img/androidmap/arraymap.jpg)

#### ArrayMap 解决碰撞冲突方式

ArrayMap 会对 key**从小到大排序**，使用**二分法查询**key 对应在数组中的下标。
先对存储 hashCode 的 int[]数组使用**二分查找法**得到相应的 index，通过 index 换算，可以得到 Object[]数组的**key 下标（index\*2）**和**value 下标（index\*2+1）**。

那么如果出现 hashCode 冲突，该怎么办呢？
ArrayMap 采用自己的**开放地址法**来解决碰撞冲突。

以 put 一个键值对 key/value 为例子：

![ArrayMap进行put](/assert/img/androidmap/arraymap_put.jpg)

hashCode 碰撞时会发生这样的情景， 存储 hashCode 的 int[]数组**二分查找**后得到的 index 不为负数（因为如果该 hashCode 存储过数据，index 必然大于等于 0，也就是能在 int[]数组中查找到值），且同时根据 index 换算后得到的 key 下标和 value 下标在 Object[]数组也已存有数据。
ArrayMap 会以当前换算好的 key 下标为**中心点**，**向后和向前遍历**，查询对比是否有**相同 key 值**。
相同则说明重复 put 了；向后和向前遍历都没有找到相同 key 值的，最后将在向后遍历的最后一个下标后插入当前 key/value。

#### ArrayMap 扩容机制

ArrayMap 默认容量为 0。

ArrayMap 扩容机制：

1. 如果容量大于等于 8，则扩容一半
2. 否则容量大于等于 4，则扩容到 8
3. 否则扩容到 4
4. 扩容时只需要数组拷贝工作，不需要重建哈希表

为什么 4 是扩容默认的 size， 因为 4 是相对效率较高的大小。

#### ArrayMap 总结

从存储方式和扩容机制可以看出，ArrayMap 将内存的使用率提高了很多，但是读取的复杂度却是**O(lgN)**（因为二分法查找）。在数据量不是很大（**千级以内**）的时候，使用 ArrapMap 可以优化内存，并且存取速度几乎是不受什么影响的。

尚未提及的一点是，ArrayMap 在删除元素时，如果集合剩余元素少于一定阈值，还有**收缩（shrunk）**功能。（等以后研究一下后补上）

可以看出 ArrayMap 和 HashMap 大体上是时间和空间的权衡罢了。

## ArraySet

### 设计目的

代替 HashSet,只保存 value，value 唯一

### 设计原理

和 ArrayMap 类似，不做展开。

## SparseArray

### 设计目的

代替 HashMap<Integer,Object>，SparseArray 只能在 key 为 int 的时候才能使用，注意是 int 而不是 Integer。
这样做就是为了进步一优化 ArrayMap，避免装箱拆箱的操作，从而节省内存和提高效率。

### 设计原理

原理和 Arraymap 基本一致，也是基于双数组和二分查找，这里就不展开。

## SparseIntArray

### 设计目的

代替 HashMap<Integer,Integer>，进一步优化 SparseArray，避免 value 为 Integer 的时候的装箱拆箱操作。

## SparseBooleanArray

### 设计目的

代替 HashMap<Integer,Boolean>，进一步优化 SparseArray，避免 value 为 Boolean 的时候的装箱拆箱操作。

## SparseLongArray

### 设计目的

代替 HashMap<Integer,Long>，进一步优化 SparseArray，避免 value 为 Long 的时候的装箱拆箱操作。
