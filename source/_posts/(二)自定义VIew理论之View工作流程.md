---
title: (二)自定义VIew理论之View工作流程
date: 2016/12/10 20:46:25
updated: 2019-09-16 13:32:20
categories:
  - Android
tags:
  - Android
  - 自定义View
---

## 一、简述

View 的工作流程主要是指 measure、layout、draw 这三大流程，即测量、布局、绘制。measure 确定 View 的测量宽高；layout 确定 View 的最终宽高和四个顶点的位置；draw 则将 View 绘制到屏幕上。

## 二、measure 过程

### 1. View 的 measure 过程

由 measure()方法来完成，measure()方法是 final 类型的方法，因此不能被重写。而 measure 方法会去调用 onMeasure()方法，因此只需要看 onMeasure()的实现即可。具体可以参看上一节相关内容。

### 2. ViewGroup 的 measure 过程

除了完成自己的 measure 外，递归地调用子元素的 measure 方法

### 3. 注意事项

View 的 measure 过程完成以后就可以通过 getMeasuredWidth/Height 方法来获取 View 的测量宽高。但是在某些极端情况下，系统可能需要多次 measure 过程才能最终确定 View 的测量宽高，因此在这种情况下获取的 View 的测量宽高可能是不准确的。最好在 onLayout()方法中去获取 View 的测量宽高。

## 三、layout 过程

ViewGroup 通过 layout()方法确定子元素的位置，当位置确定以后在 onLayout()方法中遍历所有子元素的 layout()方法，如此递归。
  layout()方法的大致流程：首先通过 setFrame()方法来设置四个顶点的位置，即初始化 mLeft、mRight、mTop、mBottom 这四个值。View 的四个顶点确定了，那么 View 在父容器的位置就确定了；然后调用 onLayout()方法，这个方法的作用是父容器确定子元素的位置。和 onMeasure()方法类似，onLayout()的具体实现同样和具体的布局有关，所以 View 和 ViewGroup 均没有实现 onLayout()方法。

## 四、draw 过程

1. 绘制背景 background.draw(canvas)
2. 绘制自己(onDraw)
3. 绘制 Children(dispatchDraw)
4. 绘制装饰(onDrawScrollBars)
