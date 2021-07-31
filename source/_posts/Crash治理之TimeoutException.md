---
title: Crash治理之TimeoutException
date: 2019-09-25 15:36:46
updated: 2019-09-25 15:36:49
categories:
  - Android
tags:
  - Crash
  - TimeoutException
---

## 产生原因

与 GC 过程相关的守护线程中的 2 个守护线程 FinalizerDaemon 和 FinalizerWatchdogDaemon 有关。

- FinalizerDaemon ：析构守护线程。对于重写了成员函数 finalize 的对象，它们被 GC 决定回收时，并没有马上被回收，而是被放入到一个队列中，等待 FinalizerDaemon 守护线程去调用它们的成员函数 finalize ，然后再被回收。
- FinalizerWatchdogDaemon ：析构监护守护线程。用来监控 FinalizerDaemon 线程的执行。一旦检测那些重定了成员函数 finalize 的对象在执行成员函数 finalize 时超出一定的时间，那么就会退出 VM 。

原因小总结：

GC 过程中 FinalizerDaemon 守护线程执行 doFinalize 方法超时。FinalizerWatchdogDaemon 检测到后产生 TimeoutException 并退出虚拟机。
（每个手机触发 Timeout 的时长不同，比如 vivo 的某些 rom 是 2 分钟，模拟器统一都是 10 秒钟）

## 解决方案

### 理想方案

1. 减少对 finalize() 方法的依赖，尽量不依靠 finalize() 方法释放资源，手动处理资源释放逻辑；如果不得已使用 finalize() 方法，尽量减少耗时以及线程同步时间。
2. 减少 finalizable 对象个数，即减少有 finalize() 方法的对象创建，降低 finalizable 对象 GC 次数；Android 5.0 以后 View 默认会实现 finalize 方法，那么减少 View 的创建就是一种解决方法。

理想方案现实情况下却不太容易完全做到。往往需要采用止损方案。

### 止损方案

止损方案本质都是不检测该异常或忽略该异常、不上报该异常，治标不治本。

#### 尝试反射去关闭 FinalizerWatchdogDaemon

**限制：**

Android 9.0 版本开始限制 Private API 调用，不能再使用反射调用 Daemons 以及 FinalizerWatchdogDaemon 类方法。

```java
public class WatchDogKiller {
    private static final String TAG = "WatchDogKiller";
    private static volatile boolean sWatchdogStopped = false;

    public static boolean checkWatchDogAlive() {
        final Class clazz;
        try {
            clazz = Class.forName("java.lang.Daemons$FinalizerWatchdogDaemon");
            final Field field = clazz.getDeclaredField("INSTANCE");
            field.setAccessible(true);
            final Object watchdog = field.get(null);
            Method isRunningMethod = clazz.getSuperclass().getDeclaredMethod("isRunning");
            isRunningMethod.setAccessible(true);
            boolean isRunning = (boolean) isRunningMethod.invoke(watchdog);
            return isRunning;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }


    public static void stopWatchDog() {
        // 建议在在debug包或者灰度包中不要stop，保留发现问题的能力。
        if (!BuildConfig.DEBUG) {
            return;
        }

        // Android P 以后不能反射FinalizerWatchdogDaemon
        if (Build.VERSION.SDK_INT >= 28) {
            Log.w(TAG, "stopWatchDog, do not support after Android P, just return");
            return;
        }
        if (sWatchdogStopped) {
            Log.w(TAG, "stopWatchDog, already stopped, just return");
            return;
        }
        sWatchdogStopped = true;
        Log.w(TAG, "stopWatchDog, try to stop watchdog");

        try {
            final Class clazz = Class.forName("java.lang.Daemons$FinalizerWatchdogDaemon");
            final Field field = clazz.getDeclaredField("INSTANCE");
            field.setAccessible(true);
            final Object watchdog = field.get(null);
            try {
                final Field thread = clazz.getSuperclass().getDeclaredField("thread");
                thread.setAccessible(true);
                thread.set(watchdog, null);
            } catch (final Throwable t) {
                Log.e(TAG, "stopWatchDog, set null occur error:" + t);

                t.printStackTrace();
                try {
                    // 直接调用stop方法，在Android 6.0之前会有线程安全问题
                    final Method method = clazz.getSuperclass().getDeclaredMethod("stop");
                    method.setAccessible(true);
                    method.invoke(watchdog);
                } catch (final Throwable e) {
                    Log.e(TAG, "stopWatchDog, stop occur error:" + t);
                    t.printStackTrace();
                }
            }
        } catch (final Throwable t) {
            Log.e(TAG, "stopWatchDog, get object occur error:" + t);
            t.printStackTrace();
        }
    }
}

```

```java
public class SampleApplication extends Application {

    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        WatchDogKiller.stopWatchDog();
    }
}
```

#### 利用 Thread.UncaughtExceptionHandler 阻断 TimeoutException 处理

```java
public class SampleApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        final Thread.UncaughtExceptionHandler defaultUncaughtExceptionHandler =
                Thread.getDefaultUncaughtExceptionHandler();
        Thread.setDefaultUncaughtExceptionHandler(new Thread.UncaughtExceptionHandler() {
            @Override
            public void uncaughtException(Thread t, Throwable e) {
                if (t.getName().equals("FinalizerWatchdogDaemon") && e instanceof TimeoutException) {
                    // ignore it
                } else {
                    defaultUncaughtExceptionHandler.uncaughtException(t, e);
                }
            }
        });
    }
}

```

---

**推荐阅读:**

1. [提升 Android 下内存的使用意识和排查能力](https://yq.aliyun.com/articles/225751)
2. [再谈 Finalizer 对象--大型 App 中内存与性能的隐性杀手](https://yq.aliyun.com/articles/225755)
3. [从 Daemons 到 finalize timed out after 10 seconds](https://www.jianshu.com/p/18950c9b0ec9)
4. [ART 运行时垃圾收集（GC）过程分析](https://blog.csdn.net/pbm863521/article/details/74451935)
5. [滴滴出行安卓端 finalize time out 的解决方案](https://segmentfault.com/a/1190000019373275)
