---
title: ViewPager + Fragment
categories:
  - Android
comments: true
date: 2017-12-29 10:33:25
updated: 2017-12-29 10:33:25
tags:
keywords:
description:
---

# ViewPager 里对 Fragment 的相关优化操作
# FragmentPagerAdapter 和 FragmentStatePagerAdapter 的区别

[参考链接](https://mp.weixin.qq.com/s/gG4BJHtb0fQcM5um_lubVg)

<!-- more -->

### 基本使用

ViewPager是support v4库中提供界面滑动的类，继承自ViewGroup。PagerAdapter是ViewPager的适配器类，为ViewPager提供界面。但是一般来说，通常都会使用PagerAdapter的两个子类：FragmentPagerAdapter和FragmentStatePagerAdapter作为ViewPager的适配器，他们的特点是界面是Fragment。

> 在support v13和support v4中都提供了FragmentPagerAdapter和FragmentStatePagerAdapter，区别在于：support v13中使用android.app.Fragment，而support v4使用android.support.v4.app.Fragment。一般都使用support v4中的FragmentPagerAdapter和FragmentStatePagerAdapter。

默认，ViewPager会缓存当前页相邻的界面，比如当滑动到第2页时，会初始化第1页和第3页的界面（即Fragment对象，且生命周期函数运行到onResume()），可以通过**setOffscreenPageLimit(count)**设置离线缓存的界面个数。

FragmentPagerAdapter和FragmentStatePagerAdapter需要重写的方法都一样，常见的重写方法如下：

* `public FragmentPagerAdapter(FragmentManager fm)`: 构造函数，参数为FragmentManager。如果是嵌套Fragment场景，子PagerAdapter的参数传入getChildFragmentManager()。

* `Fragment getItem(int position)`: 返回第position位置的Fragment，必须重写。

* `int getCount()`: 返回ViewPager的页数，必须重写。

* `Object instantiateItem(ViewGroup container, int position)`: container是ViewPager对象，返回第position位置的Fragment。

* `void destroyItem(ViewGroup container, int position, Object object)`: container是ViewPager对象，object是Fragment对象。

* `getItemPosition(Object object)`: object是Fragment对象，如果返回POSITION_UNCHANGED，则表示当前Fragment不刷新，如果返回POSITION_NONE，则表示当前Fragment需要调用`destroyItem()`和`instantiateItem()`进行销毁和重建。 默认情况下返回POSITION_UNCHANGED。

### 懒加载

懒加载主要用于ViewPager且每页是Fragment的情况，场景为微信主界面，底部有4个tab，当滑到另一个tab时，先显示”正在加载”，过一会才会显示正常界面。

默认情况，ViewPager会缓存当前页和左右相邻的界面。实现懒加载的主要原因是：用户没进入的界面需要有一系列的网络、数据库等耗资源、耗时的操作，预先做这些数据加载是不必要的。

这里懒加载的实现思路是：用户不可见的界面，只初始化UI，但是不会做任何数据加载。等滑到该页，才会异步做数据加载并更新UI。

这里就实现类似微信那种效果，整个UI布局为：底部用PagerBottomTabStrip项目实现，上面是ViewPager，使用FragmentPagerAdapter。逻辑为：当用户滑到另一个界面，首先会显示正在加载，等数据加载完毕后（这里用睡眠1秒钟代替）显示正常界面。

ViewPager默认缓存左右相邻界面，为了避免不必要的重新数据加载（重复调用`onCreateView()`），因为有4个tab，因此将离线缓存的半径设置为3，即`setOffscreenPageLimit(3)`。

懒加载主要依赖Fragment的`setUserVisibleHint(boolean isVisible)`方法，当Fragment变为可见时，会调用`setUserVisibleHint(true)`；当Fragment变为不可见时，会调用`setUserVisibleHint(false)`，且该方法调用时机：

* `onAttach()`之前，调用`setUserVisibleHint(false)`。

* `onCreateView()`之前，如果该界面为当前页，则调用`setUserVisibleHint(true)`，否则调用`setUserVisibleHint(false)`。

* 界面变为可见时，调用`setUserVisibleHint(true)`。

* 界面变为不可见时，调用`setUserVisibleHint(false)`。

懒加载Fragment的实现：

```java
    private boolean mIsInited;
    private boolean mIsPrepared;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View root = inflater.inflate(R.layout.demo, container, false);
        mIsInited = true;
        lazyLoad();
        return root;
    }
    
    public void lazyLoad(){
        if (getUserVisibleHint()&&mIsPrepared&&!mIsInited){
            //异步初始化
            loadData();
        }
    }

    private void loadData(){
        new Thread(){
            //UI
        }.start();
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (isVisibleToUser){
            lazyLoad();
        }
    }
```

注：

* 在Fragment中有两个变量控制是否需要做数据加载：
  * mIsPrepared：表示UI是否准备好，因为数据加载后需要更新UI，如果UI还没有inflate，就不需要做数据加载，因为`setUserVisibleHint()`会在`onCreateView()`之前调用一次，如果此时调用，UI还没有inflate，因此不能加载数据。
  * mIsInited：表示是否已经做过数据加载，如果做过了就不需要做了。因为`setUserVisibleHint(true)`在界面可见时都会调用，如果滑到该界面做过数据加载后，滑走，再滑回来，还是会调用`setUserVisibleHint(true)`，此时由于mIsInited=true，因此不会再做一遍数据加载。

* lazyLoad()：懒加载的核心类，在该方法中，只有界面可见（getUserVisibleHint()==true）、UI准备好（mIsPrepared==true）、过去没做过数据加载（mIsInited==false）时，才需要调`loadData()`做数据加载，数据加载做完后把mIsInited置为true。



