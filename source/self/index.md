---
title: self
date:
layout:
export_on_save:
  puppeteer: ["pdf", "png"]
---

## 专业技能

### 熟悉 Kotlin 协程，并了解其原理

优势：

**消除了并发任务之前协作的难度。**

假设有一个需求，需要请求两个网络接口，将数据合并后再显示到界面。
最合理的做法是并行请求后再融合数据后展示界面。

1. 回调式（Java Callbacks）

   回调式开发，不仅是多了缩进，还限制了能力。回调式开发要做这种工作很困难，不得不妥协，将并行请求改成了串行请求，性能差了一倍。

2. 响应式（RxJava）

   可以使用 zip 函数。有一定的学习曲线。

3. 协程（Kotlin Coroutines）

   使用命令式的风格写出异步代码，简洁可读性强。

**suspend 关键字：表示非阻塞时挂起，只起到标记作用。协程挂起恢复是由编译器完成的。**

- Kotlin 协程是一个线程框架。方便，在同一个代码块能进行多次切换。
- 挂起就是可以自动切换来的切线程。
- 非阻塞式指的是可以用看起来阻塞的代码写出非阻塞的操作。

原理：

核心关注编译器。

编译器对挂起函数的第一个改变就是对函数签名的改变，这种改变被称为 CPS（Continuation Passing Style 续体传递风格）变换，将内部要执行的逻辑封装到一个闭包里面，然后再返回给调用者。

续体（Continuation）是一个抽象概念，指包装了协程在挂起之后应该继续执行的代码。**目的是保存挂起点。它携带上下文。上下文可以添加拦截器、异常处理器。**

**一个挂起函数会被编译成一个实现了 Continuation 接口的匿名类，其中一个函数 resumeWith 实现为状态机，每一个续体对应一个状态。当函数运行到每个挂起点时，状态就会切换。**

协程的挂起和 Java 的 NIO 机制是类似的，我们在一个线程中执行了一个原本会阻塞线程的任务，但是这个调用者线程没有发生阻塞，这是因为它们**有一个专门的线程来负责这些任务的流转**，也就是说，当我们发起多个阻塞操作的时候，可能只会阻塞这一个专门的线程，它一直在等待，谁的阻塞结束了，它就把回调再分派过去，这样就完成了阻塞任务与阻塞线程的多对一，而不是以前的一对一，所以挂起也好，NIO 也好，本质上都没有彻底消灭阻塞，但是它们都使阻塞的线程大大减少，从而避免了大量的线程上下文状态切换以及避免了大量线程的产生，从而在 IO 密集型任务中大大提高了性能。

### 熟悉 Jetpack 套件中的 Lifecycle、LiveData、ViewModel，并了解其原理

#### Lifecycle

解决的问题：

解决“生命周期管理”一致性问题。更简便、更不容易出错地完成生命周期的管理。

举个例子：地图 SDK，多处调用的地方都需要手动进行 resume、pause。

实现的原理：

- 模板方法模式
- 观察者模式

Activity/Fragment 实现 LifecycleOwner 接口，通过 LifecycleRegistry 在对应生命周期分发事件 Lifecycle.Event，回调到生命周期观察者 LifecycleObserver 对应订阅方法。

ComponentActivity 默认挂载了一个 无 UI 界面的 ReportFragment。

SDK >= 29：activity registerActivityLifecycleCallbacks api，onCreate、onStart、onResume 等方法被调用后，onPause、onStop、onDestroy 等方法被调用前分发。
SDK < 29：直接通过 ReportFragment 自身生命周期回调分发。

#### LiveData

解决的问题：

解决“消息管理”的一致性问题。

EventBus 本身缺乏 Lifecycle 的加持，存在生命周期管理的一致性问题。
LiveData 持有可被观察数据，可感知生命周期。

实现的原理：

生命周期感知依赖 LifeCycle，数据通知只通知活跃的观察者，STARTED 和 RESUMED 就是活跃状态。
LiveData 内维护的 mVersion 表示的是发送信息的版本,每次发送一次信息, 它都会+1, 而 ObserverWrapper 内维护的 mLastVersion 为订阅触发的版本号, 当订阅动作生效的时候, 它的版本号会和发送信息的版本号同步。初始值都为-1。

#### ViewModel

解决的问题：

解决“状态管理”的一致性问题。

1. onSaveInstanceState()
   为小数据量设计的状态恢复。
   适用场景：

   - 程序进程在后台因为内存限制被停止。
   - 配置改变（旋转屏幕、语言改变等）。

2. Fragment.setRetainInstance(true)
   可保留大数据集如图片、或复杂的对象如网络连接。
   适用场景：

   - 配置改变（旋转屏幕、语言改变等）。

