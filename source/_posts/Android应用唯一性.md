---
title: Android应用唯一性
date: 2019-08-27 23:39:27
updated: 2019-08-27 23:39:30
categories:
- App
tags:
- Android
- App
---

# Android应用唯一性

说到应用唯一性，下面三个关键词，有何区别和联系呢？

* 包名
* applicationId
* 签名

## 包名

### 定义简介

包名指的是软件包名称（代码命名空间）。就是我们平常写代码import的package。

### 配置示例

```包名
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.example.myapp">
```

### 更改包名

Android 构建工具使用 package 属性来发挥两种作用：

* 它会将此package名称用作应用生成的 R.java 类的命名空间。如：com.example.myapp.R
* 它会使用此package名称解析清单文件中声明的任何相关类名。如：声明为 <activity android:name=".MainActivity"> 的 Activity 将解析为 com.example.myapp.MainActivity

因此package 属性中的名称应始终与项目的基础软件包名称匹配，基础软件包中保存着您的 Activity 及其他应用代码。（如果它们未保持同步，您的应用代码将无法解析 R 类，因为它不再位于同一软件包中，并且清单无法识别您的 Activity 或其他组件）

**注意：** 如果未在 build.gradle 文件中定义 applicationId 属性，构建工具会将 AndroidManifest.xml 文件中的软件包名称用作applicationId。在这种情况下，重构软件包名称也会更改applicationId。

## applicationId

### 定义简介

applicationId是每个应用的唯一应用Id，看起来就和软件包名称一样。此 ID 可以在设备上和 Google Play 商店中对应用进行唯一标识(实际上，Google Play 商店和 Android 平台会查看包名 package 属性来识别您的应用，构建工具会在编译结束时将applicationId 复制替换到 APK 的最终清单文件中)。也就是说，上传新版本的应用，applicationId必须与原始APK相同，否则会被认为是两个完全不同的应用。因此发布应用后， **绝不应该更改applicationId** 。

### 配置示例

```applicationId
android {
    defaultConfig {
        applicationId "com.example.myapp.XXX"
    }
}
```

**PS：** 如果需要在清单文件中引用applicationId，可以在任何清单属性中使用 ${applicationId} 占位符，在编译期间，Gradle 会将此标记替换为实际的applicationId。

### 命名规则限制

* 必须至少包含两段（一个或多个圆点）
* 每段必须以字母开头
* 所有字符必须为字母数字或下划线[a-zA-Z0-9_]

## 签名

### 定义简介

Android 要求所有 APK 必须先使用证书进行数字签署，然后才能安装。这个签署的过程就是我们说的签名。

* 公钥证书（也称为数字证书或身份证书），包含公钥/私钥对的公钥，以及标识密钥所有者的一些其他元数据（例如名称和位置）。证书的所有者持有对应的私钥。
* 在签署 APK 时，签署工具会将公钥证书附加到 APK，用于创建此证书的密钥称为应用签名密钥。
* 密钥库，一种包含一个或多个私钥的二进制文件。
* 每个应用在其整个生命周期内必须使用相同证书，以便用户能够以应用更新的形式安装新版本。

### 配置示例

```签名配置
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            // You need to specify either an absolute path or include the
            // keystore file in the same directory as the build.gradle file.
            storeFile file("my-release-key.jks")
            storePassword "password"
            keyAlias "my-alias"
            keyPassword "password"
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            ...
        }
    }
}
```

debug模式下，Android Studio会在$HOME/.android/debug.keystore 中自动创建调试密钥库和证书，并设置密钥库和密钥密码。

在Android Studio 生成密钥：

<center>

![New Key Store](/assert/img/app/appuniqueness/keystore-example.png)

</center>

密钥库：

* Key store pat：选择创建密钥库的位置。
* Password： 为密钥库创建并确认安全的密码。

密钥：

* Alias：密钥标识名。
* Password：为密钥创建并确认安全的密码。此密码应当与为密钥库选择的密码不同。
* Validity (years)：以年为单位设置密钥的有效时长。密钥的有效期应至少为 25 年，以便您可以在应用的整个生命期内使用相同的密钥签署应用更新。
* Certificate：为证书输入一些关于自己的信息。此信息不会显示在应用中，但会作为 APK 的一部分包含在证书中。

### keytool 命令行签名工具

keytool 位于 JDK 的 bin/ 目录中。

* 生成签名 keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias
* 查看签名 jks keytool -list -v -keystore my-release-key.jks -storepass my-storepass

## 总结

1. applicationId是应用的唯一标识。发布应用后 **绝不应该更改applicationId** 。
2. 尽管applicationId和包名在设置时完全一样，但 **applicationId和包名彼此无关，两者更改互不影响** 。
3. applicationId过去直接关联到代码的软件包名称；所以，有些 Android API 会在其方法名称和参数名称中使用“package name”一词，但这实际上是指applicationId，如 **Context.getPackageName() 方法返回的是applicationId** 。
4. 无论何时都不需要在应用代码以外分享代码的真实软件包名称。对外暴露的只有applicationId。
5. 构建工具会在 **编译结束时将applicationId 复制替换到 APK 的最终清单文件中** ，编译系统利用原始值包名（设置 R 类的命名空间并解析清单类名称）后，它会舍弃该值（包名）并将其替换为applicationId。
6. 个人推荐applicationId和包名一致或者以包名为前缀。
7. Android 要求所有 APK 必须先使用证书进行数字签署，然后才能安装。
8. Android Studio默认会自动生成debug签名，签名在$HOME/.android/debug.keystore。

综上所述，不同的APP，applicationId一定不相同，包名和签名可以相同也可以不相同。
