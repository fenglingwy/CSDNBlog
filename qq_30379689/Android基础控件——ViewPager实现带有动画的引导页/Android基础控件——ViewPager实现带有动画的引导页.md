#ViewPager实现带有动画的引导页


----------
>好了，又到我们学习基础控件的时候了，其实引导页很简单，就是五张图片而已
##**一、ViewPager实现传统的引导页**
传统的ViewPager实现引导页和ListView是一样道理的，只是把ListView的Item换成图片，把BaseAdapter换成PagerAdapter，我们先来看下传统引导页的效果图

![](http://img.blog.csdn.net/20161217001739768?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

###**步骤一：编写xml文件**
既然用到的是ViewPager，那么xml文件就必须要有ViewPager，细心的你，可能会发现最后一页还有个按钮的出现，没错，xml文件中也要有个按钮

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.ViewPager
        android:id="@+id/vp_guide"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <Button
        android:id="@+id/bt_main"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:layout_centerInParent="true"
        android:layout_marginBottom="50dp"
        android:background="@color/colorPrimary"
        android:padding="6dp"
        android:text="立即开启"
        android:textColor="#fff"
        android:textSize="16dp"
        android:visibility="gone" />
</RelativeLayout>
```

###**步骤二：编写Adapter**
开头也说了，Viewpager其实就和ListView一样的，需要一个Adapter，那么就从Adapter入手。Google提供了一个专门适配ViewPager的Adapter——PagerAdapter

```
public class GuideAdapter extends PagerAdapter {
    private List<View> views;
    private Context context;

    public GuideAdapter(List<View> views, Context context) {
        this.context = context;
        this.views = views;
    }

    public Object instantiateItem(View container, int position) {
        ((ViewPager) container).addView(views.get(position));
        return views.get(position);
    }

    public void destroyItem(View container, int position, Object object) {
        ((ViewPager) container).removeView(views.get(position));
    }

    public int getCount() {
        return views.size();
    }

    public boolean isViewFromObject(View arg0, Object arg1) {
        return (arg0 == arg1);
    }
}
```
>基本ViewPager的Adapter都是这么写的，就是往ViewPager中添加List传过来的View和删除List传过来的View，可以说是每个ViewPager的模板

###**步骤三：编写Activity**
我们找到对应的ViewPager，然后设置Adapter，代码中的initViews、initListener、initData是按顺序执行下去的，这段代码不难，很容易看懂
```
public class GuideActivity extends BaseActivity implements ViewPager.OnPageChangeListener {

    private ViewPager vp_guide;
    private int[] imgId = {R.drawable.guide_center_1, R.drawable.guide_center_2, R.drawable.guide_center_3,
            R.drawable.guide_center_4, R.drawable.guide_center_5};
    private List<View> mImageViews;
    private GuideAdapter adapter;
    private Button bt_main;

    @Override
    public void initViews() {
        setContentView(R.layout.activity_guide);
        vp_guide = (ViewPager) findViewById(R.id.vp_guide);
        bt_main = (Button) findViewById(R.id.bt_main);
    }

    @Override
    public void initListener() {
        bt_main.setOnClickListener(this);
        vp_guide.setOnPageChangeListener(this);
    }

    @Override
    public void initData() {
        //初始化引导资源
        mImageViews = new ArrayList<>();
        for (int i = 0; i < imgId.length; i++) {
            ImageView imageView = new ImageView(this);
            imageView.setScaleType(ImageView.ScaleType.CENTER_CROP);
            imageView.setImageResource(imgId[i]);
            mImageViews.add(imageView);
        }
        //设置引导页
        adapter = new GuideAdapter(mImageViews, this);
        vp_guide.setAdapter(adapter);
    }

    @Override
    public void processClick(View v) {
        switch (v.getId()) {
	        //按钮点击事件，跳转到主页面
            case R.id.bt_main:
                Intent intent = new Intent(GuideActivity.this, MainActivity.class);
                startActivity(intent);
                finish();
                break;
        }
    }

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        if (position == imgId.length - 1) {
            //最后一个，实现动画浮现
            bt_main.setVisibility(View.VISIBLE);
            AlphaAnimation aa = new AlphaAnimation(0, 1f);
            aa.setDuration(1000);
            bt_main.startAnimation(aa);
        } else {
            bt_main.setVisibility(View.GONE);
        }
    }

    @Override
    public void onPageSelected(int position) {

    }

    @Override
    public void onPageScrollStateChanged(int state) {

    }
}
```
细心的你可能也发现了该引导页是没有状态栏的，所以我们需要设置其主题为状态栏透明

```
<activity
    android:name=".Activity.GuideActivity"
    android:theme="@android:style/Theme.NoTitleBar.Fullscreen"/>
