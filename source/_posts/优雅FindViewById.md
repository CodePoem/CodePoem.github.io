---
title: 优雅FindViewById
date: 2020-04-18 13:06:23
updated: 2020-11-23 02:39:17
categories:
  - Android
tags:
  - Android
  - FindViewById
---

## findBiewById

findBiewById 是 Android 开发中在布局中查找 View 元素的 Api。

### findBiewById 基本使用

```java
class DemoActivity extends AppCompatActivity {
  TextView mTextDemo;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_demo);
    mTextDemo = (TextView) findViewById(R.id.tv_demo);
    mTextDemo.setText("Demo");
  }
}
```

因为写起来很繁琐（而且还需要手动强转类型），所以逐渐出现了各种简化或者代替它的方式。

### 省略强转

从 [Android Support Library 26.0.0 Beta 1](https://developer.android.google.cn/topic/libraries/support-library/rev-archive.html) 开始 findViewById 将不再需要强转了。

findViewById() 方法的所有实例现在会返回 \<T extends View\> T，而不是 View。

```java
class DemoActivity extends AppCompatActivity {
  TextView mTextDemo;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_demo);
    mTextDemo = findViewById(R.id.tv_demo);
    mTextDemo.setText("Demo");
  }
}
```

### findBiewById 原理

findBiewById 原理实质上是递归遍历查找匹配 Id 的 View。

Activity 中 findViewId 方法会调用获取 Window -> DecoreView -> View 的 findBiewById 方法。

最终会调用到 View 中的 findViewTraversal 方法。方法名看上去是遍历操作，在 View 类中找不到遍历逻辑；
实际上 ViewGroup 覆写了 View 的 findViewTraversal 方法，实现了递归遍历查找匹配 View 的方法。

```java
class View {
    @Nullable
    public final <T extends View> T findViewById(@IdRes int id) {
        if (id == NO_ID) {
            return null;
        }
        return findViewTraversal(id);
    }

    protected <T extends View> T findViewTraversal(@IdRes int id) {
        if (id == mID) {
            return (T) this;
        }
        return null;
    }
}
class ViewGroup {

  /**
   * {@hide}
   */
  @Override
  protected <T extends View> T findViewTraversal(@IdRes int id) {
        if (id == mID) {
            return (T) this;
        }

        final View[] where = mChildren;
        final int len = mChildrenCount;

        for (int i = 0; i < len; i++) {
            View v = where[i];

            if ((v.mPrivateFlags & PFLAG_IS_ROOT_NAMESPACE) == 0) {
                v = v.findViewById(id);

                if (v != null) {
                    return (T) v;
                }
            }
        }

        return null;
    }
}
```

## [ButterKnife](https://jakewharton.github.io/butterknife/)

ButterKnife 是 jakewharton 大神开源作品，用于替代 findViewById ，避免繁琐的写法。

### ButterKnife 基本使用

```java
class DemoActivity extends AppCompatActivity {
  @BindView(R.id.tv_demo)
  TextView mTextDemo;

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_demo);
    ButterKnife.bind(this);
    mTextDemo.setText("Demo");
  }
}
```

### ButterKnife 原理

注解编译时生成绑定类，代替我们完成 FindViewById 的操作。

PS：ButterKnife Github ReadMe 中说明已不推荐使用，推荐下文提到的 Android 官方提供的 [ViewBinding](https://developer.android.com/topic/libraries/view-binding)

## [Data Binding Library](https://developer.android.google.cn/topic/libraries/data-binding/index.html)

### Data Binding 基本使用

布局需要使用 \<layout\> 标签包裹。

```java
class DemoActivity extends AppCompatActivity {

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.demo_activity);
    ActivityDemoBinding binding = DataBindingUtil.setContentView(this, R.layout.activity_demo);
    binding.tvDemo.setText("Demo");
  }
}
```

### Data Binding 原理

自动查找所有 View 并缓存到 binding 实例中以供访问。性能超过手写的 findViewById，因为它只遍历了一遍 XML 布局，而 findViewById 每次都会去遍历 XML 布局；include 布局中的 view 也能同样能访问，并且保留结构。

## Kotlin Android Extensions

直接生成对应的 View 作为属性，不需要 findViewById，不需要定义变量，直接使用。使用时需要注意访问的 View 属于哪个 Layout，因为智能提示的候选项会提供所有布局中的 View 供你选择，然后帮你 import 对应包以便你访问这个 View；假如 import 的多个同一层级的 layout 中具有相同的 id，则这个 id 对应的 View 将无法访问。

### Kotlin Android Extensions synthetic 基本使用

```kotlin

import kotlinx.android.synthetic.main.activity_demo.*

class DemoActivity : AppCompatActivity() { {

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.demo_activity);
    tvDemo.setText("Demo");
  }
}
```

### Kotlin Android Extensions synthetic 原理

Kotlin 会自动生成类似 findViewById() 的方法：findCachedViewById()，在这个方法里面创建一个 HashMap 缓存每次查找到的 View，避免每次调用 View 的属性或方法时都会重新调用 findCachedViewById() 进行查找。

PS：在 [Kotlin 1.4.20-M2](https://github.com/JetBrains/kotlin/releases/tag/v1.4.20-M2) 中，JetBrain s废弃了 Kotlin Android Extensions 编译插件。推荐使用 ViewBinding。

## (推荐使用)[ViewBinding](https://developer.android.com/topic/libraries/view-binding)

### ViewBinding 基本使用

Android Studio 3.6 Canary 11 及更高版本中可用。

```gradle
android {
  ...
  viewBinding {
      enabled = true
  }
}
```

```kotlin
class DemoActivity : AppCompatActivity() { {

  private lateinit var binding: ActivityDemoBinding

  @Override public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    binding = ActivityDemoBinding.inflate(layoutInflater)
    setContentView(binding.root)
    binding.tvDemo.setText("Demo");
  }
}
```

### ViewBinding 原理

### ViewBinding 优缺点

#### 优点

- Null 安全：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null 指针异常的风险。此外，如果视图仅出现在布局的某些配置中，则绑定类中包含其引用的字段会使用 @Nullable 标记。

- 类型安全：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型。这意味着不存在发生类转换异常的风险。

- 更快的编译速度：视图绑定不需要处理注释，因此编译时间更短。

- 易于使用：视图绑定不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用视图绑定后，它会自动应用于该模块的所有布局。

#### 缺点与限制

- 布局和代码之间的不兼容性可能会导致编译版本在编译时（而非运行时）失败。
- 视图绑定不支持布局变量或布局表达式，因此不能用于直接在 XML 布局文件中声明动态界面内容。
- 视图绑定不支持双向数据绑定。

---

相关文章

[使用视图绑定替代 findViewById](https://juejin.im/post/5e69cb55e51d4526d87c8610#heading-4)
[你好, View Binding! 再次再见, findViewById!](https://juejin.im/post/5dd407066fb9a020366f85fa#heading-5)
[Kotlin 干掉了 findViewById，但用不好也会有性能问题](https://juejin.im/entry/5d8caedd518825093a3579b0)
[Migrating the deprecated Kotlin Android Extensions compiler plugin](https://proandroiddev.com/migrating-the-deprecated-kotlin-android-extensions-compiler-plugin-to-viewbinding-d234c691dec7)
[【译】迁移被废弃的Kotlin Android Extensions插件](https://blog.csdn.net/qq_17766199/article/details/109557820)
