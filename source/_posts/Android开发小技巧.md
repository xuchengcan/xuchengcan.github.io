---
title: Android开发小技巧
categories:
  - Android
comments: true
date: 2017-12-29 10:42:26
updated: 2017-12-29 10:42:26
tags:
keywords:
description:
---

# 一些在工作中会遇到的好方法

<!-- more -->

### 知晓当前在哪个活动之中

新建一个BaseActivity ，在OnCreate中加上

Log.e("BaseActivity", getClass().getSimpleName());

### ~~随时随地退出程序~~

~~新建一个ActivityCollector类作为活动的管理器~~

~~在Activity管理器中维护一个activity的列表，用于activity新建和销毁的记录，实现add和remove两个方法，并实现finishALl方法。~~

~~在BaseActivity的onCreated()中~~

~~ActivityCollector.add(this)~~

~~在onDestroy()中~~

~~ActivityCollector.remove(this)~~

~~在任意Activity中想退出程序都可以调用ActivityCollector.finishAll()~~

不够优雅，改为监听生命周期的方式 ActivityLifecycleCallbacks

### 启动活动的最佳写法

在目标SecondActivity中写下如下方法

```java
public static void activityStart(Context context, String data1, String data2,……){
     Intent intent = new Intent(context , SecondActivity.class);
     intent.putExtra("param1", data1);
     intent.putExtra("param2", data2);
     context.startActivity(intent);
}
```

在首FirstActivity中就可以通过 SecondActivity.activityStart(FirstActivity.this, "xxx" ,"xxx" );

完成启动的写法。这样利于多人协作，调用其他人写的Activity时不用考虑传参细节。

