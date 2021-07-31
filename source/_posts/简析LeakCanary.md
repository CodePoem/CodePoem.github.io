---
title: 简析LeakCanary
date: 2019-10-19 11:25:23
updated: 2021-08-01 11:31:55
categories:
  - Android
tags:
  - Android
  - LeakCanary
  - 内存
---

## 使用

添加依赖（release 有 no-op 版）然后在 Application 初始化。

```gradle
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:1.X'
  releaseImplementation 'com.squareup.leakcanary:leakcanary-android-no-op:1.X'
  // Optional, if you use support library fragments:
  debugImplementation 'com.squareup.leakcanary:leakcanary-support-fragment:1.X'
}
```

```java
public class ExampleApplication extends Application {

    @Override public void onCreate() {
    super.onCreate();
    if (LeakCanary.isInAnalyzerProcess(this)) {
        // This process is dedicated to LeakCanary for heap analysis.
        // You should not init your app in this process.
        return;
    }
    LeakCanary.install(this);
    // Normal app init code...
    }
}
```

## 原理

### Watch

Activity：

application.registerActivityLifecycleCallbacks 覆写 onActivityDestroyed

watch（）使用 WeakReference + ReferenceQueue 监听对象回收情况

watchedObjects（LinkedHashMap<key, KeyedWeakReference>） watch() 方法传进来的引用，尚未判定为泄露
queue（ReferenceQueue） 怀疑泄漏的对象列表

以 UUID.randomUUID().toString() 为 key 构造 KeyedWeakReference（关联 ReferenceQueue） 存入 watchedObjects。

**_弱引用一旦变得弱可达，就会立即入队 ReferenceQueue。这将在 finalization 或者 GC 之前发生。_**

watch 方法最后会调用 moveToRetained（）

### Dump

### Analysis

计算了到 GC Roots 的最短强引用路径。

## 2.0 版本

### 不需要在 Application 初始化

```gradle
dependencies {
  debugImplementation 'com.squareup.leakcanary:leakcanary-android:2.X'
}
```

#### ContentProvider 实现

leakcanary-android-process 模块的 AndroidManifest.xml 文件:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.squareup.leakcanary">

  <application>
    <service
        android:name="leakcanary.internal.HeapAnalyzerService"
        android:exported="false"
        android:process=":leakcanary" />

    <provider
        android:name="leakcanary.internal.AppWatcherInstaller$LeakCanaryProcess"
        android:authorities="${applicationId}.leakcanary-process.installer"
        android:process=":leakcanary"
        android:exported="false"/>
  </application>

</manifest>
```

看看 AppWatcherInstaller 干了啥：

```kotlin
internal sealed class AppWatcherInstaller : ContentProvider() {

  /**
   * [MainProcess] automatically sets up the LeakCanary code that runs in the main app process.
   */
  internal class MainProcess : AppWatcherInstaller()

  /**
   * When using the `leakcanary-android-process` artifact instead of `leakcanary-android`,
   * [LeakCanaryProcess] automatically sets up the LeakCanary code
   */
  internal class LeakCanaryProcess : AppWatcherInstaller() {
    override fun onCreate(): Boolean {
      super.onCreate()
      AppWatcher.config = AppWatcher.config.copy(enabled = false)
      return true
    }
  }

   override fun onCreate(): Boolean {
    val application = context!!.applicationContext as Application
    InternalAppWatcher.install(application)
    return true
  }
}
```

利用加载顺序实现自动注入：

Application->attachBaseContext =====> ContentProvider->onCreate =====> Application->onCreate =====> Activity->onCreate

优点：实现"免侵入"集成，不需要手动初始化。
缺点：无法更改初始化时机（App 启动优化按需延迟初始化第三方库对这样的集成方式就无能为力了）。考虑到 LeakCanary 是开发 debug 阶段使用的，也无可厚非。一般的 SDK 还是不建议使用这种方式。

### 添加默认对 Fragment 的支持

Fragment：

Android O 版本 androidx 都具备对 Fragment 生命周期的监听功能。

application.registerActivityLifecycleCallbacks 覆写 onActivityCreated
然后 fragmentManager.registerFragmentLifecycleCallbacks 覆写 onFragmentViewDestroyed() onFragmentDestroyed()
