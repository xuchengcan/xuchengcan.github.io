---
title: Android冷知识
categories:
  - Android
comments: true
date: 2017-12-29 10:44:17
updated: 2017-12-29 10:44:17
tags:
keywords:
description:
---

# 记录一些在面试题中可能出现的冷知识
持续更新

<!-- more -->

1、 Android应用的所有UI组件都继承了View类，View类还有一个重要的子类：VIewGroup，但ViewGroup通常作为其他组件的容器使用。

2、 ListView、GridView、Spinner、Gallery等AdapterView都只是容器，而Adapter负责提供每个“列表项”组件，AdapterView则负责采用合适的方式显示这些列表项。

3、 Android已经不推荐使用Gallery组件，推荐使用其他水平滚动组件HorizontalSorollView和ViewPager

4、 Android系统包含两套事件处理系统

* 基于监听的事件处理

沿袭Java方式有：

> 内部类形式：将事件监听器类定义成当前类的内部类
>
> 外部类形式：将事件监听器类定义成一个外部类
>
> Activity本身作为事件监听器类：让Activity本身实现监听器接口，并实现事件处理方法
>
> 匿名内部类形式：使用匿名内部类创建事件监听器对象

Android方式有：

> 直接绑定到android界面组件标签

* 基于回调的事件处理

重写View控件，对回调函数进行处理

* 基于回调的事件传播

如果处理事情的回调方法返回true，则表明改处理方法已完全处理该事件，该事件不会传播出去。反之false则会继续传播下去

**Button组件实践：**Android系统最先触发的是Button上绑定的事件监听器，接着才触发该组件提供的事件的回调方法，然后还会传播到该组件所在的Activity。

**顺序**：事件监听器—&gt;组件回调方法—&gt; Activity

对比Android提供的两种事件处理模型，不难发现基于监听的事件处理模型具有更大的优势：

* 基于监听的事件模型分工更明确，事件源、事件监听由两个类分开实现，因此具有更好的可维护性。
* Android的事件处理机制保证基于监听的事件监听器会被优先触发。

5、 Android的UI操作并不是线程安全的。故Android制定了一条简单的规则：只允许UI线程修改Activity里的UI组件。Android不允许在新的线程中访问Activity里面的界面组件。

6、若应用程序没有声明任何一个活动作为主活动，这个程序仍然可以正常安装。只是你无法在启动器中看到或者打开这个程序。这种程序一般作为第三方服务供其他应用在内部进行调用的，如支付宝快捷支付服务。

7、 在Manifest中对Activity进行theme定义，可将activity显示模式变成对话框形式。

```xml
<activity 
     android:name=".xxxx"
     android:theme="@android:style/Theme.Dialog">
```

8、res/raw和assets

* 相同点

> 两者目录下的文件在打包后会原封不动的保存在apk包中，不会被编译成二进制。

* 不同点

> res/raw中的文件会被映射到R.java文件中，访问的时候直接使用资源ID即R.id.filename；
>
> assets文件夹下的文件不会被映射到R.java中，访问的时候需要AssetManager类。
>
> res/raw不可以有目录结构，而assets则可以有目录结构，也就是assets目录下可以再建立文件夹

* 读取文件资源

读取res/raw下的文件资源，通过以下方式获取输入流来进行写操作

`InputStream is =getResources().openRawResource(R.id.filename);`

读取assets下的文件资源，通过以下方式获取输入流来进行写操作

```java
AssetManager am = null;
am = getAssets();
InputStream is = am.open("filename");
```

* 注

> 1：Google的Android系统处理Assert有个bug，在AssertManager中不能处理单个超过1MB的文件，不然会报异常，raw没这个限制可以放个4MB的Mp3文件没问题。
>
> 2：assets 文件夹是存放不进行编译加工的原生文件，即该文件夹里面的文件不会像 xml， java 文件被预编译，可以存放一些图片，html，js, css 等文件。

9、 Mipmap图像

APK分包时，mipmap资源会全部包含在APK文件中，所以Android推荐将启动器图标放进mipmap目录中

