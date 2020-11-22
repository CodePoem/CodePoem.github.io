---
title: 优雅FindViewById
date: 2020-04-18 13:06:23
updated: 2020-11-22 19:40:24
categories:
- Android
tags:
- Android
- FindViewById
---

findBiewById 是 Android 开发中在布局中查找 View 元素的 Api。

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

## 省略强转

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

## [ButterKnife](https://jakewharton.github.io/butterknife/)

注解编译时生成绑定类，代替我们完成FindViewById的操作。

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

PS：ButterKnife Github ReadMe 中说明已不推荐使用，推荐下文提到的 Android 官方提供的 [ViewBinding](https://developer.android.com/topic/libraries/view-binding)

## [Data Binding Library](https://developer.android.google.cn/topic/libraries/data-binding/index.html)

自动查找所有 View 并缓存到 binding 实例中以供访问。性能超过手写的 findViewById，因为它只遍历了一遍 XML 布局，而 findViewById 每次都会去遍历 XML 布局；include 布局中的 view 也能同样能访问，并且保留结构。

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

## Kotlin Android Extensions

直接生成对应的 View 作为属性，不需要 findViewById，不需要定义变量，直接使用。使用时需要注意访问的 View 属于哪个 Layout，因为智能提示的候选项会提供所有布局中的 View 供你选择，然后帮你 import 对应包以便你访问这个 View；假如 import 的多个同一层级的 layout 中具有相同的 id，则这个 id 对应的 View 将无法访问。

底层使用类似 findViewById 的转换来找到对应的控件，并且有缓存机制。

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

## [ViewBinding](https://developer.android.com/topic/libraries/view-binding)

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

- Null 安全：由于视图绑定会创建对视图的直接引用，因此不存在因视图 ID 无效而引发 Null 指针异常的风险。此外，如果视图仅出现在布局的某些配置中，则绑定类中包含其引用的字段会使用 @Nullable 标记。
- 类型安全：每个绑定类中的字段均具有与它们在 XML 文件中引用的视图相匹配的类型。这意味着不存在发生类转换异常的风险。

布局和代码之间的不兼容性可能会导致编译版本在编译时（而非运行时）失败。

---
相关文章

[使用视图绑定替代 findViewById](https://juejin.im/post/5e69cb55e51d4526d87c8610#heading-4)
[你好, View Binding! 再次再见, findViewById!](https://juejin.im/post/5dd407066fb9a020366f85fa#heading-5)
[Kotlin干掉了findViewById，但用不好也会有性能问题](https://juejin.im/entry/5d8caedd518825093a3579b0)
