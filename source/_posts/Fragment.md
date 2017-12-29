---
title: Fragment 详细解析
categories:
  - Android
date: 2017-12-29 10:13:13
updated: 2017-12-29 10:13:13
tags:
keywords:
description:
comments:
---

# 摘录：

[Fragment全解析系列（二）：正确的使用姿势](http://www.jianshu.com/p/fd71d65f0ec6)

[参考博客-天天P图工程师](https://mp.weixin.qq.com/s/gG4BJHtb0fQcM5um_lubVg)

<!-- more -->

注：代码来自 `com.android.support:support-v4:25.3.1`

Fragment核心的类有：

* Fragment：Fragment的基类，任何创建的Fragment都需要继承该类。

* FragmentManager：管理和维护Fragment。他是抽象类，具体的实现类是FragmentManagerImpl。

* FragmentTransaction：对Fragment的添加、删除等操作都需要通过事务方式进行。他是抽象类，具体的实现类是BackStackRecord。

Nested Fragment（Fragment内部嵌套Fragment的能力）是Android 4.2提出的，support-fragment库可以兼容到1.6。通过`getChildFragmentManager()`能够获得管理子Fragment的FragmentManager，在子Fragment中可以通过`getParentFragment()`获得父Fragment。

### 基本使用

利用AS直接生成Fragment

```java
public class BlankFragment extends Fragment {
    // TODO: Rename parameter arguments, choose names that match
    // the fragment initialization parameters, e.g. ARG_ITEM_NUMBER
    private static final String ARG_PARAM1 = "param1";
    private static final String ARG_PARAM2 = "param2";

    // TODO: Rename and change types of parameters
    private String mParam1;
    private String mParam2;

    private OnFragmentInteractionListener mListener;

    public BlankFragment() {
        // Required empty public constructor
    }

    /**
     * Use this factory method to create a new instance of
     * this fragment using the provided parameters.
     *
     * @param param1 Parameter 1.
     * @param param2 Parameter 2.
     * @return A new instance of fragment Blank2Fragment.
     */
    // TODO: Rename and change types and number of parameters
    public static BlankFragment newInstance(String param1, String param2) {
        Blank2Fragment fragment = new Blank2Fragment();
        Bundle args = new Bundle();
        args.putString(ARG_PARAM1, param1);
        args.putString(ARG_PARAM2, param2);
        fragment.setArguments(args);
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        if (getArguments() != null) {
            mParam1 = getArguments().getString(ARG_PARAM1);
            mParam2 = getArguments().getString(ARG_PARAM2);
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View root = inflater.inflate(R.layout.demo,container,false);
        return root;
    }

    // TODO: Rename method, update argument and hook method into UI event
    public void onButtonPressed(Uri uri) {
        if (mListener != null) {
            mListener.onFragmentInteraction(uri);
        }
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnFragmentInteractionListener) {
            mListener = (OnFragmentInteractionListener) context;
        } else {
            throw new RuntimeException(context.toString()
                    + " must implement OnFragmentInteractionListener");
        }
    }

    @Override
    public void onDetach() {
        super.onDetach();
        mListener = null;
    }

    /**
     * This interface must be implemented by activities that contain this
     * fragment to allow an interaction in this fragment to be communicated
     * to the activity and potentially other fragments contained in that
     * activity.
     * <p>
     * See the Android Training lesson <a href=
     * "http://developer.android.com/training/basics/fragments/communicating.html"
     * >Communicating with Other Fragments</a> for more information.
     */
    public interface OnFragmentInteractionListener {
        // TODO: Update argument type and name
        void onFragmentInteraction(Uri uri);
    }
}
```

涉及的问题

1、 Fragment需要有一个空方法

> 每一个Fragment必须有一个空的构造方法，这样当Activity恢复状态时Fragment能够被实例化。强烈建议当我们继承Fragment类时，不要添加带有参数的构造方法，因为当Fragment被重新实例化时，这些构造方法不会被调用。如果需要给Fragment传递参数，可以调用setArguments(Bundle)方法，然后在Fragment中调用getArguments()来获取参数。

如果fragment进入后台，此时横竖屏切换，会让fragment重新创建，fragment没有空的构造就会出运行异常。

2、 Layout.Inflate方法

Fragment有很多可以复写的方法，其中最常用的就是`onCreateView()`，该方法返回Fragment的UI布局，需要注意的是

`inflate()`的第三个参数是false，因为在Fragment内部实现中，会把该布局添加到container中，如果设为true，那么就会重复做两次添加，则会抛如下异常:`The specified child already has a parent. You must call removeView() on the child's parent first`

3、 传参

如果在创建Fragment时要传入参数，必须要通过`setArguments(Bundle bundle)`方式添加，而不建议通过为Fragment添加带参数的构造函数，因为通过`setArguments()`方式添加，在由于内存紧张导致Fragment被系统杀掉并恢复（re-instantiate）时能保留这些数据。官方建议如下：

> It is strongly recommended that subclasses do not have other constructors with parameters, since these constructors will not be called when the fragment is re-instantiated.

4、 绑定Activity

我们可以在Fragment的`onAttach()`中通过`getArguments()`获得传进来的参数，并在之后使用这些参数。如果要获取Activity对象，不建议调用`getActivity()`，而是在`onAttach()`中将Context对象强转为Activity对象。

```java
    Activity mActivity;

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnFragmentInteractionListener) {
            mListener = (OnFragmentInteractionListener) context;
        } else {
            throw new RuntimeException(context.toString()
                    + " must implement OnFragmentInteractionListener");
        }
        if (context instanceof Activity) {
            mActivity = (Activity) context;
        }
    }
```

### 添加到Activity

```xml
    <FrameLayout
        android:id="@+id/fragment_container"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

    </FrameLayout>
```

```java
    getSupportFragmentManager()
        .beginTransaction()
        .add(R.id.fragment_container, BlankFragment.newInstance("a","b"),"blank")
        //.addToBackStack("BackStack")
        .commit();
```

将Fragment添加到布局FrameLayout中，指定tag的好处是后续我们可以通过`BlankFragment frag = getSupportFragmentManager().findFragmentByTag("blank")`从FragmentManager中查找Fragment对象。

在一次事务中，可以做多个操作，比如同时做`add().remove().replace()`。

* `commit()`操作是异步的，内部通过`mManager.enqueueAction()`加入处理队列。对应的同步方法为`commitNow()`，`commit()`内部会有`checkStateLoss()`操作，如果开发人员使用不当（比如`commit()`操作在`onSaveInstanceState()`之后），可能会抛出异常，而`commitAllowingStateLoss()`方法则是不会抛出异常版本的`commit()`方法，但是尽量使用`commit()`，而不要使用`commitAllowingStateLoss()`。

* `addToBackStack("fname")`是可选的。FragmentManager拥有回退栈（BackStack），类似于Activity的任务栈，如果添加了该语句，就把该事务加入回退栈，当用户点击返回按钮，会回退该事务（回退指的是如果事务是add(frag1)，那么回退操作就是remove(frag1)）；如果没添加该语句，用户点击返回按钮会直接销毁Activity。

Fragment有一个常见的问题，即Fragment重叠问题，这是由于Fragment被系统杀掉，并重新初始化时再次将fragment加入activity，因此通过在外围加if语句能判断此时是否是被系统杀掉并重新初始化的情况。

#### Fragment有个常见的异常：

`java.lang.IllegalStateException: Can not perform this action after onSaveInstanceState:- Error in Fragment`

该异常出现的原因是：commit()在onSaveInstanceState()后调用。

onSaveInstanceState() 只有在系统即将要自动清理销毁Activity或Fragment前才会调用, 比如

1. 由于重力感应 手机从竖屏变为横屏
2. 手机点击Home键和长按Home键
3. 点击电源键锁屏时
4. 从当前Activity跳到另一个Activity
5. 应用内存不足即将自动销毁时等情况

因为Activity的生命周期在Honeycomb出现后有了些许的差别。在Honeycomb之前，Activity是不可被杀死了直到它们被暂停了，也就是说onSaveInstanceState()在要执行onPause()前就被调用了。而从Honeycomb开始，Activity在它们被停止了就才可以销毁，即只有Activity的onSaveInstanceState()会在onStop()之前调用。这个不同可以用如下表格总结：

|  | HoneyComb之前 | HoneyCombz之后 |
| :--- | :---: | :---: |
| Activity在onPause之前会被销毁？ | 不会 | 不会 |
| Activity在onstop之前会被销毁？ | 会 | 不会 |
| onSaveInstanceState确保在哪里调用？ | onPause | onStop |

因此在onStop()之后调用commit()会发生异常

#### 解决方案

[参考博客](http://blog.csdn.net/chenzujie/article/details/49701269%29)

* **在Activity的生命周期函数里谨慎使用commit**，大部分的应用只会在Activity的第一次调用onCreate或者相应用户输入的时候使用commit，这两者情况下都不可能遇到这个问题。但，一旦在其他Activity的生命周期函数，比如onActivityResult(),onStart(),onResume()，就变得有点麻烦了。比如，你不应该在FragmentActivity的onResume()里commit，因为存在一些情况onResume()会在Activity恢复信息前调用（具体查看[文档](http://blog.csdn.net/chenzujie/article/details/m/reference/android/support/v4/app/FragmentActivity.html#onResume%28%29)）。如果你的应用程序需要在Activity非onCreate()的生命周期函数里commit，那么就在FragmentActivity的onResumeFragments()或者Activity#onPostResume()。这两个方法都会在Activity从它的销毁状态恢复过来后才会调用，因此可以避免状态丢失。（这里有个例子，是我对[StackOverflow问题](http://stackoverflow.com/questions/16265733/failure-delivering-result-onactivityforresult)这个问题的回答，可能可以给你一点关于如何在Activity#onActivityResult()做commit的思路）。
* **防止在异步操作的回调方法里执行transaction操作**。这里包括在AsyncTask#onPostExecute() 和LoaderManager.LoaderCallbacks#onLoadFinished()等方法里的操作。在这些方法里操作commit等操作的问题在于这些方法没法活儿当前Activity的生命周期状态。比如考虑如下顺序的事件：  
  1、一个Activity执行了AsyncTask  
  2、用户按了Home键，触发了onSaveInstanceState()和onStop()方法。  
  3、AsyncTask异步操作完成回调了onPostExecute(),并且不知道Activity已经onStop,导致抛出异常。一般来说，去好的防止在这些异步回调方法里发生异常的方法是不要触发这些方法。Google似乎也很赞同这个观点。根据[这篇文章](https://groups.google.com/d/msg/android-developers/dXZZjhRjkMk/QybqCW5ukDwJ)，Android团队也认为在异步回调里做commit操作对于用户体验并不是一种好的体验。如果你的应用程序要求在这些方法里进行commit等操作，并没有合适的方式保证回调在onSaveInstanceState()后触发，你可能只能使用commitAllowingStateLoss()并处理可能随之发生的状态丢失。(可以看这两个问题来理解这个问题[链接1](http://stackoverflow.com/questions/8040280/how-to-handle-handler-messages-when-activity-fragment-is-paused)和[链接2](http://stackoverflow.com/questions/7992496/how-to-handle-asynctask-onpostexecute-when-paused-to-avoid-illegalstateexception))

* **把commitAllowingStateLoss()作为没有其他办法才使用的方法**，commit和commitAllowingStateLoss方法的差别就在于后者不会抛出在状态丢失的时候抛出异常。通常你最好不要使用这个方法，因为这种情况下可能存在状态丢失。更好的方法，是保证你的应用程序在状态丢失前使用commit，因为这会是更好的用户体验。除非状态丢失的事情无法避免，否则不要使用commitAllowingStateLoss().

### Fragment生命周期

主要体现在以下表中13个方法里，以下是按照Fragment从开始到销毁的先后执行顺序排序。

* onInflate

> public void onInflate(Activity activity, AttributeSet attrs,BundlesavedInstanceState)
>
> 在Activity.onCreate方法之前调用，可以获取除了View之外的资源

* onAttach

> public void onAttach(Activity activity)
>
> 当fragment第一次与Activity产生关联时就会调用，以后不再调用

* onCreate

> public void onCreate(Bundle savedInstanceState)
>
> 在onAttach执行完后会立刻调用此方法，通常被用于读取保存的状态值，获取或者初始化一些数据，但是该方法不执行，窗口是不会显示的，因此如果获取的数据需要访问网络，最好新开线程。

* onCreateView

> public View onCreateView(LayoutInflater inflater, ViewGroup container,Bundle savedInstanceState)
>
> 作用：创建Fragment中显示的view,其中inflater用来装载布局文件，container表示&lt;fragment&gt;标签的父标签对应的ViewGroup对象，savedInstanceState可以获取Fragment保存的状态

* onViewCreated

> public void onViewCreated(View view, Bundle savedInstanceState)
>
> 继上面后就会调用此方法，相关view绑定可以在此进行

* onActivityCreated

> public void onActivityCreated(Bundle savedInstanceState)
>
> 在Activity.onCreate方法调用后会立刻调用此方法，表示窗口已经初始化完毕，此时可以调用控件了

* onStart

> public void onStart()
>
> 开始执行与控件相关的逻辑代码，如按键点击

* onResume

> public void onResume()
>
> 这是Fragment从创建到显示的最后一个回调的方法

* onPause

> public void onPause()
>
> 当发生界面跳转时，临时暂停，暂停时间是500ms,0.5s后直接进入下面的onStop方法

* onStop

> public void onStop()
>
> 当该方法返回时，Fragment将从屏幕上消失

* onDestroyView

> public void onDestroyView()
>
> 当fragment状态被保存，或者从回退栈弹出，该方法被调用

* onDestroy

> public void onDestroy()
>
> 当Fragment不再被使用时，如按返回键，就会调用此方法

* onDetach

> public void onDetach()
>
> Fragment生命周期的最后一个方法，执行完后将不再与Activity关联，将释放所有fragment对象和资源

#### 切换例子：

两个Fragment：F1和F2，F1在初始化时就加入Activity，点击F1中的按钮调用replace替换为F2。

由 Log 可知

* Fragment的onAttach()-&gt;onCreate()-&gt;onCreateView()-&gt;onActivityCreated()-&gt;onStart()都是在Activity的onStart()中调用的。

* Fragment的onResume()在Activity的onResume()之后调用。

接下去分两种情况，分别是不加addToBackStack()和加addToBackStack()。

1、当点击F1的按钮，调用replace()替换为F2，且不加addToBackStack()时，日志如下：

**Fragment2**.onAttach-&gt;onCreate-&gt;**Fragment1**.onPause-&gt;onStop-&gt;onDestroyView-&gt;onDestroy-&gt;**onDetach**-&gt;

**Fragment2.**onCreateView-&gt;onViewCreated-&gt;onActivityCreated-&gt;onStart-&gt;onResume

可以看到，F1最后调用了onDestroy()和onDetach()。

2、当点击F1的按钮，调用replace()替换为F2，且加addToBackStack()时，日志如下：

**Fragment2**.onAttach-&gt;onCreate-&gt;**Fragment1**.onPause-&gt;onStop-&gt;**onDestroyView**

**Fragment2.**onCreateView-&gt;onViewCreated-&gt;onActivityCreated-&gt;onStart-&gt;onResume

可以看到，F1被替换时，最后只调到了onDestroyView()，并没有调用onDestroy()和onDetach()。当用户点返回按钮回退事务时，F1会调onCreateView()-&gt;onStart()-&gt;onResume()，因此在Fragment事务中加不加addToBackStack()会影响Fragment的生命周期。

#### FragmentTransaction的一些基本方法：

下面给出调用这些方法时，Fragment生命周期的变化：

* add(): onAttach()-&gt;…-&gt;onResume()。

* remove(): onPause()-&gt;…-&gt;onDetach()。

* replace(): 相当于旧Fragment调用remove()，新Fragment调用add()。

* show(): 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为true。

* hide(): 不调用任何生命周期方法，调用该方法的前提是要显示的Fragment已经被添加到容器，只是纯粹把Fragment UI的setVisibility为false。

* detach(): onPause()-&gt;onStop()-&gt;onDestroyView()。UI从布局中移除，但是仍然被FragmentManager管理。

* attach(): onCreateView()-&gt;onStart()-&gt;onResume()。

### Fragment实现原理和Back Stack

我们知道Activity有任务栈，用户通过startActivity将Activity加入栈，点击返回按钮将Activity出栈。Fragment也有类似的栈，称为回退栈（Back Stack），回退栈是由FragmentManager管理的。默认情况下，Fragment事务是不会加入回退栈的，如果想将Fragment事务加入回退栈，则可以加入`addToBackStack("")`。如果没有加入回退栈，则用户点击返回按钮会直接将Activity出栈；如果加入了回退栈，则用户点击返回按钮会回滚Fragment事务。

```java
getSupportFragmentManager()
    .beginTransaction()
    .add(R.id.fragment_container, BlankFragment.newInstance("a","b"),"blank")
    //.addToBackStack("BackStack")
    .commit();
```

查看`FragmentManager`源码 ，其由三个类组成：

当我们调用`beginTransaction`时，其由`FragmentManagerImpl`实现，

```java
public abstract class FragmentManager

final class FragmentManagerState implements Parcelable

final class FragmentManagerImpl extends FragmentManager implements LayoutInflaterFactory

    @Override
    public FragmentTransaction beginTransaction() {
        return new BackStackRecord(this);
    }

    final class BackStackRecord extends FragmentTransaction implements
        FragmentManager.BackStackEntry, FragmentManagerImpl.OpGenerator {
    }
```

从定义可以看出，BackStackRecord有三重含义：

* 继承了FragmentTransaction，即是事务，保存了整个事务的全部操作轨迹。

* 实现了BackStackEntry，作为回退栈的元素，正是因为该类拥有事务全部的操作轨迹，因此在popBackStack()时能回退整个事务。

* 待了解FragmentManagerImpl.OpGenerator，感觉是状态处理

BackStackRecord类包含了一次事务的整个操作轨迹，是以链表形式存在的，链表的元素是Op类，表示其中某个操作，定义如下：

```java
    static final class Op {
        int cmd;//操作的命令（add，remove。replace。。。）
        Fragment fragment;//操作的Fragment对象
        int enterAnim;
        int exitAnim;
        int popEnterAnim;
        int popExitAnim;
    }
```

下面我们来看`add`操作

```java
    @Override
    public FragmentTransaction add(int containerViewId, Fragment fragment) {
        doAddOp(containerViewId, fragment, null, OP_ADD);
        return this;
    }


    private void doAddOp(int containerViewId, Fragment fragment, String tag, int opcmd) {
        final Class fragmentClass = fragment.getClass();
        final int modifiers = fragmentClass.getModifiers();
        if (fragmentClass.isAnonymousClass() || !Modifier.isPublic(modifiers)
                || (fragmentClass.isMemberClass() && !Modifier.isStatic(modifiers))) {
            throw new IllegalStateException("Fragment " + fragmentClass.getCanonicalName()
                    + " must be a public static class to be  properly recreated from"
                    + " instance state.");
        }

        fragment.mFragmentManager = mManager;

        if (tag != null) {
            if (fragment.mTag != null && !tag.equals(fragment.mTag)) {
                throw new IllegalStateException("Can't change tag of fragment "
                        + fragment + ": was " + fragment.mTag
                        + " now " + tag);
            }
            fragment.mTag = tag;
        }

        if (containerViewId != 0) {
            if (containerViewId == View.NO_ID) {
                throw new IllegalArgumentException("Can't add fragment "
                        + fragment + " with tag " + tag + " to container view with no id");
            }
            if (fragment.mFragmentId != 0 && fragment.mFragmentId != containerViewId) {
                throw new IllegalStateException("Can't change container ID of fragment "
                        + fragment + ": was " + fragment.mFragmentId
                        + " now " + containerViewId);
            }
            fragment.mContainerId = fragment.mFragmentId = containerViewId;
        }

        Op op = new Op();
        op.cmd = opcmd;
        op.fragment = fragment;
        addOp(op);
    }

    void addOp(Op op) {
        mOps.add(op);//add()是将创建好的Op对象加入链表
        op.enterAnim = mEnterAnim;
        op.exitAnim = mExitAnim;
        op.popEnterAnim = mPopEnterAnim;
        op.popExitAnim = mPopExitAnim;
    }
```

接着看`commit`操作

```java
    @Override
    public int commit() {
        return commitInternal(false);
    }

    int commitInternal(boolean allowStateLoss) {
        if (mCommitted) throw new IllegalStateException("commit already called");
        if (FragmentManagerImpl.DEBUG) {
            Log.v(TAG, "Commit: " + this);
            LogWriter logw = new LogWriter(TAG);
            PrintWriter pw = new PrintWriter(logw);
            dump("  ", null, pw, null);
            pw.close();
        }
        mCommitted = true;
        if (mAddToBackStack) {
            mIndex = mManager.allocBackStackIndex(this);
        } else {
            mIndex = -1;
        }
        mManager.enqueueAction(this, allowStateLoss);
        return mIndex;
    }
```

如果mAddToBackStack为true，则调用`allocBackStackIndex(this)`将事务添加进回退栈，FragmentManager类的变量`ArrayList<BackStackRecord> mBackStackIndices;`就是回退栈。实现如下：

```java
    public int allocBackStackIndex(BackStackRecord bse) {
        synchronized (this) {
            if (mAvailBackStackIndices == null || mAvailBackStackIndices.size() <= 0) {
                if (mBackStackIndices == null) {
                    mBackStackIndices = new ArrayList<BackStackRecord>();
                }
                int index = mBackStackIndices.size();
                if (DEBUG) Log.v(TAG, "Setting back stack index " + index + " to " + bse);
                mBackStackIndices.add(bse);
                return index;

            } else {
                int index = mAvailBackStackIndices.remove(mAvailBackStackIndices.size()-1);
                if (DEBUG) Log.v(TAG, "Adding back stack index " + index + " with " + bse);
                mBackStackIndices.set(index, bse);
                return index;
            }
        }
    }

    /**
     * Adds an action to the queue of pending actions.
     *
     * @param action the action to add
     * @param allowStateLoss whether to allow loss of state information
     * @throws IllegalStateException if the activity has been destroyed
     */
    public void enqueueAction(OpGenerator action, boolean allowStateLoss) {
        if (!allowStateLoss) {
            checkStateLoss();
        }
        synchronized (this) {
            if (mDestroyed || mHost == null) {
                throw new IllegalStateException("Activity has been destroyed");
            }
            if (mPendingActions == null) {
                mPendingActions = new ArrayList<>();
            }
            mPendingActions.add(action);
            scheduleCommit();
        }
    }

    /**
     * Schedules the execution when one hasn't been scheduled already. This should happen
     * the first time {@link #enqueueAction(OpGenerator, boolean)} is called or when
     * a postponed transaction has been started with
     * {@link Fragment#startPostponedEnterTransition()}
     */
    private void scheduleCommit() {
        synchronized (this) {
            boolean postponeReady =
                    mPostponedTransactions != null && !mPostponedTransactions.isEmpty();
            boolean pendingReady = mPendingActions != null && mPendingActions.size() == 1;
            if (postponeReady || pendingReady) {
                mHost.getHandler().removeCallbacks(mExecCommit);
                mHost.getHandler().post(mExecCommit);
            }
        }
    }
```

mPendingActions就是前面说的待执行队列，`mHost.getHandler()`就是主线程的Handler，因此Runnable是在主线程执行的，mExecCommit的内部就是调用了`execPendingActions()`，即把mPendingActions中所有积压的没被执行的事务全部执行。执行队列中的事务会怎样被执行呢？就是调用BackStackRecord的`run()`方法，`run()`方法就是执行Fragment的生命周期函数，还有将视图添加进container中。




