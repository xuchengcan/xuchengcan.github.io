---
title: IPC 机制（上）
categories:
  - Android
  - IPC
comments: true
date: 2018-01-07 12:23:56
updated: 2018-01-07 12:23:56
tags: Android 开发艺术探索
keywords:
description:
---

# 本文为 《Android 开发艺术探索》 第二章-IPC机制 的笔记上篇

<!-- more -->

## 简介

IPC 是 Inter-Process Communication 的缩写，含义为进程间通信或者跨进程通信，是指两个进程直接进行数据交换的过程。

## Android 中的多进程模式

### 开启多进程模式

给四大组件在 AndroidMenifest 中指定 android:process 属性，除此之外没有其他办法，也就是说我们无法给一个线程或者一个实体类指定其运行时所在的进程。
另：非常规方式，通过 JNI 在 native 层去 fork 一个新的进程。

``` xml
        <!-- 包名加":remote"-->
        <!-- 属于当前应用的私有进程 -->
        <!-- 其他应用不可以和他跑在同一个进程中 -->
        <activity
            android:name=".activity.TestActivity1"
            android:process=":remote" />

        <!-- 完整进程名-->
        <!-- 全局进程 -->
        <!-- 其他应用可以通过 ShareUID 方式可以和它跑在同一个进程中 -->
        <activity
            android:name=".activity.TestActivity2"
            android:process="com.chennuo.remote" />
```

> 可以通过 `adb shell ps` 命令来查看进程信息

我们知道 Android 系统会为每个应用分配一个唯一的 UID， 具有相同 UID 的应用才能共享数据。这里要说明的是，两个应用通过 ShareUID 跑在同一个进程中是有要求的，需要这两个应用有相同的 ShareUID 并且签名相同才可以。在这种情况下，它们可以互相访问对方的私有数据，比如 data 目录、组件信息等，不管它们是否跑在同一个进程中。当然如果它们跑在同一个进程中，那么除了能共享 data 目录、组件信息，还可以共享内存数据，或者它们看起来就像是一个应用的两个部分。

### 多进程模式的运行机制

一般来说，使用多进程会造成如下几方面的问题：
- 静态成员和单例模式完全失效
- 线程同步机制完全失效
- SharedPreferences 的可靠性下降
- Application 会多次创建

## IPC 基础概念介绍

### Serializable 接口

> 声明 serialVersionUID 非必须，但是不声明 serialVersionUID 会对反序列化过程带来影响。

序列化的时候，系统会把当前类的 serialVersionUID 写入序列化的文件中（也可能是其他的中介），当反序列化的时候，系统会去检测文件中的 serialVersionUID ，看是否和当前类的一致。如果一致则说明两者版本相同。否则，说明当前类和序列化的类发生了某些变换，会报 `java.io.InvalidClassException` 错误

> 我们可以给 serialVersionUID 指定为 1L 或者由 AS 自动生成 serialVersionUID 的 hash 值，两者没有本质区别。

注：

- 1.静态成员变量属于类不属于对象，所以不会参与序列化过程
- 2.用 transient 关键字标记的成员变量不参与序列化过程

### Parcelable 接口

> Serializable 是 Java 中的序列化接口，其使用简单但是开销大，序列化和反序列化都需要大量的 I/O 操作。
> Parcelable 是 Android 中的序列化方式，更适合用于 Android 平台。
> 对比两者，Parcelable 主要用于内存序列化上。Serializable 更适合将序列化到存储设备或者将对象序列化后通过网络传输。

### Binder

Android 开发中，Binder 主要用在 Service 中，包括 AIDL 和 Messenger，其中普通 Service 中的 Binder 不涉及进程间通信，而 Messenger 的底层其实是 AIDL。所以我们选择用 AIDL 来分析 Binder 的工作机制。

#### 分析 AIDL

当客户端和服务端都位于同一个进程时，方法调用不会走跨进程的 `transact` 过程，而当两者位于不同的进程时，方法调用需要走 `transact` 过程，这个逻辑由 Stub 的内部代理类 Proxy 来完成。这个接口的核心实现就是它的内部类 Stub 和 Stub 的内部代理类 Proxy。

