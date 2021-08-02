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

---

5. 熟练使用 MVC/MVP/MVVM 架构，熟悉 Git 版本管理，有团队协作经验，具备 Android 模块化开发、 SDK 开发经验。

---

架构：

---

#### 自我评价

主攻 Android，并期望能在移动开发领域深入钻研，同时也不局限于此，期望能培养“全栈”思维（大前端、后端）。具备较强责任心且注重团队协作，能尽心尽责地完成分配的工作任务并力求完美，在工作中能与同事之间相互协助、友好交流，更高效的完成工作。关注个人和团队开发效率，会积极主动尝试优化开发测试流程、开发测试工具等。有较强自学能力，且求知欲旺盛，敢于接受新的挑战，遇到问题善用过往经验和搜索引擎（Google、Stack over flow、官方文档等）寻找答案。同时也具备一定的源码阅读能力，用以学习官方技术实现和解决难以搜索解决的问题。

---

1. 如何主动优化开发测试流程？举个例子。

Debug 工具的优化：接口版本控制，测试频繁需要改动版本号打包。
开发优化：资源放置问题：图片资源放置错误。
开发流程优化：checkstyle lint。

2. 怎么自学的？举个例子。

- 出现某种问题或者现象，去被动学习。

根据以往知识经验类比猜测，搜索引擎验证，看源码验证。

- 主动学习。

业务优先，但不是说代码写完了就完了，看看有没有什么优化的地方。
无视图 UI 回调。
查阅官网资料、写简单 demo。

---

#### 工作经历

##### 杭州二维火科技有限公司 （ 2017 年 11 月 ~ 至今 ）

火掌柜部门，实习并转正，Android 开发工程师。

---

- 为什么跳槽

个人发展空间受限，业务线变动，人员流动大。期望有更好的平台发展。

- 职业规划

3-5 年规划，成为 Android 技术专家，对某个领域有深刻的理解。

---

#### 项目经历

---

为什么做这个项目？做这个项目的目标是什么？我的方案是什么？相对其他方案我的方案优势是什么？项目的收益是什么？项目的架构图是否能画出来？项目中使用的主要框架原理是否前前后后都清楚？

---

##### 火掌柜 Android 端 （ToB）

- 项目简介：一款互联网智能餐厅管理系统。可远程管理员工、菜品、桌位、收银设置等。

- 参与过程：迭代并维护 App 中桌位、传菜、收银设置等主要功能，并关注和提升 App 性能与稳定性。
- 技术产出：

1. Android 端持续集成建设

   - 背景：在业务快速迭代的同时 Android 模块化也在持续进行，开发过程中不可避免的会产生一些无用资源、编译异常等缺陷，并且这些问题往往又会导致团队协作过程中多人重复解决问题， App 软件交付降速、交付的软件产品质量不佳等问题。因此迫切需要对 Android 端进行持续集成基础建设，来自动化地进行初步的质量监控并反馈。

   - 过程：调研各大成熟公司对于持续集成建设的方案并动手实践对比，结合公司具体场景最终决定持续集成方案（Gitlab CI + Jenkins）并展开实施。主要实现了自动化的 Android 端代码的静态检查和消息闭环，包括 Lint 、 CheckStyle 、 FindBugs 等代码质量检测工具的集成，以及发出通知（钉钉消息）到模块负责人的功能。

   - 成果：赋予 Android 端 App 提早发现并暴露软件问题的能力，为维稳并降低 Android 端产品的 Bugly 奔溃率、提高团队效率做出了贡献。

---

1. 主要做了什么 ？

持续集成搭建，包括 Gitlab CI runner 的配置、执行脚本的编写、Lint、CheckStyle 、FindBugs 的集成。主要是结合公司实际需求场景进行定制化的改动（策略的考量），比如触发持续集成的时机（硬件资源有限），Lint（较慢）、CheckStyle 规则指定。

2. 有什么记忆比较深刻的地方？

自定义 Lint 规则。
比如 Log.d 统一成 LogUtils，quickFix。
收敛 Bitmap decode。

- 依赖 lint-api lint-check
- api 不稳定，且没有什么 api 文档，只能参考着官方 check custom-lint 写

---

