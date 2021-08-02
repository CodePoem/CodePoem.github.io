---
title: 简析Glide
date: 2019-10-17 11:25:23
updated: 2021-08-01 11:31:55
categories:
  - Android
tags:
  - Android
  - Glide
  - 图片
---

最简洁的图片加载流程：

```java
Glide.with(context)
    .load(myUrl)
    .into(imageView);
```

1. with 传递图片加载上下文，获取图片加载请求管理器（RequestManager）。
2. load 传递图片加载数据源，构建图片加载请求建造器（RequestBuilder）。
3. into 传递图片加载目标载体，创建图片加载请求（Request），执行图片加载。

## 生命周期

with 支持传递 Context，Activity，Fragment，Glide 4.0.0 版本后 with 还支持传递 View。

1. 如果是子线程，会使用 ApplicationContext。
2. 传递 Context 如果是 ContextWrapper 直接会优先向上遍历得到 Activity。
3. 传递 View ，会优先向上遍历得到 Activity，然后根据得到的 Activity 优先遍历得到 Fragment。

### 传入 ApplicationContext

Application 对象的生命周期即应用程序的生命周期，因此 Glide 并不需要做什么特殊的处理，它自动就是和应用程序的生命周期是同步的。

### 传入非 ApplicationContext

无视图的 Fragment 接管 LifeCycle。

## 缓存机制

### 内存缓存

1. 根据图片地址，宽高，变换，签名等生成 key。
2. 根据 key 从活动缓存取图片资源。取到则完成；取不到进行下一步。
3. 根据 key 从内存缓存（LRU 实现）。取到则将取到的图片资源添加到活动缓存，完成；取不到进行下一步。
4. 返回 null，表示未取到。

- LRU 缓存。LruCache 算法（Least Recently Used），也叫近期最少使用算法。算法原理就是把最近使用的对象用强引用存储在 LinkedHashMap 中，并且把最近最少使用的对象在缓存值达到预设定值之前从内存中移除。

- 活跃缓存。弱引用缓存正在使用的图片，可以保护这些图片不会被 LruCache 算法回收掉。

缓存写入时机：

- 图片资源执行完加载解码后，写入活跃缓存。
- 图片资源回收且引用数为 0，先从活跃缓存中删除，后写入 LRU 缓存。

### 硬盘缓存

DiskCacheStrategy.NONE： 表示不缓存任何内容。
DiskCacheStrategy.SOURCE： 表示只缓存原始图片。
DiskCacheStrategy.RESULT： 表示只缓存转换过后的图片（默认选项）。
DiskCacheStrategy.ALL ： 表示既缓存原始图片，也缓存转换过后的图片。

DiskLruCache，journal 文件。

journal 文件组成：

- libcore.io.DiskLruCache （魔法值，标志着使用 DiskLruCache 技术）
- DiskLruCache 版本号
- 应用版本号
- DIRTY 数据行（正在写数据）
- CLEAN 数据行（写入数据成功）
- REMOVE 数据行（写入数据失败）
- READ 数据行（读取数据）

## Bitmap 采样和复用机制

### 采样

1. 计算缩放因子
2. 获取采样类型 内存优先（比要求的尺寸小）/ 质量优先（比要求的尺寸大）
3. 计算整型的缩放因子
4. 将整型的缩放因子转成 2 的幂

### 复用机制

BitmapPool，默认具体实现 LruBitmapPool。

不光对开销较大的 Bitmap 进行了复用，就连为了复用 Bitmap 时重复申请的 Key 对象都进行了复用，尽可能的减少了对象的创建开销，保证了应用的流畅性。

#### 复用的前提

inMutable = true
inBitmap 指定复用的 Bitmap

在 Android 4.4 之前，仅支持相同大小的 bitmap，inSampleSize 必须为 1，而且必须采用 jpeg 或 png 格式。

在 Android 4.4 之后只有一个限制，就是被复用的 bitmap 尺寸要大于新的 bitmap，简单来说就是大图可以给小图复用。

#### 复用实现

根据特定的缓存策略，根据系统版本如判断采用完全匹配或者大小匹配，从缓存池获取到 Dirty 的 Bitmap，使用 reconfigure 和 eraseColor(Color.TRANSPARENT) 来 clean 这个 Bitmap。

1. 加入复用池时机：外部加载完或者回收调用放入
2. 获取缓存时机：加载图片前
3. 删除，每次操作都会判断是否需要删除多余的缓存

## 线程池管理

- diskCacheExecutor 本地缓存任务 newDiskCacheBuilder corePoolSize=maximumPoolSize=1
- sourceExecutor 资源获取任务 corePoolSize=maximumPoolSize=处理器数量和固定值 4 之间区最小
- sourceUnlimitedExecutor 资源获取不限制任务 corePoolSize=0 maximumPoolSize=Integer.MAX_VALUE 10ms 超时 SynchronousQueue
- animationExecutor Gif 加载任务 corePoolSize=在处理器数量和固定值 4 之间区最小 maximumPoolSize=1 或 2

除了 sourceUnlimitedExecutor 都使用了 PriorityBlockingQueue 来作为等待队列。

## 使用自定义模块解耦

- setMemoryCache()
  用于配置 Glide 的内存缓存策略，默认配置是 LruResourceCache。

- setBitmapPool()
  用于配置 Glide 的 Bitmap 缓存池，默认配置是 LruBitmapPool。

- setDiskCache()
  用于配置 Glide 的硬盘缓存策略，默认配置是 InternalCacheDiskCacheFactory。

- setDiskCacheService()
  用于配置 Glide 读取缓存中图片的异步执行器，默认配置是 FifoPriorityThreadPoolExecutor，也就是先入先出原则。

- setResizeService()
  用于配置 Glide 读取非缓存中图片的异步执行器，默认配置也是 FifoPriorityThreadPoolExecutor。

- setDecodeFormat()
  用于配置 Glide 加载图片的解码模式，默认配置是 RGB_565。

Glide 3:

1. 自定义模块类，实现 GlideModule 。
2. AndroidManifest.xml 中配置

```xml
<manifest>

    ...

    <application>

        <meta-data
            android:name="com.example.glidetest.MyGlideModule"
            android:value="GlideModule" />

        ...

    </application>
</manifest>
```

解析 AndroidManifest.xml 中 value 为 GlideModule 的 meta-data ,反射实例化自定义模块类。Glide 在实例化的时候会先遍历模块列表应用配置。

Glide 4:

1. 自定义模块类，继承 AppGlideModule 。
2. 自定义模块类上添加 @GlideModule 注解。

注解处理器 auto-service + JavaPoet
