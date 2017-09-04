#TabLayout的使用、仿爱奇艺导航条


----------
>学习，学习，学以致用，让基础控件贴近实战效果

TabLayout是Google新推出的Material Design的控件之一，TabLayout的使用必须结合ViewPager和Fragment的使用，如果对ViewPager不熟悉的同学，请自行查阅资料，很简单的。我们来看下爱奇艺导航条的原效果

![](http://img.blog.csdn.net/20161228191424555?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

我们今天要实现的效果图，在真机上运行效果会更接近原图

![](http://img.blog.csdn.net/20161228192605446?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**一、集成TabLayout**
由于TabLayout是Material Design中的控件之一，集成它需要在Gradle文件中添加依赖库，由于我的compileSdkVersion为24，所以采用24的版本就不会提示错误

```
compile 'com.android.support:design:24.0.0'
```

>我不知道是我电脑项目中东西太多，还是Material Design本来就很大的库，编译的时候花了8分钟

##**二、实现布局**
TabLayout需要和ViewPager一起使用，所以在TabLayout下面放置一个ViewPager
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:background="#fff"
    android:layout_height="match_parent"
    android:orientation="vertical">
	
    <android.support.design.widget.TabLayout
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:id="@+id/tl"
        android:background="#eee"
        android:layout_width="match_parent"
        android:layout_height="45dp"
        app:tabIndicatorHeight="1dp"
        app:tabGravity="center"
        app:tabMode="scrollable"
        app:tabIndicatorColor="#24C026"
        app:tabSelectedTextColor="#24C026"
        app:tabTextColor="#000000"/>

    <android.support.v4.view.ViewPager
        android:id="@+id/vp"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
由于使用了TabLayout的自定义属性，所以记得导入资源

```
xmlns:app="http://schemas.android.com/apk/res-auto"
```
这里对上面TabLayout的参数进行介绍

* tabIndicatorHeight：Tab指示器下标的高度
* tabGravity：Tab内容的显示模式
* tabMode：Tab的展示模式
	* fixed（默认）：固定的，标签很多时候会被挤压，不能滑动
	* scrollable：可滚动的，标签多的时候可滚动
* tabIndicatorColor：Tab指示器下标的颜色
* tabSelectedTextColor：Tab文字被选中的颜色
* tabTextColor：Tab文字的颜色

>其实这些属性都可以在代码中设置，不过为了代码的阅读性和美观，所以将属性设置都在布局文件中实现

##**三、实现代码**
找到对应的控件，并添加对应的Tab和Fragment
```
public class MainActivity extends AppCompatActivity {

    private TabLayout tl;
    private ViewPager vp;

    private FragmentAdapter adapter;
    private List<Fragment> fragments;
    private List<String> strings;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tl = (TabLayout) findViewById(R.id.tl);
        vp = (ViewPager) findViewById(R.id.vp);
		
		//添加Fragment
        fragments = new ArrayList<>();
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());
        fragments.add(new FragmentOne());

		//添加Tab文字
        strings = new ArrayList<>();
        strings.add("推荐");
        strings.add("爱上超模");
        strings.add("电视剧");
        strings.add("电影");
        strings.add("综艺");
        strings.add("动漫");
        strings.add("订阅");
        strings.add("北京");
        strings.add("资讯");
        strings.add("娱乐");
        strings.add("搞笑");
        strings.add("儿童");
        strings.add("原创");
		
		//添加Tab
        for (String str : strings) {
            tl.addTab(tl.newTab().setText(str));
        }

		//绑定ViewPager
        adapter = new FragmentAdapter(getSupportFragmentManager(), fragments, strings);
        vp.setAdapter(adapter);
        tl.setupWithViewPager(vp);
    }
}
```
可以发现，创建Tab都是通过声明的该TabLayout生成的，这里我们采用的是高级for循环遍历

```
for (String str : strings) {
    tl.addTab(tl.newTab().setText(str));
}
```
然后通过TabLayout的setupWithViewPager()方法绑定一个ViewPager，记得ViewPager是要有东西的，所以需要一个Adapter，ViewPager的使用和ListView大同小异，如果对ViewPager不懂的同学，请先学习ViewPager后再来理解

```
adapter = new FragmentAdapter(getSupportFragmentManager(), fragments, strings);
vp.setAdapter(adapter);
tl.setupWithViewPager(vp);
```
下面我们可以创建很多个不同的Fragment来跟我们的标签绑定，这里为了方便演示，所以只创建同一个Fragment

```
public class FragmentOne extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_one, null);
        return view;
    }
}
```
##**四、实现Adapter**

```
public class FragmentAdapter extends FragmentPagerAdapter {

    private List<Fragment> fragments;
    private List<String> strings;

    public FragmentAdapter(FragmentManager fm, List<Fragment> fragments ,List<String> strings) {
        super(fm);
        this.fragments = fragments;
        this.strings = strings;
    }

	//返回Tab对应的Fragment
    @Override
    public Fragment getItem(int position) {
        return fragments.get(position);
    }

	//返回Tab数目
    @Override
    public int getCount() {
        return strings.size();
    }
	
	//返回Tab文字
    @Override
    public CharSequence getPageTitle(int position) {
        return strings.get(position);
    }

}

```
这里有个方法需要注意：需要重写getPageTitle()方法来为TabLayout的Tab添加上文字，否则会显示不出来文字，这也是为什么我们需要在构造方法中传进来List< String> strings，其他的与ListView的Adapter大同小异

[部分源码下载](http://download.csdn.net/detail/qq_30379689/9723882)

>到这里的课程就结束，如果对基础控件感兴趣的朋友可以关注我博客的基础控件系列