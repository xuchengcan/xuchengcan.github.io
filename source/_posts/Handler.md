---
title: Handler 详解
categories:
  - Android
  - 进程线程
date: 2017-12-29 10:24:25
updated: 2017-12-29 10:24:25
tags:
keywords:
description:
comments: true
---

> Android的UI操作并不是线程安全的。故Android制定了一条简单的规则：只允许UI线程修改Activity里的UI组件。Android不允许在新的线程中访问Activity里面的界面组件。

为了让主线程能“适时”地处理新启动的线程所发送的消息，显然只能通过回调的方式来实现——开发者只要重写Handler类中处理消息的方法，当新启动的线程发送消息时，消息会发送到与之关联的MessageQueue，而Handler会不断从MessageQueue中获取并处理消息——这将导致Handler类中处理消息的方法被回调。

<!-- more -->

Handler类的主要作用有两个：

1. 在新启动的线程中发送消息。
2. 在主线程中获取、处理消息。

### 关键类

* Looper：

每个线程只有一个Looper，它负责管理MessageQueue，会不断的从MessageQueue中取出消息，并将消息分给对应的Handler处理。

* MessageQueue：

有Looper负责管理。它采取先进先出的方式来管理Message。

* Handler：

它能把消息发送给Looper管理的MessageQueue，并负责处理Looper分给它的消息。

---

如果希望Handler正常工作，必须在当前线程中有一个Looper对象，为了保证当前线程中有Looper对象，可以分如下两种情况处理。

1. 主UI线程中，系统已经初始化了一个Looper对象，因此程序直接创建Handler即可。
2. 如果是自己启动的子线程，就必须自己创建一个Looper对象，并启动它。创建Looper对象调用它的Prepare()方法即可。（Prepare()方法保证每个线程最多只有一个Looper对象）

```java
    Public void run(){
        //创建Looper
        //prepare()方法中通过ThreadLocal对象实现Looper实例与线程的绑定（源码）
        Looper .prepare();
        //创建Handler子类实例
        Handler mHandler = new Handler()   {…………};
        //启动Looper
        Looper .loop(); 
    }
```

* Looper.prepare()

