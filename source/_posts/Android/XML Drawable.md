---
title: XML Drawable
date: 2017-12-29 09:23:21
updated: 2017-12-29 09:23:22
tags:
categories:
    - Android
    - UI系列
keywords:
description:
comments:
---

# 资源文件 XML 的详细介绍

[参考链接](http://blog.csdn.net/crazymo_/article/details/48941781)

<!-- more -->

* StateListDrawable资源

```xml
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 指定获得焦点时的颜色 -->
    <item android:state_focused="true"
    android:color="#f44"/>
    <!-- 指定失去焦点时的颜色 -->
    <item android:state_focused="false"
    android:color="#eee"/>
</selector>
```

* LayerDrawable资源

```xml
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 定义轨道的背景 -->
    <item android:id="@android:id/background"
    android:drawable="@drawable/grow" />
    <!-- 定义轨道上已完成部分的外观-->
    <item android:id="@android:id/progress"
    android:drawable="@drawable/ok" />
</layer-list>
```

* ClipDrawable资源

```xml
<clip xmlns:android="http://schemas.android.com/apk/res/android">
    android:drawable="@drawable/shuangta"//截取的源Drawable对象
    android:clipOrientation="horizontal"//指定截取方向
    android:gravity="center">//指定截取的对齐方式
</clip>
可通过setLevel（int level）方法控制截取的图片部分，0<= level <= 10000，零到整张图。可实现幻灯片特效
```

* 样式资源和主题资源

```xml
<resources>
    <!--定义一个样式，指定字体大小、字体颜色-->
    <style name="style1">
        <item name="android:textSize">20sp</item>
        <item name="android:textColor">#00d</item>
    </style>
    <!--定义一个样式，继承前一个颜色-->
    <style name="style2" parent="@style/style1">
        <item name="android:background">#ee6</item>
        <item name="android:padding">8dp</item>
        <!--覆盖父样式中指定的属性-->
        <item name="android:textColor">#000</item>
    </style>
    <style name="CrazyTheme">
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowFrame">@drawable/window_border</item>
        <item name="android:windowBackground">@drawable/star</item>
    </style>
</resources>
```
