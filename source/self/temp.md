2. Flutter

Widget 、 Element 、RenderObject

Widget 与 Element 是一对多
Element 与 RenderObject 是一一对应的关系

Widget 不可变，每次保持在一帧，如果发生改变是通过 State 实现跨帧状态保存，而真实完成布局和绘制数组的是 RenderObject ， Element 充当两者的桥梁， State 就是保存在 Element 中。

setState 其实是调用了 markNeedsBuild ，该方法内部标记此 Element 为 Dirty ，然后在下一帧 WidgetsBinding.drawFrame 才会被绘制，这可以看出 setState 并不是立即生效的。

状态共享 InheritedWidget

Flutter UI 是直接通过 skia 渲染

Platform Channel 原生通信

Stream

JIT（Just In Time）
每次启动应用都需要重新编译

AOT（Ahead Of Time)
4.4 推出了全新的虚拟机运行环境 ART，5.0 后 Dalvik 被替换

应用安装比较耗时
优化后的文件会占用额外的存储空间

1. 工程改动。热重载模块会逐一扫描工程中的文件，检查是否有新增、删除或者改动，直到找到在上次编译之后，发生变化的 Dart 代码。
2. 增量编译。热重载模块会将发生变化的 Dart 代码，通过编译转化为增量的 Dart Kernel 文件。
3. 推送更新。热重载模块将增量的 Dart Kernel 文件通过 HTTP 端口，发送给正在移动设备上运行的 Dart VM。
4. 代码合并。Dart VM 会将收到的增量 Dart Kernel 文件，与原有的 Dart Kernel 文件进行合并，然后重新加载 4 新的 Dart Kernel 文件。
5. Widget 重建。在确认 Dart VM 资源加载成功后，Flutter 会将其 UI 线程重置，通知 Flutter Framework 重建 Widget。

---

1. Apk 瘦身做了哪些优化？

闪屏页默认 Logo 图障眼法。实际并没有缩短启动时间。

- 打包资源文件，生成 R.java 文件（使用工具 AAPT）
- 处理 AIDL 文件，生成 java 代码（没有 AIDL 则忽略）
- 编译 java 文件，生成对应.class 文件（java compiler）
- .class 文件转换成 dex 文件（dex）
- 打包成没有签名的 apk（使用工具 apkbuilder）
- 使用签名工具给 apk 签名（使用工具 Jarsigner）
- 对签名后的.apk 文件进行对齐处理，不进行对齐处理不能发布到 Google Market（使用工具 zipalign）

第 4 步，将 class 文件转换成 dex 文件，默认只会生成一个 dex 文件，单个 dex 文件中的方法数不能超过 65536。

闪屏页开个子线程去加载 dex，难维护，不推荐；
今日头条的方案，在单独一个进程加载 dex，加载完主进程再继续。

在第一次启动的时候，直接加载没有经过 OPT 优化的原始 DEX，先使得 APP 能够正常启动。然后在后台启动一个单独进程，慢慢地做完 DEX 的 OPT 工作，尽可能避免影响到前台 APP 的正常使用。

第三方库延迟加载。没必要在 Application 初始化的。Glide 自身就在框架中初始化。

2. 还做了哪些其他性能优化？App 启动、ANR？

- 启动优化

闪屏页设置默认 Logo 图障眼法。这样打开桌面图标会马上显示 logo，不会出现黑/白屏，直到 Activity 启动完成，替换主题，logo 消失，但是总的启动时间并没有改变。

- 内存优化

减少 OOM LeakCanary

减少卡顿

刷新原理
底层每间隔 16 毫秒会发出 vsyn 信号，应用界面要更新，必须先向底层请求 vsync 信号，这样下一个 16 毫秒 vsync 信号来的时候，底层会通过 JNI 通知到应用，然后通过主线程 Handler 执行 View 的绘制任务。所以两个地方会造成卡顿，一个是主线程在执行耗时操作导致 View 的绘制任务没有及时执行，还有一个是 View 绘制太久，可能是层级太多，或者里面绘制算法太复杂，导致没能在下一个 vsync 信号来临之前准备完数据，导致掉帧卡顿。

监控卡段
BlockCanary Handlder 取出一条消息前 Printer logging 不为空
AOP 插桩

---

4. 熟悉 Glide 、 OkHttp 、 LeakCanary 等著名开源库，并阅读过部分源码。

Glide

---

1. Glide

为什么用 Glide？

- 支持多种格式。（Gif、WebP）
- 生命周期集成。
- 高效处理 Bitmap
- 高效缓存策略。

Fesco 5.0 以下有优势，在 Native 管理内存，更质量的图片展示，但是包大小更大，2-3M。

假如让你写一个图片加载框架，说说思路?

