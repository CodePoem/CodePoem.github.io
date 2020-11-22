---
title: ViewPager下Fragment预加载与懒加载
date: 2020-01-11 11:16:48
updated: 2020-01-11 11:16:52
categories:
  - Android
tags:
  - ViewPager
  - Fragment
  - 预加载
  - 懒加载
---

## 预加载

ViewPager 控件有一个预加载机制，即默认情况下当前页面左右两侧的 1 个页面会被预加载，以方便用户滑动切换到相邻的界面时，更流畅地加载界面（节省了初始化时间）。

从源码里可以看到，ViewPager 的预加载机制是不可取消的，预加载数量 limit 至少为 1，如果外部设置小于 1，内部会自动置为 1。

```java
public class ViewPager extends ViewGroup {

    // 省略其他代码

    public void setOffscreenPageLimit(int limit) {
        if (limit < 1) {
            Log.w("ViewPager", "Requested offscreen page limit " + limit + " too small; defaulting to " + 1);
            limit = 1;
        }

        if (limit != this.mOffscreenPageLimit) {
            this.mOffscreenPageLimit = limit;
            this.populate();
        }
    }
}
```

## 懒加载

在 ViewPager 预加载的机制下，ViewPager 本身比较难以实现 Fragment 实例化懒加载。既然 Fragment 的实例化难以懒加载，那么退而求其次，只要求实现 Fragment 请求数据的懒加载。

我们需要从 Fragment 本身入手。

主要用到 Fragment 两个方法：

- public void onViewCreated(View view, @Nullable Bundle savedInstanceState)
- public void setUserVisibleHint(boolean isVisibleToUser)

```java
public abstract class BaseFragment {
    protected boolean isViewCreated;
    protected boolean isVisibleToUser;
    protected boolean isDataInitiated;

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        isViewCreated = true;
        lazyLoad();
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        this.isVisibleToUser = isVisibleToUser;
        if (isVisibleToUser) {
            lazyLoad();
        }
    }

    public boolean lazyLoad() {
        return lazyLoad(false);
    }

    public boolean lazyLoad(boolean forceRefresh) {
        if (isVisibleToUser && isViewCreated && (!isDataInitiated || forceRefresh)) {
            fetchData();
            isDataInitiated = true;
            return true;
        }
        return false;
    }

    protected abstract void fetchData();
}
```
