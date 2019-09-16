---
title: (一)自定义View理论之View绘制原理
date: 2016/12/9 20:46:25
updated: 2019-09-16 13:32:15
categories:
- Android自定义View
tags:
- Android
- 自定义View
---

# 一、简述
  自定义View是Android进阶之路不可避免的难关。此刻下定决心攻克这个难关，以此为证。在学习绘制自定义View之前先来思考一下系统是如何绘制出这些View的。
  推荐《Android群英传》、《Android开发艺术探索》这两本书，本系列文中很多都来源于这两本的阐述。

# 二、View的测量
  设想这么一个游戏：一个人蒙着眼睛，另一个人通过说话来指示蒙着眼睛的人去画他想画图案。比如你会指导他在画板左上角水平竖直都为10厘米处开始画一个边长为10厘米的正方形，那么他大致就能画准确。而如果你只告诉他画一个矩形，那么他就画不准确了。事实上，Android就是那个蒙着眼睛的人，我们需要细致地高速他怎么绘制。
  生活中，画一个图形必须知道他的大小和位置。同样，Android系统在绘制View前也必须对View进行测量，即告诉系统该画一个多大的View，这个过程在onMeasure()方法中进行。
  Android系统为我们提供了一个设计短小精悍却功能强大的类—— __MeasureSpec__，来帮助我们绘制View。
  __MeasureSpec__是一个32位的int值，高2位表示测量的模式，低30位表示测量的大小。位运算是为了提高效率。
__测量的模式：__
  __EXACTLY__：精确模式(当View宽高设置为具体数值或者指定为match_parent)
  __AT MOST__：最大值模式(当View宽高设置为指定为wrap_content)
  __UNSPECIFIED__：不指定测量大小模式(通常自定义View才会用到)

自定义View测量，重写onMeasure()方法。
系统的super.onMeasure()在最终会调用setMeasureDimension(int measuredWidth， int  measuredHeight)方法将测量的宽高值设置进去，从而完成测量工作。所以在重写onMeasure()方法后，最终要做的工作就是把测量后的宽高值作为参数设置给setMeasureDimension()方法。

模板如下：

```java
class CustomView extends View{
    
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(measureSize(widthMeasureSpec, 200), measureSize(heightMeasureSpec,
                200));
    }
    
    /**
     * 测量View尺寸
     *
     * @param measureSpec 要测量的Spec
     * @param defaultSize 默认View大小
     * @return View尺寸
     */
    private int measureSize(int measureSpec, int defaultSize) {
        int result = defaultSize;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);
    
        switch (specMode) {
            case MeasureSpec.UNSPECIFIED:
                result = defaultSize;
                break;
            case MeasureSpec.AT_MOST:
                result = Math.min(result, specSize);
                break;
            case MeasureSpec.EXACTLY:
                result = specSize;
                break;
        }
        return result;
    }
}
```

# 三、View的绘制
  测量好View后，我们就可以重写onDraw()方法，在Canvas对象上来绘制图案了。无论多复杂、精美的控件，它都可以拆分成一个个小小的图形单元。而那些具体的图形单元的绘制就只需要调用系统的API来完成，这里就不详细展开了。

# 四、ViewGroup的测量
  ViewGroup的宽高指定为warp_content时，需要遍历子View，获得所有子View的大小，从而来决定自己的大小。其他模式则会通过具体的指定值来设置自身的大小。

# 五、ViewGroup的绘制
  ViewGroup通常情况下不需要绘制，因为它本身没有需要绘制的东西。如果不是指定了ViewGroup的背景颜色，那么ViewGroup的onDraw()方法都不会被调用。GroupView会使用dispatchDraw()方法来绘制其子View，其过程同样是通过遍历子View，并调用子View的绘制方法来实现绘制。
