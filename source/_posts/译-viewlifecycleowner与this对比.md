---
title: 译-viewlifecycleowner与this对比
date: 2020-11-24 13:03:26
updated: 2020-11-24 13:03:29
categories:
  - Android
tags:
  - Android
  - 译文
  - lifecycle
---

哈喽，Android 小伙伴们~

这篇文章将提到 Fragment 中的两种 lifecycleowners。因为作为一个 Android 开发者，我们能够简单地使用 Fragment 作为 lifecycle owner，但某些情境下这可能会使你烦恼。

让我们先来看一下 Fragment 的生命周期：

<center>

![FragmentLifecycle](/assert/img/lifecycle/fragment_lifecycle.png)

</center>

从图中我们可以看到 onCreate 和 onDestroy 只调用了一次。这些是 Fragment 的主要的生命周期方法。 onCreateView 和 onDestroyView 根据 Fragment 的状态来被调用，因为它们是 Fragment 中 View 的主要生命周期方法。

因此，如果我们在 onCreate 中绑定 LiveData ，仅仅注册一次似乎是很合适很健康的。

```kotlin
liveData.observe(this, observer)
```

然而当我们跳转到另一个 Fragment 并且改变 LiveData 的值后再回到之前的 Fragment 时会发生什么呢?

返回到之前的 Fragment 时，我们不能获取到 LiveData 最新的数据，需要去请求它。

因此似乎使用 View 的 LifeCycle 方法是更有用的。在这种条件下，我们可能犯的一个共同的错误是使用 Fragment 作为 LifeCycle owner。

```kotlin
// 在 onViewCreated, onCreateView, onActivityCreated 中调用
// 错误!!! 它可能造成重复观察或其他问题
liveData.observe(this, observer)
```

我们应该使用 viewLifecycleOwner :

```kotlin
// 在 onViewCreated, onCreateView, onActivityCreated 中调用
liveData.observe(viewLifecycleOwner, observer)
```

---

[原文](https://medium.com/@cs.ibrahimyilmaz/viewlifecycleowner-vs-this-a8259800367b)
