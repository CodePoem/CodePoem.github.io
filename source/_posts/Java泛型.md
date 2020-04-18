---
title: Java泛型
date: 2019-09-10 07:49:26
updated: 2019-09-10 07:49:29
categories:
- Java
tags:
- Java
- 泛型
---

# Java泛型

## 概述

泛型：参数化类型（将类型由原来的具体的类型参数化）。

特点：

* 适用于多种数据类型执行相同的代码。
* 使用泛型时，在实际使用之前类型就已经确定了，不需要强制类型转换。
* 泛型只在编译阶段有效。

泛型命名标识字母（常见字母）：

* T Type
* K V Key Value
* E Element

## 简单使用

### 泛型类

```泛型类
class 类名称<泛型命名标识> {
  private 泛型命名标识 /*（成员变量类型）*/ var;
  .....
}
```

### 泛型接口

```泛型接口
public interface 接口名称<泛型命名标识> {
    public 泛型命名标识 method();
}
```

### 泛型方法

```泛型方法
public class Method {
  public static <泛型命名标识> void method(泛型命名标识 var){
      .....
  }
```

## 类型擦除

类型擦除：Java的泛型机制是在编译级别实现的。编译器生成的字节码在运行期间并不包含泛型的类型信息。
泛型信息只存在于代码编译阶段，在进入 JVM 之前，与泛型相关的信息会被擦除掉。

## 上界&下界/协变&逆变

无界：通配符?，表示泛型类型是一个未知类型。
上界：上界通配符extends，限制了泛型未知类型的上界。
下界：下界通配符super，限制了泛型未知类型的下界。

型变：协变、逆变和不变的统称，用来描述类型转换后的继承关系。比如：Sub是Super的子类型，List<Sub>是否是List<Super>的子类型。

协变（covariance）：满足Sub是Super的子类型，List<Sub>也是List<? extends Super>的子类型。
逆变（covariance）：满足Sub是Super的子类型，List<Super>也是List<? super Sub>的子类型。
不变（invariance）：表示Sub是Super的子类型，List<Sub>和List<Super>不存在型变关系。

ps：子类(subclass)和子类型(subtype)不是同一个概念。子类说明了新类是继承自父类，而子类型强调的是新类具有父类一样的行为（未必是继承，还有可能是实现接口）。
