---
title: Activity四种启动模式
categories:
  - Android
  - 四大组件
comments: true
date: 2017-12-29 11:06:31
updated: 2019-02-25 11:06:31
tags:
keywords:
description:
---

#### 在 Activity 的多 Activity 开发中，Activity 之间的跳转可能需要有多种方式，有时是普通的生成一个新实例，有时希望跳转到原来某个 Activity 实例，而不是生成大量的重复的 Activity。加载模式便是决定以哪种方式启动一个跳转到原来某个 Activity 实例。

在 Activity 里，有 4 种 Activity 的启动模式，分别为：

* standard: 标准模式，一调用startActivity()方法就会产生一个新的实例。在这种模式下，activity默认会进入启动它的activity所属的任务栈中。注意：在非activity类型的context (如 ApplicationContext )并没有所谓的任务栈，所以不能通过ApplicationContext去启动standard模式的activity。

* singleTop: 栈顶复用模式。如果已经有一个实例位于Activity栈的顶部时，就不产生新的实例，而只是调用Activity中的newInstance()方法。如果不位于栈顶，会产生一个新的实例。注意：这个activity的onCreate，onStart，onResume不会被回调，因为他们并没有发生改变。

* singleTask: 栈内复用模式。首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈。
  - 如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去。
  - 如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例 。
    - 如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法。
    - 如果不存在该实例，则新建Activity，并入栈


* singleInstance: 这个跟singleTask基本上是一样，只有一个区别：这种模式下的Activity会单独占用一个Task栈，具有全局唯一性，即整个系统中就这么一个实例。以singleInstance模式启动的Activity在整个系统中是单例的，如果在启动这样的Activiyt时，已经存在了一个实例，那么会把它所在的任务调度到前台，重用这个实例。


<!-- more -->

### 应用场景

* standard：默认启动模式

* singleTop: **当且仅当启动的 Activity 和上一个 Activity 一致的时候才会通过调用`onNewIntent()`方法重用 Activity。**使用场景：资讯阅读类 APP 的内容界面。

* singleTask：浏览器的主页面，或者大部分 APP 的主页面。

这些启动模式可以在功能清单文件AndroidManifest.xml中进行设置，中的launchMode属性。相关的代码中也有一些标志可以使用,比如我们想只启用一个实例,则可以使用 Intent.FLAG_ACTIVITY_REORDER_TO_FRONT 标志，这个标志表示：如果这个activity已经启动了，就不产生新的activity，而只是把这个activity实例加到栈顶来就可以了。

```java
Intent intent = new Intent(ReorderFour.this, ReorderTwo.class);
intent.addFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT);
startActivity(intent);
```

Activity的加载模式受启动Activity的Intent对象中设置的Flag和manifest文件中Activity的元素的特性值交互控制。

下面是影响加载模式的一些特性。核心的Intent Flag有：

* `FLAG_ACTIVITY_NEW_TASK`

* `FLAG_ACTIVITY_CLEAR_TOP`

* `FLAG_ACTIVITY_RESET_TASK_IF_NEEDED`

* `FLAG_ACTIVITY_SINGLE_TOP`

核心的特性有：

taskAffinity

launchMode

allowTaskReparenting

clearTaskOnLaunch

alwaysRetainTaskState

finishOnTaskLaunch

### 四种加载模式的区别

* 所属task的区别

> 一般情况下，standard 和 singleTop 的 activity 的目标task，和收到的 Intent 的发送者在同一个 task 内，就相当于谁调用它，它就跟谁在同一个Task中。除非Intent包括参数FLAG_ACTIVITY_NEW_TASK。如果提供了FLAG_ACTIVITY_NEW_TASK参数，会启动到别的task里。singleTask 和 singleInstance 总是把要启动的 activity 作为一个 task 的根元素，他们不会被启动到一个其他 task 里。

* 是否允许多个实例

> standard 和 singleTop 可以被实例化多次，并且是可以存在于不同的 task 中；这种实例化时一个 task 可以包括一个 activity 的多个实例；
>
> singleTask 和 singleInstance 则限制只生成一个实例，并且是 task 的根元素。
>
> singleTop 要求如果创建intent的时候栈顶已经有要创建的Activity的实例，则将intent发送给该实例，而不创建新的实例。

* 是否允许其它activity存在于本task内

> singleInstance 独占一个 task，其它 activity 不能存在那个 task 里；如果它启动了一个新的 activity，不管新的 activity 的 launch mode 如何，新的 activity 都将会到别的 task 里运行（如同加了FLAG_ACTIVITY_NEW_TASK参数）。

而另外三种模式，则可以和其它activity共存。

* 是否每次都生成新实例

> standard 对于每一个启动 Intent 都会生成一个 activity 的新实例；
>
> singleTop”的activity如果在task的栈顶的话，则不生成新的该activity的实例，直接使用栈顶的实例，否则，生成该activity的实例。比如：现在task栈元素为A - B - C - D（D在栈顶），这时候给D发一个启动intent，如果D是 “standard”的，则生成D的一个新实例，栈变为A－B－C－D－D。如果D是singleTop的话，则不会生产D的新实例，栈状态仍为A-B-C-D
>
> 如果这时候给B发Intent的话，不管B的launchmode是”standard” 还是 “singleTop” ，都会生成B的新实例，栈状态变为A-B-C-D-B。
>
> singleInstance 是其所在栈的唯一activity，它会每次都被重用。
>
> singleTask” 如果在栈顶，则接受intent，否则，该intent会被丢弃，但是该task仍会回到前台。 当已经存在的activity实例处理新的intent时候，会调用onNewIntent()方法，如果收到intent生成一个activity实例，那么用户可以通过back键回到上一个状态；如果是已经存在的一个activity来处理这个intent的话，用户不能通过按back键返回到这之前的状态。