**DESCRIPTOR**
Binder 的唯一标识，一般用当前 Binder 的类名表示。

**asInterface(android.os.IBinder obj)**
用于将服务端的 Binder 对象转换成客户端所需的 AIDL 接口类型的对象，这种转换过程是区分进程的。如果两端位于同一进程，则返回服务端的 Stub 对象本身，否则返回的是系统封装后的 Stub.proxy 对象

```java
        /**
         * Cast an IBinder object into an chen.easyview.aidl.IBookManager interface,
         * generating a proxy if needed.
         */
        public static chen.easyview.aidl.IBookManager asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof chen.easyview.aidl.IBookManager))) {
                return ((chen.easyview.aidl.IBookManager) iin);
            }
            return new chen.easyview.aidl.IBookManager.Stub.Proxy(obj);
        }
```

**asBinder**
用于返回当前 Binder 对象

**onTransact**
该方法运行在服务端中的 Binder 线程池中，当客户端发起跨进程请求时，远程请求会通过系统底层封装后交由此方法来处理。如果此方法返回 false，那么客户端的请求会失败，因此我们可以利用这个特性来做权限验证

```java
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(DESCRIPTOR);
                    return true;
                }
                case TRANSACTION_getBookList: {
                    data.enforceInterface(DESCRIPTOR);
                    java.util.List<chen.easyview.aidl.Book> _result = this.getBookList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                case TRANSACTION_addBook: {
                    data.enforceInterface(DESCRIPTOR);
                    chen.easyview.aidl.Book _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = chen.easyview.aidl.Book.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addBook(_arg0);
                    reply.writeNoException();
                    return true;
                }
            }
            return super.onTransact(code, data, reply, flags);
        }
```

**Proxy#getBookList**

```java
            @Override
            public java.util.List<chen.easyview.aidl.Book> getBookList() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<chen.easyview.aidl.Book> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getBookList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(chen.easyview.aidl.Book.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
```

这个方法运行在客户端，当客户端远程调用此方法时，会调用 transact 方法来发起 RPC（远程过程调用） 请求，同时当前线程挂起；然后服务端的 onTransact 方法会被调用，直到 RPC 过程返回后，当前线程继续执行，并从 _reply 中取出 RPC 过程的返回结果；最后返回 _reply 中的数据、

#### 自己写 AIDL

AIDL 不是实现 Binder 的必需品。如果是我们手写的 Binder，那么在服务端只需要创建一个 BookManagerImpl 的对象并在 Service 的 onBind 方法中返回即可。

### Binder 的两个重要方法 `linkToDeath` `unlinkTODeath`

> 这两个方法用于监听 Binder 连接是否断裂。

通过 linkToDeath 我们可以给 Binder 设置一个死亡代理，当 Binder 死亡时，我们就会收到通知。首先，我们声明一个 DeathRecipient 对象，其为接口，内部只有一个方法 binderDied，我们需要实现这个方法，当 Binder 死亡的时候，系统就会回调 binderDied 方法，然后我们就可以移除之前绑定的 binder 代理并重新绑定远程服务。

```java
// 写在客户端中

    IBookManager mBookManager;

    private IBinder.DeathRecipient mDeathRecipient = new IBinder.DeathRecipient() {
        @Override
        public void binderDied() {
            if(mBookManager == null){
                return;
            }
            mBookManager.asBinder().unlinkToDeath(mDeathRecipient,0);
            mBookManager = null;
            bindService(new Intent("chen.easyview.aidl.IBookManager").setPackage("chen.easyview")
            ,conn, BIND_AUTO_CREATE);
        }
    };

    private ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mBookManager = IBookManager.Stub.asInterface(service);
            try {
                service.linkToDeath(mDeathRecipient,0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

// 另通过 Binder 的方法 isBinderAlive 也可以判断 Binder 是否死亡。
```

> 以上为 IPC 的基础知识，下篇将继续记录 Android 中的进程间通信