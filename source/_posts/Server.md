---
title: Server
categories:
  - Android
  - 四大组件
comments: true
date: 2017-12-29 10:58:46
updated: 2017-12-29 10:58:46
tags:
keywords:
description:
---

Service可以在和多场合的应用中使用，比如播放多媒体的时候用户启动了其他Activity这个时候程序要在后台继续播放，比如检测SD卡上文件的变化，再或者在后台记录你地理信息位置的改变等等，总之服务嘛，总是藏在后头的。

Service是在一段不定的时间运行在后台，不和用户交互应用组件。每个Service必须在manifest中 通过&lt;service&gt;来声明。可以通过contect.startservice和contect.bindserverice来启动。

Service和其他的应用组件一样，运行在进程的主线程中。这就是说如果service需要很多耗时或者阻塞的操作，需要在其子线程中实现。

<!-- more -->

### service的两种模式（startService()/bindService()不是完全分离的）

* 本地服务 Local Service 用于应用程序内部。

它可以启动并运行，直至有人停止了它或它自己停止。在这种方式下，它以调用Context.startService()启动，而以调用Context.stopService()结束。它可以调用Service.stopSelf() 或 Service.stopSelfResult()来自己停止。不论调用了多少次startService()方法，你只需要调用一次stopService()来停止服务。用于实现应用程序自己的一些耗时任务，比如查询升级信息，并不占用应用程序比如Activity所属线程，而是单开线程后台执行，这样用户体验比较好。

* 远程服务 Remote Service 用于android系统内部的应用程序之间。

它可以通过自己定义并暴露出来的接口进行程序操作。客户端建立一个到服务对象的连接，并通过那个连接来调用服务。连接以调用Context.bindService()方法建立，以调用 Context.unbindService()关闭。多个客户端可以绑定至同一个服务。如果服务此时还没有加载，bindService()会先加载它。可被其他应用程序复用，比如天气预报服务，其他应用程序不需要再写这样的服务，调用已有的即可。

### 启动方式

> 不论调用了多少次startService()方法，你只需要调用一次stopService()来停止服务。

* 使用context.startService() 启动Service是会会经历:

context.startService() -&gt;onCreate()- &gt;onStart()-&gt;Service running

context.stopService() | -&gt;onDestroy() -&gt;Service stop

如果Service还没有运行，则android先调用onCreate()然后调用onStart()；如果Service已经运行，则只调用onStart()，所以一个Service的onStart方法可能会重复调用多次。

stopService的时候直接onDestroy，如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行。该Service的调用者再启动起来后可以通过stopService关闭Service。

所以调用startService的生命周期为：onCreate --&gt; onStart(可多次调用) --&gt; onDestroy



* 使用context.bindService() 启动Service是会会经历:

context.bindService()-&gt;onCreate()-&gt;onBind()-&gt;Service running

onUnbind() -&gt; onDestroy() -&gt;Service stop

onBind将返回给客户端一个IBind接口实例，IBind允许客户端回调服务的方法，比如得到Service运行的状态或其他操作。这个时候把调用者（Context，例如Activity）会和Service绑定在一起，Context退出了，Srevice就会调用onUnbind-&gt;onDestroy相应退出。

所以调用bindService的生命周期为：onCreate --&gt; onBind(只一次，不可多次绑定) --&gt; onUnbind --&gt; onDestory。

在Service每一次的开启关闭过程中，只有onStart可被多次调用(通过多次startService调用)，其他onCreate，onBind，onUnbind，onDestory在一个生命周期中只能被调用一次。

