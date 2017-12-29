---
title: AsyncTask
categories:
  - Android
  - 进程线程
date: 2017-12-29 10:08:27
updated: 2017-12-29 10:08:27
tags: 
keywords:
description:
comments:
---

# AsyncTask 的介绍与使用

<!-- more -->

AsyncTask&lt;&gt;是一个抽象类，通常用于被继承。适用于简单的异步处理，不需要借助线程和Handler即可实现。

它定义了如下三种泛型类型。

* Params：启动任务执行的输入参数的类型

* Progress：后台任务完成的进度值的类型

* Result：后台执行任务完成后返回结果的类型

### 使用方式

1. 创建AsyncTask的子类，并为三个泛型参数指定类型。如果某个泛型参数不需要指定类型，可将它指定为Void

2. 根据需要可使用如下方法

   * doInBackground(Params…)：重写该方法就是后台线程将要完成的任务。该方法可以调用publiceProgress ( Progress…values)方法更新任务的执行进度。

   * onProgressUpdate (Progress…values)：在doInBackground()方法中调用publicProgress()方法更新任务的执行进度后，将会触发该方法。

   * onPreExecute()：该方法将在执行后台耗时操作前被调用。通常该方法用于完成一些初始化的准备工作，比如在界面上显示进度条。

   * onPostExecute( Result result)：当doInBackground()完成后，系统会自动调用该方法，并将doInBackground()的返回值传给该方法

3. 调用AsyncTask子类的实例的execute(Params…params)开始执行耗时任务

### 注：

* 必须在UI线程中创建AsyncTask的实例

* 必须在UI线程中调用AsyncTask的execute()方法

* AsyncTask的onPreExecute()、onPostExecute(Result result)、doInBackground(Params…params),onProgressUpdate(Progress…values)方法，不应该由程序员代码调用，要由系统调用

* 每个AsyncTask只能被执行一次，多次调用将会引发异常

### AsyncTask中的三个泛型参数的类型与四个方法中参数的关系

第一个参数类型为Params的泛型：

> AsyncTask子类的实例调用execute()方法中传入参数类型，参数为该类型的可变长数组doInBackground()方法的参数类型，参数为该类型的可变长数组

第二个参数类型为Progress的泛型:

> 在doInBackground方法中手动调用publishProgress方法中传入参数的类型，参数为该类型的可变长数组
>
> onProgressUpdate()方法中的参数类型，参数为该类型的可变长数组

第三个参数类型为Result的泛型:

> doInBackground()方法的返回值的类型
>
> onPostExecute方法的参数的类型









