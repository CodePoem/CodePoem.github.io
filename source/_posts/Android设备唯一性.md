---
title: Android设备唯一性
date: 2019-08-28 23:21:55
updated: 2019-08-28 23:21:58
categories:
  - Android
tags:
  - Android
  - App
---

如何标识唯一的 Android 设备呢？

## 硬件标识符

### IMEI

国际移动设备身份码(International Mobile Equipment Identity)，是由 15 位数字组成的"电子串号"，它与每台手机一一对应，而且该码是全世界唯一的。

IMEI 所需权限：

```IMEI所需权限
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
```

### MEID

全球唯一的 56bit CDMA 制式移动终端标识号（ Mobile Equipment IDentifier ）。标识号会被烧入终端里，并且不能被修改。可用来对 CDMA 制式移动式设备进行身份识别和跟踪。

### ESN

电子序列号（electronic serial number），在 CDMA 系统中，是鉴别一个物理硬件设备唯一的标识。

### DEVICE_ID

不同的手机设备返回上述的 IMEI，MEID 或者 ESN 码。

### MAC 地址

设备网卡的唯一设别码，该码全球唯一，一般称为物理地址，硬件地址用来定义设备的位置。

MAC 地址所需权限：

```MAC地址所需权限
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
```

**PS：** Android6.0 及以上 8.0 以下获取不到准确的 MAC 地址。8.0 及以上获取不到 MAC 地址。

### ANDROID_ID

在设备首次运行的时候，系统会随机生成一 64 位的数字，并把这个数值以 16 进制保存下来，这个 16 进制的数字就是 ANDROID_ID，但是如果手机恢复出厂设置这个值会发生改变。

**PS：** 国内厂商可能不提供 ANDROID_ID。

### SN

设备序列号（Serial Number）。

**PS：** 国内厂商不提供 SN 可能为垃圾数据。

### 自定义全局唯一 ID (GUID)

自定义全局唯一 ID，通常采用随机算法生成。

```GUID
UUID.randomUUID().toString();
```

## 总结

硬件标识符没有哪一个能说是准确且可靠的，因此通常都是使用上述几个的组合来标识唯一的 Android 设备（组合的时候可以加一些自定义的算法，如截取定长、MD5、Base64 等）。
