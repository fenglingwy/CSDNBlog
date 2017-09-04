#Dialog的自定义，仿淘宝、天猫、京东等商城广告弹窗


----------
>学习，学习，学以致用

Dialog已经是安卓开发者的家常便饭了，经常用来作为一些提醒功能，但是它也可以拓展为广告弹窗效果，让我们来自定义一个广告的Dialog吧。下面是今天要实现的效果图

![这里写图片描述](http://img.blog.csdn.net/20170718003435410?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**一、自定义Dialog布局的实现**

Dialog的布局很简单，其实就是一个充满容器的图片，view_dialog_advertisement.xml
```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <ImageView
        android:id="@+id/iv_advertisement"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="12dp"
        android:src="@drawable/welcome_advertisement" />

</RelativeLayout>
```
在Dialog里面放置我们要显示的效果图即可

![这里写图片描述](http://img.blog.csdn.net/20170717235910381?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**二、自定义Dialog代码的实现**

```
public class MyAdvertisementView extends Dialog implements View.OnClickListener {

    public MyAdvertisementView(Context context) {
        super(context);
        setContentView(R.layout.view_dialog_advertisement);
        //设置点击布局外则Dialog消失
        setCanceledOnTouchOutside(true);
    }

    public void showDialog() {
        Window window = getWindow();
        //设置弹窗动画
        window.setWindowAnimations(R.style.style_dialog);
        //设置Dialog背景色
        window.setBackgroundDrawableResource(R.color.colorTransparent);
        WindowManager.LayoutParams wl = window.getAttributes();
        //设置弹窗位置
        wl.gravity = Gravity.CENTER;
        window.setAttributes(wl);
        show();
        findViewById(R.id.iv_advertisement).setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        dismiss();
    }
}
```
上面代码所做的事情

1. 初始化设置Dialog显示的布局
2. 提供一个方法弹出Dialog
3. 设置Dialog里面控件的点击事件


##**三、自定义Dialog的弹窗动画**

在代码中通过window.setWindowAnimations()设置我们的弹窗动画，下面来style.xml的代码

```
<!-- dialog动画 -->
<style name="style_dialog">
    <item name="android:windowEnterAnimation">@anim/anim_popup_zoom_enter</item>
    <item name="android:windowExitAnimation">@anim/anim_popup_zoom_exit</item>
</style>
```
anim_popup_zoom_enter.xml

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <scale
        android:duration="500"
        android:fromXScale="0.8"
        android:fromYScale="0.8"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="1.0"
        android:toYScale="1.0" />

    <alpha
        android:duration="500"
        android:fromAlpha="0"
        android:toAlpha="1.0" />
</set>
```

anim_popup_zoom_exit.xml

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">

    <scale
        android:duration="300"
        android:fromXScale="1.0"
        android:fromYScale="1.0"
        android:pivotX="50%"
        android:pivotY="50%"
        android:toXScale="0.8"
        android:toYScale="0.8" />

    <alpha
        android:duration="300"
        android:fromAlpha="1.0"
        android:toAlpha="0" />
</set>
```


##**四、使用自定义Dialog**

在代码中只需要创建我们的Dialog对象，然后调用我们提供的弹窗方法即可

```
MyAdvertisementView myAdvertisementView = new MyAdvertisementView(this);
myAdvertisementView.showDialog();
```

[部分源码下载](http://download.csdn.net/detail/qq_30379689/9901857)

>到这里的课程就结束，如果对基础控件感兴趣的朋友可以关注我博客的基础控件系列