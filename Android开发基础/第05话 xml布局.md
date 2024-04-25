---
author:
- LTSlw
tags:
- android
date: 2024-04-25
lastmod: 2024-04-25
---

# xml布局

## Android控件

所有的Android控件的基类都是`View`，其中有一个子类`ViewGroup`可以嵌套其他的`View`和`ViewGroup`以实现布局

### 盒子模型

一个控件有外到内分为`margin`、`padding`、`content`三部分，`margin`使控件外围额外占用一些空白，`content`是控件边缘向内收缩使`content`和边缘保持一段距离

- View支持padding，但是不支持margin
- ViewGroup支持padding和margin

## 布局单位

### Density-independent pixels

`Density-independent pixels`，简称`dp`，dp与dpi相关，1dp在160dpi下等价于1px（`px = dp * (dpi / 160)`），目的是在不同像素密度的屏幕上相同控件显示大小一致，一般用于调整控件的大小和间距

### Scalable pixels

`Scalable pixels`简称`sp`，默认情况下`1sp = 1dp`，但会根据用户设置调整，一般用于显示文字

## 颜色

### 预定义颜色

- BLACK
- BLUE
- CYAN
- DKGRAY
- GRAY
- GREEN
- LTGRAY
- MEGENTA
- RED
- TRANSPARENT
- WHITE
- YELLOW

### 十六进制表示

布局文件中可以使用`#RRGGBB`表示颜色，代码中可以使用`0xAARRGGBB`表示

### Color类

导入Color类可以使用更丰富的颜色

```
import android.graphics.Color
```

[相关文档](https://developer.android.com/reference/android/graphics/Color)

## Android Studio中编辑布局

![](imgs/05_00_ui.png)

- `Code`/`Design`/`Split`（右上角）：只查看源xml/只查看可视化的界面/同时查看两者
- `Palette`：可以浏览所有可以使用的控件，拖动到右侧添加控件
- `Attributes`：查看和编辑选中控件的属性，比如宽高、文字大小等
- `Component Tree`：控件列表

> 调整控件宽高时除了指定dp值，还可以使用`wrap_content`和`match_parent`，分别表示恰好包裹content和填充父控件

## 使用代码创建控件

实际开发中，应该使用XML为设计UI的主要方式，代码作为辅助手段

```
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val textView = TextView(this)
        textView.text = "完全用代码创建的界面"
        setContentView(textView)
    }
}
```

### 使用代码的优点

- 完全可控，具有较强的灵活性
- 免去了Inflation过程，性能不受损失（XML定义的界面，最后也是要转换为代码来实现布局的）
- 可读性比较差；容易写出耦合性强的代码，可维护性较低

### 使用xml的优点

- 直观简洁，可读性强
- 实现了UI界面和逻辑代码的初步分离
