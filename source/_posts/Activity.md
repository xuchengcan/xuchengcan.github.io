---
title: Activity 基本介绍
categories:
  - Android
  - 四大组件
comments: true
date: 2017-12-29 11:00:45
updated: 2017-12-29 11:00:45
tags:
keywords:
description:
---

### 内容包括

- 启动顺序
- 生命周期
- 三个时期
- Activity 栈
- Activity 状态
- 面试题

<!-- more -->

# Activity

### 启动顺序

* 第一个Activity启动时：

onCreate()——&gt;onStart()——&gt;onResume()

* 当另一个Activity启动时:

第一个Activity onPause()——&gt;第二个Activity onCreate()——&gt;onStart()——&gt;onResume()

——&gt;第一个Activity onStop()

* 当返回到第一个Activity时：

第二个Activity onPause() ——&gt; 第一个Activity　onRestart()——&gt;onStart()——&gt;onResume()

——&gt;第二个Activity onStop()——&gt;onDestroy()

#### 一个Activity的销毁顺序:

（情况一）onPause()——&gt;&lt;Process Killed&gt;

（情况二）onPause()——&gt;onStop()——&gt;&lt;Process Killed&gt;

（情况三）onPause()——&gt;onStop()——&gt;onDestroy()

但是当一个活动的状态发生改变的时候，开发者可以通过调用 onXX() 的方法获取到相关的通知信息。

在实现 Activity 类的时候，通过覆盖（ override ）这些方法即可在你需要处理的时候来调用。

### 生命周期

1. **onCreate** ：当活动第一次启动的时候，触发该方法，可以在此时完成活动的初始化工作。onCreate 方法有一个参数，该参数可以为空（ null ），也可以是之前调用 onSaveInstanceState （）方法保存的状态信息。
2. **onStart** ：该方法的触发表示所属活动将被展现给用户。**活动由不可见变为可见的时候调用**
3. **onResume** ：当一个活动和用户发生交互的时候，触发该方法。**此时的活动一定位于返回栈的栈顶，并且处于运行状态。**
4. **onPause** ：当一个正在前台运行的活动因为其他的活动需要前台运行而转入后台运行的时候，触发该方法。这时候需要将活动的状态持久化，比如正在编辑的数据库记录等。**我们通常会在这个方法中将一些及其消耗 CPU 的资源释放掉（比如显示地图或者大规模图形），以及保存一些关键数据（比如用户输入的数据等等），但这个方法的执行速度一定要快，不然会影响到新的栈顶活动的使用。**
5. **onStop** ：当一个活动不再需要展示给用户的时候，触发该方法。如果内存紧张，系统会直接结束这个活动，而不会触发 onStop 方法。 所以保存状态信息是应该在onPause时做，而不是onStop时做。活动如果没有在前台运行，都将被停止或者Linux管理进程为了给新的活动预留足够的存储空间而随时结束这些活动。因此对于开发者来说，在设计应用程序的时候，必须时刻牢记这一原则。在一些情况下，onPause方法或许是活动触发的最后的方法，因此开发者需要在这个时候保存需要保存的信息。
6. **onRestart** ：当处于停止状态的活动需要再次展现给用户的时候，触发该方法。
7. **onDestroy** ：当活动销毁的时候，触发该方法。和 onStop 方法一样，如果内存紧张，系统会直接结束这个活动而不会触发该方法。

8. **onSaveInstanceState**：系统调用该方法，允许活动保存之前的状态，比如说在一串字符串中的光标所处的位置等。通常情况下，开发者不需要重写覆盖该方法，在默认的实现中，已经提供了自动保存活动所涉及到的用户界面组件的所有状态信息。

### 三个时期

* 完整生存期  

* 活动在`onCreate()`方法和`onDestroy()`方法之间所经历的，就是完整生存期。一般情况下，一个活动会在`onCreate()`方法中完成各种初始化操作，而在`onDestroy()`方法中完成释放内存的操作。

* 可见生存期  
  **划重点！！！这个问题我就在面试中遇到了，其实我知晓这个题，怎奈误解了面试官的意思答非所问了。**活动在`onStart()`方法和`onStop()`方法之间所经历的，就是可见生存期。在可见生存期内， 活动对于用户总是可见的， 即便有可能无法和用户进行交互。 我们可以通过这两个方法，合理地管理那些对用户可见的资源。比如在`onStart()`方法中对资源进行加载，而在`onStop()`方法中对资源进行释放， 从而保证处于停止状态的活动不会占用过多内存。

* 前台生存期  
  活动在`onResume()`方法和`onPause()`方法之间所经历的，就是前台生存期。在前台生存期内， 活动总是处于运行状态的， 此时的活动是可以和用户进行相互的， 我们平时看到和接触最多的也这个状态下的活动。

### Activity 栈

上面提到开发者是无法控制Activity的状态的，那Activity的状态又是按照何种逻辑来运作的呢？这就要知道 Activity 栈。每个Activity的状态是由它在Activity栈（是一个后进先出LIFO，包含所有正在运行Activity的队列）中的位置决定的。当一个新的Activity启动时，当前的活动的Activity将会移到Activity栈的顶部。如果用户使用后退按钮返回的话，或者前台的Activity结束，活动的Activity就会被移出栈消亡，而在栈上的上一个活动的Activity将会移上来并变为活动状态。一个应用程序的优先级是受最高优先级的Activity影响的。当决定某个应用程序是否要终结去释放资源，Android内存管理使用栈来决定基于Activity的应用程序的优先级。

### Activity状态

一般认为Activity有以下四种状态：

* 活动的：当一个Activity在栈顶，它是可视的、有焦点、可接受用户输入的。Android试图尽最大可能保持它活动状态，杀死其它Activity来确保当前活动Activity有足够的资源可使用。当另外一个Activity被激活，这个将会被暂停。

* 暂停：在很多情况下，你的Activity可视但是它没有焦点，换句话说它被暂停了。有可能原因是一个透明或者非全屏的Activity被激活。当被暂停，一个Activity仍会当成活动状态，只不过是不可以接受用户输入。在极特殊的情况下，Android将会杀死一个暂停的Activity来为活动的Activity提供充足的资源。当一个Activity变为完全隐藏，它将会变成停止。

* 停止：当一个Activity不是可视的，它“停止”了。这个Activity将仍然在内存中保存它所有的状态和会员信息。尽管如此，当其它地方需要内存时，它将是最有可能被释放资源的。当一个Activity停止后，一个很重要的步骤是要保存数据和当前UI状态。一旦一个Activity退出或关闭了，它将变为待用状态。

* 待用： 在一个Activity被杀死后和被装在前，它是待用状态的。待用Acitivity被移除Activity栈，并且需要在显示和可用之前重新启动它。

### 面试题：

* Activity A 通过 Intent 显示启动了 Activity B，当 B 处于可见状态后，A 是否一定会调用`onStop()`?

不一定，有可能A此时仍然可见，例如B是一个DialogActivity，或者其他A仍可见的情况。



