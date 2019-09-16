---
title: 自定义View
date: 2019-09-16 12:34:07
updated: 2019-09-16 12:34:09
categories:
- Android自定义View
tags:
- Android
- 自定义View
---

# 自定义View

## 什么是自定义View

自定义 View 就是通过继承 View 或者 View 的子类，并在继承的类里面实现自定义的处理逻辑（重写相应的方法），以达到自己想要的效果。

## 为什么要自定义View

* 让界面有特定的显示效果、交互
* 优化布局（减少布局层次）
* 封装

## 如何自定义View

### 自定义View主要流程

1. 测量阶段 measure
2. 布局阶段 layout
3. 绘制阶段 draw

背景：DecorView是视图的顶级View，我们添加的布局文件是它的一个子布局，而ViewRootImpl则负责渲染视图，它调用了一个performTraveals方法使得ViewTree开始三大工作流程（performMeasure、performLayout、performDraw），然后使得View展现在我们面前。

#### 测量过程

ViewRootImpl.performMeasure() -> View.measure() -> View.onMesure()

measure()：调度方法，主要做一些前置和优化工作，并最终会调用 onMeasure() 方法执行实际的测量工作。
onMesure()：实际执行测量任务的方法，主要用与测量 View 尺寸和位置。在自定义 View 的 onMeasure() 方法中，View 根据自己的特性和父 View 对自己的尺寸要求算出自己的期望尺寸，并通过 setMeasuredDimension() 方法告知父 View 自己的期望尺寸。

MeasureSpec：封装View的规格尺寸，由32位整型表示测量模式（高2位）+测量大小（低30位）。

#### 布局过程

ViewRootImpl.performLayout() -> View.layout() -> View.onLayout()

layout：保存 View 的实际尺寸。调用 setFrame() 方法保存 View 的实际尺寸，调用 onSizeChanged() 通知开发者 View 的尺寸更改了，并最终会调用 onLayout() 方法让子 View 布局（因为自定义 View 中没有子 View，所以自定义 View 的 onLayout() 方法是一个空实现）。
onLayout： 空实现，什么也不做，因为它没有子 View。

#### 绘制过程

ViewRootImpl.performDraw() -> View.draw() -> View.onDraw()

draw()：绘制阶段的总调度方法，在其中会调用绘制背景的方法 drawBackground()、绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()。
drawBackground()：绘制背景的方法，不能重写，只能通过 xml 布局文件或者 setBackground() 来设置或修改背景。
onDraw()：绘制 View 主体内容的方法，通常情况下，在自定义 View 的时候，只用实现该方法即可。
dispatchDraw()：绘制子 View 的方法。同 onLayout() 方法一样，在自定义 View 中它是空实现，什么也不做。
onDrawForeground()：绘制 View 前景的方法，也就是说，想要在主体内容之上绘制东西的时候就可以在该方法中实现。

### 自定ViewGroup主要流程

大部分流程和自定义View一致，就不再赘述。这里只列出和自定义View不一样的地方。

1. onMeasure()：在自定义 ViewGroup 的 onMeasure() 方法中，ViewGroup 会递归调用子 View 的 measure() 方法，并通过 measure() 将 ViewGroup 对子 View 的尺寸要求（ViewGroup 会根据开发者对子 View 的尺寸要求、自己的父 View（ViewGroup 的父 View） 对自己的尺寸要求和自己的可用空间计算出自己对子 View 的尺寸要求）传入，对子 View 进行测量，并把测量结果临时保存，以便在布局阶段使用。测量出子 View 的实际尺寸之后，ViewGroup 会根据子 View 的实际尺寸计算出自己的期望尺寸，并通过 setMeasuredDimension() 方法告知父 View（ViewGroup 的父 View） 自己的期望尺寸。

2. layout()：保存 ViewGroup 的实际尺寸。调用 setFrame() 方法保存 ViewGroup 的实际尺寸，调用 onSizeChanged() 通知开发者 ViewGroup 的尺寸更改了，并最终会调用 onLayout() 方法让子 View 布局。

3. onLayout()：ViewGroup 会递归调用每个子 View 的 layout() 方法，把测量阶段计算出的子 View 的实际尺寸和位置传给子 View，让子 View 保存自己的实际尺寸和位置。

4. draw() -> dispatchDraw()：绘制子 View 的方法。在自定义 ViewGroup 中，它会调用 ViewGroup.drawChild() 方法，在 ViewGroup.drawChild() 方法中又会调用每一个子 View 的 View.draw() 让子 View 进行自我绘制。
