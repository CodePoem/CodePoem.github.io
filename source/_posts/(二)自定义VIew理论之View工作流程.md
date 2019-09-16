---
title: (二)自定义VIew理论之View工作流程
date: 2016/12/10 20:46:25
updated: 2019-09-16 13:32:20
categories:
- Android自定义View
tags:
- Android
- 自定义View
---

# 一、简述
  View的工作流程主要是指measure、layout、draw这三大流程，即测量、布局、绘制。measure确定View的测量宽高；layout确定View的最终宽高和四个顶点的位置；draw则将View绘制到屏幕上。

# 二、measure过程

## 1. View的measure过程
  由measure()方法来完成，measure()方法是final类型的方法，因此不能被重写。而measure方法会去调用onMeasure()方法，因此只需要看onMeasure()的实现即可。具体可以参看上一节相关内容。

## 2. ViewGroup的measure过程
  除了完成自己的measure外，递归地调用子元素的measure方法

## 3. 注意事项
  View的measure过程完成以后就可以通过getMeasuredWidth/Height方法来获取View的测量宽高。但是在某些极端情况下，系统可能需要多次measure过程才能最终确定View的测量宽高，因此在这种情况下获取的View的测量宽高可能是不准确的。最好在onLayout()方法中去获取View的测量宽高。

# 三、layout过程
  ViewGroup通过layout()方法确定子元素的位置，当位置确定以后在onLayout()方法中遍历所有子元素的layout()方法，如此递归。
  layout()方法的大致流程：首先通过setFrame()方法来设置四个顶点的位置，即初始化mLeft、mRight、mTop、mBottom这四个值。View的四个顶点确定了，那么View在父容器的位置就确定了；然后调用onLayout()方法，这个方法的作用是父容器确定子元素的位置。和onMeasure()方法类似，onLayout()的具体实现同样和具体的布局有关，所以View和ViewGroup均没有实现onLayout()方法。

# 四、draw过程

1. 绘制背景background.draw(canvas)
2. 绘制自己(onDraw)
3. 绘制Children(dispatchDraw)
4. 绘制装饰(onDrawScrollBars)