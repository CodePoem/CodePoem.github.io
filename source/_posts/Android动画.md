---
title: Android动画
date: 2019-09-16 23:48:10
updated: 2019-09-16 23:48:12
categories:
- Android
tags:
- Android
- 动画
---

Android中动画划分为两大类 视图动画（View Animation）和 属性动画（Property Animation）。其中 属性动画（Property Animation）需要在Android3.0之后（API >= 11）使用。
视图动画（View Animation）又包括了 帧动画（Frame Animation） 和 补间动画（Tweened Animation）。

## 视图动画（View Animation）

### 帧动画（Frame Animation）

帧动画（Frame Animation） 有时也叫 Drawable动画，这种动画的实质其实是Drawable。像播放幻灯片一样，利用视觉残留产生动画效果。

核心类：AnimationDrawable

使用方式：

#### XML定义帧动画

推荐使用XML定义动画：

在res/drawable下新建animation_frame_test.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<animation-list xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">
    <item
        android:drawable="@color/colorPrimary"
        android:duration="300" />
    <item
        android:drawable="@color/colorPrimaryDark"
        android:duration="300" />
</animation-list>
```

#### Java代码创建帧动画

创建AnimationDrawable对象，addFrame(Drawable frame,int duration)向动画中添加帧，调用start()和stop()。

### 补间动画（Tweened Animation）

补间动画（Tweened Animation）只能应用于View对象，而且只支持一部分属性，它只是改变了View对象绘制的位置，而没有改变View对象本身（所以点击响应区域还在原来的位置）。多用于Window切换动画或Activity跳转动画。

核心类：android.view.animation.Animator

补间动画可以分为四种形式，分别是 alpha（淡入淡出），translate（位移），scale（缩放大小），rotate（旋转）。

#### java类

* AlphaAnimation 渐变透明度动画效果
* RotateAnimation 画面转移旋转动画效果
* ScaleAnimation 渐变尺寸伸缩动画效果
* TranslateAnimation 画面转换位置移动动画效果
* AnimationSet 一个持有其它动画元素alpha、scale、translate、rotate或者其它set元素的容器

插值器：

* LinearInterpolator：动画以均匀的速度改变
* AccelerateInterpolator：在动画开始的地方改变速度较慢，然后开始加速
* AccelerateDecelerateInterpolator：在动画开始、结束的地方改变速度较慢，中间时加速
* CycleInterpolator：动画循环播放特定次数，变化速度按正弦曲线改变： Math.sin(2  mCycles  Math.PI * input)
* DecelerateInterpolator：在动画开始的地方改变速度较快，然后开始减速
* AnticipateInterpolator：反向，先向相反方向改变一段再加速播放
* AnticipateOvershootInterpolator：开始的时候向后然后向前甩一定值后返回最后的值
* BounceInterpolator： 跳跃，快到目的值时值会跳跃，如目的值100，后面的值可能依次为85，77，70，80，90，100
* OvershottInterpolator：回弹，最后超出目的值然后缓慢改变到目的值

#### XML

在res/anim下新建animation_tweened_test_rotate.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromDegrees="0"
    android:interpolator="@android:anim/accelerate_decelerate_interpolator"
    android:pivotX="50%"
    android:pivotY="50%"
    android:repeatCount="infinite"
    android:toDegrees="360" />
```

AnimationUtils.loadAnimation(Context context, @AnimRes int id)创建Animation，然后给指定的View调用view.startAnimation(Animation animation) view.clearAnimation()

## 属性动画（Property Animation）

属性动画实现原理就是修改控件的属性值实现的动画。

核心类：android.animation.Animator

* Animator 创建属性动画的基类
* ValueAnimator  Animator子类。其内部采用一种时间循环的机制来计算值与值之间的动画过度，我们只需将初始值以及结束值提供给该类，并告诉其动画所需时间长度，该类就会自动帮我们从初始值平滑过度到结束。该类还能管理动画的播放次数、模式和监听器等。
* ObjectAnimator  ValueAnimator的子类，允许我们对指定对象的属性执行动画，用起来更简单，实际中用得较多。
* AnimatorSet  Animator子类，可以组合多个Animator，并制定Animator的播放次序。
* Interpolator 插值器，同补间动画。
* Evaluator 计算器，告诉动画系统如何从初始值过度到结束值。提供了一下的几种Evaluator：

IntEvaluator：用于计算int属性
FloatEvaluator：用于计算float属性
ArgbEvaluator：用于计算16进制表示颜色值的计算器
TypeEvaluator：上述计算类的公共接口，可以自己实现接口完成自定义

监听器：

AnimatorUpdateListener： 当值状态发生改变时候会回调onAnimationUpdate方法。

AnimatorListener、AnimatorListenerAdapter：

* onAnimationStart()：动画开始
* onAnimationRepeat()：动画重复执行
* onAnimationEnd()：动画结束
* onAnimationCancel()：动画取消

平常开发，属性动画多使用ValueAnimator和ObjectAnimator。
ValueAnimator有个缺点，就是只能对数值对动画计算。
为了能让动画直接与对应控件相关联，以使我们从监听动画过程中解放出来，可以使用ValueAnimator的子类ObjectAnimator。

使用ObjectAnimator实现动画也有一些要求和限制：

1. 动画显示的属性必须带有一个 setter 方法（以骆驼拼写法命名）。 因为 ObjectAnimator 会在动画期间自动更新属性值，它必须能够用此 setter 方法访问到该属性。 例如：假设属性名称为color，则需要有一个setColor()方法。
2. 如果在调用 ObjectAnimator 的某个工厂方法时，我们只为 values... 参数指定了一个值，那此值将被认定为动画属性的结束值。 这样的话，动画显示的属性必须带有一个 getter 方法，用于获取动画的起始值。例如：假设属性名为color，则需要有一个getColor()方法。
3. 动画属性的 getter 方法（如果必要的话）和 setter 方法所操作数据的类型必须与 ObjectAnimator 中设定的起始和结束值相同。
