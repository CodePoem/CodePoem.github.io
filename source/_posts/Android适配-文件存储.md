---
title: Android适配-文件存储
date: 2021-05-06 07:36:56
updated: 2021-05-07 08:06:21
categories:
  - Android
tags:
  - Android
  - Android适配
  - 文件存储
---

## FileProvider

历史问题：File Uri 访问控制需要开放底层文件系统权限，开放的权限直到下次关闭之对任何 App 应用都可用。这种级别的访问从根本上说是不安全的。

解决方案：FileProvider

Content Uri 访问控制，“路由映射”设计，无需修改开放底层文件系统权限，只需授予运行时级别的临时权限。

1. AndroidManifest 中声明 FileProvider。考虑到与依赖代码的兼容，不建议直接使用 FileProvider，建议自定义类继承 FileProvider，并制定唯一的 android:authorities 属性。

   ```java
   public class AppleFileProvider extends FileProvider {
   }
   ```

   AndroidManifest.xml:

   ```xml
   <provider
        android:name="com.XXX.XXX.sample.AppleFileProvider"
        android:authorities="${applicationId}.file.provider"
        android:exported="false"
        android:grantUriPermissions="true">

    </provider>
   ```

   - android:name 属性为 FileProvider 组件名。
   - android:authorities 属性为控制域，一般是由包名+自定义的标识构成。
   - android:exported 属性设置为 false; FileProvider 不需要公开。
   - android:grantUriPermissions 属性设置为 true，以允许授予对文件的临时访问权限。

2. 定义“路由映射”关系，即指定 android.support.FILE_PROVIDER_PATHS \<meta-data\> 元素.

   参考官方文档[Specifying Available Files](https://developer.android.google.cn/reference/androidx/core/content/FileProvider#SpecifyFiles)

   AndroidManifest.xml:

   ```xml
   <provider
        android:name="com.XXX.XXX.sample.AppleFileProvider"
        android:authorities="${applicationId}.file.provider"
        android:exported="false"
        android:grantUriPermissions="true">

        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            android:resource="@xml/apple_file_provider"/>

    </provider>
   ```

   apple_file_provider.xml:

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
    <paths>
        <files-path
            path="."
            name="files_temp" />
        <cache-path
            path="."
            name="cache_temp" />
        <external-path
            path="."
            name="external_temp" />
        <external-cache-path
            path="."
            name="external_cache_temp" />
        <external-files-path
            path="."
            name="external_files_temp" />
        <external-media-path
            path="."
            name="external-media_temp" />
        <root-path
            path="."
            name="root_content_temp" />
    </paths>
   ```

   - files-path 表示应用程序内部存储区域子目录中的文件。对应 API Context.getFilesDir()。
   - cache-path 表示应用程序内部存储区域的缓存子目录中的文件。对应 API Context.getCacheDir()。
   - external-path 表示外部存储区根目录下的文件。对应 API Environment.getExternalStorageDirectory()。
   - external-files-path 表示应用程序外部存储区域根目录中的文件。对应 API ContextCompat.getExternalFilesDirs(context, null)。
   - external-cache-path 表示应用程序外部缓存区域根目录中的文件。对应 API ContextCompat.getExternalCacheDirs(context)。
   - external-media-path 表示应用程序外部媒体区域根目录中的文件。对应 API Context.getExternalMediaDirs()。（API 21+）

   注意：root-path 表示设备根目录，可以获取到外置 SD 卡文件。root-path 官方文档里没有提到，但在 FileProvider 源码里有涉及。

   ```java
   public class FileProvider extends ContentProvider {
       private static final String TAG_ROOT_PATH = "root-path";
       private static final String TAG_FILES_PATH = "files-path";
       private static final String TAG_CACHE_PATH = "cache-path";
       private static final String TAG_EXTERNAL = "external-path";
       private static final String TAG_EXTERNAL_FILES = "external-files-path";
       private static final String TAG_EXTERNAL_CACHE = "external-cache-path";
       private static final String TAG_EXTERNAL_MEDIA = "external-media-path";
   }
   ```

   具体“路由映射”逻辑可自行查看 FileProvider 源码中 parsePathStrategy 方法的实现。

3. 运行时获取 content uri。

   FileProvider.getUriForFile() 即可将 file Uri 转换为传输时所需的 content Uri。

   ```java
   public static Uri getContentUri(Context context, String authority, File file) {
       Uri contentUri;
       if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.N) {
           contentUri = FileProvider.getUriForFile(context, authority, file);
       } else {
           contentUri = Uri.fromFile(file);
       }
       return contentUri;
   }
   ```

## 分区存储

存储目录可以参考之前写的文章 [Android 存储目录.md](https://codepoem.fun/2019/05/05/Android%E5%AD%98%E5%82%A8%E7%9B%AE%E5%BD%95/)。

历史问题：App 开发者未遵循规范，滥用公有存储目录，导致用户相册等公有目录下文件混乱不堪。

解决方案：分区存储。

在分区存储的强制规范下，媒体文件要存放到公有目录，就必须直接或间接地使用 MediaStore 的 API 来统一存储到媒体库。

两个规则强制执行，来规范 App 开发者存储文件行为。

1. 强制要求指定相对路径，可以理解为子目录，且必须是公有目录默认媒体目录分类（DCIM、Picture、Music、Video、Download、Document）中的一个。
2. Android 11 强制执行分区存储，即使申请了外部空间读写权限，直接通过 File Uri 来读写也会抛出异常。

适配方案：

- 操作私有目录（包括内部存储私有目录和外部存储私有目录）。
  使用 Context 相关 API，无需权限。

- 操作外部存储公有目录。
  1. SAF （全称 Storage Access Framework，存储访问框架），无需权限即可访问自己或其他 App 的公有媒体文件。SAF 的缺点是界面单调、操作不便、不可定制交互。
  2. 根据 API 适配。
     1. Android 4.4 - 10 仍旧使用 File API Environment.getExternalStorageDirectory()。
        Android 6、0 需要动态申请外部存储权限。
        Android 10 须通过在 manifest 的 Application 节点添加 requestLegacyExternalStorage = true 来声明对 File Api 方式的兼容。
     2. Android 11 使用 MediaStore 读写，访问自己 App 的文件无需权限。读取其他 App 媒体文件需要 READ_EXTERNAL_STORAGE 权限，写入其他 App 媒体文件需要 MANAGE_EXTERNAL_STORAGE 权限。

---

相关文章：
[Android 存储目录.md](https://codepoem.fun/2019/05/05/Android%E5%AD%98%E5%82%A8%E7%9B%AE%E5%BD%95/)
[记录一次 Android7.0 适配 FileProvider 踩坑.md](https://codepoem.fun/2019/11/16/%E8%AE%B0%E5%BD%95%E4%B8%80%E6%AC%A1Android7.0%E9%80%82%E9%85%8DFileProvider%E8%B8%A9%E5%9D%91/)
