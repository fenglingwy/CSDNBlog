#SeekBar的使用、仿淘宝滑动验证


----------
>学习，学习，学以致用

SeekBar是一个拖动条控件，最简单的案例就是我们的调节音量，还有音频视频的播放，传统的SeekBar样式，如图

![](http://img.blog.csdn.net/20161122124551159)

>传统的实现太简单，不足以让我们到能装逼的地步。本来是打算实现滴滴出行滑动完成订单的效果，可惜找不到效果图，今天也就用淘宝的滑动验证来作为实例

![](http://img.blog.csdn.net/20161122122930666)

##**1.1 实现分析**

* SeekBar：使用progressDrawable属性自定义SeekBar
* 拖动块：使用thumb属性更改，其实就是一张图片
* 文字：使用RelativeLayout嵌套在一起


##**1.2 实现布局**


```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="16dp">

    <SeekBar
        android:id="@+id/sb"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:max="100"
        android:progress="0"
        android:progressDrawable="@drawable/seekbar_bg"
        android:thumb="@drawable/thumb"
        android:thumbOffset="0dp" />

    <TextView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:gravity="center"
        android:text="请按住滑块，拖动到最右边"
        android:textColor="#888888"
        android:textSize="14dp" />
</RelativeLayout>

```
>其效果是

![](http://img.blog.csdn.net/20161122125408660)

###**SeekBar属性介绍**

* android:max：设置进度条最大的进度值
* android:progress：设置当前的进度值
* android:progressDrawable：设置进度条的Drawable样式
* android:thumb：设置进度条滑块
* android:thumbOffset：设置进度条滑块的偏移量

##**1.3 SeekBar样式**
>这里是android:progressDrawable里面的seekbar_bg.xml
```
<?xml version="1.0" encoding="UTF-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--seekBar背景-->
    <item android:id="@android:id/background">
        <!--形状-->
        <shape android:shape="rectangle">
            <!--大小-->
            <size android:height="29dp" />
            <!--圆角-->
            <corners android:radius="2dp" />
            <!--背景-->
            <solid android:color="#E7EAE9" />
            <!--边框-->
            <stroke
                android:width="1dp"
                android:color="#C3C5C4" />
        </shape>
    </item>
    <!--seekBar的进度条-->
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="2dp" />
                <solid android:color="#7AC23C" />
                <stroke
                    android:width="1dp"
                    android:color="#C3C5C4" />
            </shape>
        </clip>
    </item>
</layer-list>
```

##**1.4 代码实现逻辑**
>代码也非常简单，seekBar提供了一个监听事件OnSeekBarChangeListener，在对应的回调中实现文字的出现和消失、文本内容的修改
```
public class MainActivity extends AppCompatActivity implements SeekBar.OnSeekBarChangeListener {

    private TextView tv;
    private SeekBar seekBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tv = (TextView) findViewById(R.id.tv);
        seekBar = (SeekBar) findViewById(R.id.sb);
        seekBar.setOnSeekBarChangeListener(this);
    }

    /**
     * seekBar进度变化时回调
     *
     * @param seekBar 
     * @param progress 
     * @param fromUser 
     */
    @Override
    public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
        if (seekBar.getProgress() == seekBar.getMax()) {
            tv.setVisibility(View.VISIBLE);
            tv.setTextColor(Color.WHITE);
            tv.setText("完成验证");
        } else {
            tv.setVisibility(View.INVISIBLE);
        }
    }

    /**
     * seekBar开始触摸时回调
     *
     * @param seekBar
     */
    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {

    }

    /**
     * seekBar停止触摸时回调
     *
     * @param seekBar
     */
    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
        if (seekBar.getProgress() != seekBar.getMax()) {
            seekBar.setProgress(0);
            tv.setVisibility(View.VISIBLE);
            tv.setTextColor(Color.GRAY);
            tv.setText("请按住滑块，拖动到最右边");
        }
    }
}
```

>好了，今天的SeekBar的使用就到这里，如果对其他基础控件感兴趣的，可以关注我的博客，基础控件系列，欢迎提供大家idea

[源码下载](http://download.csdn.net/detail/qq_30379689/9689724)