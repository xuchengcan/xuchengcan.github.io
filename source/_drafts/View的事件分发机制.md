---
title: View的事件分发机制
categories:
  - Android
  - View
comments: true
date: 2018-07-26 14:24:40
updated: 2018-07-26 14:24:40
tags: Android开发艺术探索,自定义View
keywords:
description:
---

# View 的事件分发机制整理笔记

参考
- 《Android 开发艺术探索》
- 《Android 高级进阶》

<!-- more -->

## 点击事件的传递规则

| 分发 | 拦截 | 处理 |
| ------ | ------ | ------ |
| dispatchTouchEvent | onInterceptTouchEvent | onTouchEvent |

> onInterceptTouchEvent 仅在 ViewGroup 及其子类中存在，在 View 和 Activity 中是不存在的。

```java
    // 伪代码
    public boolean dispatchTouchEvent(MotionEvent event){
        boolean consume = false;
        if (onInterceptTouchEvent(event)){
            consume = onTouchEvent(event);
        }else {
            consume = child.dispatchTouchEvent(event);
        }

        return consume;
    }
```

onTouchListener 的优先级比 onTouchEvent 高