#ViewFlipper的使用，仿淘宝头条垂直滚动广告条


----------
>学习，学习，学以致用

ViewFlipper是安卓自带的控件，很多人可能很少知道这个控件，这个控件很简单，也很好理解，能不能用上实战就看你们的本事了。下面是淘宝头条广告的原效果

![](http://img.blog.csdn.net/20170107153843305?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面是我们今天要实现的效果，图片是Gif，运行效果是很流畅的，由于这个图片反应有点慢，会浪费大家点时间，所以我把它调快了，大家可以掏出手机打开淘宝看，一模一样的

![](http://img.blog.csdn.net/20170107183100898?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

从源码可以看出，其实ViewFlipper间接的继承了FrameLayout，也可以说ViewFlipper其实就是个FrameLayout，只不过在内部封装了动画实现和Handler实现一个循环而已

##**一、ViewFlipper的布局实现**


布局的编写很简单，跟普通布局一样的
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ViewFlipper
        android:id="@+id/vf"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:autoStart="true"
        android:background="#fff"
        android:flipInterval="3000"
        android:inAnimation="@anim/anim_marquee_in"
        android:outAnimation="@anim/anim_marquee_out"
        android:paddingLeft="30dp" />
</RelativeLayout>
```
这里介绍ViewFlipper用到的属性，这些属性其实都可以使用代码实现，只不过这里为了代码看上去美观，才放在布局里的

* android:autoStart：设置自动加载下一个View
* android:flipInterval：设置View之间切换的时间间隔
* android:inAnimation：设置切换View的进入动画
* android:outAnimation：设置切换View的退出动画


下面是ViewFlipper常用的方法介绍，除了可以设置上面的属性之外，还提供了其他方法

* isFlipping： 判断View切换是否正在进行
* setFilpInterval：设置View之间切换的时间间隔
* startFlipping：开始View的切换，而且会循环进行
* stopFlipping：停止View的切换
* setOutAnimation：设置切换View的退出动画
* setInAnimation：设置切换View的进入动画
* showNext： 显示ViewFlipper里的下一个View
* showPrevious：显示ViewFlipper里的上一个View

这里还涉及到两个动画其实就是一个平移的动画，它们都保存在anim文件夹中

anim_marquee_in.xml

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="1500"
        android:fromYDelta="100%p"
        android:toYDelta="0"/>
</set>
```

anim_marquee_out.xml

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <translate
        android:duration="1500"
        android:fromYDelta="0"
        android:toYDelta="-100%p"/>
</set>
```
>当然，如果你对动画xml比较熟悉，自己可以实现更多好看的效果

##**二、自定义ViewFlipper的广告条**
当我们准备好了ViewFlipper之后，就应该在ViewFlipper里面添加我们的广告条了，下面是其中一个广告条的布局文件，另外两个雷同，只不过改了文字而已

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/marqueeview_bg"
            android:text="热议"
            android:textColor="#F14C00"
            android:textSize="12sp" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:padding="3dp"
            android:singleLine="true"
            android:text="小米6来了：晓龙835+8G运存！"
            android:textColor="#333"
            android:textSize="14sp" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:orientation="horizontal">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/marqueeview_bg"
            android:text="热议"
            android:textColor="#F14C00"
            android:textSize="12sp" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:ellipsize="end"
            android:padding="3dp"
            android:singleLine="true"
            android:text="227斤的胖MM，掀起上衣后，美爆全场！"
            android:textColor="#333"
            android:textSize="14sp" />
    </LinearLayout>
</LinearLayout>

```

其效果是

![](http://img.blog.csdn.net/20170107183634015?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**三、代码为ViewFlipper添加广告条**
所有的准备条件都准备好了，该开始使用代码将准备好的东西黏在一起了，代码很简单，这里就不多解释了

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ViewFlipper vf = (ViewFlipper) findViewById(R.id.vf);

        vf.addView(View.inflate(this, R.layout.view_advertisement01, null));
        vf.addView(View.inflate(this, R.layout.view_advertisement02, null));
        vf.addView(View.inflate(this, R.layout.view_advertisement03, null));
    }
}
```

[源码下载](http://download.csdn.net/detail/qq_30379689/9731413)

>到这里的课程就结束，如果对基础控件感兴趣的朋友可以关注我博客的基础控件系列