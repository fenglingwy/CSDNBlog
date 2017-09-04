#Android自定义View——贝塞尔曲线实现水波纹效果


----------
>本篇文章包含以下内容：

>* 简单介绍
>* 原理分析
>* 实现步骤


##简单介绍


----------

###效果图展示

![](http://img.blog.csdn.net/20161109122450416)

###贝塞尔曲线

该View涉及到Path类的运用，如果对Path类不熟悉的，可以看http://blog.sina.com.cn/s/blog_4d9c3fec0102vyhs.html，这篇文章通熟易懂

我们使用到的是Path类的quadTo(x1, y1, x2, y2)方法，属于二阶贝塞尔曲线，使用一张图来展示二阶贝塞尔曲线，这里的（x1,y1）是控制点，（x2,y2）是终止点，起始点默认是Path的起始点（0,0）

![](http://img.blog.csdn.net/20161109123021372)

##原理分析


----------
一、水波纹实现原理

1. 通过for循环画出两个波纹，需要波纹的-mWL点、-3/4*mWL点、-1/2*mWL、-1/4*mWL四个点，通过path的quadTo画出

2. 接着通过ValueAnimator对offset递增，实现平移效果，并无限重复

![](http://img.blog.csdn.net/20161109125401891)

实现一次循环波纹：

![](http://img.blog.csdn.net/20161109125938867)

实现无限次循环波纹：

![](http://img.blog.csdn.net/20161109130017273)

接下来在波纹下方的空白处画上一个矩形：

![](http://img.blog.csdn.net/20161109130313495)

>去掉红色点就是最终效果

##实现步骤


----------

##一、分析变量信息

```
//波浪画笔
private Paint mPaint;
//测试红点画笔
private Paint mCyclePaint;

//波浪Path类
private Path mPath;
//一个波浪长度
private int mWaveLength = 1000;
//波纹个数
private int mWaveCount;
//平移偏移量
private int mOffset;
//波纹的中间轴
private int mCenterY;

//屏幕高度
private int mScreenHeight;
//屏幕宽度
private int mScreenWidth;
```
##二、初始化画笔

```
public WaveView(Context context, AttributeSet attrs) {
    super(context, attrs);
    mPath = new Path();
    mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    mPaint.setColor(Color.LTGRAY);
    mPaint.setStyle(Paint.Style.FILL_AND_STROKE);
    setOnClickListener(this);

    mCyclePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
    mCyclePaint.setColor(Color.RED);
    mCyclePaint.setStyle(Paint.Style.FILL_AND_STROKE);
}
```

##三、获取View的宽和高

```
@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    mScreenHeight = h;
    mScreenWidth = w;
    //加1.5：至少保证波纹有2个，至少2个才能实现平移效果
    mWaveCount = (int) Math.round(mScreenWidth / mWaveLength + 1.5);
    mCenterY = mScreenHeight / 2;
}
```

##四、实现onDraw方法，进行我们的绘制

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    mPath.reset();
    //移到屏幕外最左边
    mPath.moveTo(-mWaveLength + mOffset, mCenterY);
    for (int i = 0; i < mWaveCount; i++) {
        //正弦曲线
        mPath.quadTo((-mWaveLength * 3 / 4) + (i * mWaveLength) + mOffset, mCenterY + 60, (-mWaveLength / 2) + (i * mWaveLength) + mOffset, mCenterY);
        mPath.quadTo((-mWaveLength / 4) + (i * mWaveLength) + mOffset, mCenterY - 60, i * mWaveLength + mOffset, mCenterY);
        //测试红点
        canvas.drawCircle((-mWaveLength * 3 / 4) + (i * mWaveLength) + mOffset, mCenterY + 60, 5, mCyclePaint);
        canvas.drawCircle((-mWaveLength / 2) + (i * mWaveLength) + mOffset, mCenterY, 5, mCyclePaint);
        canvas.drawCircle((-mWaveLength / 4) + (i * mWaveLength) + mOffset, mCenterY - 60, 5, mCyclePaint);
        canvas.drawCircle(i * mWaveLength + mOffset, mCenterY, 5, mCyclePaint);
    }
    //填充矩形
    mPath.lineTo(mScreenWidth, mScreenHeight);
    mPath.lineTo(0, mScreenHeight);
    mPath.close();
    canvas.drawPath(mPath, mPaint);
}
```

##五、点击事件，实现平移效果

```
@Override
public void onClick(View view) {
    ValueAnimator animator = ValueAnimator.ofInt(0, mWaveLength);
    animator.setDuration(1000);
    animator.setRepeatCount(ValueAnimator.INFINITE);
    animator.setInterpolator(new LinearInterpolator());
    animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        @Override
        public void onAnimationUpdate(ValueAnimator animation) {
            mOffset = (int) animation.getAnimatedValue();
            postInvalidate();
        }
    });
    animator.start();
}
```

整个类源码：[源码下载](http://download.csdn.net/detail/qq_30379689/9677445)

```
public class WaveView extends View implements View.OnClickListener {

    //波浪画笔
    private Paint mPaint;
    //测试红点画笔
    private Paint mCyclePaint;

    //波浪Path类
    private Path mPath;
    //一个波浪长度
    private int mWaveLength = 1000;
    //波纹个数
    private int mWaveCount;
    //平移偏移量
    private int mOffset;
    //波纹的中间轴
    private int mCenterY;

    //屏幕高度
    private int mScreenHeight;
    //屏幕宽度
    private int mScreenWidth;

    public WaveView(Context context) {
        super(context);
    }

    public WaveView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    public WaveView(Context context, AttributeSet attrs) {
        super(context, attrs);
        mPath = new Path();
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.LTGRAY);
        mPaint.setStyle(Paint.Style.FILL_AND_STROKE);
        setOnClickListener(this);

//        mCyclePaint = new Paint(Paint.ANTI_ALIAS_FLAG);
//        mCyclePaint.setColor(Color.RED);
//        mCyclePaint.setStyle(Paint.Style.FILL_AND_STROKE);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mScreenHeight = h;
        mScreenWidth = w;
        //加1.5：至少保证波纹有2个，至少2个才能实现平移效果
        mWaveCount = (int) Math.round(mScreenWidth / mWaveLength + 1.5);
        mCenterY = mScreenHeight / 2;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        mPath.reset();
        //移到屏幕外最左边
        mPath.moveTo(-mWaveLength + mOffset, mCenterY);
        for (int i = 0; i < mWaveCount; i++) {
            //正弦曲线
            mPath.quadTo((-mWaveLength * 3 / 4) + (i * mWaveLength) + mOffset, mCenterY + 60, (-mWaveLength / 2) + (i * mWaveLength) + mOffset, mCenterY);
            mPath.quadTo((-mWaveLength / 4) + (i * mWaveLength) + mOffset, mCenterY - 60, i * mWaveLength + mOffset, mCenterY);
            //测试红点
//            canvas.drawCircle((-mWaveLength * 3 / 4) + (i * mWaveLength) + mOffset, mCenterY + 60, 5, mCyclePaint);
//            canvas.drawCircle((-mWaveLength / 2) + (i * mWaveLength) + mOffset, mCenterY, 5, mCyclePaint);
//            canvas.drawCircle((-mWaveLength / 4) + (i * mWaveLength) + mOffset, mCenterY - 60, 5, mCyclePaint);
//            canvas.drawCircle(i * mWaveLength + mOffset, mCenterY, 5, mCyclePaint);
        }
        //填充矩形
        mPath.lineTo(mScreenWidth, mScreenHeight);
        mPath.lineTo(0, mScreenHeight);
        mPath.close();
        canvas.drawPath(mPath, mPaint);
    }

    @Override
    public void onClick(View view) {
        ValueAnimator animator = ValueAnimator.ofInt(0, mWaveLength);
        animator.setDuration(1000);
        animator.setRepeatCount(ValueAnimator.INFINITE);
        animator.setInterpolator(new LinearInterpolator());
        animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                mOffset = (int) animation.getAnimatedValue();
                postInvalidate();
            }
        });
        animator.start();
    }
}
```