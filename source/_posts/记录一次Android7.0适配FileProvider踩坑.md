---
title: 记录一次Android7.0适配FileProvider踩坑
date: 2019-11-16 11:25:23
updated: 2019-11-16 11:25:27
categories:
  - Android
tags:
  - Android
  - FileProvider
---

- FileProvider 重复

自定义 FileProvider 继承 FileProvider。

```java
public class AppleFileProvider extends FileProvider {
}
```

AndroidManifest.xml 中 application 节点下添加 provider 节点。

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

- 多个 FileProvider authorities 重复

authorities 一般是由包名+自定义的标识构成。

Uri uri = FileProvider.getUriForFile(getContext(), context.getPackageName() + ".file.provider", file);

例子：

AppleFileProvider 和 BoyFileProvider 的 authorities 重复了，都为${applicationId}.file.provider。
默认会取 apk 中合并后的 AndroidManifest.xml 的第一个 authorities 匹配的 FileProvider。

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

<provider
    android:name="com.XXX.XXX.sample.BoyFileProvider"
    android:authorities="${applicationId}.file.provider"
    android:exported="false"
    android:grantUriPermissions="true">

    <meta-data
        android:name="android.support.FILE_PROVIDER_PATHS"
        android:resource="@xml/boy_file_provider"/>

</provider>
```

此时 Uri uri = FileProvider.getUriForFile(getContext(), context.getPackageName() + ".file.provider", file);会与 AppleFileProvider 匹配，也就会去取 apple_file_provider.xml 中的配置。
如果 AppleFileProvider 与 BoyFileProvider 在 AndroidManifest 的顺序互换一下，那么就会与 BoyFileProvider 匹配，也就会去取 boy_file_provider.xml 中的配置。