```java
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

可以看到，首先判断sThreadLocal中是否已经存在Looper了，如果还没有则创建一个新的Looper设置进去。这样也就完全解释了为什么我们要先调用Looper.prepare()方法，才能创建Handler对象。同时也可以看出每个线程中最多只会有一个Looper对象。

* Looper.loop()

```java
    /**
     * Run the message queue in this thread. Be sure to call
     * {@link #quit()} to end the loop.
     */
    public static void loop() {
        final Looper me = myLooper();
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            try {
                msg.target.dispatchMessage(msg);
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
    }
```

Looper.loop()中一直在不断的从消息队列中通过MessageQueue的next方法获取Message，然后通过代码msg.target.dispatchMessage(msg)让该msg所绑定的Handler（Message.target）执行dispatchMessage方法以实现对Message的处理。

```java
    /**
     * Handle system messages here.
     */
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

我们可以看到Handler提供了三种途径处理Message，而且处理有前后优先级之分：首先尝试让postXXX中传递的Runnable执行，其次尝试让Handler构造函数中传入的Callback的handleMessage方法处理，最后才是让Handler自身的handleMessage方法处理Message。

---

### 另外除了发送消息之外，我们还有以下几种方法可以在子线程中进行UI操作：

1. Handler.post()
2. View.post()
3. Activity.runOnUiThread()

我们先来看下Handler中的post()方法，代码如下所示：

```java
    /**
     * Causes the Runnable r to be added to the message queue.
     * The runnable will be run on the thread to which this handler is 
     * attached. 
     *  
     * @param r The Runnable that will be executed.
     * 
     * @return Returns true if the Runnable was successfully placed in to the 
     *         message queue.  Returns false on failure, usually because the
     *         looper processing the message queue is exiting.
     */
    public final boolean post(Runnable r)
    {
       return  sendMessageDelayed(getPostMessage(r), 0);
    }

    private static Message getPostMessage(Runnable r) {
        Message m = Message.obtain();
        m.callback = r;
        return m;
    }

    /**
     * Enqueue a message into the message queue after all pending messages
     * before (current time + delayMillis). You will receive it in
     * {@link #handleMessage}, in the thread attached to this handler.
     *  
     * @return Returns true if the message was successfully placed in to the 
     *         message queue.  Returns false on failure, usually because the
     *         looper processing the message queue is exiting.  Note that a
     *         result of true does not mean the message will be processed -- if
     *         the looper is quit before the delivery time of the message
     *         occurs then the message will be dropped.
     */
    public final boolean sendMessageDelayed(Message msg, long delayMillis)
    {
        if (delayMillis < 0) {
            delayMillis = 0;
        }
        return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
    }
```

上面调用了sendMessageDelayed()方法去发送一条消息，并且还使用了getPostMessage()方法将Runnable对象转换成了一条消息。

getPostMessage() 方法中将消息的callback字段的值指定为传入的Runnable对象。

这个callback字段看起来有些眼熟，我们回Handler，在dispatchMessage()方法中原来有做一个检查，如果Message的callback等于null才会去调用handleMessage()方法，否则就调用handleCallback()方法。下面是handleCallback()方法中的代码

```java
    private static void handleCallback(Message message) {
        message.callback.run();
    }
```

我们可以看到。这里直接调用了一开始传入的Runnable对象的run()方法。因此在子线程中通过Handler的post()方法进行UI操作就可以这么写：

```java
        handler = new Handler();
        new Thread(new Runnable() {
            @Override
            public void run() {
                handler.post(new Runnable() {
                    @Override
                    public void run() {
                        //在这里进行UI操作
                    }
                });
            }
        }).start();

//回见
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```

虽然写法上相差很多，但是原理是完全一样的，我们在Runnable对象的run()方法里更新UI，效果完全等同于在handleMessage()方法中更新UI。因为dispatchMessage()中会先判断 msg.callback（其实就是Runnable）对象是否存在，存在就直接调用run()方法。

然后再来看一下View中的post()方法，代码如下所示：

```java
    /**
     * <p>Causes the Runnable to be added to the message queue.
     * The runnable will be run on the user interface thread.</p>
     *
     * @param action The Runnable that will be executed.
     *
     * @return Returns true if the Runnable was successfully placed in to the
     *         message queue.  Returns false on failure, usually because the
     *         looper processing the message queue is exiting.
     *
     * @see #postDelayed
     * @see #removeCallbacks
     */
    public boolean post(Runnable action) {
        final AttachInfo attachInfo = mAttachInfo;
        if (attachInfo != null) {
            return attachInfo.mHandler.post(action);
        }

        // Postpone the runnable until we know on which thread it needs to run.
        // Assume that the runnable will be successfully placed after attach.
        getRunQueue().post(action);
        return true;
    }
```

其实也就是调用了Handler中的post()方法。

最后再来看一下Activity中的runOnUiThread()方法，代码如下所示：

```java
    /**
     * Runs the specified action on the UI thread. If the current thread is the UI
     * thread, then the action is executed immediately. If the current thread is
     * not the UI thread, the action is posted to the event queue of the UI thread.
     *
     * @param action the action to run on the UI thread
     */
    public final void runOnUiThread(Runnable action) {
        if (Thread.currentThread() != mUiThread) {
            mHandler.post(action);
        } else {
            action.run();
        }
    }
```

如果当前的线程不等于UI线程(主线程)，就去调用Handler的post()方法，否则就直接调用Runnable对象的run()方法。

通过以上所有源码的分析，我们已经发现了，不管是使用哪种方法在子线程中更新UI，其实背后的原理都是相同的，必须都要借助异步消息处理的机制来实现。





















