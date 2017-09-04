##仿1号店垂直滚动广告条实现##


----------
##效果图展示，图片有点卡，耐心看会，原程序是很流畅的
![](http://img.blog.csdn.net/20161012103501209)

实现步骤：

>* 声明变量
>* 初始化画笔、文本大小和坐标
>* onMeasure()适配wrap_content的宽高
>* onDraw()画出根据坐标画出两段Text
>* 监听点击事件
>* 在Activity中实现点击事件

实现原理（坐标变换原理）：整个过程都是基于坐标Y的增加和交换进行处理的，Y值都会一直增加到endY，然后进行交换逻辑

![](http://img.blog.csdn.net/20161011224319583)

----------


##步骤一：声明变量##

>由于1号店是两句话的滚动，所以我们也是使用两句话来实现的

```
private Paint mPaint;
private float x, startY, endY, firstY, nextStartY, secondY;

//整个View的宽高是以第一个为标准的，所以第二句话长度必须小于第一句话
private String[] text = {"今日特卖：毛衣3.3折>>>", "公告：全场半价>>>"};
private float textWidth, textHeight;

//滚动速度
private float speech = 0;
private static final int CHANGE_SPEECH = 0x01;
//是否已经在滚动
private boolean isScroll = false;
```


----------


##步骤二：初始化画笔、文本大小和坐标##

>以第一句话为标准来做控件的宽高标准

```
//初始化画笔
mPaint = new Paint();
mPaint.setColor(Color.RED);
mPaint.setTextSize(30);
//测量文字的宽高，以第一句话为标准
Rect rect = new Rect();
mPaint.getTextBounds(text[0], 0, text[0].length(), rect);
textWidth = rect.width();
textHeight = rect.height();
//文字开始的x,y坐标
//由于文字是以基准线为基线的，文字底部会突出一点，所以向上收5px
x = getX() + getPaddingLeft();
startY = getTop() + textHeight + getPaddingTop() - 5;
//文字结束的x,y坐标
endY = startY + textHeight + getPaddingBottom();
//下一个文字滚动开始的y坐标
//由于文字是以基准线为基线的，文字底部会突出一点，所以向上收5px
nextStartY = getTop() - 5;
//记录开始的坐标
firstY = startY;
secondY = nextStartY;
```


----------


##步骤三：onMeasure()适配wrap_content的宽高##

>如果学习过自定义View的话，下面的代码应该很熟悉，就是适配warp_content的模板代码：

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    int width = measureWidth(widthMeasureSpec);
    int height = measureHeight(heightMeasureSpec);
    setMeasuredDimension(width, height);
}

private int measureHeight(int heightMeasureSpec) {
    int result = 0;
    int size = MeasureSpec.getSize(heightMeasureSpec);
    int mode = MeasureSpec.getMode(heightMeasureSpec);
    if (mode == MeasureSpec.EXACTLY) {
        result = size;
    } else {
        result = (int) (getPaddingTop() + getPaddingBottom() + textHeight);
        if (mode == MeasureSpec.AT_MOST) {
            result = Math.min(result, size);
        }
    }
    return result;
}

private int measureWidth(int widthMeasureSpec) {
    int result = 0;
    int size = MeasureSpec.getSize(widthMeasureSpec);
    int mode = MeasureSpec.getMode(widthMeasureSpec);
    if (mode == MeasureSpec.EXACTLY) {
        result = size;
    } else {
        result = (int) (getPaddingLeft() + getPaddingRight() + textWidth);
        if (mode == MeasureSpec.AT_MOST) {
            result = Math.min(result, size);
        }
    }
    return result;
}
```


----------


##步骤四：onDraw()画出根据坐标画出两段Text（已修复：Text停下来时闪一下的bug）##

>* 通过Handler来改变速度
>* 通过isScroll锁，来控制Handler只改变一次
>* 通过invalidate一直重绘两句话的文字

```
@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);

    //启动滚动
    if (!isScroll) {
        mHandler.sendEmptyMessageDelayed(CHANGE_SPEECH, 2000);
        isScroll = true;
    }

    canvas.drawText(text[0], x, startY, mPaint);
    canvas.drawText(text[1], x, nextStartY, mPaint);
    startY += speech;
    nextStartY += speech;

    //超出View的控件时
    if (startY > endY || nextStartY > endY) {
        if (startY > endY) {
            //第一次滚动过后交换值
            startY = secondY;
            nextStartY = firstY;
        } else if (nextStartY > endY) {
            //第二次滚动过后交换值
            startY = firstY;
            nextStartY = secondY;
        }
        speech = 0;
        isScroll = false;
    }
    invalidate();
}
```

```
private Handler mHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case CHANGE_SPEECH:
                speech = 1f;
                break;
        }
    }
};
```


----------


##步骤五：监听点击事件（已修复：点击事件错乱的问题）##

>在自定义View重写dispatchTouchEvent处理点击事件，这个也是模板代码：

```
public onTouchListener listener;

public interface onTouchListener {
    void touchListener(String s);
}

public void setListener(onTouchListener listener) {
    this.listener = listener;
}

