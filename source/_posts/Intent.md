---
title: Intent
categories:
  - Android
comments: true
date: 2017-12-29 10:27:14
updated: 2017-12-29 10:27:14
tags:
keywords:
description:
---

# 介绍 Intent 的一些特殊使用（未完待续）

<!-- more -->

### 1、&lt;intent-filter&gt;与Action和Category项（动作和附加类别信息）

例如:

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

> 一个Intent对象最多只能包含一个Action属性，通过setAction()来设置属性值；
>
> 一个Intent对象可以包含多个Category属性，通过addCategory()来添加Category属性
>
> 但是在AndroidManifest.xml中，Activity里的&lt;intent-filter&gt;可以有多个Action属性和Category属性

当Intent被创建时，该Intent默认启动Category属性值为`Intent.CATEGORY_DEFAULT`属性（常量值为`android.intent.category.DEFAULT`）的组件。

实际上，Android内部提供了大量标准Action、Catetory常量。（电话、短信、web）

### 2、&lt;intent-filter&gt;与Data和Type项

Data属性通常用于向Action属性提供操作的数据。Data数据接受一个Uri对象。

Type属性用于指定该Data所指定Uri对应的MIME类型（可为任意自定义类型，须符合abc/xyz格式即可）

注：

> 两属性直接存在相互覆盖的情况（按设置顺序，后者覆盖前者），若要两者同时存在，则使用setDataAndType()方法。

### 3、Extra属性

该属性通常用于多个Activity之间进行数据交换，Intent的Extra属性值应该是一个Bundle对象。

### 4、Flag属性

该属性用于为该Intent添加一些额外的控制旗标。



