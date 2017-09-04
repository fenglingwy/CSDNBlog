# Android群英传知识点回顾——第十二章：Android5.X新特性详解 #
----------
##知识点目录##
>* **12.1 Android5.X UI设计初步**
	*  12.1.1 材料的形态模拟
	*  12.1.2 更加真实的动画
	*  12.1.3 大色块的使用
>* **12.2 Material Design主题**
>* **12.3 Palette**
>* **12.4 视图与阴影**
>* **12.5 Tinting和Clipping**
	*  12.5.1 Tinting（着色）
	*  12.5.2 Clipping（裁剪）
>* **12.6 列表与卡片**
	*  12.6.1 RecyclerView
	*  12.6.2 CardView
>* **12.7 Android过渡动画**
>* **12.8 Material Design动画效果**
	*  12.8.1 Ripple效果
	*  12.8.2 Circular Reveal
	*  12.8.3 View state changes Animation
>* **12.9 Toolbar**
>* **12.10 Notification**
	*  12.10.1 基本的Notification
	*  12.10.2 折叠式Notification
	*  12.10.3 悬挂式Notification
	*  12.10.4 显示等级的Notification

----------
##知识点回顾##
>转眼间，安卓7.0已经出了，而我还在学习5.0的知识，此刻的心情是蓝瘦、香菇，看来得加把劲了，这篇文章较长，不过有很多实例的动画确实很好看，今年是个UI火热的一年，掌握了这些动画，可谓是你的看家本领，也是你进阶的一部分

###**12.1 Android5.X UI设计初步**
>Android5.X系列开始使用新的设计风格Material Design

###12.1.1 材料的形态模拟
>Google通过模拟自然界的形态变化、关线与阴影、纸与纸之间的空间层级关系，带来一种真实的空间感