3. ViewModel
   事实上 ViewModels 在幕后使用 setRetainInstance 设置为 true 的 Fragment。 实现原理？？？
   适用场景：

   - 配置改变（旋转屏幕、语言改变等）。

ViewModel SavedStateHandle 可代替 onSaveInstanceState。 实现原理是封装了 onSaveInstanceState，在 Activity / Fragment 存储和恢复时分发给 SavedStateHandle。
SavedStateHandle 负责接收 Activity 重建时缓存的数据，并将缓存的数据以 LiveData 的方式暴露给开发者，也向外部提供了获取最新键值对的入口。SavedStateRegistry 则负责将 Activity 原生的数据缓存机制串联起来，向外部暴露了提交数据和消费数据的入口。

实现的原理：

ViewModel 能在 Activity(Fragment) 在由于配置重建时恢复数据的实现原理是：Activity(ComponentActivity) 会将 ViewModelStore 在 Activity(Fragment) 重建之前交给 ActivityThread 中的 ActivityClientRecord 持有，待 Activity(Fragment) 重建完成之后，再从 ActivityClientRecord 中获取 ViewModelStore。

如果应用的进程位于后台时，由于系统内存不足被销毁了。即使利用 ViewModel 的也不能在 Activity(Fragment) 重建时恢复数据。因为存储 ViewModel 的 ViewModelStore 是交给 ActivityThread 中的 ActivityClientRecord 暂存的，进程被回收了，ActivityThread 也就会被回收，ViewModelStore 也就被回收了，ViewModel 自然不复存在了。

### 熟悉 Glide 、 OkHttp 、 LeakCanary 等知名开源库，并阅读过部分源码

[Glide](/post/简析Glide)

[OkHttp](/post/简析OkHttp)

[LeakCanary](/post/简析LeakCanary)

依赖 Java 的一个原生特性，即 WeakReference 和 ReferenceQueue 的一个组合特性：在声明一个 WeakReference 对象时如果同时传入了 ReferenceQueue 作为构造参数的话，那么当 WeakReference 持有的对象被 GC 回收时，JVM 就会把这个弱引用存入与之关联的引用队列之中。

### 掌握 MVC/MVP/MVVM 架构，并对其有一定的理解

关注点分离原则。

MVP 核心思想是依赖倒转，解决逻辑复用难问题。
MVVM 核心思想是数据驱动。

### 了解 Flutter，有过相关业务页面开发经验

三棵树：

- Widget 是用户界面的不可变描述。
- Element 是实例化的 Widget 对象。
- RenderObject 用于应用界面的布局和绘制，保存了元素的大小，布局等信息。

Widget 与 Element 是一对多
Element 与 RenderObject 是一一对应的关系

为了性能，为了复用 Element 从而减少频繁创建和销毁 RenderObject。因为实例化一个 RenderObject 的成本是很高的，频繁的实例化和销毁 RenderObject 对性能的影响比较大，所以当 Widget 树改变的时候，Flutter 使用 Element 树来比较新的 Widget 树和原来的 Widget 树。

Widget 不可变，每次保持在一帧，如果发生改变是通过 State 实现跨帧状态保存，而真实完成布局和绘制数组的是 RenderObject ， Element 充当两者的桥梁， State 就是保存在 Element 中。

setState 其实是调用了 markNeedsBuild ，该方法内部标记此 Element 为 Dirty ，然后在下一帧 WidgetsBinding.drawFrame 才会被绘制，这可以看出 setState 并不是立即生效的。

Flutter 热重载：

1. 工程改动。热重载模块会逐一扫描工程中的文件，检查是否有新增、删除或者改动，直到找到在上次编译之后，发生变化的 Dart 代码。
2. 增量编译。热重载模块会将发生变化的 Dart 代码，通过编译转化为增量的 Dart Kernel 文件。
3. 推送更新。热重载模块将增量的 Dart Kernel 文件通过 HTTP 端口，发送给正在移动设备上运行的 Dart VM。
4. 代码合并。Dart VM 会将收到的增量 Dart Kernel 文件，与原有的 Dart Kernel 文件进行合并，然后重新加载 4 新的 Dart Kernel 文件。
5. Widget 重建。在确认 Dart VM 资源加载成功后，Flutter 会将其 UI 线程重置，通知 Flutter Framework 重建 Widget。

声明式 UI 通过对视图实例的屏蔽，来规避视图实例的 null 安全问题，可用于替代 DataBinding、ViewBinding 等框架。

声明式 UI：

声明式 UI 的本质是函数式编程。
函数式编程的基石是纯函数。
纯函数的特性是 只有一个入口、只有一个出口，且无副作用。

### 了解 CI / CD ，有一定的持续化集成实战经验