```
>特别注意：这里需要注意的是图片的大小问题，如果图片高清太大，可能会出现内存溢出的错误。

##**二、ViewPager实现带有动画的引导页**
带有动画的引导页编写步骤和传统是一模一样的，只不过给ViewPager设置一个动画。Google提供ViewPager.setPageTransformer(boolean reverseDrawingOrder, PageTransformer transformer)方法来设置引导页的切换效果，这里先看Google提供的切换Demo

![](http://img.blog.csdn.net/20161217005112875?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


###**步骤一：编写PageTransformer**
从上面效果看出，只是在引导页之间添加了一个动画而已，而Google提供的PageTransformer就可以对当前位置的引导页进行操作，比如：设置透明度的变化，设置缩放的变化，就能实现切换的动画效果

```
public class DepthPageTransformer implements ViewPager.PageTransformer {

    private static final float MIN_SCALE = 0.75f;

    @Override
    public void transformPage(View view, float position) {
        int pageWidth = view.getWidth();
        if (position < -1) {
            view.setAlpha(0);
        } else if (position <= 0) {
            view.setAlpha(1);
            view.setTranslationX(0);
            view.setScaleX(1);
            view.setScaleY(1);
        } else if (position <= 1) {
            view.setAlpha(1 - position);
            view.setTranslationX(pageWidth * -position);
            float scaleFactor = MIN_SCALE + (1 - MIN_SCALE) * (1 - Math.abs(position));
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
        } else {
            view.setAlpha(0);
        }
    }
}
```
###**步骤二：分析PageTransformer**
从上面的代码中，可以知道在ViewPager滑动的时候，会触发transformPage这个方法，并且会将当前的position和View传递过来，下面就是我们的对View的操作

**① position**

1. position < -1（即-无穷到-1）：让引导页消失，透明度为0
2. position <= 0（即-1到0）：让引导页出现
3. position <= 1（即0到1）：让引导页根据position做动画
4. 剩下else（即1到无穷）：让引导页消失，透明度为0

**② 图解position**

![](http://img.blog.csdn.net/20161217123804708?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>原谅我画图不好看，不生动，如果还不理解的话可以自己打印Log信息，把View和Position都打印出来帮助理解

###**步骤三：使用PageTransformer**
使用PageTransformer非常简单，只要通过ViewPager设置即可

```
vp_guide.setPageTransformer(true, new DepthPageTransformer());
```
##**三、其他动画的引导页的参考**
Google还为我们提供了另一个动画效果，看效果图

![](http://img.blog.csdn.net/20161217130632748?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

实现步骤其实和上面的步骤是一样的，具体我们来看PageTransformer的编写

```
public class ZoomOutPageTransformer implements ViewPager.PageTransformer {

    private static final float MIN_SCALE = 0.85f;
    private static final float MIN_ALPHA = 0.5f;

    @SuppressLint("NewApi")
    public void transformPage(View view, float position) {
        int pageWidth = view.getWidth();
        int pageHeight = view.getHeight();
        if (position < -1) {
            view.setAlpha(0);
        } else if (position <= 1) {
            float scaleFactor = Math.max(MIN_SCALE, 1 - Math.abs(position));
            float vertMargin = pageHeight * (1 - scaleFactor) / 2;
            float horzMargin = pageWidth * (1 - scaleFactor) / 2;
            if (position < 0) {
                view.setTranslationX(horzMargin - vertMargin / 2);
            } else {
                view.setTranslationX(-horzMargin + vertMargin / 2);
            }
            view.setScaleX(scaleFactor);
            view.setScaleY(scaleFactor);
            view.setAlpha(MIN_ALPHA + (scaleFactor - MIN_SCALE) / (1 - MIN_SCALE) * (1 - MIN_ALPHA));
        } else {
            view.setAlpha(0);
        }
    }
}
```
这里的原理就不分析了，和上面是一样的，只不过操作不同而已。除了Google提供的Demo之外，我们可以模仿谷歌的Demo，编写出我们自己的动画效果

```
public class RotateDownPageTransformer implements ViewPager.PageTransformer {

    private static final float ROT_MAX = 20.0f;
    private float mRot;

    public void transformPage(View view, float position) {
        if (position < -1) {
            ViewHelper.setRotation(view, 0);
        } else if (position <= 1) {
            //[-1,1]
            mRot = (ROT_MAX * position);
            ViewHelper.setPivotX(view, view.getMeasuredWidth() * 0.5f);
            ViewHelper.setPivotY(view, view.getMeasuredHeight());
            ViewHelper.setRotation(view, mRot);
        } else {
            ViewHelper.setRotation(view, 0);
        }
    }
}
```

效果如图

![](http://img.blog.csdn.net/20161217131224894?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

>好了，今天基础控件就到这里了，如果不懂的话可以自己实践一下，然后用纸笔思考思考，你就会有收获的。我也是通过博客学习别人的博客，然后通过自己的方式，将学习的内容写出来。我们一起加油，后来者们