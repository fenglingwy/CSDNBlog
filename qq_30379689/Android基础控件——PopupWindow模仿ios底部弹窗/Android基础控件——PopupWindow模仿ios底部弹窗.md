#PopupWindow模仿ios底部弹窗#


----------
##**前言**##
>在H5火热的时代，许多框架都出了底部弹窗的控件，在H5被称为弹出菜单ActionSheet，今天我们也来模仿一个ios的底部弹窗，取材于苹果QQ的选择头像功能
##**正文**##
>废话不多说，先来个今天要实现的效果图

![](http://img.blog.csdn.net/20161023101707938)

>整个PopupWindow的开启代码

```
private void openPopupWindow(View v) {
    //防止重复按按钮
    if (popupWindow != null && popupWindow.isShowing()) {
        return;
    }
    //设置PopupWindow的View
    View view = LayoutInflater.from(this).inflate(R.layout.view_popupwindow, null);
    popupWindow = new PopupWindow(view, RelativeLayout.LayoutParams.MATCH_PARENT,
            RelativeLayout.LayoutParams.WRAP_CONTENT);
    //设置背景,这个没什么效果，不添加会报错
    popupWindow.setBackgroundDrawable(new BitmapDrawable());
    //设置点击弹窗外隐藏自身
    popupWindow.setFocusable(true);
    popupWindow.setOutsideTouchable(true);
    //设置动画
    popupWindow.setAnimationStyle(R.style.PopupWindow);
    //设置位置
    popupWindow.showAtLocation(v, Gravity.BOTTOM, 0, navigationHeight);
    //设置消失监听
    popupWindow.setOnDismissListener(this);
    //设置PopupWindow的View点击事件
    setOnPopupViewClick(view);
    //设置背景色
    setBackgroundAlpha(0.5f);
}
```


步骤分析：

* PopupWindow的布局
* PopupWindow的创建
* PopupWindow添加动画效果
* PopupWindow设置背景阴影
* PopupWindow监听点击事件
* 获取NavigationBar的高度

>PopupWindow的布局：在Layout中，设计我们需要的布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:background="@drawable/popup_shape"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="16dp"
            android:text="你可以将照片上传至照片墙"
            android:textColor="#666"
            android:textSize="14sp" />

        <View
            android:layout_width="match_parent"
            android:layout_height="0.1px"
            android:background="#888" />

        <TextView
            android:id="@+id/tv_pick_phone"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="16dp"
            android:text="从手机相册选择"
            android:textColor="#118"
            android:textSize="18sp" />

        <View
            android:layout_width="match_parent"
            android:layout_height="0.1px"
            android:background="#888" />

        <TextView
            android:id="@+id/tv_pick_zone"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="16dp"
            android:text="从空间相册选择"
            android:textColor="#118"
            android:textSize="18sp" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="8dp"
        android:background="@drawable/popup_shape">

        <TextView
            android:id="@+id/tv_cancel"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            android:padding="16dp"
            android:text="取消"
            android:textColor="#118"
            android:textSize="18sp"
            android:textStyle="bold" />
    </LinearLayout>
</LinearLayout>

```
>其效果是：

![](http://img.blog.csdn.net/20161023102932968)


>PopupWindow的创建：这个创建与我们普通的控件创建方法是一样的

```
//设置PopupWindow的View
View view = LayoutInflater.from(this).inflate(R.layout.view_popupwindow, null);
popupWindow = new PopupWindow(view, RelativeLayout.LayoutParams.MATCH_PARENT,
                RelativeLayout.LayoutParams.WRAP_CONTENT);
```

> PopupWindow添加动画效果：我们创建一个anim文件夹，创建我们的out和in动画效果，然后在style添加我们的动画

```

<?xml version="1.0" encoding="utf-8"?>
<!--进入动画-->
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_interpolator"
    android:fromYDelta="100%" android:toYDelta="0"
    android:duration="200"/>
```

```
<?xml version="1.0" encoding="utf-8"?>
<!--退出动画-->
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:interpolator="@android:anim/accelerate_interpolator"
    android:fromYDelta="0" android:toYDelta="100%"
    android:duration="200"/>
```

```
<!--弹窗动画-->
<style name="PopupWindow">
    <item name="android:windowEnterAnimation">@anim/window_in</item>
    <item name="android:windowExitAnimation">@anim/window_out</item>
</style>
```

```
//设置动画
popupWindow.setAnimationStyle(R.style.PopupWindow);
```

>PopupWindow设置背景阴影：弹窗打开时设置透明度为0.5，弹窗消失时设置透明度为1

```
//设置屏幕背景透明效果
public void setBackgroundAlpha(float alpha) {
    WindowManager.LayoutParams lp = getWindow().getAttributes();
    lp.alpha = alpha;
    getWindow().setAttributes(lp);
}
```

>PopupWindow监听点击事件：监听我们PopupWindow里面控件的点击事件

```
//设置PopupWindow的View点击事件
setOnPopupViewClick(view);
```

```
private void setOnPopupViewClick(View view) {
    TextView tv_pick_phone, tv_pick_zone, tv_cancel;
    tv_pick_phone = (TextView) view.findViewById(R.id.tv_pick_phone);
    tv_pick_zone = (TextView) view.findViewById(R.id.tv_pick_zone);
    tv_cancel = (TextView) view.findViewById(R.id.tv_cancel);
    tv_pick_phone.setOnClickListener(this);
    tv_pick_zone.setOnClickListener(this);
    tv_cancel.setOnClickListener(this);
}
```

>获取NavigationBar的高度：这里需要适配有些手机没有NavigationBar有些手机有，这里以存在NavigationBar来演示

```
int resourceId = getResources().getIdentifier("navigation_bar_height", "dimen", "android");
navigationHeight = getResources().getDimensionPixelSize(resourceId);
```
>对存在NavigationBar的手机上，设置其PopupWindow的出现位置

```
//设置位置
popupWindow.showAtLocation(v, Gravity.BOTTOM, 0, navigationHeight);
```

>对没有NavigationBar的手机上，设置其PopupWindow的出现位置

```
//设置位置
popupWindow.showAtLocation(v, Gravity.BOTTOM, 0, 0);
```


----------
##**源码**##
> Github：https://github.com/AndroidHensen/IOSPopupWindow

```
public class MainActivity extends AppCompatActivity implements View.OnClickListener, PopupWindow.OnDismissListener {

    private Button bt_open;
    private PopupWindow popupWindow;
    private int navigationHeight;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bt_open = (Button) findViewById(R.id.bt_open);
        bt_open.setOnClickListener(this);

        int resourceId = getResources().getIdentifier("navigation_bar_height", "dimen", "android");
        navigationHeight = getResources().getDimensionPixelSize(resourceId);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.bt_open:
                openPopupWindow(v);
                break;
            case R.id.tv_pick_phone:
                Toast.makeText(this, "从手机相册选择", Toast.LENGTH_SHORT).show();
                popupWindow.dismiss();
                break;
            case R.id.tv_pick_zone:
                Toast.makeText(this, "从空间相册选择", Toast.LENGTH_SHORT).show();
                popupWindow.dismiss();
                break;
            case R.id.tv_cancel:
                popupWindow.dismiss();
                break;
        }
    }

    private void openPopupWindow(View v) {
        //防止重复按按钮
        if (popupWindow != null && popupWindow.isShowing()) {
            return;
        }
        //设置PopupWindow的View
        View view = LayoutInflater.from(this).inflate(R.layout.view_popupwindow, null);
        popupWindow = new PopupWindow(view, RelativeLayout.LayoutParams.MATCH_PARENT,
                RelativeLayout.LayoutParams.WRAP_CONTENT);
        //设置背景,这个没什么效果，不添加会报错
        popupWindow.setBackgroundDrawable(new BitmapDrawable());
        //设置点击弹窗外隐藏自身
        popupWindow.setFocusable(true);
        popupWindow.setOutsideTouchable(true);
        //设置动画
        popupWindow.setAnimationStyle(R.style.PopupWindow);
        //设置位置
        popupWindow.showAtLocation(v, Gravity.BOTTOM, 0, navigationHeight);
        //设置消失监听
        popupWindow.setOnDismissListener(this);
        //设置PopupWindow的View点击事件
        setOnPopupViewClick(view);
        //设置背景色
        setBackgroundAlpha(0.5f);
    }

    private void setOnPopupViewClick(View view) {
        TextView tv_pick_phone, tv_pick_zone, tv_cancel;
        tv_pick_phone = (TextView) view.findViewById(R.id.tv_pick_phone);
        tv_pick_zone = (TextView) view.findViewById(R.id.tv_pick_zone);
        tv_cancel = (TextView) view.findViewById(R.id.tv_cancel);
        tv_pick_phone.setOnClickListener(this);
        tv_pick_zone.setOnClickListener(this);
        tv_cancel.setOnClickListener(this);
    }

    //设置屏幕背景透明效果
    public void setBackgroundAlpha(float alpha) {
        WindowManager.LayoutParams lp = getWindow().getAttributes();
        lp.alpha = alpha;
        getWindow().setAttributes(lp);
    }

    @Override
    public void onDismiss() {
        setBackgroundAlpha(1);
    }
}
```