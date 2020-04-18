---
title: (三)自定义View实践之自定义属性
date: 2016/12/11 20:46:25
updated: 2019-09-16 13:32:24
categories:
- Android
tags:
- Android
- 自定义View
---

# 一、定义与声明

在资源文件夹values下新建attr.xml文件

```
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <attr name="text" format="string"/>
    <attr name="textColor" format="color"/>
    <attr name="textSize" format="dimension"/>

    <declare-styleable name="CodeView">
        <attr name="text"></attr>
        <attr name="textColor"></attr>
        <attr name="textSize"></attr>
    </declare-styleable>

</resources>
```

其中CodeView是自定义styleable名，可以随意取
format有以下几种：
1. reference：参考某一资源ID
2. color：颜色值
3. boolean：布尔值
4. dimension：尺寸值
5. float：浮点值
6. integer：整型值
7. string：字符串
8. fraction：百分数
9. enum：枚举值
10. flag：位或运算

# 二、具体使用

具体的解析需要自行来做对应的修改（以我自己写的parseAttr方法为例子）：
自定义View的构造器中调用parseAttr()自定义解析并处理。

```java
class CustomView extends View {
    
    public CodeView(Context context) {
        this(context, null);
    }
    
    public CodeView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CodeView(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        parseAttr(context, attrs);
    }
    
    /**
     * 解析自定义属性
     *
     * @param context 调用方上下文
     * @param attrs 属性集
     */
    private void parseAttr(Context context, @Nullable AttributeSet attrs) {
        TypedArray attributes = context.obtainStyledAttributes(attrs, R.styleable.CodeView);
        int num = attributes.getIndexCount();
        for (int i = 0; i < num; i++) {
            int attr = attributes.getIndex(i);
            switch (attr) {
                case R.styleable.CodeView_text:
                    mText = attributes.getString(attr);
                    break;
                case R.styleable.CodeView_textColor:
                    //默认设置为黑色
                    mTextColor = attributes.getColor(attr, Color.BLACK);
                    break;
                case R.styleable.CodeView_textSize:
                    //默认设置为16sp，TypeValue也可以把sp转化为px
                    mTextSize = attributes.getDimensionPixelSize(attr, (int) TypedValue
                            .applyDimension(TypedValue.COMPLEX_UNIT_SP, 16, getResources()
                            .getDisplayMetrics()));
                    break;
            }
        }
        attributes.recycle();
    }
}
```