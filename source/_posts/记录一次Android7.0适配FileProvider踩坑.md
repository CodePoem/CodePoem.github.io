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

* FileProvider重复

自定义FileProvider继承FileProvider。

```AppleFileProvider
public class AppleFileProvider extends FileProvider {
}
```

AndroidManifest.xml中application节点下添加provider节点。

```File Provider
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

* 多个FileProvider authorities重复

authorities一般是由包名+自定义的标识构成。

Uri uri = FileProvider.getUriForFile(getContext(), context.getPackageName() + ".file.provider", file);

例子：

AppleFileProvider 和 BoyFileProvider 的authorities重复了，都为${applicationId}.file.provider。
默认会取apk中合并后的AndroidManifest.xml的第一个authorities匹配的FileProvider。

```File Provider
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

此时Uri uri = FileProvider.getUriForFile(getContext(), context.getPackageName() + ".file.provider", file);会与AppleFileProvider匹配，也就会去取apple_file_provider.xml中的配置。
如果AppleFileProvider与BoyFileProvider在AndroidManifest的顺序互换一下，那么就会与BoyFileProvider匹配，也就会去取boy_file_provider.xml中的配置。
