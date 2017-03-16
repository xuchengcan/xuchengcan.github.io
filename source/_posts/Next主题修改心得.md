---
title: 博客修改心得
date: 2017-03-13 23:52:36
tags: hexo-Next主题
categories: 代码
comments:
keywords: hexo-Next
description:
---

其实就是记录下自己修改博客相关内容和 Next 主题的一些笔记

<!-- more -->

### 网站标题设置
>/source/css/_schemes/Muse/_logo.styl   
.site-title

### 网站副标题设置
>/source/css/_common/components/header/site-meta.styl    
.site-subtitle

### 网站 banner 设置
>/layout/_partials/header.swig    
/source/css/_schemes/Muse/_logo.styl  背景图 Url 设置    
关于 banner，我是在 header.swig 中重新布局的，仅针对 Muse 主题。其中的设计来自「[夏末](https://notes.wanghao.work/)」，也就是 Next 主题作者的 banner。   

~~如果有人知道那个背景跟随鼠标移动式怎么实现的，还希望能得到分享~~   
已在下面的 hexo-Next 主题优化中得到解決

### 网站 sidebar 设置
>/layout/_macro/sidebar.swig    //插入网易云音乐，放弃该操作，改为在个人页显示
/source/css/_common/components/sidebar/sidebar.styl

### 參考 XuewenDing 写的 hexo-Next 主题优化
>link: [hexo-Next主题优化](http://www.dingxuewen.com/2017/03/01/Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2%E7%9A%84%E4%B8%AA%E6%80%A7%E5%8C%96%E8%AE%BE%E7%BD%AE%E4%B8%80/)   

首先找到 \themes\next\layout\_layout.swig，在末尾前加上下面一句:（这里提供两种样式，当然你也可以自由更改）。    
- 默认灰色线条    


    <script type="text/javascript"src="/js/src/particle.js">
    </script>


- 浅蓝色线条


    <script type="text/javascript" src="/js/src/particle.js" count="50" zindex="-2" opacity="1" color="0,104,183">
    </script>

然后在 themes\source\js\src\下新建文件 particle.js 写上以下代码:

    !function(){function n(n,e,t){return n.getAttribute(e)||t}function e(n){return document.getElementsByTagName(n)}function t(){var t=e("script"),o=t.length,i=t[o-1];return{l:o,z:n(i,"zIndex",-1),o:n(i,"opacity",.5),c:n(i,"color","0,0,0"),n:n(i,"count",99)}}function o(){c=u.width=window.innerWidth||document.documentElement.clientWidth||document.body.clientWidth,a=u.height=window.innerHeight||document.documentElement.clientHeight||document.body.clientHeight}function i(){l.clearRect(0,0,c,a);var n,e,t,o,u,d,x=[w].concat(y);y.forEach(function(i){for(i.x+=i.xa,i.y+=i.ya,i.xa*=i.x>c||i.x<0?-1:1,i.ya*=i.y>a||i.y<0?-1:1,l.fillRect(i.x-.5,i.y-.5,1,1),e=0;e<x.length;e++)n=x[e],i!==n&&null!==n.x&&null!==n.y&&(o=i.x-n.x,u=i.y-n.y,d=o*o+u*u,d<n.max&&(n===w&&d>=n.max/2&&(i.x-=.03*o,i.y-=.03*u),t=(n.max-d)/n.max,l.beginPath(),l.lineWidth=t/2,l.strokeStyle="rgba("+m.c+","+(t+.2)+")",l.moveTo(i.x,i.y),l.lineTo(n.x,n.y),l.stroke()));x.splice(x.indexOf(i),1)}),r(i)}var c,a,u=document.createElement("canvas"),m=t(),d="c_n"+m.l,l=u.getContext("2d"),r=window.requestAnimationFrame||window.webkitRequestAnimationFrame||window.mozRequestAnimationFrame||window.oRequestAnimationFrame||window.msRequestAnimationFrame||function(n){window.setTimeout(n,1e3/45)},x=Math.random,w={x:null,y:null,max:2e4};u.id=d,u.style.cssText="position:fixed;top:0;left:0;z-index:"+m.z+";opacity:"+m.o,e("body")[0].appendChild(u),o(),window.onresize=o,window.onmousemove=function(n){n=n||window.event,w.x=n.clientX,w.y=n.clientY},window.onmouseout=function(){w.x=null,w.y=null};for(var y=[],s=0;m.n>s;s++){var f=x()*c,h=x()*a,g=2*x()-1,p=2*x()-1;y.push({x:f,y:h,xa:g,ya:p,max:6e3})}setTimeout(function(){i()},100)}();
