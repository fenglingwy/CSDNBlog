# Android群英传知识点回顾——第五章：Android Scroll分析 #
----------
##知识点目录##
>* **5.1 滑动效果是如何产生的**
	*  5.1.1 Android坐标系
	*  5.1.2 视图坐标系
	*  5.1.3 触控事件——MotionEvent
>* **5.2 实现滑动的七种方法**
	*  5.2.1 layout方法
	*  5.2.2 offsetLeftAndRight()与offsetTopAndBottom()
	*  5.2.3 LayoutParams
	*  5.2.4 scrollTo与scrollBy
	*  5.2.5 Scroller
	*  5.2.6 属性动画
	*  5.2.7 ViewDragHelper

----------
##知识点回顾##

###**5.1 滑动效果是如何产生的**
>要实现View的滑动，就必须监听用户触摸的事件，并根据事件传入的坐标，动态且不断地改变View的坐标，从而实现View跟随用户触摸的滑动而滑动

>特别注意：View.getX()和event.getX()两个不同的位置获取的区别

###5.1.1 Android坐标系
> Android坐标系以屏幕左上角作为原点，可通过以下方法获得x和y的坐标：

>* getRawX()：获取Android坐标系的x坐标，请注意这里和event.getRawX()的区别
>* getRawY()：获取Android坐标系的y坐标，请注意这里和event.getRawY()的区别

