---
title: IPC机制（下）
categories:
  - Android
  - IPC
comments: true
date: 2018-01-09 21:07:08
updated: 2018-01-09 21:07:08
tags: Android 开发艺术探索
keywords:
description:
---

# 本文为 《Android 开发艺术探索》 第二章-IPC机制 的笔记下篇

<!-- more -->

## Android 中的 IPC 方式

### Bundle

Activity、Service、Receiver 都是支持在 Intent 中传递 Bundle 数据的。例如相机

### 使用文件共享

将某些类序列化成文件存储，通过文件共享信息

其中使用 SharedPreferences 需要注意，SharedPreferences 是 Android 中采用的轻量级存储方案，它底层通过 XML 文件来存储键值对。但是，由于系统对其读写有一定的缓存策略，即内存中会有一份 SharedPreferences 文件的缓存，因此在多进程模式下，其工作不可靠，面对高并发的读写访问，SharedPreferences 有可能会丢失数据。

### 使用 Messenger 信使

一种轻量级的 IPC 方案，底层实现为 AIDL。

服务端：构建包含 Handler 的 Messenger ，在 Handler 里面对 Message 进行处理。通过 `onBind` 返回一个包含服务端调用的 Binder 对象。

客户端：在 `onServiceConnected(ComponentName className, IBinder service)` 中拿到服务端的调用 `mService = new Messenger(service)`， Messenger(IBinder service) 内部实现为 `IMessenger.Stub.asInterface(service)`

若客户端需要对服务端的通信做出回应，则在发送的时候构建处理返回消息的 Messenger，并在通过 msg.replyto = mMessenger 赋值在传送过去的 Message 对象中，服务端通过获取 msg.replyto 重新获取 Messenger 对象，并对其进行操作，从而完成客户端与服务端之间的通信。

### 使用 AIDL

Messenger 的作用主要是为了传递消息，而很多时候我们需要跨进程调用服务端的方法，这种情况下 Messenger 无法做到，但是我们可以通过 AIDL 来实现跨进程的方法调用。AIDL 也是 Messenger 的底层实现。

#### 服务端

服务端首先要创建一个 Service 用来监听客户端的连接请求，然后创建一个 AIDL 文件，将暴露给客户端的接口在这个 AIDL 文件中声明，最后在 Service 中实现这个 AIDL 接口即可。

#### 客户端

首先需要绑定服务端的 Service，绑定成功后，将服务端返回的 Binder 对象转换成 AIDL 接口所属的类型，接着就可以调用 AIDL 中的方法。

#### AIDL 接口的创建

支持类型如下

- 基本数据类型（int、long、char、boolean、double等）
- String 和 CharSequence
- List：只支持 ArrayList，里面每个元素都必须能够被 AIDL 支持
- Map：只支持 HashMap，里面每个元素都必须能够被 AIDL 支持，包括 key 和 value
- Parcelable：所有实现了 Parcelable 接口的对象
- AIDL：所有的 AIDL 接口本身也可以在 AIDL 文件中使用

其中自定义的 Parcelable 对象和 AIDL 对象必须要显示 import 进来，不管是否在同一个包中。如果 AIDL 文件中用到了自定义的 Parcelable 对象，那么必须新建一个和它同名的 AIDL 文件，并在其中声明它为 Parcelable 类型。除此之外，AIDL 中除了基本数据类型，其他类型的参数必须标上方向： `in、out、inout` 用以表示数据流向

#### 服务端和客户端的实现

简单的 C/S 模式下，AIDL 中 List 数据的传递，我们在 Service 中采用 CopyOnWriteArrayList 来完成，它支持并发读/写。我们知道  `AIDL 方法是在服务端的 Binder 线程池中执行的` ，因此当多个客户端同时连接的时候，会存在多个线程同时访问的情形，所以我们要在 AIDL 方法中处理线程同步，而 CopyOnWriteArrayList 能帮我们进行自动的线程同步。

> 前面我们提过 AIDL 中能够使用的 List 只有 ArrayList，但是我们这里却使用 CopyOnWriteArrayList （它并不继承于 ArrayList）。Binder 中会按照 List 的规范去访问数据并最终形成一个新的 ArrayList 传递给客户端。与此类似的还有 ConcurrentHashMap。

我们还可以使用 AIDL 接口，完成观察者模式。对应的应用例如系统服务的监听（定位...）

对象的跨进程传输本质都是反序列化的过程，所以 AIDL 中自定义对象都必须要实现 Parcelable 接口。
RemoteCallbackList 是系统专门提供的用于删除跨进程 listenr 的接口。RemoteCallbackList 是一个泛型，支持管理任意 AIDL 接口。当客户端进程终止后，它能够自动移除客户端所注册的 listener，且内部实现了线程同步的功能。
遍历 RemoteCallbackList 需要如下方式

```java
final int N = mListenerList.beginBroadcast();
        for (int i = 0; i < N; i++) {
            IOnNewBookArrivedListener listener = mListenerList.getBroadcastItem(i);
            if (listener != null){
                try {
                    listener.onNewBookArrived(book);
                }catch (RemoteException e){
                    e.printStackTrace();
                }
            }
        }
        mListenerList.finishBroadcast();
```

#### AIDL 特别点

- 客户端调用远程服务的方法，被调用的方法运行在 Binder 线程池中，所以服务端本身可以执行大量耗时的操作。客户端线程在调用的时候会被挂起，所以调用耗时服务端方法时需另开线程完成。
- 客户端的 onServiceConnected 和 onServiceDisconnected 方法都运行在 UI 线程中，不要在其中调用服务端的耗时方法，否则会导致 ANR。
- 服务端同样不能调用客户端中的耗时方法
- 服务端调用客户端的方法，客户端的方法运行在客户端的 Binder 线程池中，所以不能在里面去访问 UI 相关的内容，需使用 Handler 切换到 UI 线程。

#### Binder 死亡处理

方法一：通过 DeathRecipient 监听，在 binderDied 方法中重连，具体内容见上篇。（binderDied 在客户端的 Binder 线程池中被回调）

方法二：在 onServiceDisconnected 中重连远程服务。（在客户端的 UI 线程中被回调）

#### AIDL 权限验证

> 定义 permission

```xml
    <permission
        android:name="chennuo.easyview.aidl.permissson.ACCESS_BOOK_SERVICE"
        android:protectionLevel="normal" />
```

> 使用 permission

```xml
    <uses-permission android:name="chennuo.easyview.aidl.permissson.ACCESS_BOOK_SERVICE" />
```

> 服务端 onBind 验证 permission

```java
    @Override
    public IBinder onBind(Intent intent) {
        //验证权限
        int check = checkCallingOrSelfPermission("chennuo.easyview.aidl.permissson.ACCESS_BOOK_SERVICE");
        //未授权
        if (check == PackageManager.PERMISSION_DENIED){
            return null;
        }
        return mBinder;
    }
```

### 使用 ContentProvider

通过数据库的方式进行进程间通信

### Socket

## Binder 连接池

通过创建一个 Service 即可完成多个 AIDL 接口的工作