#Android基础控件——SwipeRefreshLayout最简单的下拉刷新


----------
还在使用传统的下拉刷新，觉得不够漂亮，怕被产品经理骂吗？

![这里写图片描述](http://img.blog.csdn.net/20161127173636096)

还在忧愁自己技术不够好，不会改造带动画的下拉刷新吗？

![这里写图片描述](http://img.blog.csdn.net/20161127173719237)

那么不要担心，使用SwipeRefreshLayout最简单的下拉刷新，既不失美观又简洁

![](http://img.blog.csdn.net/20161127174035316)

SwipeRefreshLayout下拉刷新是Google自家的下拉刷新控件，使用过程跟开源库PullToRefresh差不多，废话不多说，开车啦

##**一、创建布局**
SwipeRefreshLayout实质上是一个ViewGroup，所以我们将其作为我们的根布局进行演示

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v4.widget.SwipeRefreshLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/refresh"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <!--数据View-->
    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</android.support.v4.widget.SwipeRefreshLayout>
```
经过这个步骤之后，其实在页面上就已经能够下拉看到组件

##**二、代码编写**

```
public class HomeActivity extends AppCompatActivity implements SwipeRefreshLayout.OnRefreshListener {

    private static final int REFRESH_COMPLETE = 0x01;
    private SwipeRefreshLayout refreshLayout;

    private Handler mHandler = new Handler() {
        public void handleMessage(android.os.Message msg) {
            switch (msg.what) {
                case REFRESH_COMPLETE:
                    refreshLayout.setRefreshing(false);
                    break;
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_home);

        refreshLayout = (SwipeRefreshLayout) findViewById(R.id.refresh);
        refreshLayout.setColorSchemeResources(android.R.color.holo_blue_light,
                android.R.color.holo_red_light, android.R.color.holo_orange_light);
        refreshLayout.setSize(SwipeRefreshLayout.LARGE);
        refreshLayout.setOnRefreshListener(this);
    }
	
	/**
     * 当刷新的时候进行回调
     */
    @Override
    public void onRefresh() {
        //在这里执行操作的更新等操作
        ...
        //刷新成功
        mHandler.sendEmptyMessageDelayed(REFRESH_COMPLETE, 2000);
    }
}
```
SwipeRefreshLayout方法介绍

* setColorSchemeResources(int... args)：设置刷新时圆圈的颜色变化，为int数组
* setSize(int size)：设置刷新时圆圈的大小，有DEFAULT和LARGE两个值，默认是DEFAULT
* setOnRefreshListener(OnRefreshListener listener)：设置刷新时回调监听事件，刷新时调用
* setRefreshing(boolean refreshing)：设置是否继续正在刷新

>好了，今天就讲这么多了，如果对基础控件有兴趣的同学，可以关注我的博客的基础控件篇