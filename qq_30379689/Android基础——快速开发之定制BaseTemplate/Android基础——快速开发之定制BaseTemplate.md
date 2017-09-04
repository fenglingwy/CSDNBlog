#Android基础——快速开发之定制BaseTemplate


----------
>本篇内容有：

>* 定制BaseActivity
* 定制BaseFragment
* 定制BaseApplication

##**前言**

初学者肯定会遇到一个日常任务，那么就是findViewById，setOnClickListener（暂且把它们称为日常任务），而且很多人会把他们混在一起，导致项目结构混乱，最主要的是写多了会烦，不觉得吗？当项目的Activity越多时，每次添加控件都要重新写一次，想想都累

```
tv_cz_10 = (TextView) findViewById(R.id.tv_cz_10);
tv_cz_20 = (TextView) findViewById(R.id.tv_cz_20);
tv_cz_30 = (TextView) findViewById(R.id.tv_cz_30);
tv_cz_50 = (TextView) findViewById(R.id.tv_cz_50);
tv_cz_10.setOnClickListener(this);
tv_cz_20.setOnClickListener(this);
tv_cz_30.setOnClickListener(this);
tv_cz_50.setOnClickListener(this);
```

* 定制解决的问题：尽量写少的代码，做更多事
* 定制的目的：理清代码结构，让你编程更有逻辑性
* 定制的内容：一切都是根据项目的需求去实现

##**定制BaseActivity**

我们就针对日常任务简单的定制一份我们的BaseActivity
```
public abstract class BaseActivity extends FragmentActivity implements View.OnClickListener {

    private SparseArray<View> mViews;

    public abstract int getLayoutId();

    public abstract void initViews();

    public abstract void initListener();

    public abstract void initData();

    public abstract void processClick(View v);

    public void onClick(View v) {
        processClick(v);
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mViews = new SparseArray<>();
        setContentView(getLayoutId());
        initViews();
        initListener();
        initData();
    }
    
	/**
     * 通过ID找到View
     */
    public <E extends View> E findView(int viewId) {
        E view = (E) mViews.get(viewId);
        if (view == null) {
            view = (E) findViewById(viewId);
            mViews.put(viewId, view);
        }
        return view;
    }
    
	/**
     * View设置OnClick事件
     */
    public  <E extends View> void setOnClick(E view){
        view.setOnClickListener(this);
    }
}
```
代码其实很简单，光从代码可能不知道这段代码的意思，那么就从实现这段代码来理解它的真正作用，下面是实现BaseActivity的代码

```
public class SearchActivity extends BaseActivity {

	BannerController bannerController;
    ShopController shopController;
    
    private ImageView iv_zxing;
    private TextView tv_sure;

    @Override
    public int getLayoutId() {
	    //这里用来获取Activity的layout
        return R.layout.activity_search;
    }

    @Override
    public void initViews() {
	    //这里用来初始化View
        iv_zxing = findView(R.id.iv_zxing);
        tv_sure = findView(R.id.tv_sure);
    }

    @Override
    public void initListener() {
	    //这里用来初始化点击事件
        setOnClick(iv_zxing);
        setOnClick(tv_sure);
    }

    @Override
    public void initData() {
		//这里用来设置数据、获取数据、读取网络数据、这里所做的一切都可以在Controller实现
		bannerController = new BannerController(getActivity());
        shopController = new ShopController(getActivity());

        initShop();
        initBanner();
    }


    @Override
    public void processClick(View v) {
	    //这里用来处理点击事件
        switch (v.getId()) {
            case R.id.iv_zxing:
                
                break;
            case R.id.tv_sure:
                
                break;
        }
    }

    private void initShop() {

    }
    
    private void initBanner() {

    }
}
```

是不是觉得代码结构很清晰，而且比起之前的日常任务来说，代码确实少了不少，各个方法都放着自己应该做的事情，这样能保证你在编程的时候逻辑不会出错，让别人读起来也很轻松，当然，除了常用的setOnClickListener还有setOnItemClickListener，这就需要根据项目需要而定制

