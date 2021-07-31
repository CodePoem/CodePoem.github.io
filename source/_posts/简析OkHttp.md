---
title: 简析OkHttp
date: 2019-10-18 11:25:23
updated: 2021-08-01 11:31:55
categories:
  - Android
tags:
  - Android
  - OkHttp
  - 网络
---

是对 Socket 的封装。URLConnection 在 4.4 以后底层也使用了 OkHttp。

Android 源码中 /external/okhttp/jarjar-rules.txt 中表示 com.squareup 开关的包会在编译时打包成 com.android 开头的包。

```text
rule com.squareup.** com.android.@1
rule okio.** com.android.okio.@1
```

## 基本用法

```java
String url = "http://www.xxx.com";
//  生成 OkHttpClient 实例对象
OkHttpClient okHttpClient = new OkHttpClient();
//  生成 Request 对象
Request request = new Request.Builder()
    .url(url)
    .post(RequestBody.create(MediaType.parse("application/json; charset=utf-8"),"test content"))
    .build();
//  生成 Call 对象
Call call = okHttpClient.newCall(request);

call.enqueue(new Callback() {
    @Override
    public void onFailure(@NonNull Call call, @NonNull IOException e) {
    }

    @Override
    public void onResponse(@NonNull Call call, @NonNull Response response)  {

    }
});
```

## 流程

1. 创建请求对象。 (url, method, headers, body, tags)--> Request--> Call

2. 线程池分发请求。同步使用 call.excute(),异步使用 call.enqueue()请求事件队列, 都交给 Dispatcher 分发， enqueue-->Runnable-->ThreadPoolExecutor

3. 递归 Interceptor 拦截器，发送请求。 getResponseWithInterceptorChain()，InterceptorChain

4. 请求回调，数据解析。 Respose --> (code,message,requestBody)

不管是异步还是同步，都是一样的三部曲:

1. 加入到 Dispatche r 里面的同步(或异步)队列。
2. 执行 getResponseWithInterceptorChain 方法。（只不过同步操作是直接运行了 getResponseWithInterceptorChain 方法，而异步是通过线程池执行 R unnabl e 再去执行 getResponseWithInterceptorChain 方法）
3. 从 Dispatcher 里面的同步(或异步)队列移除。

Dispatcher 内部维护 3 个队列及 1 个线程池:

- readyAsyncCalls

待访问请求队列，里面存储准备执行的请求。

- runningSyncCalls

同步请求队列，正在执行的请求，包含已经取消但是还没有结束的请求。

- runningAsyncCalls

异步请求队列，里面存储正在执行，包含已经取消但是还没有结束的请求。

- ExecutorService

线程池，最小 0，最大 Max 的线程池。

## 拦截器

### RetryAndFollowUpInterceptor

重试与重定向拦截器。

通过 while (true) 的死循环来进行对异常结果或者响应结果判断是否要进行重新请求。

### BirdgeInterceptor

桥接拦截器。

为用户构建的一个 Request 请求转化为能够进行网络访问的请求，同时将网络请求回来的响应 Response 转化为用户可用的 Response。初始化信息，添加请求头等，比如涉及的网络文件的类型和网页的编码，返回的数据的解压处理等等。

### CacheInterceptor

缓存拦截器。

根据 OkHttpClient 对象的配置以及缓存策略对请求值进行缓存。

内部有 Cache 类，处理缓存操作，intercache 内部类，disklrucache 算法等
重点是不缓存非 get 的请求。
CacheStrategy 缓存策略类，通过工厂模式获取。

### ConnectionInterceptor

网络连接拦截器。

底层是通过 SOCKET 的方式于服务端进行连接的，并且在连接建立之后会通过 OKIO 获取通向 server 端的输入流 Source 和输出流 Sink。

### CallServerInterceptor

服务请求的拦截器。

负责与服务器建立 Socket 连接，并且创建了一个 HttpStream 它包括通向服务器的输入流和输出流。

### 自定义拦截器

#### Application Interceptor

在 retryAndFollowUpInterceptor 之前，处于拦截器第一个位置。

它是第一个触发拦截的，这里拦截到的 url 请求的信息都是最原始的信息。所以我们可以在该拦截器中添加一些我们请求中需要的通用信息，打印一些我们需要的日志。可以定义多个这样的拦截器，例如一个处理 header 信息，一个处理 接口请求的 加解密 。

#### NetwrokInterceptor

在 ConnectInterceptor 和 CallServerInterceptor 之间，处于拦截器倒数第二个位置。

会经过 RetryAndFollowIntercptor 进行重定向并且也会通过 BridgeInterceptor 进行 request 请求头和 响应 resposne 的处理，因此这里可以得到的是更多的信息。在打印结果可以看到它内部重定向操作和失败重试，这里会有比 Application Interceptor 更多的日志。
