# Android群英传知识点回顾——第三章：Android控件架构与自定义控件详解 #
----------
##知识点目录##
>* **3.1 Android控件架构**
>* **3.2 View的测量**
>* **3.3 View的绘制**
>* **3.4 ViewGroup的测量**
>* **3.5 ViewGroup的绘制**
>* **3.6 自定义View**
	* 3.6.1 对现有的空间进行拓展
	* 3.6.2 创建复合控件
	* 3.6.3 重写View来实现全新的空间
>* **3.7 自定义ViewGroup**
>* **3.8 事件拦截机制分析**

----------
##知识点回顾##

###**3.1 Android控件架构**
> 控件大致非为两类：

>* view控件：视图控件
>* viewGroup控件：包含多个View控件，并管理其包含的View控件
>* 两者之间的关系：上层控件负责下层子控件的测量与绘制，并传递交互事件

> UI界面架构：

>* Activity都包含一个Window对象，通常由PhoneWindow来实现
>* PhoneWindow将一个DecorView设置为整个应用窗口的根View
	* DecorView为整个Window界面的最顶层View
	* DecorView只有一个子元素为LinearLayout，代表整个Window界面，包含通知栏，标题栏，内容显示栏三块区域
	* LinearLayout里有两个FrameLayout子元素：
		* 标题栏显示界面。只有一个TextView显示应用的名称
		* 内容栏显示界面。就是setContentView()方法载入的布局界面
###**3.2 View的测量**
> MeasureSpec类：32位的int值，高2位为测量模式，低30位为测量大小

> MeasureSpec模式：

>* EXACTLY：精确模式 ，当控件的layout_width属性或layout_height属性指定为具体值，控件大小也是该具体值
>* AT_MOST：最大值模式，当控件layout_width属性或layout_height属性指定为warp_content时，控件的尺寸不要超过父控件允许的最大尺寸
>* UNSPECIFIED：未指定模式，控件要多大就多大，通常情况下再绘制自定义View中才会使用

>View类默认的onMeasure()方法只支持EXACTLY模式，View需要支持warp_content属性，那么就必须重写onMeasure()方法，来制定warp_content的大小

>下面我们通过一个简单的实例，演示如何进行View的测量，首先，需要重写onMeasure()方法：

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}
```
>可以发现，onMeasure方法调用了父类的onMeasure方法，代码跟踪父类onMeasure方法

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```
>可以发现，系统最终会调用setMeasuredDimension(int measuredWidth，int measuredHeight)方法将测量后的宽高值设置进去，我们调用自定义的measureWidth()方法和measureHeight()方法，分别对宽高进行重新定义

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(measureWidth(widthMeasureSpec),measureHeight(heightMeasureSpec));
}
```
> 下面以measureWidth()方法为例：

>  第一步：从MeasureSpec对象中提取出具体的测量模式和大小

```
int specMode = MeasureSpec.getMode(measureSpec);
int specSize = MeasureSpec.getSize(measureSpec);
```
>第二步：通过不同的测量模式给出不同的测量值：

>* EXACTLY：使用指定的specSize即可
>* AT_MOST：取出我们指定的大小和SpecSize的最小值
>* UNSPECIFIED：200px

>下面这段代码基本上可以作为模板代码：

```
private int measureWidth(int measureSpec) {
   int result = 0;
   int specMode = MeasureSpec.getMode(measureSpec);
   int specSize = MeasureSpec.getSize(measureSpec);

   if (specMode == MeasureSpec.EXACTLY) {
       result = specSize;

   } else {
       result = 200;
       if (specMode == MeasureSpec.AT_MOST) {
           result = Math.min(result, specSize);
       }
   }
   return result;
}
```
>可以发现，当指定warp_content属性时，View就获得一个默认值200px

###**3.3 View的绘制**
>当测量好了一个View之后，我们通过重写View类中的onDraw()方法来绘图，要想绘制相应的图像，就必须在Canvas上进行绘制

```
Canvas canvas = new Canvas(Bitmap);
```
>Canvas就像是一个画板，我们传进去一个bitmap，通过这个bitmap创建的Canvas画布紧紧联系在一起，这个过程我们称之为装载画布，这个bitmap用来存储所有绘制在Canvas上的像素信息，所以当你在后面调用所有的Canvas.drawxxx方法都会发生在这个bitmap上
###**3.4 ViewGroup的测量**
>ViewGroup在测量时通过遍历所有子View，从而调用子View的Measure方法来获得每一个子View的结果

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int childCount = getChildCount();
    for (int i = 0; i < childCount; i++) {
        getChildAt(i).measure(widthMeasureSpec, heightMeasureSpec);
    }
 }
```
>ViewGroup测量完毕后，通常会去重写onLayout()方法来控制其子View显示位置的逻辑

