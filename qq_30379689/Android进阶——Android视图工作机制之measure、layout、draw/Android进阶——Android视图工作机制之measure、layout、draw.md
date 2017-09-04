#Android视图工作机制之measure、layout、draw


----------

>本篇文章包括以下内容：

>* [前言](#a)
>* [Android视图工作机制简介](#b)
>* [Android视图工作机的相关概念](#d)
>* [Android视图工作机制之MeasureSpec](#e)
>* [Android视图工作机制之measure过程](#f)
>* [Android视图工作机制之layout过程](#g)
>* [Android视图工作机制之draw过程](#h)
>* [Android视图工作机制中的重绘](#j)
>* [结语](#i)

##**<span id="a">前言</span>**
自定义View一直是初学者们最头疼的事情，因为他们并没有了解到真正的实现原理就开始试着做自定义View，碰到很多看不懂的代码只能选择回避，做多了会觉得很没自信。其实只要了解了View的工作机制后，会发现是挺简单的，自定义View就是借助View的工作机制开始将View绘制出来的
##**<span id="b">Android视图工作机制简介</span>**
Android视图工作机制按顺序分为以下三步：

1. measure：确定View的宽高
2. layout：确定View的位置
3. draw：绘制出View的形状

##**<span id="d">Android视图工作机制的相关概念</span>**
Android视图工作机制其实挺人性化的，当你真正理解之后，就跟我们画画是一个道理的，下面为了更好的理解，我将自定义View的过程拟物化

![](http://img.blog.csdn.net/20170119233820894?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

相关概念：

* View（照片框）：自定义View
* measure（尺子）：测量View大小
* MeasureSpec（尺子刻度）：测量View大小的测量单位
* layout（照片框的位置）：View的具体位置
* draw（笔）：绘制View

画图步骤：

1. 首先画一个100 x 100的照片框，需要尺子测量出宽高的长度（measure过程）
2. 然后确定照片框在屏幕中的位置（layout过程）
3. 最后借助尺子用手画出我们的照片框（draw过程）

>细心的你会发现，现实中的画图步骤和View工作机制步骤是一样的

##**<span id="e">Android视图工作机制之MeasureSpec</span>**
>我们知道，自定义View第一步是测量，而测量需要测量规格（或测量标准）才能知道View的宽高，所以在测量之前需要认识MeasureSpec类

MeasureSpec类是决定View的measure过程的测量规格（比喻：尺子），它由以下两部分组成

* SpecMode：测量模式（比喻：直尺、三角尺等不同类型）
* SpecSize：测量模式下的规格大小（比喻：尺子的刻度）

MeasureSpec的表示形式是32位的int值

* 高2位（前面2位）：表示测量模式，即SpecMode
* 低30位（后面30位）：表示在测量模式下的测量规格大小，即SpecSize

>其实就是MeasureSpec通过将SpecMode和SpecSize打包成一个int值来避免过多的对象内存分配，这点可以从其源码中看出

```
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;
    public static final int EXACTLY     = 1 << MODE_SHIFT;
    public static final int AT_MOST     = 2 << MODE_SHIFT;

    public static int makeMeasureSpec(int size, int mode) {
        if (sUseBrokenMakeMeasureSpec) {
            return size + mode;
        } else {
            return (size & ~MODE_MASK) | (mode & MODE_MASK);
        }
    }

    public static int makeSafeMeasureSpec(int size, int mode) {
        if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
            return 0;
        }
        return makeMeasureSpec(size, mode);
    }

    public static int getMode(int measureSpec) {
        return (measureSpec & MODE_MASK);
    }

    public static int getSize(int measureSpec) {
        return (measureSpec & ~MODE_MASK);
    }
}
```

我们都知道SpecMode的尺子类型有很多，不同的尺子有不同的功能，而SpecSize刻度是固定的一种，所以SpecMode又分为三种模式

* UNSPECIFIED：父容器不对View有任何大小的限制，这种情况一般用于系统内部，表示一种测量状态
* EXACTLY：父容器检测出View所需要的精确大小，这时候View的值就是SpecSize
* AT_MOST：父容器指定了一个可用大小即SpecSize，View的大小不能大于这个值

**一、结论：子View的MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同决定的**

1. 首先要知道LayoutParams有三种情况：MATCH_PARENT、WARP_CONTENT、100dp（精确大小）
2. 只要子View的MeasureSpec被确定，那么就可以在measure过程中，测量出子View的宽高

**二、通过例子来解释结论**

1. 假如父容器LinearLayout的MeasureSpec：EXACTLY、AT_MOST的任意一种
子View的LayoutParams：**精确大小（100dp）**
也就是说：**子View必须是指定大小，不管父容器载不载得下子View**
所以返回子View的MeasureSpec：EXACTLY
所以返回子View测量出来的大小：子View自身精确大小

2. 假如父容器LinearLayout的MeasureSpec：EXACTLY、AT_MOST的任意一种
子View的LayoutParams：**MATCH_PARENT**
也就是说：**子View必须占满整个父容器，那么父容器多大，子View就多大**
所以返回子View的MeasureSpec：跟父容器一致
所以返回子View测量出来的大小：父容器可用大小

3. 假如父容器LinearLayout的MeasureSpec：EXACTLY、AT_MOST的任意一种
子View的LayoutParams：**WARP_CONTENT**
也就是说：**子View必须自适应父容器，父容器不管多小，你都不能超过它，只能自适应的缩小**
所以返回子View的MeasureSpec：AT_MOST（不能超过父容器本身）
所以返回子View测量出来的大小：父容器可用大小

>至于第4种情况，父容器是UNSPECIFIED的时候，由于父容器不知道自己多大，而子View又采用MATCH_PARENT、WARP_CONTENT的时候，子View肯定也不知道自己多大，所以只有当子View采用EXACTLY的时候，才知道自己多大

**三、通过图片分析结论结果**

通过上面的例子总结，我们可以通过父容器的测量规格和子View的布局参数来确定子View的MeasureSpec，这样便确立了子View的宽高，下面是父容器测量规格和子View布局参数确立子ViewMeasureSpec的结果图

![](http://img.blog.csdn.net/20170123001028935?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>知道怎么测量宽高了，那么下面就开始测量呗

##**<span id="f">Android视图工作机制之measure过程</span>**

measure过程其实和事件分发有点类似，也包括ViewGroup和View，我们通过各自的源码来分析其measure的过程

###**一、ViewGroup的measure过程**
ViewGroup源码中，提供了一个measureChildren的方法来遍历调用子View的measure方法，而各个子View再递归去执行这个过程

```
protected void measureChild(View child, int parentWidthMeasureSpec,
        int parentHeightMeasureSpec) {
    final LayoutParams lp = child.getLayoutParams();
	//分析这里
    final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
            mPaddingLeft + mPaddingRight, lp.width);
    final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
            mPaddingTop + mPaddingBottom, lp.height);
	//开始子View的measure过程
    child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
}
```
从源码中分析，ViewGroup的measure过程特别简单，并且可以看到子View的MeasureSpec确实是通过父容器的MeasureSpec和自身的LayoutParams决定的，这也就印证了结论所说的话，但是measure的过程还是依靠子View自身的MeasureSpec

###**二、View的measure过程**

View的源码中，由于measure方法是个final类型的，所以子类不能重写此方法

```
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
	......
    if ((mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ||
            widthMeasureSpec != mOldWidthMeasureSpec ||
            heightMeasureSpec != mOldHeightMeasureSpec) {

        int cacheIndex = (mPrivateFlags & PFLAG_FORCE_LAYOUT) == PFLAG_FORCE_LAYOUT ? -1 :
                mMeasureCache.indexOfKey(key);
        if (cacheIndex < 0 || sIgnoreMeasureCache) {
            //这里调用onMeasure
            onMeasure(widthMeasureSpec, heightMeasureSpec);
            mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        } else {
            long value = mMeasureCache.valueAt(cacheIndex);
            // Casting a long to int drops the high 32 bits, no mask needed
            setMeasuredDimensionRaw((int) (value >> 32), (int) value);
            mPrivateFlags3 |= PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
        }
    }
	......
}
```
可以发现，View的measure方法中，会调用自身的onMeasure方法（平时，自定义View重写这个方法，就是对自定义的View根据自己定的规则来确定测量大小），下面继续追踪

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```
从onMeasure方法中，有getDefaultSize()、getSuggestedMinimumWidth()、getSuggestedMinimumHeight()，它们之间又是什么呢，继续追踪

① getDefaultSize()
```
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        result = specSize;
        break;
    }
    return result;
}
```
很显然，如果你自定义不重写onMeasure()方法的话，那么系统就会采用默认的测量模式来确定你的测量大小，即getDefaultSize()，它的逻辑很简单，不去看UNSPECIFIED模式，它就是返回specSize，即View测量后的大小

② getSuggestedMinimumWidth()和getMinimumHeight()
```
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}

protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}
```
getSuggestedMinimumWidth和getSuggestedMinimumHeight原理是一样的，如果View没有设置背景，那么View的宽度为mMinWidth，而mMinWidth对应的就是android:minWidth这个属性的值，如果这个属性不指定，那么mMinWidth默认为0；如果指定了背景，那么View的宽度就是max(mMinWidth, mBackground.getMinimumWidth())，而这里的getMinimumWidth()又是什么，继续追踪

```
public int getMinimumWidth() {
    final int intrinsicWidth = getIntrinsicWidth();
    return intrinsicWidth > 0 ? intrinsicWidth : 0;
}
```
getMinimumWidth是在Drawable类中的，它返回的是Drawable的原始宽度，如果没有Drawable，则返回0

>到这里measure过程就结束了，如果是自定义View的话，就重写onMeasure方法，将其默认的测量方式改为我们自己规定的测量方式，最后获得我们的宽高

##**<span id="g">Android视图工作机制之layout过程</span>**
layout过程就比measure过程简单多了，因为它不用什么规格之类的东西，下面是View的layout源码

```
public void layout(int l, int t, int r, int b) {
    if ((mPrivateFlags3 & PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT) != 0) {
        onMeasure(mOldWidthMeasureSpec, mOldHeightMeasureSpec);
        mPrivateFlags3 &= ~PFLAG3_MEASURE_NEEDED_BEFORE_LAYOUT;
    }

    int oldL = mLeft;
    int oldT = mTop;
    int oldB = mBottom;
    int oldR = mRight;

    boolean changed = isLayoutModeOptical(mParent) ?
            setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

    if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) {
	    //调用onLayout
        onLayout(changed, l, t, r, b);
        mPrivateFlags &= ~PFLAG_LAYOUT_REQUIRED;

        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnLayoutChangeListeners != null) {
            ArrayList<OnLayoutChangeListener> listenersCopy =
                    (ArrayList<OnLayoutChangeListener>)li.mOnLayoutChangeListeners.clone();
            int numListeners = listenersCopy.size();
            for (int i = 0; i < numListeners; ++i) {
                listenersCopy.get(i).onLayoutChange(this, l, t, r, b, oldL, oldT, oldR, oldB);
            }
        }
    }

    mPrivateFlags &= ~PFLAG_FORCE_LAYOUT;
    mPrivateFlags3 |= PFLAG3_IS_LAID_OUT;
}
```
可以发现，View只需要4个点即可确定一个矩形，就是View的位置，然后调用onLayout()

```
protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
}
```
onLayout()方法其实就是一个空方法，当我们在自定义View时重写onLayout()方法，其实就是让我们重新设置View的位置

##**<span id="h">Android视图工作机制之draw过程</span>**
draw过程也很简单，就是将View绘制到屏幕上，它有如下几个步骤

1. 绘制背景：drawBackground(canvas)
2. 绘制自己：if (!dirtyOpaque) onDraw(canvas)
3. 绘制children：dispatchDraw(canvas)
4. 绘制装饰：onDrawForeground(canvas)
```
public void draw(Canvas canvas) {
    final int privateFlags = mPrivateFlags;
    final boolean dirtyOpaque = (privateFlags & PFLAG_DIRTY_MASK) == PFLAG_DIRTY_OPAQUE &&
            (mAttachInfo == null || !mAttachInfo.mIgnoreDirtyState);
    mPrivateFlags = (privateFlags & ~PFLAG_DIRTY_MASK) | PFLAG_DRAWN;
    // Step 1, draw the background, if needed
    int saveCount;

    if (!dirtyOpaque) {
        drawBackground(canvas);
    }

    // skip step 2 & 5 if possible (common case)
    final int viewFlags = mViewFlags;
    boolean horizontalEdges = (viewFlags & FADING_EDGE_HORIZONTAL) != 0;
    boolean verticalEdges = (viewFlags & FADING_EDGE_VERTICAL) != 0;
    if (!verticalEdges && !horizontalEdges) {
        // Step 3, draw the content
        if (!dirtyOpaque) onDraw(canvas);

        // Step 4, draw the children
        dispatchDraw(canvas);

        // Overlay is part of the content and draws beneath Foreground
        if (mOverlay != null && !mOverlay.isEmpty()) {
            mOverlay.getOverlayView().dispatchDraw(canvas);
        }

        // Step 6, draw decorations (foreground, scrollbars)
        onDrawForeground(canvas);

        // we're done...
        return;
    }
    ......
}
```
>我们常常就是重写onDraw()方法来绘制我们的自定义View，否则是没有图像的，这点在源码中也是提供了onDraw()的空实现方法给我们去绘制图像

##**<span id="j">Android视图工作机制中的重绘</span>**

###**一、invalidate()和requestLayout()**

invalidate()和requestLayout()，常用于View重绘和更新，其主要区别如下

* invalidate方法只会执行onDraw方法
* requestLayout方法只会执行onMeasure方法和onLayout方法，并不会执行onDraw方法。

所以当我们进行View更新时，若仅View的显示内容发生改变且新显示内容不影响View的大小、位置，则只需调用invalidate方法；若View宽高、位置发生改变且显示内容不变，只需调用requestLayout方法；若两者均发生改变，则需调用两者，按照View的绘制流程，推荐先调用requestLayout方法再调用invalidate方法

###**二、invalidate()和postInvalidate()**

* invalidate方法用于UI线程中重新绘制视图
* postInvalidate方法用于非UI线程中重新绘制视图，省去使用handler

##**<span id="i">结语</span>**
Android的自定义其实很简单，对于初学者，可能就是measure过程比较难以理解，不过不要紧，每个人初学都是这样的，建议多多实践，花点时间去研究，你会更加熟能生巧，根本不用死记硬背，只要有思路便可以画出你想要的自定义View，当然，能结合动画那就更完美了，加油