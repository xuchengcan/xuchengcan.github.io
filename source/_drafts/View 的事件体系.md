---
title: View 的事件体系
categories:
  - Android
  - View
comments: true
date: 2018-01-20 13:44:11
updated: 2018-01-20 13:44:11
tags: Android 开发艺术探索
keywords:
description:
---

# 本文为 《Android 开发艺术探索》 第三章-View的事件体系 的笔记

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

通过 MotionEvent 对象我们可以得到点击事件发生的 x 和 y 坐标。我们可以通过 getX/getY 返回相对于当前 View 左上角的 x 和 y 坐标，也可以通过 getRawX/getRawY 返回相对于手机屏幕左上角的 x 和 y 坐标。

TouchSlop 是系统所能识别出的被认为是滑动的最小距离。

## 弹性滑动

### 使用 Scroller

Scroller 本身并不能实现 View 的滑动，它需要配合 View 的 computeScroll 方法才能完成弹性滑动的效果，它不断的让 View 重绘，而每次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔 Scroller 就可以得出 View 当前的滑动位置，知道了滑动位置就可以通过 scrollTo 方法来完成 View 的滑动。就这样， View 的每一次重绘都会导致 View 进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，这就是 Scroller 的工作机制。`整个过程对 View 没有丝毫引用，甚至在它内部连计时器都没有。`

### 通过动画

### 使用延时策略

## View 的事件分发机制





















