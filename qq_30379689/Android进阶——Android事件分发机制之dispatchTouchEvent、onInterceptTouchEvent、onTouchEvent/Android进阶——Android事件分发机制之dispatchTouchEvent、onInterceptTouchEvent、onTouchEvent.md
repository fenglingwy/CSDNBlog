#Android事件分发机制之dispatchTouchEvent、onInterceptTouchEvent、onTouchEvent


----------

>本篇文章包括以下内容：

>* [前言](#a)
>* [Android事件分发机制的简介](#b)
>* [Android事件分发机制的相关概念](#c)
>* [Android事件分发机制的分发例子](#d)
>* [Android事件分发机制的分发流程](#e)
>* [ViewGroup事件分发源码分析](#f)
>* [View事件分发源码分析](#g)
>* [View事件方法执行顺序](#k)
>* [面试题](#i)
>* [结语](#j)

##**<span id="a">前言</span>**

Android事件分发机制可以说是我们Android工程师面试题中的必考题，弄懂它的原理是我们避不开的任务，所以长痛不如短痛，花点时间干掉他，废话不多说，开车啦

##**<span id="b">Android事件分发机制的简介</span>**
Android事件分发机制的发生在View与View之间或者ViewGroup与View之间具有镶嵌的视图上，而且视图上必须为点击可用。当一个点击事件产生后，它的传递过程遵循如下顺序：Activity->Window->View，即事件先传递给Activity，再到Window，再到顶级View，才开始我们的事件分发

##**<span id="c">Android事件分发机制的相关概念</span>**
Android事件分发机制主要由三个重要的方法共同完成的

* dispatchTouchEvent：用于进行点击事件的分发
* onInterceptTouchEvent：用于进行点击事件的拦截
* onTouchEvent：用于处理点击事件

**这里需要注意的是View中是没有onInterceptTouchEvent()方法的**

##**<span id="d">Android事件分发机制的分发例子</span>**
这里以两个ViewGroup嵌套View来演示，下面是演示图

![](http://img.blog.csdn.net/20170101234918740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**一、MyView**
继承View并覆写其三个构造方法，覆写dispatchTouchEvent和onTouchEvent，前面已经说了View是没有onInterceptTouchEvent方法的
```
public class MyView extends View {
    public MyView(Context context) {
        super(context);
    }

    public MyView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public MyView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        System.out.println("MyView dispatchTouchEvent");
        return super.dispatchTouchEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        System.out.println("MyView onTouchEvent");
        return super.onTouchEvent(event);
    }
}
```
###**二、MyViewGroup01和MyViewGroup02**
MyViewGroup01和MyViewGroup02是一样的代码，这里以01为例，继承ViewGroup并覆写其三个构造方法，覆写dispatchTouchEvent和onTouchEvent和onInterceptTouchEvent方法
```
public class MyViewGroup01 extends LinearLayout {

    public MyViewGroup01(Context context) {
        super(context);
    }

    public MyViewGroup01(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public MyViewGroup01(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        System.out.println("MyViewGroup01 dispatchTouchEvent");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        System.out.println("MyViewGroup01 onInterceptTouchEvent");
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        System.out.println("MyViewGroup01 onTouchEvent");
        return super.onTouchEvent(event);
    }
}
```
###**三、MyView和MyViewGroup布局文件**
这里以ViewGroup和Group嵌套，由上面可以知道事件最后分配到布局的顶级View，这里的顶级View是MyViewGroup02，然后开始事件的传递
```xml
<?xml version="1.0" encoding="utf-8"?>
<com.handsome.boke2.TouchEvent.MyViewGroup02 xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:background="#0f0">

    <com.handsome.boke2.TouchEvent.MyViewGroup01
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="#f00">

        <com.handsome.boke2.TouchEvent.MyView
            android:layout_width="50dp"
            android:layout_height="50dp"
            android:background="#00f" />
    </com.handsome.boke2.TouchEvent.MyViewGroup01>
</com.handsome.boke2.TouchEvent.MyViewGroup02>
```
###**四、分析事件传递**
点击MyView（即蓝色部分）：其事件会从顶级View（MyViewGroup02）往下分发，而事件的分发过程中分为两步骤

* 分发过程
* 处理过程

其正常的分发事件结果为
```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
MyViewGroup01 onInterceptTouchEvent
MyView dispatchTouchEvent
//处理过程
MyView onTouchEvent
MyViewGroup01 onTouchEvent
MyViewGroup02 onTouchEvent
```


**1、dispatchTouchEvent（分发事件）**
如果在MyViewGroup01的dispatchTouchEvent方法中返回true，表示需要在MyViewGroup01消费了整个事件，即不会再分发，也不会再处理。dispatchTouchEvent方法中返回true的打印信息

```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
```
如果在MyViewGroup01的dispatchTouchEvent方法中返回false，表示在MyViewGroup01点击事件在本层不再继续进行分发，并交由上层控件的onTouchEvent方法进行消费。dispatchTouchEvent方法中返回false的打印信息

```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
//处理过程
MyViewGroup02 onTouchEvent

```

**2、onInterceptTouchEvent（拦截事件）**

如果在MyViewGroup01的onInterceptTouchEvent方法中返回true，表示需要在MyViewGroup01拦截这个点击事件，不再继续往下分发，即MyView不再执行dispatchTouchEvent方法。但是只是分发结束了而已，接着开始处理事件。下面是onInterceptTouchEvent方法中返回true的打印信息

```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
MyViewGroup01 onInterceptTouchEvent
//处理过程
MyViewGroup01 onTouchEvent
MyViewGroup02 onTouchEvent
```

如果在MyViewGroup01的onInterceptTouchEvent方法中返回false，表示需要在MyViewGroup01不会拦截这个点击事件，继续往下分发。下面是onInterceptTouchEvent方法中返回false的打印信息
```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
MyViewGroup01 onInterceptTouchEvent
MyView dispatchTouchEvent
//处理过程
MyView onTouchEvent
MyViewGroup01 onTouchEvent
MyViewGroup02 onTouchEvent
```
**3、onTouchEvent（消费事件）**

如果MyViewGroup01的onTouchEvent方法中返回true，表示MyViewGroup01可以将该事件直接消费掉了，即分发结束后，处理事件的时候，直接处理到MyViewGroup01就可以结束了。下面是onTouchEvent方法中返回true的打印信息

```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
MyViewGroup01 onInterceptTouchEvent
MyView dispatchTouchEvent
//处理过程
MyView onTouchEvent
MyViewGroup01 onTouchEvent
```
如果MyViewGroup01的onTouchEvent方法中返回false，表示MyViewGroup01不可以将该事件直接消费掉，即事件继续往上处理。下面是onTouchEvent方法中返回false的打印信息
```java
//分发过程
MyViewGroup02 dispatchTouchEvent
MyViewGroup02 onInterceptTouchEvent
MyViewGroup01 dispatchTouchEvent
MyViewGroup01 onInterceptTouchEvent
MyView dispatchTouchEvent
//处理过程
MyView onTouchEvent
MyViewGroup01 onTouchEvent
MyViewGroup02 onTouchEvent
```

总结起来：

* **dispatchTouchEvent**
	* return true：表示该View内部消化掉了所有事件
	* return false：表示事件在本层不再继续进行分发，并交由上层控件的onTouchEvent方法进行消费
	* return super.dispatchTouchEvent(ev)：默认事件将分发给本层的事件拦截onInterceptTouchEvent 方法进行处理
* **onInterceptTouchEvent**
	* return true：表示将事件进行拦截，并将拦截到的事件交由本层控件 的 onTouchEvent 进行处理
	* return false：表示不对事件进行拦截，事件得以成功分发到子View
	* return super.onInterceptTouchEvent(ev)：默认表示不拦截该事件，并将事件传递给下一层View的dispatchTouchEvent
* **onTouchEvent**
	* return true：表示onTouchEvent处理完事件后消费了此次事件
	* return fasle：表示不响应事件，那么该事件将会不断向上层View的onTouchEvent方法传递，直到某个View的onTouchEvent方法返回true
	* return super.dispatchTouchEvent(ev)：表示不响应事件，结果与return false一样

##**<span id="e">Android事件分发机制的分发流程</span>**
这里以网上的图片来说明，如果对上面分发例子还不太懂的同学，看这张图片已经很生动的说明了整个过程

![](http://img.blog.csdn.net/20170101220052950?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**<span id="f">ViewGroup事件分发源码分析</span>**
我们这里以ViewGroup的dispatchTouchEvent()方法开始讲解，这里面主要有两件事情

* 询问是否拦截事件
* 遍历子View并分发事件

**一、dispatchTouchEvent源码询问拦截事件**

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
		
	......
	
	// Check for interception.
	final boolean intercepted;
	//这里检查是否拦截事件
	if (actionMasked == MotionEvent.ACTION_DOWN
	        || mFirstTouchTarget != null) {
	    final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
	    if (!disallowIntercept) {
	        intercepted = onInterceptTouchEvent(ev);
	        ev.setAction(action); // restore action in case it was changed
	    } else {
	        intercepted = false;
	    }
	} else {
	    // There are no touch targets and this action is not an initial down
	    // so this view group continues to intercept touches.
	    intercepted = true;
	}

	......
}
```
ViewGroup在两种情况下都会判断是否要拦截当前事件

* 事件类型为ACTION_DOWN：当前由我们触发的点击事件，也即是说ACTION_MOVE和ACTION_UP事件来时，则不触发拦截事件
* mFirstTouchTarget != null：当ViewGroup不拦截事件并将事件交给子View的时候该不等式成立。反过来，事件被ViewGroup拦截时，该不等式不成立

**二、dispatchTouchEvent源码遍历子View并分发事件**

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
		
	......
	
	final View[] children = mChildren;
	//遍历所有子View
	for (int i = childrenCount - 1; i >= 0; i--) {
	    final int childIndex = getAndVerifyPreorderedIndex(
	            childrenCount, i, customOrder);
	    final View child = getAndVerifyPreorderedView(
	            preorderedList, children, childIndex);
	
	    //判断子View是否能接收点击事件
	    if (childWithAccessibilityFocus != null) {
	        if (childWithAccessibilityFocus != child) {
	            continue;
	        }
	        childWithAccessibilityFocus = null;
	        i = childrenCount - 1;
	    }
	
		//判断子元素在播放动画时落在子元素的区域内
	    if (!canViewReceivePointerEvents(child)
	            || !isTransformedTouchPointInView(x, y, child, null)) {
	        ev.setTargetAccessibilityFocus(false);
	        continue;
	    }
	    
		//判断子元素点击事件是否落在子元素的区域内
	    newTouchTarget = getTouchTarget(child);
	    if (newTouchTarget != null) {
	        // Child is already receiving touch within its bounds.
	        // Give it the new pointer in addition to the ones it is handling.
	        newTouchTarget.pointerIdBits |= idBitsToAssign;
	        break;
	    }
	
	    resetCancelNextUpFlag(child);
	    //事件传递到子View，下面追踪该方法
	    if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {
	        // Child wants to receive touch within its bounds.
	        mLastTouchDownTime = ev.getDownTime();
	        if (preorderedList != null) {
	            // childIndex points into presorted list, find original index
	            for (int j = 0; j < childrenCount; j++) {
	                if (children[childIndex] == mChildren[j]) {
	                    mLastTouchDownIndex = j;
	                    break;
	                }
	            }
	        } else {
	            mLastTouchDownIndex = childIndex;
	        }
	        mLastTouchDownX = ev.getX();
	        mLastTouchDownY = ev.getY();
	        newTouchTarget = addTouchTarget(child, idBitsToAssign);
	        alreadyDispatchedToNewTouchTarget = true;
	        break;
	    }
	
	    // The accessibility focus didn't handle the event, so clear
	    // the flag and do a normal dispatch to all children.
	    ev.setTargetAccessibilityFocus(false);
	}

	......
}
```
ViewGroup直接使用for遍历所有子View，对子View的各种状态进行判断，最后调用dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)将事件传递给子View，下面是dispatchTransformedTouchEvent()方法的部分源码

```
if (child == null) {
    handled = super.dispatchTouchEvent(event);
} else {
    handled = child.dispatchTouchEvent(event);
}
```
其最后就是分发给子View的dispatchTouchEvent()方法，那么事件就分发到子View中去了

##**<span id="g">View事件分发源码分析</span>**
View对点击事件的处理过程主要有

* View的dispatchTouchEvent()：判断分发事件
* View的onTouchEvent()：处理事件的具体做法


**一、dispatchTouchEvent源码判断分发事件部分**
```
public boolean dispatchTouchEvent(MotionEvent event) {

	boolean result = false;
	......
	if (onFilterTouchEventForSecurity(event)) {
	    if ((mViewFlags & ENABLED_MASK) == ENABLED && handleScrollBarDragging(event)) {
	        result = true;
	    }
	    //noinspection SimplifiableIfStatement
	    ListenerInfo li = mListenerInfo;
	    //这里开始判断
	    if (li != null && li.mOnTouchListener != null
	            && (mViewFlags & ENABLED_MASK) == ENABLED
	            && li.mOnTouchListener.onTouch(this, event)) {
	        result = true;
	    }
	
	    if (!result && onTouchEvent(event)) {
	        result = true;
	    }
	}
	
	......
	return result;
}
```
从源码判断处看出，首先会判断有没有设置mOnTouchListener，如果mOnTouchListener不为空，那么onTouchEvent就不会被调用，这里可以得到一个结论，若在View中设置了OnTouchListener，那么它的优先级是高于onTouchEvent的，这样可以更好的让我们自己setOnTouchEventListener()处理点击事件

**二、onTouchEvent源码处理事件的具体做法部分**

```
public boolean onTouchEvent(MotionEvent event) {
	......
	//当View处于不可用状态下，也会消耗点击事件
	if ((viewFlags & ENABLED_MASK) == DISABLED) {
        if (action == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
            setPressed(false);
        }
        // A disabled view that is clickable still consumes the touch
        // events, it just doesn't respond to them.
        return (((viewFlags & CLICKABLE) == CLICKABLE
                || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
    }

	......
	//对点击事件的具体处理
	if (((viewFlags & CLICKABLE) == CLICKABLE ||
            (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE) ||
            (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE) {
        switch (action) {
            case MotionEvent.ACTION_UP:
                boolean prepressed = (mPrivateFlags & PFLAG_PREPRESSED) != 0;
                if ((mPrivateFlags & PFLAG_PRESSED) != 0 || prepressed) {
                    ......
                    if (!mHasPerformedLongPress && !mIgnoreNextUpEvent) {
                        // This is a tap, so remove the longpress check
                        removeLongPressCallback();

                        // Only perform take click actions if we were in the pressed state
                        if (!focusTaken) {
                            // Use a Runnable and post this rather than calling
                            // performClick directly. This lets other visual state
                            // of the view update before click actions start.
                            if (mPerformClick == null) {
                                mPerformClick = new PerformClick();
                            }
                            if (!post(mPerformClick)) {
                                performClick();
                            }
                        }
                    }
					......
				}
        }

        return true;
    }
    ......
}
```
从对点击事件的具体处理中看出，只要View的CLICKABLE和LONG_CLICKABLE有一个为true，那么它就会消耗这个事件，即onTouchEvent方法返回true。在ACTION_UP事件中，会触发PerformClick()方法，如果View设置了OnClickListener，那么PerformClick()方法内部会调用它的onClick()方法，这里就不分析它的点击事件了

##**<span id="k">View事件方法执行顺序</span>**

onTouchListener > onTouchEvent > onLongClickListener > onClickListener 

##**<span id="i">面试题</span>**
**一、简要的谈谈Android的事件分发机制？**

当点击事件发生时，首先Activity将TouchEvent传递给Window，再从Window传递给顶层View。TouchEvent会最先到达最顶层 view 的 dispatchTouchEvent ，然后由  dispatchTouchEvent 方法进行分发，如果dispatchTouchEvent返回true ，则交给这个view的onTouchEvent处理，如果dispatchTouchEvent返回 false ，则交给这个 view 的 interceptTouchEvent 方法来决定是否要拦截这个事件，如果 interceptTouchEvent 返回 true ，也就是拦截掉了，则交给它的 onTouchEvent 来处理，如果 interceptTouchEvent 返回 false ，那么就传递给子 view ，由子 view 的 dispatchTouchEvent 再来开始这个事件的分发。如果事件传递到某一层的子 view 的 onTouchEvent 上了，这个方法返回了 false ，那么这个事件会从这个 view 往上传递，都是 onTouchEvent 来接收。而如果传递到最上面的 onTouchEvent 也返回 false 的话，这个事件就会“消失”，而且接收不到下一次事件。

**二、为什么View有dispatchTouchEvent方法？**

因为View可以注册很多事件的监听器，如长按、滑动、点击等，它也需要一个管理者来分发

**三、ViewGroup中可能有很多个子View，如何判断应该分配给哪一个？**

根据源码可知，它会分配给在点击范围内的子View

**四、当点击时，子View重叠应该如何分配？**

一般分配给最上层的子View，这是由于安卓的渲染机制导致的

##**<span id="j">结语</span>**
事件分发机制就犹如数学的定理是一样道理的，只有记住定理，才能在具体应用中具体分析，有人可能不知道在什么地方会用到，如果你做的项目中，比如有一个控件点击不能反应，那么就有可能是事件分发的结果。至于对源码的分析可能内容比较复杂，内容也多，源码部分有些也让我不是很懂，这里是浅析一下，做个开头，希望大家下去有时间可以翻源码去理解这一机制，将它运用在实战中吧