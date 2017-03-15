---
title: 博客修改心得
date: 2017-03-13 23:52:36
tags: hexo-Next主题
categories: 代码
comments:
keywords: hexo-Next
description: 
---

其实就是记录下自己修改博客相关内容和Next主题的一些笔记

<!-- more -->

### 网站标题设置
>/source/css/_schemes/Muse/_logo.styl   
.site-title

### 网站副标题设置
>/source/css/_common/components/header/site-meta.styl    
.site-subtitle

### 网站banner设置
>/layout/_partials/header.swig    
/source/css/_schemes/Muse/_logo.styl  背景图url设置    
关于banner，我是在header.swig中重新布局的，仅针对Muse主题。其中的设计来自[夏末](https://notes.wanghao.work/),也就是next主题作者的banner   

**如果有人知道那个背景跟随鼠标移动式怎么实现的，还希望能得到分享**

### 网站sidebar设置
>/layout/_macro/sidebar.swig    插入网易云音乐，放弃该操作，改为在个人页显示
/source/css/_common/components/sidebar/sidebar.styl

### 网站Seo相关设置
>link： http://www.jianshu.com/p/86557c34b671