如果你是很酷很有性格的人，那么也可以尝试下面这种用法，用一个字母作为方法，一切定制因你心情而定
```
public <E extends View> E F(int viewId) {
    E view = (E) mViews.get(viewId);
    if (view == null) {
        view = (E) findViewById(viewId);
        mViews.put(viewId, view);
    }
    return view;
}

public  <E extends View> void C(E view){
    view.setOnClickListener(this);
}

//用起来也很帅哦
@Override
public void initViews() {
    iv_zxing = F(R.id.iv_zxing);
    tv_sure = F(R.id.tv_sure);
}

@Override
public void initListener() {
    C(iv_zxing);
    C(tv_sure);
}
```


##**定制BaseFragment**

介绍完了Activity，那么Fragment就很简单了，可以模仿Activity实现，如果和上面的一模一样那么就没有乐趣了，这里由于个人项目原因，我把Fragment默认设置成了懒加载模式，并且只加载一次数据

```
public abstract class BaseFragment extends Fragment implements View.OnClickListener {

    private boolean isVisible = false;
    private boolean isInitView = false;
    private boolean isFirstLoad = true;

    public View convertView;
    private SparseArray<View> mViews;

    public abstract int getLayoutId();

    public abstract void initViews();

    public abstract void initListener();

    public abstract void initData();

    public abstract void processClick(View v);

    @Override
    public void onClick(View v) {
        processClick(v);
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (isVisibleToUser) {
            isVisible = true;
            lazyLoad();
        } else {
	        //设置已经不是可见的
            isVisible = false;
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        mViews = new SparseArray<>();
        convertView = inflater.inflate(getLayoutId(), container, false);
        initViews();
        
        isInitView = true;
        lazyLoad();
        return convertView;
    }
	
	//懒加载
    private void lazyLoad() {
        if (!isFirstLoad || !isVisible || !isInitView) {
            //如果不是第一次加载、不是可见的、不是初始化View，则不加载数据
            return;
        }
        //加载数据
        initListener();
        initData();
		//设置已经不是第一次加载
        isFirstLoad = false;
    }

    public <E extends View> E findView(int viewId) {
        if (convertView != null) {
            E view = (E) mViews.get(viewId);
            if (view == null) {
                view = (E) convertView.findViewById(viewId);
                mViews.put(viewId, view);
            }
            return view;
        }
        return null;
    }

    public  <E extends View> void setOnClick(E view){
        view.setOnClickListener(this);
    }
}
```
这里和Activity最大的区别

1. convertView：由于Fragment的findID需要convertView，我们只好抽取出来
2. setUserVisibleHint：这个方法当切换Fragment时会调用，会返回当前Fragment是否用户可见

```
public class HomeFragment extends BaseFragment {

    ShopController shopController;

    private ImageView iv_speech;
    private TextView tv_search;


    @Override
    public int getLayoutId() {
        return R.layout.fragment_home;
    }

    @Override
    public void initViews() { 
        iv_speech = findView(R.id.iv_speech);
        tv_search = findView(R.id.tv_search);
    }

    @Override
    public void initData() {
        shopController = new ShopController(getActivity());

        initShop();
    }

    @Override
    public void initListener() {
        setOnClick(iv_speech);
        setOnClick(tv_search);
    }

    @Override
    public void processClick(View v) {
        switch (v.getId()) {
            case R.id.iv_speech:
               
                break;
            case R.id.tv_search:
               
                break;
        }
    }

    private void initShop() {
        
    }
}
```
这里采用的是ViewPager+Fragment，如果需要让Fragment进行缓存，那么必须对ViewPager进行缓存设置

```
//设置缓存页面
viewPager.setOffscreenPageLimit(5);
```

下面是设置了缓存的懒加载模式的效果图，可以看到第一次切换需要加载数据，而第二次切回去则界面不会变化，效果和手机淘宝一样

![](http://img.blog.csdn.net/20170227133452653?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**定制BaseApplication**

定制BaseApplication那就简单了，在Application中经常用到的就是第三方的设置、数据库的加载，具体可以根据项目需求进行定制

```
public abstract class BaseApplication extends Application {

    public abstract void initConfigs();

    @Override
    public void onCreate() {
        super.onCreate();
        initConfigs();
    }

}
```

##**结语**
学习完之后，建议大家将BaseTemplate用到你们的项目中，当然从中也要学习抽象方法，抽取常用的方法，比如：在加载数据的时候可以抽取BaseController，在Adapter中可以抽取通用的BaseAdapter，具体还需要大家去研究


[代码下载](http://download.csdn.net/detail/qq_30379689/9765138)