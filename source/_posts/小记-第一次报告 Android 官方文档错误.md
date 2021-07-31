---
title: 小记-第一次报告 Android 官方文档错误
date: 2021-06-10 22:43:00
updated: 2021-06-10 22:43:03
categories:
  - Android
tags:
  - 小计
  - Android
---

## 背景

最近在阅读 Android 官方文档[《应用架构指南》](https://developer.android.com/jetpack/guide)一篇，在实操过程中发现示例代码的一处错误。

## 问题

[UserProfileViewModel](https://developer.android.com/jetpack/guide#connect-viewmodel-repository) 的示例代码中

代码行：

```kotlin
val user = LiveData<User> = _user
```

应该为：

```kotlin
val user :LiveData<User> = _user
```

## 后记

本次发现也已向 Google 提交了 [issue](https://issuetracker.google.com/issues/190329948)，并得到了反馈。

虽然只是一处简单的文档笔误，但是也见证了我仔细阅读文档并实操的过程。

---

另外，这是我第一次对 Android 官方文档反馈问题，算是拿到“一血”，仅此记录一下~
