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

### 启动方式

#### 不论调用了多少次startService()方法，你只需要调用一次stopService()来停止服务。  

**使用context.startService() 启动Service时会经历:**

context.startService() -&gt;onCreate()- &gt;onStart()-&gt;Service running

context.stopService() | -&gt;onDestroy() -&gt;Service stop

如果Service还没有运行，则android先调用onCreate()然后调用onStart()；如果Service已经运行，则只调用onStart()，所以一个Service的onStart方法可能会重复调用多次。

stopService的时候直接onDestroy，如果是调用者自己直接退出而没有调用stopService的话，Service会一直在后台运行。该Service的调用者再启动起来后可以通过stopService关闭Service。

所以调用startService的生命周期为：onCreate --&gt; onStart(可多次调用) --&gt; onDestroy

**特点：**  
一旦服务开启就跟调用者（开启者）没有任何关系了。开启者退出了，开启者挂了，服务还在后台长期的运行，开启者不能调用服务里面的方法。

**使用context.bindService() 启动Service时会经历:**

context.bindService()-&gt;onCreate()-&gt;onBind()-&gt;Service running

onUnbind() -&gt; onDestroy() -&gt;Service stop

onBind将返回给客户端一个IBind接口实例，IBind允许客户端回调服务的方法，比如得到Service运行的状态或其他操作。这个时候把调用者（Context，例如Activity）会和Service绑定在一起，Context退出了，Srevice就会调用onUnbind-&gt;onDestroy相应退出。

所以调用bindService的生命周期为：onCreate --&gt; onBind(只一次，不可多次绑定) --&gt; onUnbind --&gt; onDestory。

在Service每一次的开启关闭过程中，只有onStart可被多次调用(通过多次startService调用)，其他onCreate，onBind，onUnbind，onDestory在一个生命周期中只能被调用一次。

**特点：**  
bind的方式开启服务，绑定服务，调用者挂了，服务也会跟着挂掉。绑定者可以调用服务里面的方法。

### service的两种模式（startService()/bindService()不是完全分离的）  

#### 本地服务 Local Service 用于应用程序内部。

它可以启动并运行，直至有人停止了它或它自己停止。在这种方式下，它以调用Context.startService()启动，而以调用Context.stopService()结束。它可以调用Service.stopSelf() 或 Service.stopSelfResult()来自己停止。不论调用了多少次startService()方法，你只需要调用一次stopService()来停止服务。用于实现应用程序自己的一些耗时任务，比如查询升级信息，并不占用应用程序比如Activity所属线程，而是单开线程后台执行，这样用户体验比较好。

> 调用者和service在同一个进程里，所以运行在主进程的main线程中。所以不能进行耗时操作，可以采用在service里面创建一个Thread来执行任务。service影响的是进程的生命周期，讨论与Thread的区别没有意义。
> 任何 Activity 都可以控制同一 Service，而系统也只会创建一个对应 Service的实例。

#### 远程服务 Remote Service 用于android系统内部的应用程序之间。

它可以通过自己定义并暴露出来的接口进行程序操作。客户端建立一个到服务对象的连接，并通过那个连接来调用服务。连接以调用Context.bindService()方法建立，以调用 Context.unbindService()关闭。多个客户端可以绑定至同一个服务。如果服务此时还没有加载，bindService()会先加载它。可被其他应用程序复用，比如天气预报服务，其他应用程序不需要再写这样的服务，调用已有的即可。

> 调用者和service不在同一个进程中，service在单独的进程中的main线程，是一种垮进程通信方式

#### IntentService

IntentService是Service的子类，比普通的Service增加了额外的功能。先看Service本身存在两个问题：

* Service不会专门启动一条单独的进程，Service与它所在应用位于同一个进程中；
* Service也不是专门一条新线程，因此不应该在Service中直接处理耗时的任务；

IntentService特征:

* 会创建独立的worker线程来处理所有的Intent请求；
* 会创建独立的worker线程来处理onHandleIntent()方法实现的代码，无需处理多线程问题；
* 所有请求处理完成后，IntentService会自动停止，无需调用stopSelf()方法停止Service；
* 为Service的onBind()提供默认实现，返回null；
* 为Service的onStartCommand提供默认实现，将请求Intent添加到队列中；




参考资料： 
https://www.jianshu.com/p/51aaa65d5d25