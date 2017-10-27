---
title: 第4章面向切面的Spring
date: 2016/11/23 20:46:25
updated: 2016/11/23 20:46:25
categories:
- Spring实战第4版读书笔记
---
# 一、什么是面向切面编程

## (1)简述
如果要重用通用功能的话，最常见的面向对象技术是继承或委托。继承往往会导致一个脆弱的对象体系；委托则可能需要委托对象进行复杂的调用。而切面提供了另外一种方案。

## (2)定义AOP术语
* __通知(advice):__定义了切面是__什么__以及__何时__使用。

![通知类型](http://upload-images.jianshu.io/upload_images/3828003-4ca159200bd2ef47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

* __切点(pointcut):__定义了切面在__何处__。
* __连接点(join point):__在应用程序执行过程中能够插入切面的一个点。这个点可以是调用方式时、抛出异常时、甚至修改一个字段时。
![联系图](http://upload-images.jianshu.io/upload_images/3828003-59ae2d1bbf0b0f4f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## (3)Spring对AOP的支持
Spring只支持__方法级别__的连接点。如果你的AOP需求超过了简单的方法调用(如构造器或属性拦截)，那么你需要考虑AspectJ来实现切面。
![Spring对AOP的支持类型](http://upload-images.jianshu.io/upload_images/3828003-01fc09e75f492e5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 二、通过切点来选择连接点
![Spring借助AspectJ的切点表达式语言来定义Spring切面](http://upload-images.jianshu.io/upload_images/3828003-00888045047903b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

exxcution指示器是实际执行匹配的，其他指示器都是用来限定切点的。

# 三、使用注解创建切面

## (1)定义切面
__@AspectJ注解，表示一个POJO不仅仅是一个POJO，还是一个切面。__

![Spring使用AspectJ注解来声明通知方法](http://upload-images.jianshu.io/upload_images/3828003-01c45864d71a8963.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

样例：

![切面样例](http://upload-images.jianshu.io/upload_images/3828003-7cbe28c2d2e42327.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__@Pointcut注解可以声明频繁使用的切点表达式__

![省略频繁使用的切点表达式](http://upload-images.jianshu.io/upload_images/3828003-a78f1c16ac5b417d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__光是完成上述步骤还不够，还需要启动AspectJ注解的自动代理。__

![JavaConfig](http://upload-images.jianshu.io/upload_images/3828003-e8b59c3ea35c26d5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![XML配置](http://upload-images.jianshu.io/upload_images/3828003-fb95aec402a73e1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## (2)环绕通知
![环绕通知](http://upload-images.jianshu.io/upload_images/3828003-ef738c4def743a0f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__当你需要把控制权交给被通知的方法时，它需要调用ProceedingJoinPoint的proceed()方法。如果不调用这个方法，就会阻塞被通知的方法。__

## (3)处理通知中的参数
![声明参数](http://upload-images.jianshu.io/upload_images/3828003-f1593074872e4ac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![样例](http://upload-images.jianshu.io/upload_images/3828003-77353f49644e1b8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## (4)通过注解引入新功能
__在Spring中，切面只是实现了它们所包装bean相同接口的代理，如果除了实现这些接口，代理也能暴露新接口的话，就相当于实现了新功能。__
![原理图](http://upload-images.jianshu.io/upload_images/3828003-c9f3e618b34b4c78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__@DeclareParents注解的使用来实现引入新功能:__
第一步：假设我们要在所有的Performance实现引入以下接口：
![准备引入的新接口](http://upload-images.jianshu.io/upload_images/3828003-7cbb7fad149bbca9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二步：我们需要创建一个新的切面
![创建的切面](http://upload-images.jianshu.io/upload_images/3828003-4bc0596b73d4d3e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![@DeclareParents注解解释](http://upload-images.jianshu.io/upload_images/3828003-e901055a0af68a0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步：在Spring应用中声明这个bean

![声明bean](http://upload-images.jianshu.io/upload_images/3828003-206f07876486ba22.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
到此，我们就完成了引入新功能，是不是很简单。

# 四、在XML中声明切面
__面向注解的切面声明有一个明显的劣势：你必须能够为通知类添加注解，也就是说，你必须有源码。__
__因此尽管基于注解的配置要优于基于Java的配置，基于Java的配置要优于基于XML的配置,但是，如果你需要要声明切面，又不能为通知类添加注解的时候，你就需要XML配置了。__

![Spring的AOP配置元素以非侵入式的方式声明切面](http://upload-images.jianshu.io/upload_images/3828003-afef47bb2de4ea50.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

样例：


![纯净的POJO](http://upload-images.jianshu.io/upload_images/3828003-6d7b30e272cd9052.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![XML将无注解的Audience声明为切面](http://upload-images.jianshu.io/upload_images/3828003-b0994618bb791938.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

__同基于AspectJ注解的通知中的@Pointcut注解一样，XML配置中的<aop:pointcut>元素可以用来减少频繁定义的切点。__
![省略频繁定义的切点](http://upload-images.jianshu.io/upload_images/3828003-4eaf66833ea5b77a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

XML配置声明环绕通知、为通知传递参数、通过切面引入新功能就不过多阐述，过于累赘。

# 五、注入AspectJ切面
AspectJ提供了Spring AOP所不能支持的许多类型的切点。
em...这一小节没看懂，跳。。。过。。。吧。