![](http://img.blog.csdn.net/20161009103632909)

###5.1.2 视图坐标系
> 视图坐标系由子控件以父控件的左上角作为原点，可通过以下方法获得x和y的坐标：

>* getX()：获取视图坐标系的x坐标，请注意这里和event.getX()的区别
>* getY()：获取视图坐标系的y坐标，请注意这里和event.getY()的区别

![](http://img.blog.csdn.net/20161009103826286)

###5.1.3 触控事件——MotionEvent
> MotionEvent封装的一些常用的事件常量：

>* ACTION_DOWN：单点触摸按下的动作
>* ACTION_UP：单点触摸离开的动作
>* ACTION_MOVE：单点触摸移动的动作
>* ACTION_CANCEL：单点触摸取消
>* ACTION_OUTSIDE：单点触摸超出边界
>* ACTION_POINTER_DOWN：多点触摸按下的动作
>* ACTION_POINTER_UP：多点触摸离开的动作 

> 通常会在onTouchEvent(MotionEvent event)方法中通过event.getAction()方法获取触控事件类型，以下代码为模板式子：

```
@Override
public boolean onTouchEvent(MotionEvent event) {
    //获取当前输入点的X,Y坐标（视图坐标）
    int x = (int) event.getX();
    int y = (int) event.getY();
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
            //处理输入的按下动作
            break;
        case MotionEvent.ACTION_MOVE:
            //处理输入的移动动作
            break;
        case MotionEvent.ACTION_UP:
            //处理输入的离开动作
            break;
    }
    return true;
}
```
![](http://img.blog.csdn.net/20161009122424691)
> 系统海提供了非常多的方法来获取坐标值、相对距离等：

> View提供的获取坐标方法：
>
>* getTop()：获取到的是View自身的顶边到其父布局顶边的距离
>* getLeft()：获取到的是View自身的左边到其父布局左边的距离
>* getRight()：获取到的是View自身的右边到其父布局右边的距离
>* getBottom()：获取到的是View自身的底边到其父布局底边的距离

> MotionEvent提供的方法：
> 
>* getX()：获取点击事件距离控件左边的距离，即视图坐标
>* getY()：获取点击事件距离控件顶边的距离，即视图坐标
>* getRawX：获取点击事件距离整个屏幕左边的距离，即绝对坐标
>* getRawY：获取点击事件距离整个屏幕顶边的距离，即绝对坐标

###**5.2 实现滑动的七种方法**
>无知识点

##5.2.1 layout方法
>当我们手指触摸时记录下触摸点，在滑动时也记录下触摸点，通过两次的差值，计算出偏移量，使用View本身的layout方法，让他不断的位移偏移量即可

>Layout滑动......见经典代码回顾案例一



###5.2.2 offsetLeftAndRight()与offsetTopAndBottom()

>offsetLeftAndRight滑动......见经典代码回顾案例二



###5.2.3 LayoutParams
> 通过getLayoutParams()方法来获取布局参数，需要注意的是：

>* 不同的View应采用不同的View的getLayoutParams()方法
>* 前提必须要有一个父布局，否则系统无法获取LayoutParams
>* 使用ViewGroup.MarginLayoutParams不需要考虑父布局类型

>LayoutParams滑动......见经典代码回顾案例三

###5.2.4 scrollTo与scrollBy

>scrollTo与scrollBy滑动......见经典代码回顾案例四

>* scrollTo：移动到一个具体的坐标点（x，y）
>* scrollBy：表示移动的增量dx，dy

> scrollTo、scrollBy方法移动的是View的content，即让View的内容移动了，如果在ViewGroup中使用scrollTo、scrollBy方法，那么移动的将是所有子View，例如：

>* TextView，content就是它的文本
>* ImageView，content就是它的drawable对象

> 也就是说我们要用getParent()来使用scrollTo、scrollBy方法，否则，TextView只是偏移它的文本，ImageView只是偏移它的图片

###5.2.5 Scroller
> Scroller使用步骤：

>* 初始化Scroller
>* 重写computeScroll()方法
>* startScroll开启滑动

>Scroller滑动......见经典代码回顾案例五

###5.2.6 属性动画
> 无知识点

###5.2.7 ViewDragHelper
> ViewDragHelper使用步骤：

>* 初始化ViewDragHelper
>* 重写拦截事件，将事件传递给ViewDragHelper处理
>* 处理computeScroll()
>* 处理回调ViewDragHelper.Callback
>* 开启滑动，并更新UI

>在ViewDragHelper.Callback中，系统定义了大量的监听事件来帮助我们处理各种事件：

>* onViewCaptured() ：用户触摸到view回调
>* onViewDragStateChanged()：拖拽状态改变时回调，比如idle，dragging等状态
>* onViewPositionChanged()：位置改变时回调，常用于滑动时更改scale进行缩放等效果

>ViewDragHelper滑动......见经典代码回顾案例六


##经典代码回顾##


----------


###案例一：Layout滑动
```
public class DragView2 extends View {

    private int lastX;
    private int lastY;

    public DragView2(Context context) {
        super(context);
        ininView();
    }

    public DragView2(Context context, AttributeSet attrs) {
        super(context, attrs);
        ininView();
    }

    public DragView2(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        ininView();
    }

    private void ininView() {
        setBackgroundColor(Color.BLUE);
    }

    // 绝对坐标方式
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int rawX = (int) (event.getRawX());
        int rawY = (int) (event.getRawY());
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = rawX;
                lastY = rawY;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = rawX - lastX;
                int offsetY = rawY - lastY;
                // 在当前left、top、right、bottom的基础上加上偏移量
                layout(getLeft() + offsetX,
                        getTop() + offsetY,
                        getRight() + offsetX,
                        getBottom() + offsetY);
                // 重新设置初始坐标
                lastX = rawX;
                lastY = rawY;
                break;
        }
        return true;
    }
}
```
> 一定要重新设置初始坐标，这样才能准确地获取偏移量

###5.2.1-5.2.4 效果图
![](http://img.blog.csdn.net/20161009140243720)

----------


###案例二：offsetLeftAndRight滑动

```
public class DragView1 extends View {

    private int lastX;
    private int lastY;

    public DragView1(Context context) {
        super(context);
        ininView();
    }

    public DragView1(Context context, AttributeSet attrs) {
        super(context, attrs);
        ininView();
    }

    public DragView1(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        ininView();
    }

    private void ininView() {
        // 给View设置背景颜色，便于观察
        setBackgroundColor(Color.BLUE);
    }

    // 视图坐标方式
    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = x;
                lastY = y;
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                // 在当前left、top、right、bottom的基础上加上偏移量
                layout(getLeft() + offsetX,
                        getTop() + offsetY,
                        getRight() + offsetX,
                        getBottom() + offsetY);
                        offsetLeftAndRight(offsetX);
                        offsetTopAndBottom(offsetY);
                break;
        }
        return true;
    }
}
```


----------


###案例三：LayoutParams滑动

```
public class DragView3 extends View {

    private int lastX;
    private int lastY;

    public DragView3(Context context) {
        super(context);
        ininView();
    }

    public DragView3(Context context, AttributeSet attrs) {
        super(context, attrs);
        ininView();
    }

    public DragView3(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        ininView();
    }

    private void ininView() {
        setBackgroundColor(Color.BLUE);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                // 记录触摸点坐标
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                // 计算偏移量
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ViewGroup.MarginLayoutParams layoutParams = (ViewGroup.MarginLayoutParams) getLayoutParams();
//                LinearLayout.LayoutParams layoutParams = (LinearLayout.LayoutParams) getLayoutParams();
                layoutParams.leftMargin = getLeft() + offsetX;
                layoutParams.topMargin = getTop() + offsetY;
                setLayoutParams(layoutParams);
                break;
        }
        return true;
    }
}
```


----------


###案例四：scrollTo与scrollBy滑动

```
public class DragView4 extends View {

    private int lastX;
    private int lastY;

    public DragView4(Context context) {
        super(context);
        ininView();
    }

    public DragView4(Context context, AttributeSet attrs) {
        super(context, attrs);
        ininView();
    }

    public DragView4(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        ininView();
    }

    private void ininView() {
        setBackgroundColor(Color.BLUE);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ((View) getParent()).scrollBy(-offsetX, -offsetY);
                break;
        }
        return true;
    }
}
```
>要注意的是，scrollTo、scrollBy是以Android坐标系移动的，所以要使用我们正常的坐标系，则在x和y加一个负号


----------


###案例五：Scroller滑动

```
public class DragView5 extends View {

    private int lastX;
    private int lastY;
    private Scroller mScroller;

    public DragView5(Context context) {
        super(context);
        ininView(context);
    }

    public DragView5(Context context, AttributeSet attrs) {
        super(context, attrs);
        ininView(context);
    }

    public DragView5(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        ininView(context);
    }

    private void ininView(Context context) {
        setBackgroundColor(Color.BLUE);
        // 初始化Scroller
        mScroller = new Scroller(context);
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        // 判断Scroller是否执行完毕
        if (mScroller.computeScrollOffset()) {
            ((View) getParent()).scrollTo(
                    mScroller.getCurrX(),
                    mScroller.getCurrY());
            // 通过重绘来不断调用computeScroll
            invalidate();
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                lastX = (int) event.getX();
                lastY = (int) event.getY();
                break;
            case MotionEvent.ACTION_MOVE:
                int offsetX = x - lastX;
                int offsetY = y - lastY;
                ((View) getParent()).scrollBy(-offsetX, -offsetY);
                break;
            case MotionEvent.ACTION_UP:
                // 手指离开时，执行滑动过程
                View viewGroup = ((View) getParent());
                mScroller.startScroll(
                        viewGroup.getScrollX(),
                        viewGroup.getScrollY(),
                        -viewGroup.getScrollX(),
                        -viewGroup.getScrollY());
                invalidate();
                break;
        }
        return true;
    }
}
```
###效果图
![](http://img.blog.csdn.net/20161009140148251)


----------


###案例六：ViewDragHelper滑动

```
public class DragViewGroup extends FrameLayout {

    private ViewDragHelper mViewDragHelper;
    private View mMenuView, mMainView;
    private int mWidth;

    public DragViewGroup(Context context) {
        super(context);
        initView();
    }

    public DragViewGroup(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public DragViewGroup(Context context,
                         AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView();
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();
        mMenuView = getChildAt(0);
        mMainView = getChildAt(1);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = mMenuView.getMeasuredWidth();
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        return mViewDragHelper.shouldInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        //将触摸事件传递给ViewDragHelper,此操作必不可少
        mViewDragHelper.processTouchEvent(event);
        return true;
    }

    private void initView() {
        mViewDragHelper = ViewDragHelper.create(this, callback);
    }

    private ViewDragHelper.Callback callback =
            new ViewDragHelper.Callback() {

                // 何时开始检测触摸事件
                @Override
                public boolean tryCaptureView(View child, int pointerId) {
                    //如果当前触摸的child是mMainView时开始检测
                    return mMainView == child;
                }

                // 触摸到View后回调
                @Override
                public void onViewCaptured(View capturedChild,
                                           int activePointerId) {
                    super.onViewCaptured(capturedChild, activePointerId);
                }

                // 当拖拽状态改变，比如idle，dragging
                @Override
                public void onViewDragStateChanged(int state) {
                    super.onViewDragStateChanged(state);
                }

                // 当位置改变的时候调用,常用与滑动时更改scale等
                @Override
                public void onViewPositionChanged(View changedView,
                                                  int left, int top, int dx, int dy) {
                    super.onViewPositionChanged(changedView, left, top, dx, dy);
                }

                // 处理垂直滑动
                @Override
                public int clampViewPositionVertical(View child, int top, int dy) {
                    return 0;
                }

                // 处理水平滑动
                @Override
                public int clampViewPositionHorizontal(View child, int left, int dx) {
                    return left;
                }

                // 拖动结束后调用
                @Override
                public void onViewReleased(View releasedChild, float xvel, float yvel) {
                    super.onViewReleased(releasedChild, xvel, yvel);
                    //手指抬起后缓慢移动到指定位置
                    if (mMainView.getLeft() < 500) {
                        //关闭菜单
                        //相当于Scroller的startScroll方法
                        mViewDragHelper.smoothSlideViewTo(mMainView, 0, 0);
                        ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
                    } else {
                        //打开菜单
                        mViewDragHelper.smoothSlideViewTo(mMainView, 300, 0);
                        ViewCompat.postInvalidateOnAnimation(DragViewGroup.this);
                    }
                }
            };

    @Override
    public void computeScroll() {
        if (mViewDragHelper.continueSettling(true)) {
            ViewCompat.postInvalidateOnAnimation(this);
        }
    }
}
```
>布局文件必须放置两个子View

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.handsome.qunyingzhuang.chapter_5.DragViewGroup
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/holo_blue_light">

            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Menu" />
        </FrameLayout>

        <FrameLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/holo_orange_dark">

            <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Main" />
        </FrameLayout>
    </com.handsome.qunyingzhuang.chapter_5.DragViewGroup>
</RelativeLayout>

```

###效果图
![](http://img.blog.csdn.net/20161009141758337)

>经典回顾源码下载

>github：https://github.com/CSDNHensen/QunYingZhuang