```
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    for (int i = 0; i < getChildCount(); i++) {
        this.getChildAt(i).layout(l, t, r, b);
    }
}
```
###**3.5 ViewGroup的绘制**
>ViewGroup通常不需要绘制，如果不是指定ViewGroup的背景颜色，那么ViewGroup的onDraw()方法都不会被调用，但是，ViewGroup会使用dispatchDraw()方法来绘制子View

```
@Override
protected void dispatchDraw(Canvas canvas) {
    super.dispatchDraw(canvas);
}
```
###**3.6 自定义View**
>在View中通常有以下一些重要的回调方法：

>* onFinishInflate()：从XML加载组件后回调
>* onSizeChanged()：组件大小改变时回调
>* onMeasure()：回调该方法来进行测量
>* onLayout()：回调该方法来确定显示的位置
>* onTouchEvent()：监听到触摸事件时回调

>通常情况下，有以下三种方法来实现自定义的控件：

>* 对现有的控件进行拓展
>* 通过组合来实现新的控件
>* 重写View来实现全新的控件

**3.6.1 对现有的控件进行拓展**
>* 自定义修改TextView......见经典代码回顾，案例一
>* 闪动的文字效果......见经典代码回顾，案例二

**3.6.2 创建复合控件**
>* 自定义ToolBar的实现......见经典代码回顾，案例三

**3.6.3 重写View来实现全新的控件**
>* 弧线展示图......见经典代码回顾，案例四
>* 音频条形图......见经典代码回顾，案例五

###**3.7 自定义ViewGroup**
>* 自定义ViewGroup，仿ScrollView......见经典代码回顾，案例六

###**3.8 事件拦截机制分析**
 >事件拦截机制三个重要方法
 
 >* dispatchTouchEvent()：分发事件
 >* onInterceptTouchEvent()：拦截事件
 >* onTouchEvent()：处理事件

>举一个例子说明事件分发机制：

>* ViewGroupA：处于视图最下层
>* ViewGroupB：处于视图中间层
>* View：处于视图最上层

>正常的事件分发机制流程：

>* ViewGroupA dispatchTouchEvent
>* ViewGroupA onInterceptTouchEvent
>* ViewGroupB dispatchTouchEvent
>* ViewGroupB onInterceptTouchEvent
>* View dispatchTouchEvent
>* View onTouchEvent
>* ViewGroupB onTouchEvent
>* ViewGroupA onTouchEvent

>若ViewGroupB的onInterceptTouchEvent()方法返回true的分发机制流程：

>* ViewGroupA dispatchTouchEvent
>* ViewGroupA onInterceptTouchEvent
>* ViewGroupB dispatchTouchEvent
>* ViewGroupB onInterceptTouchEvent
>* ViewGroupB onTouchEvent
>* ViewGroupA onTouchEvent

>若View的onTouchEvent()方法返回true的分发机制流程：

>* ViewGroupA dispatchTouchEvent
>* ViewGroupA onInterceptTouchEvent
>* ViewGroupB dispatchTouchEvent
>* ViewGroupB onInterceptTouchEvent
>* View dispatchTouchEvent
>* View onTouchEvent

>若ViewGroupB的onTouchEvent()方法返回true的分发机制流程：

>* ViewGroupA dispatchTouchEvent
>* ViewGroupA onInterceptTouchEvent
>* ViewGroupB dispatchTouchEvent
>* ViewGroupB onInterceptTouchEvent
>* View dispatchTouchEvent
>* View onTouchEvent
>* ViewGroupB onTouchEvent

>简单的说dispatchTouchEvent()和onInterceptTouchEvent()是从下往上一层一层分发下去的，而onTouchEvent()是从上往下一层一层分发下去的

----------


##经典代码回顾##

###**案例一：自定义修改TextView**