- 封装请求参数
- 使用线程池异步加载 两个线程池（本地缓存、网络）
- 缓存 内存缓存、硬盘缓存 LruCache DiskLruCache
- OOM 问题

预防：减少内存使用

图片压缩

处理：
onLowMemory

onTrimMemory

- 内存泄露 注意 ImageView 的正确引用，生命周期管理
- 列表滑动加载的问题

复用机制加载错乱
给 ImageView 设置 tag，tag 一般是图片地址，更新 ImageView 之前判断 tag 是否跟 url 一致。

2. OkHttp

是对 Socket 的封装。URLConnection 在 4.4 以后底层也使用了 OkHttp。

Android 源码中 /external/okhttp/jarjar-rules.txt 中表示 com.squareup 开关的包会在编译时打包成 com.android 开头的包。

- 创建请求对象。 (url, method, headers, body, tags)–> Request–> Call

- 线程池分发请求。同步使用 call.excute(),异步使用 call.enqueue()请求事件队列, 都交给 Dispatcher 分发， enqueue–>Runnable–>ThreadPoolExecutor

- 递归 Interceptor 拦截器，发送请求。 getResponseWithInterceptorChain()，InterceptorChain

- 请求回调，数据解析。 Respose –> (code,message,requestBody)

3. LeakCanary

Watch
application.registerActivityLifecycleCallbacks
WeakReference + ReferenceQueue

弱引用一旦变得弱可达，就会立即入队 ReferenceQueue。这将在 finalization 或者 GC 之前发生。

Dump 计算了到 GC Roots 的最短强引用路径。

fragmentManager.registerFragmentLifecycleCallbacks

#### 项目经历

---

为什么做这个项目？做这个项目的目标是什么？我的方案是什么？相对其他方案我的方案优势是什么？项目的收益是什么？项目的架构图是否能画出来？项目中使用的主要框架原理是否前前后后都清楚？

---

Android 端图片选择内部库实现（支持多图选择）

- 背景：限于项目中已有的图片选择库扩展性不强，不能满足产品多图选择的需求，于是封装设计实现新的图片选择库。

- 过程：查看项目中已存图片选择库源码，发现是通过调用系统方法来实现的图片选择功能，不能满足多图选择和 UI 定制的需求。于是查看第三方多图选择开源库 Matisse、TakePhoto 、Boxing 源码进行调研对比，最终考虑兼容性、UI 定制性和解决 issue 的积极度等问题，决定自己实现基于 Matisse 和 Boxing 的图片多选库。

- 成果：产出了一个较为健壮的通用内部库，满足了选择图片相关多变需求，同时优化了旧版图片选择框架的缺陷。

---

1. 为什么自己实现

需要定制化选择界面的 UI。
Matisse 未集成运行时权限，支持不同的图片加载库，不支持选择，UI 和加载逻辑耦合，难以修改 UI；
Boxing 集成运行时权限，选择视频，在 Android Q 有兼容性问题，会崩溃。

两者都是在 onActivityResult()接受回调。

2. 怎么设计的

建造者模式，构建请求。

适配器模式，封装一层图片选择器适配层，实现无缝切换，无需改动老代码，同时新代码可直接使用。

策略模式，可以更换具体的图片选择实现（UI）。

是否预留裁剪接口。

具体的图片选择器实现：

分层：

逻辑层：将图片选择器的核心加载选择逻辑封装成独立模块
UI 层：提供一套展示图片选择的 UI 模块

建造者模式提供选择选项配置。

3. 加载列表图片卡顿怎么优化

建议项目统一图片库。策略模式

如果自己实现考虑以下问题。

高效加载单张大图片步骤：

肯定不能直接把一张大图完全加载到内存中，根据展示这张图片的控件的实际大小来按需加载。

1. 预先获取原图宽高。inJustDecodeBounds 可以避免禁止为 bitmap 分配内存，返回值也不再是一个 Bitmap 对象，而是 null，而 BitmapFactory.Options 的 outWidth、outHeight 和 outMimeType 属性都会被赋值。

2. 计算采样率。inSampleSize，inSampleSize 默认会取靠近 2 的幂次。

3. 按采样率重新解析大图。

大量大图加载：

为了保证内存的使用始终维持在一个合理的范围，通常会把被移除屏幕的图片进行回收处理。此时垃圾回收器也会认为你不再持有这些图片的引用，从而对这些图片进行 GC 操作。用这种思路来解决问题是非常好的，可是为了能让程序快速运行，在界面上迅速地加载图片，你又必须要考虑到某些图片被回收之后，用户又将它重新滑入屏幕这种情况。这时重新去加载一遍刚刚加载过的图片无疑是性能的瓶颈。可以考虑使用内存缓存技术 LRUCache。

---

最难的问题：

7.0 适配 多个 ContentProvider authorities 重复 覆盖
