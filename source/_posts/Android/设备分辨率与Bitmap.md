---
title: 设备分辨率与Bitmap
categories:
  - Android
keywords: Bitmap
comments: true
date: 2019-02-22 10:28:13
updated: 2019-02-22 10:28:13
tags: Bitmap
description:
---

### 设备分辨率

|density|	0.75|	1	|1.5|	2|	3	|3.5|	4|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|densityDpi	|120|	160|	240|	320	|480	|560|	640|
|DpiFolder	|ldpi	|mdpi	|hdpi	|xhdpi	|xxhdpi	|xxxhdpi	|xxxxhdpi

|Dpi范围|	密度|
|:---:|:---:|
0dpi ~ 120dpi	|ldpi
120dpi ~ 160dpi	|mdpi
160dpi ~ 240dpi	|hdpi
240dpi ~ 320dpi	|xhdpi
320dpi ~ 480dpi	|xxhdpi
480dpi ~ 640dpi	|xxxhdpi

### 内存占用

> Bitmap内存占用 ≈ 像素数据总大小 = 图片宽 × 图片高× (设备分辨率/资源目录分辨率)^2 × 每个像素的字节大小

<!-- more -->

### 单个像素的字节大小

|Config |占用字节大小（byte）|说明|
|:---:|:---:|:---:|
|ALPHA_8 (1)|1|单透明通道|
|RGB_565 (3)|2|简易RGB色调|
|ARGB_4444 (4)|4|已废弃|
|ARGB_8888 (5)|4|24位真彩色|
|RGBA_F16 (6)|8|Android 8.0 新增（更丰富的色彩表现HDR）|
|HARDWARE (7)|Special|Android 8.0 新增 （Bitmap直接存储在graphic memory）注1|


### 不同Android版本时的Bitmap内存模型

|API级别	|API 10 -|	API 11 ~ API 25|	API 26 +|
|:---:|:---:|:---:|:---:|
|Bitmap对象存放	|Java heap|	Java heap	|Java heap|
像素(pixel data)数据存放|	native heap|	Java heap	|native heap

[Android Bitmap变迁与原理解析（4.x-8.x）](https://www.jianshu.com/p/d5714e8987f3)

> 如果没有在AndroidManifest中启用largeheap，那么Java 堆内存达到192M的时候就会崩溃，对于现在动辄4G的手机而言，存在严重的资源浪费，ios的一个APP几乎能用近所有的可用内存（除去系统开支），8.0之后，Android也向这个方向靠拢，最好的下手对象就是Bitmap，因为它是耗内存大户。图片内存被转移到native之后，一个APP的图片处理不仅能使用系统绝大多数内存，还能降低Java层内存使用，减少OOM风险。不过，内存无限增长的情况下，也会导致APP崩溃，但是这种崩溃已经不是OOM崩溃了，Java虚拟机也不会捕获，按道理说，应该属于linux的OOM了。

#### 8.0之后的Bitmap内存回收机制

NativeAllocationRegistry是Android 8.0引入的一种辅助自动回收native内存的一种机制，当Java对象因为GC被回收后，NativeAllocationRegistry可以辅助回收Java对象所申请的native内存

参考资料：   
https://www.jianshu.com/p/3f6f6e4f1c88  
https://www.jianshu.com/p/d5714e8987f3
https://blog.csdn.net/guolin_blog/article/details/50727753 