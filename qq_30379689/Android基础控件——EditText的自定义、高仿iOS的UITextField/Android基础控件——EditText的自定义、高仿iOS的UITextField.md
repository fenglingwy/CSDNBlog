#EditText的自定义、高仿iOS的UITextField


----------


>学习，学习，学以致用，让基础控件贴近实战效果

EditText是常用的文本框输入控件，它没有像iOS的UITextField那样，只要一个属性就可以在输入的时候弹出一个清除所有文本的图标，那么我们就要自定义一个，废话不多说，Hensen老师开车啦


iOS的效果

![这里写图片描述](http://img.blog.csdn.net/20170308170856092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

要实现的效果（有点卡，耐心看）

![这里写图片描述](http://img.blog.csdn.net/20170627152745923?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##1.1 实现分析

1. 使用EditText的setCompoundDrawablesWithIntrinsicBounds()填充图标
2. 使用EditText监听TextChangedListener
3. 使用Rect监听图标的点击事件

##1.2 实现代码

1、成员变量

```
//用户图标 20x20 具体分辨率根据不同机子适配
private Drawable imageUser;
//删除图标 15x15 具体分辨率根据不同机子适配
private Drawable imageGray;
private Context mContext;
```

2、构造方法

```
public MyEditText(Context context) {
    this(context, null);
}

public MyEditText(Context context, AttributeSet attrs) {
    this(context, attrs, 0);
}

public MyEditText(Context context, AttributeSet attrs, int defStyleAttr) {
    super(context, attrs, defStyleAttr);
    this.mContext = context;
    init(context);
}
```

3、初始化并监听文本变化

```
private void init(Context context) {
    imageUser = ContextCompat.getDrawable(context, R.drawable.user);
    imageGray = ContextCompat.getDrawable(context, R.drawable.gray);
    //初始化
    setImage();

    addTextChangedListener(new TextWatcher() {
        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {

        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {

        }

        @Override
        public void afterTextChanged(Editable s) {
            setImage();
        }
    });
}

private void setImage() {
    //setCompoundDrawablesWithIntrinsicBounds(left, top, right, bottom)
    //左边图标放置用户图标，右边图标放置删除图标，即第一个参数和第三个参数
    if (length() > 0) {
        setCompoundDrawablesWithIntrinsicBounds(imageUser, null, imageGray, null);
    } else {
        setCompoundDrawablesWithIntrinsicBounds(imageUser, null, null, null);
    }
}
```

4、监听点击事件

```
public boolean onTouchEvent(MotionEvent event) {
    switch (event.getAction()) {
        case MotionEvent.ACTION_UP:
            //得到手指离开EditText时的X Y坐标
            int x = (int) event.getRawX();
            int y = (int) event.getRawY();
            //创建一个长方形
            Rect rect = new Rect();
            //让长方形的宽等于edittext的宽，让长方形的高等于edittext的高
            getGlobalVisibleRect(rect);
            //把长方形缩短至右边30dp，约等于（padding+图标分辨率）
            rect.left = rect.right - dip2px(mContext, 30);
            //如果x和y坐标在长方形当中，说明你点击了右边的xx图片,清空输入框
            if (rect.contains(x, y)) {
                setText("");
            }
            break;
        default:
            break;
    }
    return super.onTouchEvent(event);
}

public static int dip2px(Context context, float dpValue) {
    final float scale = context.getResources().getDisplayMetrics().density;
    return (int) (dpValue * scale);
}
```


##1.3 布局实现

注意：

1. 由于不同机子的分辨率不同，需要对EditText的drawablePadding和Padding做相应的处理
2. 由于不同机子的分辨率不同，需要对图标的分辨率进行适配

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <com.handsome.boke2.EditText.MyEditText
        android:layout_width="200dp"
        android:layout_height="35dp"
        android:background="@drawable/bg_gray_border"
        android:drawablePadding="8dp"
        android:hint="请输入用户名"
        android:padding="8dp" />
</RelativeLayout>
```

而这个bg_gray_border其实就是个xml的背景图，代码如下

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <solid android:color="#ffffff" />
    <stroke
        android:width="0.01dp"
        android:color="#DEDEDE" />
    <corners android:radius="6dp" />
</shape>
```

[源码和图标下载](http://download.csdn.net/detail/qq_30379689/9881923)

>到这里的课程就结束，如果对基础控件感兴趣的朋友可以关注我博客的基础控件系列