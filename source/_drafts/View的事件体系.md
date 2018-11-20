---
title: View 的事件体系
categories:
  - Android
  - View
comments: true
date: 2018-07-20 13:44:11
updated: 2018-07-20 13:44:11
tags: Android 开发艺术探索
keywords:
description:
---

# 本文为 《Android 开发艺术探索》 第三章-View 的事件体系 的笔记

<!-- more -->

## View 基础

### View 的位置参数

View 的位置参数是相对于它的父布局来说的

- width = right - left
- height = bottom - top
- Left = getLeft()
- Right = getRight()
- Top = getTop()
- Bottom = getBottom()

x 和 y 是 View 左上角的坐标，而 translationX 和 translationY 是 View 左上角相对于父容器的偏移量。translationX 和 translationY 的默认值是 0。

> x = left + translationX  
> y = top + translationY

### MotionEvent 和 TouchSlop

通过 MotionEvent 对象我们可以得到点击事件发生的 x 和 y 坐标。
> 我们可以通过 getX/getY 返回相对于当前 View 左上角的 x 和 y 坐标，也可以通过 getRawX/getRawY 返回相对于手机屏幕左上角的 x 和 y 坐标。

TouchSlop 是系统所能识别出的被认为是滑动的最小距离。

### VelocityTracker、GestureDetector 和 Scroller

VelocityTracker 速度追踪器，用于追踪手指在滑动过程中的速度，在 view 的 onTouchEvent 方法中使用

```java
        VelocityTracker velocityTracker = VelocityTracker.obtain();
        velocityTracker.addMovement(event);

        // 设置时间间隔为 1s
        velocityTracker.computeCurrentVelocity(1000);

        // 速度 = ( 终点位置 - 起始位置 ) / 时间段  ；位置单位为像素值
        int xVelocity = (int) velocityTracker.getXVelocity();
        int yVelocity = (int) velocityTracker.getYVelocity();

        // 资源回收
        velocityTracker.clear();
        velocityTracker.recycle();
```

GestureDetector 手势检测，用于辅助检测用户的单击、滑动、长按、双击等行为。

如果只是监听滑动相关的，建议在 onTouchEvent 中实现，如果要监听双击这种行为的话，使用 GestureDetector

Scroller 见下方

## View 的滑动

### scrollTo/scrollBy

scrollBy 实际也是调用了 scrollTo 方法，它实现了基于当前位置的相对滑动，而 scrollTo 则实现了基于所传递参数的绝对滑动。

> scrollTo/scrollBy 只能将 View 的内容进行移动，并不能将 View 本身进行移动

### 使用动画

- 帧动画
- 补间动画
- 属性动画

### 改变布局参数

通过改变 LayoutParams 的方式去实现 View 的滑动

## 弹性滑动

### 使用 Scroller

```java
    private Scroller mScroller = new Scroller(context);

    public void zoomIn() {
        // Revert any animation currently in progress
        mScroller.forceFinished(true);
        // Start scrolling by providing a starting point and
        // the distance to travel
        mScroller.startScroll(0, 0, 100, 0);
        // Invalidate to request a redraw
        invalidate();
    }

    @Override
    public void computeScroll() {
        if (mScroller.computeScrollOffset()) {
            // Get current x and y positions
            int currX = mScroller.getCurrX();
            int currY = mScroller.getCurrY();
            scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
            postInvalidate();
        }
    }
```

当我们构造一个 Scroller 对象并且调用它的 startScroll 方法时，Scroller 内部只是保存了我们传递的几个参数， 让 View 滑动的根本原因在于接下去调用了 invalidate 方法，让 View 重绘，在 View 的 draw 方法中又会去调用 computeScroll 方法，而 computerScroll 又会去向 Scroller 获取当前的 scrollX 和 scrollY ，然后通过 scrollTo 方法实现滑动。接着调用 postInvalidate 方法来进行第二次重绘，这一次重绘的过程和第一次重绘一样，还是会导致 computeScroll 方法被调用，然后继续获取新的位置，如此反复知道 mScroller.computeScrollOffset() 返回 false 表示滑动结束。

所以说 Scroller 本身并不能实现 View 的滑动，它需要配合 View 的 computeScroll 方法才能完成弹性滑动的效果，它不断的让 View 重绘，而每次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔 Scroller 就可以得出 View 当前的滑动位置，知道了滑动位置就可以通过 scrollTo 方法来完成 View 的滑动。就这样， View 的每一次重绘都会导致 View 进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，这就是 Scroller 的工作机制。`整个过程对 View 没有丝毫引用，甚至在它内部连计时器都没有。`

### 通过动画

略

### 使用延时策略

使用 Handler 或 View 的 postDelayed 方法，具体略。

## View 的事件分发机制

[View 的事件分发机制](https://chennuo.online/2018/View的事件分发机制)

## View 的滑动冲突