@Override
public boolean dispatchTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_DOWN:
        case MotionEvent.ACTION_MOVE:
            //点击事件
            if (listener != null) {
                if (startY >= firstY && nextStartY < firstY) {
                    listener.touchListener(text[0]);
                } else if (nextStartY >= firstY && startY < firstY) {
                    listener.touchListener(text[1]);
                }
            }
            break;
    }
    return true;
}
```


----------


##步骤六：在Activity中实现点击事件##

```
public class VerTextViewActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_ver_text_view);

        VerTextView tv_ver = (VerTextView) findViewById(R.id.tv_ver);
        tv_ver.setListener(new VerTextView.onTouchListener() {
            @Override
            public void touchListener(String s) {
                Toast.makeText(VerTextViewActivity.this, s, Toast.LENGTH_LONG).show();
            }
        });
    }
}
```

>布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="120dp"
        android:layout_height="30dp"
        android:background="@drawable/vertextview" />

    <com.handsome.app3.Custom.VerTextView.VerTextView
        android:id="@+id/tv_ver"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="#ffffff"
        android:padding="8dp" />
</LinearLayout>
```


----------


>整个类的源码：

```
/**
 * =====作者=====
 * 许英俊
 * =====时间=====
 * 2016/10/11.
 */
public class VerTextView extends View {

    private Paint mPaint;
    private float x, startY, endY, firstY, nextStartY, secondY;

    //整个View的宽高是以第一个为标准的，所以第二句话长度必须小于第一句话
    private String[] text = {"今日特卖：毛衣3.3折>>>", "公告：全场半价>>>"};
    private float textWidth, textHeight;

    //滚动速度
    private float speech = 0;
    private static final int CHANGE_SPEECH = 0x01;
    //是否已经在滚动
    private boolean isScroll = false;

    private Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            super.handleMessage(msg);
            switch (msg.what) {
                case CHANGE_SPEECH:
                    speech = 1f;
                    break;
            }
        }
    };

    public VerTextView(Context context, AttributeSet attrs) {
        super(context, attrs);
        //初始化画笔
        mPaint = new Paint();
        mPaint.setColor(Color.RED);
        mPaint.setTextSize(30);
        //测量文字的宽高，以第一句话为标准
        Rect rect = new Rect();
        mPaint.getTextBounds(text[0], 0, text[0].length(), rect);
        textWidth = rect.width();
        textHeight = rect.height();
        //文字开始的x,y坐标
        //由于文字是以基准线为基线的，文字底部会突出一点，所以向上收5px
        x = getX() + getPaddingLeft();
        startY = getTop() + textHeight + getPaddingTop() - 5;
        //文字结束的x,y坐标
        endY = startY + textHeight + getPaddingBottom();
        //下一个文字滚动开始的y坐标
        //由于文字是以基准线为基线的，文字底部会突出一点，所以向上收5px
        nextStartY = getTop() - 5;
        //记录开始的坐标
        firstY = startY;
        secondY = nextStartY;
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
        int width = measureWidth(widthMeasureSpec);
        int height = measureHeight(heightMeasureSpec);
        setMeasuredDimension(width, height);
    }

    private int measureHeight(int heightMeasureSpec) {
        int result = 0;
        int size = MeasureSpec.getSize(heightMeasureSpec);
        int mode = MeasureSpec.getMode(heightMeasureSpec);
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else {
            result = (int) (getPaddingTop() + getPaddingBottom() + textHeight);
            if (mode == MeasureSpec.AT_MOST) {
                result = Math.min(result, size);
            }
        }
        return result;
    }

    private int measureWidth(int widthMeasureSpec) {
        int result = 0;
        int size = MeasureSpec.getSize(widthMeasureSpec);
        int mode = MeasureSpec.getMode(widthMeasureSpec);
        if (mode == MeasureSpec.EXACTLY) {
            result = size;
        } else {
            result = (int) (getPaddingLeft() + getPaddingRight() + textWidth);
            if (mode == MeasureSpec.AT_MOST) {
                result = Math.min(result, size);
            }
        }
        return result;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        //启动滚动
        if (!isScroll) {
            mHandler.sendEmptyMessageDelayed(CHANGE_SPEECH, 2000);
            isScroll = true;
        }

        canvas.drawText(text[0], x, startY, mPaint);
        canvas.drawText(text[1], x, nextStartY, mPaint);
        startY += speech;
        nextStartY += speech;

        //超出View的控件时
        if (startY > endY || nextStartY > endY) {
            if (startY > endY) {
                //第一次滚动过后交换值
                startY = secondY;
                nextStartY = firstY;
            } else if (nextStartY > endY) {
                //第二次滚动过后交换值
                startY = firstY;
                nextStartY = secondY;
            }
            speech = 0;
            isScroll = false;
        }
        invalidate();
    }

    public onTouchListener listener;

    public interface onTouchListener {
        void touchListener(String s);
    }

    public void setListener(onTouchListener listener) {
        this.listener = listener;
    }

    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
            case MotionEvent.ACTION_MOVE:
                //点击事件
                if (listener != null) {
                    if (startY >= firstY && nextStartY < firstY) {
                        listener.touchListener(text[0]);
                    } else if (nextStartY >= firstY && startY < firstY) {
                        listener.touchListener(text[1]);
                    }
                }
                break;
        }
        return true;
    }
}
```