2. Android 端图片选择内部库实现（支持多图选择）

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

##### 火拼拼 Android 端 （ToC）

- 项目简介：一款餐饮生活类 App。主打扫码点餐拼团、外卖、排队，餐饮一站式服务平台。

- 参与过程：推进 Android 端在原生开发基础上混合使用 Flutter 进行开发来提高开发效率。使用 Flutter 编写部分新业务页面（购物袋设置模块等）。

- 项目成果：项目业务迭代原本需要 iOS/Android 两端编写的大部分业务逻辑和 UI 界面只需 Flutter 统一实现，节省几乎近半项目开发人力成本。

- 技术产出：封装其他开发人员可复用的 Flutter 基础 UI 组件；改进现有 Flutter 开发不规范问题（图片资源放置错误等）；仿照 gradlew 引入 flutterw 脚本，用于管理跟随项目 Flutter SDK 版本。

---

1. Flutter 项目参与了什么？

2. 做了哪些工作，优化，有什么不一样的？

---

---

最难的问题：

华为手机图片选择 OOM

java.lang.IllegalArgumentException
column '\_data' does not exist

TimeoutException

7.0 适配 多个 ContentProvider authorities 重复 覆盖

---

#### 项目经历

##### 火掌柜 Android 端 （ToB）

- 项目简介：一款互联网智能餐厅管理系统。可远程管理员工、菜品、桌位、收银设置等。

- 参与过程：迭代并维护 App 中桌位、传菜、收银设置等主要功能，并关注和提升 App 性能与稳定性。
- 技术产出：

1. Android 端持续集成建设

   - 背景：在业务快速迭代的同时 Android 模块化也在持续进行，开发过程中不可避免的会产生一些无用资源、编译异常等缺陷，并且这些问题往往又会导致团队协作过程中多人重复解决问题， App 软件交付降速、交付的软件产品质量不佳等问题。因此迫切需要对 Android 端进行持续集成基础建设，来自动化地进行初步的质量监控并反馈。

   - 过程：调研各大成熟公司对于持续集成建设的方案并动手实践对比，结合公司具体场景最终决定持续集成方案（Gitlab CI + Jenkins）并展开实施。主要实现了自动化的 Android 端代码的静态检查和消息闭环，包括 Lint 、 CheckStyle 、 FindBugs 等代码质量检测工具的集成，以及发出通知（钉钉消息）到模块负责人的功能。

   - 成果：赋予 Android 端 App 提早发现并暴露软件问题的能力，为维稳并降低 Android 端产品的 Bugly 奔溃率、提高团队效率做出了贡献。

2. Android 端图片选择内部库实现（支持多图选择）

   - 背景：限于项目中已有的图片选择库扩展性不强，不能满足产品多图选择的需求，于是封装设计实现新的图片选择库。

   - 过程：查看项目中已存图片选择库源码，发现是通过调用系统方法来实现的图片选择功能，不能满足多图选择和 UI 定制的需求。于是查看第三方多图选择开源库 Matisse、TakePhoto 、Boxing 源码进行调研对比，最终考虑兼容性、UI 定制性和解决 issue 的积极度等问题，决定自己实现基于 Matisse 和 Boxing 的图片多选库。

   - 成果：产出了一个较为健壮的通用内部库，满足了选择图片相关多变需求，同时优化了旧版图片选择框架的缺陷。

##### 火拼拼 Android 端 （ToC）

- 项目简介：一款餐饮生活类 App。主打扫码点餐拼团、外卖、排队，餐饮一站式服务平台。

- 参与过程：推进 Android 端在原生开发基础上混合使用 Flutter 进行开发来提高开发效率。使用 Flutter 编写部分新业务页面（购物袋设置模块等）。

- 项目成果：项目业务迭代原本需要 iOS/Android 两端编写的大部分业务逻辑和 UI 界面只需 Flutter 统一实现，节省几乎近半项目开发人力成本。

- 技术产出：封装其他开发人员可复用的 Flutter 基础 UI 组件；改进现有 Flutter 开发不规范问题（图片资源放置错误等）；仿照 gradlew 引入 flutterw 脚本，用于管理跟随项目 Flutter SDK 版本。
