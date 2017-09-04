##刮刮卡效果##

----------

>效果图：

![](http://img.blog.csdn.net/20161011144443221)

>想要红包的实现效果的可以关注我的博客，[仿饿了么红包](http://blog.csdn.net/qq_30379689/article/details/52356145)

----------

实现原理：

>* 下层图片：我们的红包的图片
>* 上层图片：有两部分
	* 一部分是灰色背景
	* 一部分是拥有透明度为0，并且模式为交集的画笔
>* 使用滑动监听，滑动时，用透明度为0的画笔画出透明和上层图片灰色的交集，交集即是透明的背景
>* 监听我们滑动的距离是否达到标准，达到标准后，使用子线程1s后自动展开剩余的灰色

实现步骤：

>* 初始化画笔
>* 初始化顶层图片和底层图片
>* 监听滑动事件，判断是否滑完
>* 使用onDraw()画出我们的上层图片和底层图片


----------

##步骤一##

初始化画笔：

```
mPaint = new Paint();
mPaint.setAlpha(0);
mPaint.setStyle(Paint.Style.STROKE);
mPaint.setStrokeWidth(50);
```
画笔设置XferMode，DST_IN模式是交集模式，由于图片底层是灰色，上层是我们drawPath出来的透明度为0的路径，所以取交集，即透明的路径，达到涂刮的效果：

```
mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
```

这两个属性可使画笔四个角变为圆角：

```
mPaint.setStrokeJoin(Paint.Join.ROUND);
mPaint.setStrokeCap(Paint.Cap.ROUND);
```


----------

##步骤二##

初始化底层图片和顶层图片：

```
//初始化路径
mPath = new Path();
//初始化底层图片
mBgBitmap = BitmapFactory.decodeResource(getResources(),
        R.drawable.test);
//获取底层宽度
mBgBitmapWidth = mBgBitmap.getWidth();
//创建顶层图片
mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(),
mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
```
为顶层图片加上画布，并画上灰色：

```
 //创建顶层画布
mCanvas = new Canvas(mFgBitmap);
//顶层画布画上灰色
mCanvas.drawColor(Color.GRAY);
```

##步骤三##

监听滑动事件，判断是否滑完：

```
case MotionEvent.ACTION_DOWN:
    mPath.reset();
    mPath.moveTo(event.getX(), event.getY());
    break;
case MotionEvent.ACTION_MOVE:
    mPath.lineTo(event.getX(), event.getY());
    mCanvas.drawPath(mPath, mPaint);
    //重新绘制画面
    invalidate();

    dx = Math.abs(event.getX() - LastX);
    if (dx > 0) {
        //监听左右滑
        sumX += dx;
    }
    break;
```
当手指松开时，判断是否滑完，以每一次左右滑累加得到的sumX和图片的宽度4倍比较，即是用手指左右滑动两个来回，并通过1秒延迟，让子线程重绘，涂刮剩余的灰色：

```
case MotionEvent.ACTION_UP:
     new Thread() {
         @Override
         public void run() {
             try {
                 if (sumX > mBgBitmapWidth * 4) {
                     isFinish = true;
                     Thread.sleep(1000);
                     postInvalidate();
                 }
             } catch (InterruptedException e) {
                 e.printStackTrace();
             }
         }
     }.start();
     break;
```
记得在监听最后面重置LastY：

```
LastX = event.getX();
return true;
```


----------
##步骤四##

我们的ondraw方法，只要判断是否滑完了，滑完了就不画上层图片：

```
@Override
protected void onDraw(Canvas canvas) {
    canvas.drawBitmap(mBgBitmap, 0, 0, null);
    if (!isFinish) {
        canvas.drawBitmap(mFgBitmap, 0, 0, null);
    }
}
```


----------


在布局文件中使用：

```
<com.handsome.app3.Custom.GuaGuaKa
     android:layout_width="wrap_content"
     android:layout_height="wrap_content" />
```

整个类的源码：

```
public class GuaGuaKa extends View {

    private Bitmap mBgBitmap, mFgBitmap;
    private Paint mPaint;
    private Canvas mCanvas;
    private Path mPath;
    private float mBgBitmapWidth;
    private float LastX, dx, sumX;
    private boolean isFinish = false;

    public GuaGuaKa(Context context) {
        super(context);
        init();
    }

    public GuaGuaKa(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public GuaGuaKa(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        //初始化透明画笔
        mPaint = new Paint();
        mPaint.setAlpha(0);
        mPaint.setXfermode(
                new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(50);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        //初始化路径
        mPath = new Path();
        //初始化底层图片
        mBgBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test);
        //获取底层宽度
        mBgBitmapWidth = mBgBitmap.getWidth();
        //创建顶层图片
        mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(),
                mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        //创建顶层画布
        mCanvas = new Canvas(mFgBitmap);
        //顶层画布画上灰色
        mCanvas.drawColor(Color.GRAY);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPath.reset();
                mPath.moveTo(event.getX(), event.getY());
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(event.getX(), event.getY());
                mCanvas.drawPath(mPath, mPaint);
                //重新绘制画面
                invalidate();

                dx = Math.abs(event.getX() - LastX);
                if (dx > 0) {
                    //监听左右滑
                    sumX += dx;
                }
                break;
            case MotionEvent.ACTION_UP:
                new Thread() {
                    @Override
                    public void run() {
                        try {
                            if (sumX > mBgBitmapWidth * 4) {
                                isFinish = true;
                                Thread.sleep(1000);
                                postInvalidate();
                            }
                        } catch (InterruptedException e) {
                            e.printStackTrace();
                        }
                    }
                }.start();
                break;
        }
        LastX = event.getX();
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawBitmap(mBgBitmap, 0, 0, null);
        if (!isFinish) {
            canvas.drawBitmap(mFgBitmap, 0, 0, null);
        }
    }
}
```