![](http://img.blog.csdn.net/20161015004743010)

###12.1.2 更加真实的动画

![](http://img.blog.csdn.net/20161015005250293)


###12.1.3 大色块的使用
>Material Design中采用大量高饱和、适中亮度的大色块来突出界面的主次

![](http://img.blog.csdn.net/20161015010333347)

>希望了解更多的Material Design设计风格的读者可以访问网站：https://www.google.com/design/#resources

###**12.2 Material Design主题**
>Material Design提供默认的三种主题

```
@android:style/Theme.Material (dark version)        
@android:style/Theme.Material.Light (light version)     
@android:style/Theme.Material.Light.DarkActionBar
```

![](http://img.blog.csdn.net/20161015010736300)

>同时也提出了Color Palette的概念，让开发者自己设定系统区域的颜色

![](http://img.blog.csdn.net/20161015010811895)

>可以通过使用自定义Style来创建自己的Color Palette颜色主题

```
<resources>
	<!-- inherit from the material theme-->
    <style name="AppTheme" parent="android:Theme.Material">
        <!-- Main theme color-->
        <!-- your app branding color for the app bar-->
        <item name="colorPrimary">#BEBEBE</item>
        <!-- derker variant for thr status bar and contextual app bars-->
        <item name="colorPrimaryDark">#FF5AEBFF</item>
        <!--theme ui controls like checkBoxs and text fields-->
        <item name="colorAccent">#FFFF4130</item>
    </style>
</resources>
```

![](http://img.blog.csdn.net/20161015011247080)


###**12.3 Palette**
>Android5.X使用Palette来提取颜色，从而让主题能够动态适应当前页面的色调

>Android内置了几种提取颜色的种类：

>* Vibrant（充满活力的）
>* Vibrant dark（充满活力的黑）
>* Vibrant light（充满活力的白）
>* Muted(柔和的)
>* Muted dark(柔和的黑)
>* Muted light(柔和的白)

>使用Palette需要在Dependencies中添加依赖：

```
compile 'com.android.support:palette-v7:21.0.2'
```
>这这可以给Palette传进一个Bitmap对象，并调用它的Palette.generate()静态方法或者Palette.generateAsync()方法创建一个Palette

>下面例子演示如何通过加载图片的柔和色调来改变状态栏和Actionbar的色调：

```
public class PaletteActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.palette);
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test);
        // 创建Palette对象
        Palette.generateAsync(bitmap, new Palette.PaletteAsyncListener() {
            @Override
            public void onGenerated(Palette palette) {
                // 通过Palette来获取对应的色调
                Palette.Swatch vibrant =
                        palette.getDarkVibrantSwatch();
                // 将颜色设置给相应的组件
                getSupportActionBar().setBackgroundDrawable(
                        new ColorDrawable(vibrant.getRgb()));
                Window window = getWindow();
                window.setStatusBarColor(vibrant.getRgb());
            }
        });
    }
}
```

>使用书上源码会出现空指针错误，我们稍微把程序修改一下，查看效果图：

![](http://img.blog.csdn.net/20161015145902276)

###**12.4 视图与阴影**
>Material Design最重要的特点就是拟物扁平化

###**12.4.1 阴影效果**
>Android5.X在以外的View中只有x和y的方向上添加了z方向，View的z值由两部分组成：

```
Z = elevation + translationZ;
```
 >通常可以在XML布局文件中设置View的视图高度

```
android:elevation="XXdp"
```

>下面通过实例演示不同视图高度的效果：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher" />

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="2dp" />

    <TextView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_margin="10dp"
        android:background="@mipmap/ic_launcher"
        android:elevation="10dp" />
</LinearLayout>
```

![](http://img.blog.csdn.net/20161015150438872)

>在程序中也可以用代码变换视图高度：

```
view.setTranslationZ(xxx);
```

>通常和还可以用属性动画来改变视图高度：

```
if(flag){
    view.animate().translationZ(100);
    flag = false;
}else {
    view.animate().translationZ(0);
    flag = true;
}
```

###**12.5 Tinting和Clipping**

>Android5.X提供了两个对操作图像的新功能：Tinting和Clipping

###12.5.1 Tinting（着色）
>下面有例子说明，注意tint和tintMode属性：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher" />

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright" />
	
    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright"
        android:tintMode="add" />

    <ImageView
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:elevation="5dp"
        android:src="@mipmap/ic_launcher"
        android:tint="@android:color/holo_blue_bright"
        android:tintMode="multiply" />

</LinearLayout>
```
>效果图：

![](http://img.blog.csdn.net/20161015151934737)

###12.5.2 Clipping（裁剪）
>使用Clipping，首先需要使用ViewOutlineProvider来修改outline，然后再通过setOutlinProvider将outline作用给视图

>下面通过例子理解Clipping：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical">

    <TextView
        android:id="@+id/tv_rect"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />

    <TextView
        android:id="@+id/tv_circle"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_marginTop="20dp"
        android:elevation="1dp"
        android:gravity="center" />

</LinearLayout>
```
>在代码中：

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
		View v1 = findViewById(R.id.tv_rect);
        View v2 = findViewById(R.id.tv_circle);
        //获取Outline
        ViewOutlineProvider viewOutlineProvider1 = new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                //修改outline为特定形状
                outline.setRoundRect(0, 0, view.getWidth(), view.getHeight(), 30);
            }
        };
        //获取Outline
        ViewOutlineProvider viewOutlineProvider2 = new ViewOutlineProvider() {
            @Override
            public void getOutline(View view, Outline outline) {
                //修改outline为特定形状
                outline.setOval(0, 0, view.getWidth(), view.getHeight());
            }
        };
        //重新设置形状
        v1.setOutlineProvider(viewOutlineProvider1);
        v2.setOutlineProvider(viewOutlineProvider2);
    }
}
```
>效果图：

![](http://img.blog.csdn.net/20161015152119113)

###**12.6 列表与卡片**
>关于这章，可以在我博客上已经写过了，那么就不再多写了

>传送门：

>* [Android基础控件——RecyclerView实现拖拽排序侧滑删除效果](http://blog.csdn.net/qq_30379689/article/details/52463139)
>* [Android基础控件——CardView的使用、仿支付宝银行卡](http://blog.csdn.net/qq_30379689/article/details/52822195)

###**12.7 Activity过渡动画**
>在Activity之间的跳转中，通过overridePendingtransition(int inId，int outId)这个方法给Activity添加一些切换动画，效果也是差强人意

>Android5.X提供了三种Transition类型的动画：

>* 进入：一个进入的过渡动画决定Activity中的所有视图怎么进入屏幕
>* 退出：一个退出的过渡动画决定Activity中的所有视图怎么退出屏幕
>* 共享元素：一个共享元素过渡动画决定两个Activity之间的过渡，怎么共享他们的视图

>其中，进人和退出效果包括：

>* explode(分解)一一从屏幕中间进或出，移动视图
>* slide(滑动)——从屏幕边缘进或出，移动视图
>* fade(淡出) 一一通过改变屏幕上视图的不透明度达到添加或者移除视图

>共享元素包括（简单的说ActivityA的指定的View不会消失，带着View跳转到ActivityB中）：

>* changeBounds——改变目标视图的布局边界
>* changeClipBounds——裁剪目标视图边界
>* changeTransfrom——改变目标视图的缩放比例和旋转角度
>* changeImageTransfrom——改变目标图片的大小和缩放比例

>这里比如ActivityA跳转到ActivityB，运用这三种过渡动画，则在ActivityA中修改：

```
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this).toBundle());
```
>而在ActivityB，只需要设置以下代码：

```
getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
```

>或者在ActivityB的样式文件设置如下代码：

```
<item name="android:windowContentTransitions">true</item>
```

>那么接下来可以设置进入ActivityB的动画效果：

```
getWindow().setEnterTransition(new Explode());
getWindow().setEnterTransition(new Slide());
getWindow().setEnterTransition(new Fade());
```

>或者是设置离开ActivityB的动画效果：

```
getWindow().setExitTransition(new Explode());
getWindow().setExitTransition(new Slide());
getWindow().setExitTransition(new Fade());
```

###三种普通切换动画的效果图
![](http://img.blog.csdn.net/20161015172210627)

>下面是共享元素的使用，首先在ActivityA中设置共享元素的属性：

```
android:transitionName="XXX"
```

>同时在ActivityB中设置相同的共享元素的属性：

```
android:transitionName="XXX"
```

>在代码中使用共享元素，首先是单个共享元素：

```
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this,view,"share").toBundle());
```
>或者一个视图中有多个共享元素，通过Pair.create()来创建多个共享元素：

```
startActivity(intent, ActivityOptions.makeSceneTransitionAnimation(this, 
		Pair.create(view,"share"),Pair.create(fab,"fab")).toBundle());
```

###共享元素的效果图
![](http://img.blog.csdn.net/20161015172931025)

>Android5.X切换动画例子......见经典代码回顾案例一

###**12.8 Material Design动画效果**
>无知识点

###12.8.1 Ripple效果
>Ripple效果：点击后的波纹效果，可以通过代码设置波纹背景

```
//波纹有边界
android:background="?android:attr/selectableItemBackground"
//波纹无边界
android:background="?android:attr/selectableItemBackgroundBorderless"
```

>下面通过例子来演示一下这两个效果：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/holo_blue_light"
    android:gravity="center"
    android:orientation="vertical">

    <Button
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="?android:attr/selectableItemBackground"
        android:text="有界波纹"
        android:textColor="@android:color/white" />


    <Button
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="?android:attr/selectableItemBackgroundBorderless"
        android:text="无界波纹"
        android:textColor="@android:color/white" />

</LinearLayout>
```
>效果图：

![](http://img.blog.csdn.net/20161015184102639)

>同样的，也可以通过XML来声明一个ripple，然后在布局文件中使用：

```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:colorPrimary">
    <item>
        <shape android:shape="oval">
            <solid android:color="?android:colorAccent" />
        </shape>
    </item>
</ripple>
```

```
<Button
    android:layout_width="100dp"
    android:layout_height="100dp"
    android:background="@drawable/ripple" />
```
>效果图：

![](http://img.blog.csdn.net/20161015184408030)

###12.8.2 Circular Reveal
>这个动画效果具体变现为一个View以圆形的形式展开，揭示出来，通过ViewAnimationUtils.createCircularReveal()来创建动画，代码如下：
```
public static Animator createCircularReveal(View view, int centerX, 
int centerY, float startRadius, float endRadius) {
     return new RevealAnimator(view,centerX,centerY,startRadius,endRadius);
}
```
> RevealAnimator的使用特别简单，主要就是几个关键的坐标点：

>* centerX 动画开始的中心点X
>* centerY 动画开始的中心点Y
>* startRadius 动画开始半径
>* endRadius 动画结束半径

>接下来我们来看一下例子，了解下RevealAnimator的使用，首先用XML创建一个圆和方形：

```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:colorControlHighlight">
    <item>
        <shape android:shape="oval">
            <solid android:color="#738ffe" />
        </shape>
    </item>
</ripple>
```

```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:colorControlHighlight">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="#738ffe" />
        </shape>
    </item>
</ripple>
```
>然后在布局文件中使用这两个图形：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center">

    <ImageView
        android:id="@+id/oval"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@drawable/oval" />

    <ImageView
        android:id="@+id/rect"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:background="@drawable/rect" />

</LinearLayout>
```

>主程序：

```
public class CircularReveal extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_circular_reveal);

        final View oval = findViewById(R.id.oval);
        oval.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Animator animator = ViewAnimationUtils.createCircularReveal(oval,
                        oval.getWidth() / 2, oval.getHeight() / 2, oval.getWidth(), 0);
                animator.setInterpolator(new AccelerateDecelerateInterpolator());
                animator.setDuration(2000);
                animator.start();
            }
        });

        final View rect = findViewById(R.id.rect);
        rect.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Animator animator = ViewAnimationUtils.createCircularReveal(rect, 0, 0, 0,
                        (float) Math.hypot(rect.getWidth(), rect.getHeight()));
                animator.setInterpolator(new AccelerateInterpolator());
                animator.setDuration(2000);
                animator.start();
            }
        });
    }
}
```

>效果图：

![](http://img.blog.csdn.net/20161015190323915)

###12.8.3 View state changes Animation

>* StateListAnimator：视图动画效果，以前的Selector用来修改背景达到反馈效果，现在支持Selector使用动画效果

>在XML定义一个StateListAnimator，添加到Selector中：

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <set>
            <objectAnimator android:property="rotationX" 
	    android:duration="@android:integer/config_shortAnimTime" 
	    android:valueTo="360" 
	    android:valuyeType="floatType" />
        </set>
    </item>
    <item android:state_pressed="false">
        <set>
            <objectAnimator android:property="rotationX"
	    android:duration="@android:integer/config_shortAnimTime" 
	    android:valueTo="0" 
	    android:valuyeType="floatType" />
        </set>
    </item>
</selector>
```
>在布局文件中直接使用：

```
<Button
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:background="@drawable/anim_change" />
```

>特别要注意：Animator是要开启的，所以必须在主代码中使用AnimatorInflater.loadStateListAnimator()方法，并且通过View.setStateListAnimator()方法分配到视图中


----------


>* animated-selector：同样作为一种状态改变的动画效果Selector

>首先得有一套状态变换图：

![](http://img.blog.csdn.net/20161015192233439)
>然后在XML中定义这些图片组合：

```
<animated-selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/state_on"
        android:state_checked="true">
        <bitmap android:src="@drawable/ic_done_anim_030" />
    </item>
    <item android:id="@+id/state_off">
        <bitmap android:src="@drawable/ic_plus_anim_030" />
    </item>
    <transition
        android:fromId="@+id/state_on"
        android:toId="@+id/state_off">
        <animation-list>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_000" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_001" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_002" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_003" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_004" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_005" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_006" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_007" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_008" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_009" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_010" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_011" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_012" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_013" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_014" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_015" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_016" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_017" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_018" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_019" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_020" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_021" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_022" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_023" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_024" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_025" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_026" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_027" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_028" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_029" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_plus_anim_030" />
            </item>
        </animation-list>
    </transition>
    <transition
        android:fromId="@+id/state_off"
        android:toId="@+id/state_on">
        <animation-list>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_000" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_001" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_002" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_003" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_004" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_005" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_006" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_007" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_008" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_009" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_010" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_011" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_012" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_013" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_014" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_015" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_016" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_017" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_018" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_019" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_020" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_021" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_022" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_023" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_024" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_025" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_026" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_027" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_028" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_029" />
            </item>
            <item android:duration="16">
                <bitmap android:src="@drawable/ic_done_anim_030" />
            </item>
        </animation-list>
    </transition>
</animated-selector>

```
>最后在主程序中使用即可：

```
public class AnimatedSelectorActivity extends AppCompatActivity {

    private boolean mIsCheck;
    private static final int[] STATE_CHECKED = new int[]{
            android.R.attr.state_checked};
    private static final int[] STATE_UNCHECKED = new int[]{};
    private ImageView mImageView;
    private Drawable mDrawable;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_animated_selector);
        mImageView = (ImageView) findViewById(R.id.image);
        mDrawable = getResources().getDrawable(
                R.drawable.fab_anim);
        mImageView.setImageDrawable(mDrawable);
    }

    public void anim(View view) {
        if (mIsCheck) {
            mImageView.setImageState(STATE_UNCHECKED, true);
            mIsCheck = false;
        } else {
            mImageView.setImageState(STATE_CHECKED, true);
            mIsCheck = true;
        }
    }
}
```
>效果图：

![](http://img.blog.csdn.net/20161015192630575)

###**12.9 Toolbar**
>Toolbar与ActionBar最大的区别就是Toolbar更加自由、可控，这也是Google逐渐使用Toolbar替代Actionbar的原因，使用Toolbar必须引入appcompat-v7包，并设置主题为NoActionBar，使用以下代码进行设置：

```
<resources>

    <style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
        <!-- toolbar颜色-->
        <item name="colorPrimary">@color/colorPrimary</item>
        <!-- 状态栏颜色-->
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <!-- 窗口的背景颜色-->
        <item name="android:windowBackground">@android:color/white</item>
        <!-- add SearchView-->
        <item name="android:searchViewStyle">@style/MySearchView</item>
    </style>

    <style name="MySearchView" parent="Widget.AppCompat.SearchView" />
</resources>
```
>Toolbar的使用......见经典代码回顾案例二


###**12.10 Notification**
>Android5.0对Notification进行了优化：

>* 长按Notification可以显示消息来源
>* 锁屏状态下，可以看到Notification的通知

###12.10.1 基本的Notification
>基本的Notification......见经典代码回顾案例三

###12.10.2 折叠式Notification
>折叠式Notification......见经典代码回顾案例四
###12.10.3 悬挂式Notification
>悬挂式Notification......见经典代码回顾案例五
###12.10.4 显示等级的Notification
>Notification分为三个等级：

>* VISIBILITY_PRIVATE：表明只有当没有锁屏的时候会显示
>* VISIBILITY_PUBLIC：表明在任何情况下都会显示
>* VISIBILITY_SECRET：表明在pin、password等安全锁和没有锁屏的情况下显示

>设置Notification的等级非常简单，只需在案例三四五增加一句：

```
builder.setVisibility(Notification.VISIBILITY_PUBLIC);
```
>同时，Notificatio还提供了其他方法：

>* 设置Notification背景颜色

```
builder.setColor(Color.RED);
```

>* 设置Notification的category接口，用来确定Notification的显示位置

```
builder.setCategory(Notification.CATEGORY_MESSAGE);
```

##经典代码回顾##

###案例一：Android5.X切换动画例子
>分别看下ActivityA和ActivityB的布局文件，注意transitionName在哪个View：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:orientation="vertical"
    android:layout_height="match_parent">


    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="explode"
        android:text="explode" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="slide"
        android:text="slide" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:onClick="fade"
        android:text="fade" />

    <Button
        android:layout_width="match_parent"
        android:layout_height="100dp"
        android:transitionName="share"
        android:onClick="share"
        android:text="share" />

    <Button
        android:id="@+id/fab_button"
        android:layout_width="56dp"
        android:transitionName="fab"
        android:layout_height="56dp"
        android:background="@drawable/ripple_round"
        android:elevation="5dp"/>
</LinearLayout>

```

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin">

    <View
        android:id="@+id/holder_view"
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:transitionName="share"
        android:background="?android:colorPrimary" />

    <Button xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@+id/fab_button"
        android:transitionName="fab"
        android:layout_width="56dp"
        android:layout_height="56dp"
        android:layout_marginRight="@dimen/activity_horizontal_margin"
        android:background="@drawable/ripple_round"
        android:elevation="5dp"
        android:layout_below="@+id/holder_view"
        android:layout_marginTop="-26dp"
        android:layout_alignParentEnd="true" />

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:paddingTop="10dp"
        android:layout_below="@id/holder_view">

        <Button
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:id="@+id/button"
            android:layout_below="@+id/button4"
            android:layout_marginTop="10dp" />

        <Button
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:layout_marginTop="10dp"
            android:id="@+id/button4"
            android:layout_alignParentStart="true" />


    </RelativeLayout>

</RelativeLayout>

```

>分别看下ActivityA和ActivityB的Activity文件：

```
public class TransitionsA extends Activity {

    private Intent intent;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_transition);
    }

    // 设置不同动画效果
    public void explode(View view) {
        intent = new Intent(this, TransitionsB.class);
        intent.putExtra("flag", 0);
        startActivity(intent,
                ActivityOptions.makeSceneTransitionAnimation(this)
                        .toBundle());
    }
    // 设置不同动画效果
    public void slide(View view) {
        intent = new Intent(this, TransitionsB.class);
        intent.putExtra("flag", 1);
        startActivity(intent,
                ActivityOptions.makeSceneTransitionAnimation(this)
                        .toBundle());
    }
    // 设置不同动画效果
    public void fade(View view) {
        intent = new Intent(this, TransitionsB.class);
        intent.putExtra("flag", 2);
        startActivity(intent,
                ActivityOptions.makeSceneTransitionAnimation(this)
                        .toBundle());
    }
    // 设置不同动画效果
    public void share(View view) {
        View fab = findViewById(R.id.fab_button);
        intent = new Intent(this, TransitionsB.class);
        intent.putExtra("flag", 3);
        // 创建单个共享元素
//        startActivity(intent,
//                ActivityOptions.makeSceneTransitionAnimation(
//                        this, view, "share").toBundle());
        startActivity(intent,
                ActivityOptions.makeSceneTransitionAnimation(
                        this,
                        // 创建多个共享元素
                        Pair.create(view, "share"),
                        Pair.create(fab, "fab")).toBundle());
    }
}
```

```
public class TransitionsB extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getWindow().requestFeature(Window.FEATURE_CONTENT_TRANSITIONS);
        int flag = getIntent().getExtras().getInt("flag");
        // 设置不同的动画效果
        switch (flag) {
            case 0:
                getWindow().setEnterTransition(new Explode());
                break;
            case 1:
                getWindow().setEnterTransition(new Slide());
                break;
            case 2:
                getWindow().setEnterTransition(new Fade());
                getWindow().setExitTransition(new Fade());
                break;
            case 3:
                break;
        }
        setContentView(R.layout.activity_transition_to);
    }
}
```

###效果图

![](http://img.blog.csdn.net/20161015182329979)


----------


###案例二：Toolbar的使用
>记得先将主题设置为NoActionBar主题，在布局中声明ToolBar

```
<android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize" />
```
>在Menu文件夹中创建一个Menu的XML文件

```
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    tools:context=".MainActivity" >

    <item
        android:id="@+id/ab_search"
        android:orderInCategory="80"
        android:title="action_search"
        app:actionViewClass="android.support.v7.widget.SearchView"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/action_share"
        android:orderInCategory="90"
        android:title="action_share"
        app:actionProviderClass="android.support.v7.widget.ShareActionProvider"
        app:showAsAction="ifRoom"/>
    <item
        android:id="@+id/action_settings"
        android:orderInCategory="100"
        android:title="action_settings"
        app:showAsAction="never"/>

</menu>
```

>在代码中使用

```
public class ToolbarActivity extends AppCompatActivity {

    private ShareActionProvider mShareActionProvider;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_toolbar);

        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        toolbar.setLogo(R.mipmap.ic_launcher);
        toolbar.setTitle("主标题");
        toolbar.setSubtitle("副标题");
        setSupportActionBar(toolbar);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
        return super.onCreateOptionsMenu(menu);
    }
}
```

###效果图

![](http://img.blog.csdn.net/20161015205127577)

>修改一下布局，实现Toolbar侧滑菜单

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.handsome.qunyingzhuang.chapter_12.Anli02.ToolbarActivity">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize" />

    <android.support.v4.widget.DrawerLayout
        android:id="@+id/drawer"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <!-- 内容界面 -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="@android:color/holo_blue_light"
            android:orientation="vertical">

            <Button
                android:layout_width="100dp"
                android:layout_height="match_parent"
                android:text="内容界面" />
        </LinearLayout>

        <!-- 侧滑菜单内容 必须指定其水平重力 -->
        <LinearLayout
            android:id="@+id/drawer_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="start"
            android:orientation="vertical">

            <Button
                android:layout_width="200dp"
                android:layout_height="match_parent"
                android:text="菜单界面" />
        </LinearLayout>
    </android.support.v4.widget.DrawerLayout>
</LinearLayout>

```
>在主代码中实现：

```
public class ToolbarActivity extends AppCompatActivity {

    private ShareActionProvider mShareActionProvider;
    private DrawerLayout mDrawerLayout;
    private ActionBarDrawerToggle mDrawerToggle;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_toolbar);

        Toolbar toolbar = (Toolbar) findViewById(R.id.toolbar);
        toolbar.setLogo(R.mipmap.ic_launcher);
        toolbar.setTitle("主标题");
        toolbar.setSubtitle("副标题");
        setSupportActionBar(toolbar);

        getSupportActionBar().setDisplayHomeAsUpEnabled(true);
        mDrawerLayout = (DrawerLayout) findViewById(R.id.drawer);
        mDrawerToggle = new ActionBarDrawerToggle(
                this, mDrawerLayout, toolbar,
                R.string.abc_action_bar_home_description,
                R.string.abc_action_bar_home_description_format);
        mDrawerToggle.syncState();
        mDrawerLayout.setDrawerListener(mDrawerToggle);
    }

    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.menu_main, menu);
		/* ShareActionProvider配置 */
        mShareActionProvider = (ShareActionProvider) MenuItemCompat.getActionProvider(menu
                .findItem(R.id.action_share));
        Intent intent = new Intent(Intent.ACTION_SEND);
        intent.setType("text/*");
        mShareActionProvider.setShareIntent(intent);
        return super.onCreateOptionsMenu(menu);
    }
}
```
###效果图

![](http://img.blog.csdn.net/20161015205620256)

----------


###案例三：基本的Notification

```
public class Notification01 extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_notification01);
        //第一步：初始化
        Notification.Builder builder = new Notification.Builder(this);
        //第二步：构建点击之后的意图
        Intent intent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://www.baidu.com"));
        //构造pendingdintent
        PendingIntent pendingIntent = PendingIntent.getActivities(this, 0, new Intent[]{intent}, 0);
        //第三步：设置通知栏的各种消息
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        builder.setContentText("Title");
        builder.setContentText("内容");
        builder.setSubText("text");
        //第四步：通过NotificationManager来发出
        NotificationManager notificationManager = (NotificationManager) getSystemService(NOTIFICATION_SERVICE);
        notificationManager.notify(0, builder.build());
    }
}
```
###效果图

![](http://img.blog.csdn.net/20161015210020809)

----------


###案例四：折叠式Notification

```
public class Notification02 extends AppCompatActivity {

    private static final int NOTIFICATION_ID_COLLAPSE = 0x01;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_notification02);
        Intent intent = new Intent(Intent.ACTION_VIEW,
                Uri.parse("http://www.sina.com"));
        PendingIntent pendingIntent = PendingIntent.getActivity(
                this, 0, intent, 0);
        Notification.Builder builder = new Notification.Builder(this);
        builder.setSmallIcon(R.mipmap.ic_launcher);
        builder.setContentIntent(pendingIntent);
        builder.setAutoCancel(true);
        builder.setLargeIcon(BitmapFactory.decodeResource(
                getResources(), R.mipmap.ic_launcher));
        // 通过RemoteViews来创建自定义的Notification视图
        RemoteViews contentView =
                new RemoteViews(getPackageName(),
                        R.layout.notification);
        contentView.setTextViewText(R.id.textView,
                "show me when collapsed");

        Notification notification = builder.build();
        notification.contentView = contentView;
        // 通过RemoteViews来创建自定义的Notification视图
        RemoteViews expandedView =
                new RemoteViews(getPackageName(),
                        R.layout.notification_expanded);
        notification.bigContentView = expandedView;

        NotificationManager nm = (NotificationManager)
                getSystemService(NOTIFICATION_SERVICE);
        nm.notify(NOTIFICATION_ID_COLLAPSE, notification);
    }
}
```
![](http://img.blog.csdn.net/20161015235637916)

----------


###案例五：悬挂式Notification

```
public class Notification03 extends AppCompatActivity {

    private static final int NOTIFICATION_ID_HEADSUP = 0x01;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_notification03);
        Notification.Builder builder = new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setPriority(Notification.PRIORITY_DEFAULT)
                .setCategory(Notification.CATEGORY_MESSAGE)
                .setContentTitle("Headsup Notification")
                .setContentText("I am a Headsup notification.");

        Intent push = new Intent();
        push.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        push.setClass(this, MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(
                this, 0, push, PendingIntent.FLAG_CANCEL_CURRENT);
        builder.setContentText("Heads-Up Notification on Android 5.0")
                .setFullScreenIntent(pendingIntent, true);

        NotificationManager nm = (NotificationManager)
                getSystemService(NOTIFICATION_SERVICE);
        nm.notify(NOTIFICATION_ID_HEADSUP, builder.build());
    }
}
```
![](http://img.blog.csdn.net/20161015235908030)

----------



>经典回顾源码下载

>github：https://github.com/CSDNHensen/QunYingZhuang