```
public class CustomTextView extends TextView {

    private Paint paint1, paint2;

    public CustomTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initPaint();
    }

    /**
     * 初始化画笔
     */
    private void initPaint() {
        paint1 = new Paint();
        paint1.setColor(getResources().getColor(android.R.color.holo_blue_light));
        paint1.setStyle(Paint.Style.FILL);
        paint2 = new Paint();
        paint2.setColor(Color.YELLOW);
        paint2.setStyle(Paint.Style.FILL);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        //绘制外层矩形
        canvas.drawRect(0, 0, getMeasuredWidth(), getMeasuredHeight(), paint1);
        //绘制内层矩形
        canvas.drawRect(10, 10, getMeasuredWidth() - 10, getMeasuredHeight() - 10, paint2);
        canvas.save();
        //绘制文字前平移10像素
        canvas.translate(10, 0);
        //父类完成的方法，即绘制文本
        super.onDraw(canvas);
        canvas.restore();
    }
}
```
###**效果图**
![](http://img.blog.csdn.net/20161008143806162)


----------


###**案例二：闪动的文字效果**

```
public class FlashTextView extends TextView {

    int mViewWidth = 0;
    private Paint mPaint;
    private LinearGradient mLinearGradient;
    private Matrix matrix;
    private int mTranslate;

    public FlashTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        if (mViewWidth == 0) {
            mViewWidth = getMeasuredWidth();
            if (mViewWidth > 0) {
                mPaint = getPaint();
                mLinearGradient = new LinearGradient(0, 0, mViewWidth, 0, new int[]{Color.BLUE, 0xffffffff, Color.BLUE},
                        null, Shader.TileMode.CLAMP);
                mPaint.setShader(mLinearGradient);
                matrix = new Matrix();
            }
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        if (matrix != null) {
            mTranslate += mViewWidth + 5;
            if (mTranslate > 2 * mViewWidth / 5) {
                mTranslate = -mViewWidth;
            }
            matrix.setTranslate(mTranslate, 0);
            mLinearGradient.setLocalMatrix(matrix);
            postInvalidateDelayed(100);
        }
    }
}
```
###**效果图**
![](http://img.blog.csdn.net/20161008143855803)


----------


###**案例三：自定义ToolBar的实现**
> 在values文件夹中创建一个attrs.xml文件来自定义属性

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="TopBar">
        <attr name="title" format="string" />
        <attr name="titleTextSize" format="dimension" />
        <attr name="titleTextColor" format="color" />
        <attr name="leftTextColor" format="color" />
        <attr name="leftBackground" format="reference|color" />
        <attr name="leftText" format="string" />
        <attr name="rightTextColor" format="color" />
        <attr name="rightBackground" format="reference|color" />
        <attr name="rightText" format="string" />
    </declare-styleable>
</resources>
```
>开始创建我们的ToolBar

```
public class ToolBar extends RelativeLayout {

    private int mLeftTextColor;
    private Drawable mLeftBackground;
    private String mLeftText;

    private int mRightTextColor;
    private Drawable mRightBackgroup;
    private String mRightText;

    private float mTitleTextSize;
    private int mTitleTextColor;
    private String mTitle;

    private Button mLeftButton;
    private Button mRightButton;
    private TextView mTitleView;

    private RelativeLayout.LayoutParams mLeftParams;
    private RelativeLayout.LayoutParams mRightParams;
    private RelativeLayout.LayoutParams mTitleParams;

    //带参构造方法
    public ToolBar(Context context, AttributeSet attrs) {
        super(context, attrs);

        //通过这个方法，将你在attrs.xml文件中定义的declare-styleable
        //的所有属性的值存储到TypedArray中
        TypedArray ta = context.obtainStyledAttributes(attrs, R.styleable.TopBar);
        //从TypedArray中取出对应的值来设置的属性赋值
        mLeftTextColor = ta.getColor(R.styleable.TopBar_leftTextColor, 0);
        mLeftBackground = ta.getDrawable(R.styleable.TopBar_leftBackground);
        mLeftText = ta.getString(R.styleable.TopBar_leftText);

        mRightTextColor = ta.getColor(R.styleable.TopBar_rightTextColor, 0);
        mRightBackgroup = ta.getDrawable(R.styleable.TopBar_rightBackground);
        mRightText = ta.getString(R.styleable.TopBar_rightText);

        mTitleTextSize = ta.getDimension(R.styleable.TopBar_titleTextSize, 10);
        mTitleTextColor = ta.getColor(R.styleable.TopBar_titleTextColor, 0);
        mTitle = ta.getString(R.styleable.TopBar_title);

        //获取完TypedArray的值之后，一般要调用recycle方法来避免重复创建时候的错误
        ta.recycle();

        mLeftButton = new Button(context);
        mRightButton = new Button(context);
        mTitleView = new TextView(context);

        //为创建的元素赋值
        mLeftButton.setTextColor(mLeftTextColor);
        mLeftButton.setBackground(mLeftBackground);
        mLeftButton.setText(mLeftText);

        mRightButton.setTextColor(mRightTextColor);
        mRightButton.setBackground(mRightBackgroup);
        mRightButton.setText(mRightText);

        mTitleView.setText(mTitle);
        mTitleView.setTextColor(mTitleTextColor);
        mTitleView.setTextSize(mTitleTextSize);
        mTitleView.setGravity(Gravity.CENTER);

        //为组件元素设置相应的布局元素
        mLeftParams = new RelativeLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
        mLeftParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT);
        addView(mLeftButton, mLeftParams);
        mRightParams = new RelativeLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
        mRightParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT);
        addView(mRightButton, mRightParams);
        mTitleParams = new RelativeLayout.LayoutParams(LayoutParams.WRAP_CONTENT, LayoutParams.MATCH_PARENT);
        mTitleParams.addRule(RelativeLayout.CENTER_IN_PARENT);
        addView(mTitleView, mTitleParams);
    }

}
```
>在布局文件中使用

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.handsome.qunyingzhuang.chapter_3.ToolBar.ToolBar xmlns:custom="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        custom:leftBackground="#0000ff"
        custom:leftText="back"
        custom:leftTextColor="#00ff00"
        custom:rightBackground="#333333"
        custom:rightText="more"
        custom:rightTextColor="#ff0000"
        custom:title="我的标题"
        custom:titleTextColor="#ff0000"
        custom:titleTextSize="14sp" />
</RelativeLayout>
```
###**效果图**
![](http://img.blog.csdn.net/20161008144245382)


----------


###**案例四：弧线展示图**

```
public class CircleProgressView extends View {

    private int mCircleXY;
    private int length;
    private float mRadius;

    private Paint mCirclePaint;
    private Paint mArcPaint;
    private Paint mTextPaint;
    private String mShowText = "Hensen_";

    private int mTextSize = 25;
    private float mSweepValue = 270;

    public CircleProgressView(Context context, AttributeSet attrs) {
        super(context, attrs);
        //获取屏幕高宽
        WindowManager wm = (WindowManager) getContext()
                .getSystemService(Context.WINDOW_SERVICE);
        length = wm.getDefaultDisplay().getWidth();
        init();
    }

    private void init() {
        mCircleXY = length / 2;
        mRadius = (float) (length * 0.5 / 2);

        mCirclePaint = new Paint();
        mCirclePaint.setColor(Color.BLUE);

        mArcPaint = new Paint();
        mArcPaint.setStrokeWidth(50);
        mArcPaint.setStyle(Paint.Style.STROKE);
        mArcPaint.setColor(Color.BLUE);

        mTextPaint = new Paint();
        mTextPaint.setColor(Color.WHITE);
        mTextPaint.setTextSize(mTextSize);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //矩形
        RectF mArcRectF = new RectF((float) (length * 0.1), (float) (length * 0.1), (float) (length * 0.9), (float) (length * 0.9));
        //绘制圆
        canvas.drawCircle(mCircleXY, mCircleXY, mRadius, mCirclePaint);
        //绘制弧线
        canvas.drawArc(mArcRectF, 270, mSweepValue, false, mArcPaint);
        //绘制文字
        canvas.drawText(mShowText, 0, mShowText.length(), mCircleXY, mCircleXY + (mTextSize / 4), mTextPaint);
    }

    public void setSweepValue(float sweepValue) {
        if (sweepValue != 0) {
            mSweepValue = sweepValue;
        } else {
            mSweepValue = 25;
        }
        invalidate();
    }
}
```
>当用户不指定具体的比例值时，可以调用以下代码来设置相应的比例值

```
CircleProgressView circleProgressView = (CircleProgressView) findViewById(R.id.circle);
circleProgressView.setSweepValue(270);
```
###**效果图**
![](http://img.blog.csdn.net/20161008144443289)


----------


###**案例五：音频条形图**

```
public class MusicView extends View {

    private int mWidth;
    private int mRectHeight;
    private int mRectWidth;
    private int mRectCount = 20;
    private LinearGradient mLinearGradient;
    private Paint mPaint=new Paint();

    private float currentHeight;
    private int offset = 5;
    private double mRandom;

    public MusicView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mWidth = getWidth();
        mRectHeight = getHeight();
        mRectWidth = (int) (mWidth * 0.6 / mRectCount);
        mLinearGradient = new LinearGradient(0, 0, mRectWidth, mRectHeight, Color.YELLOW, Color.BLUE, Shader.TileMode.CLAMP);
        mPaint.setShader(mLinearGradient);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        //遍历绘制矩形，留中间间隔
        for (int i = 0; i < mRectCount; i++) {
            //开始绘制
            canvas.drawRect((float) (mWidth * 0.4 / 2 + mRectWidth * i + offset),
                    currentHeight, (float) (mWidth * 0.4 / 2 + mRectWidth * (i + 1)), mRectHeight, mPaint);
        }
        //获取随机数
        mRandom = Math.random();
        currentHeight = ((float) (mRectHeight * mRandom));
        //延迟300去刷新
        postInvalidateDelayed(300);
    }

}
```
###**效果图**
![](http://img.blog.csdn.net/20161008144532755)


----------


###**案例六：自定义ViewGroup，仿ScrollView**
>自定义的ScrollView没有系统自带的性能好，毕竟很多因素都没考虑到，这里只是适用于练手使用

```
public class CustomScrollView extends ViewGroup {

    private int mScreenHeight;
    private Scroller mScroller;
    private int mLastY;
    private int mStart;
    private int mEnd;

    public CustomScrollView(Context context, AttributeSet attrs) {
        super(context, attrs);
        //获取屏幕高宽
        WindowManager wm = (WindowManager) getContext()
                .getSystemService(Context.WINDOW_SERVICE);
        mScreenHeight = wm.getDefaultDisplay().getHeight();
        mScroller = new Scroller(getContext());
    }

    @Override
    protected void onLayout(boolean b, int i, int i1, int i2, int i3) {
        int childCount = getChildCount();
        MarginLayoutParams mlp = (MarginLayoutParams) getLayoutParams();
        mlp.height = mScreenHeight * childCount;
        setLayoutParams(mlp);

        for (int j = 0; j < childCount; j++) {
            View child = getChildAt(j);
            if (child.getVisibility() != View.GONE) {
                child.layout(i, j * mScreenHeight, i2, (j + 1) * mScreenHeight);
            }
        }
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int count = getChildCount();
        for (int i = 0; i < count; ++i) {
            View childView = getChildAt(i);
            measureChild(childView, widthMeasureSpec, heightMeasureSpec);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mLastY = y;
                mStart = getScrollY();
                break;
            case MotionEvent.ACTION_MOVE:
                if (!mScroller.isFinished()) {
                    mScroller.abortAnimation();
                }
                int dy = mLastY - y;
                if (getScrollY() < 0) {
                    dy = 0;
                }
                if (getScrollY() > getHeight() - mScreenHeight) {
                    dy = 0;
                }
                scrollBy(0, dy);
                mLastY = y;
                break;
            case MotionEvent.ACTION_UP:
                mEnd = getScrollY();
                int dScrollY = mEnd - mStart;
                if (dScrollY > 0) {
                    if (dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(0, getScrollY(), 0, -dScrollY);
                    } else {
                        mScroller.startScroll(0, getScrollY(), 0, mScreenHeight - dScrollY);
                    }
                } else {
                    if (-dScrollY < mScreenHeight / 3) {
                        mScroller.startScroll(0, getScrollY(), 0, -dScrollY);
                    } else {
                        mScroller.startScroll(0, getScrollY(), 0, -mScreenHeight - dScrollY);
                    }
                }
                break;
        }
        postInvalidate();
        return true;
    }

    @Override
    public void computeScroll() {
        super.computeScroll();
        if (mScroller.computeScrollOffset()) {
            scrollTo(0, mScroller.getCurrY());
        }
    }
}
```
>在布局中使用

```
<?xml version="1.0" encoding="utf-8"?>
<com.handsome.qunyingzhuang.chapter_3.CustomScrollView.CustomScrollView 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#ff0000"
        android:text="Hensen_博客(1)" />

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="#00ff00"
        android:text="Hensen_博客(2)" />

</com.handsome.qunyingzhuang.chapter_3.CustomScrollView.CustomScrollView>
```
###**效果图**
![](http://img.blog.csdn.net/20161008144749852)

>经典回顾源码下载

>github：https://github.com/CSDNHensen/QunYingZhuang
