##CardView的使用、仿支付宝银行卡##


----------
>今天有空学习了下CardView的使用，既然是使用，不凡使用一个实例操作一下

CardView是Android5.0的新控件，所以我们需要在dependencies中添加支持：

```
compile 'com.android.support:cardview-v7:24.2.1'
```
CardView是继承FrameLayout的一个布局控件，从源码可以看出CardView支持的属性有：

>* card_view:cardElevation 阴影的大小
>* card_view:cardMaxElevation 阴影最大高度
>* card_view:cardBackgroundColor 卡片的背景色
>* card_view:cardCornerRadius 卡片的圆角大小
>* card_view:contentPadding 卡片内容于边距的间隔
	* card_view:contentPaddingBottom
	* card_view:contentPaddingTop
	* card_view:contentPaddingLeft
	* card_view:contentPaddingRight
	* card_view:contentPaddingStart
	* card_view:contentPaddingEnd
>* card_view:cardUseCompatPadding 设置内边距，V21+的版本和之前的版本仍旧具有一样的计算方式
>* card_view:cardPreventConrerOverlap 在V20和之前的版本中添加内边距，这个属性为了防止内容和边角的重叠

我们看一下今天要实现的效果图：

![](http://img.blog.csdn.net/20161015121820736)

>有兴趣的朋友可以尝试使用ViewPager+CardView实现卡片画廊的效果

其实CardView的使用相当于加了一个布局使用，其CardView里面内容的实现，还是在布局中设计

>银行卡布局：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#ffffff"
    android:padding="16dp">

    <android.support.v7.widget.CardView
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="180dp"
        app:cardBackgroundColor="#099A8C"
        app:cardCornerRadius="10dp"
        app:cardElevation="10dp"
        app:contentPadding="16dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:orientation="horizontal">

            <ImageView
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:background="@drawable/icon_01" />

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:layout_weight="1"
                android:orientation="vertical"
                android:padding="8dp">

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="中国农业银行"
                    android:textColor="#ffffff"
                    android:textSize="18sp" />

                <TextView
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:text="储蓄卡"
                    android:textColor="#ffffff"
                    android:textSize="16sp" />
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:textColor="#ffffff"
                    android:gravity="center_vertical"
                    android:textSize="22sp"
                    android:text="**** **** **** 1234"/>
            </LinearLayout>

            <ImageView
                android:layout_width="60dp"
                android:layout_height="15dp"
                android:background="@drawable/icon_02" />
        </LinearLayout>
    </android.support.v7.widget.CardView>

</RelativeLayout>

```

>特别注意的是：使用CardView的属性时，记得加上命名空间的声明

```
xmlns:app="http://schemas.android.com/apk/res-auto"
```

>好了，今天CardView的使用就到这里