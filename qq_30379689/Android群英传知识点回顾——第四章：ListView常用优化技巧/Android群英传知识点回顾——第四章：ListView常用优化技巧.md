# Android群英传知识点回顾——第四章：ListView常用优化技巧 #
----------
##知识点目录##
>* **4.1 ListView常用优化技巧**
	*  4.1.1 使用ViewHolder模式提高效率
	*  4.1.2 设置项目间分割线
	*  4.1.3 隐藏ListView的滚动条
	*  4.1.4 取消ListView的Item点击效果
	*  4.1.5 设置ListView需要显示在第几项
	*  4.1.6 动态修改ListView
	*  4.1.7 遍历ListView中的所有Item
	*  4.1.8 处理空ListView
	*  4.1.9 ListView滑动监听 
>* **4.2 ListView常用拓展**
	*  4.2.1 具有弹性的ListView
	*  4.2.2 自动显示、隐藏布局的ListView
	*  4.2.3 聊天ListView
	*  4.2.4 动态改变ListView布局


----------
##知识点回顾##

###**4.1 ListView常用优化技巧**
>无知识点

###4.1.1 使用ViewHolder模式提高效率
>ViewHolder模式充分利用了ListView的视图缓存机制，避免了每次在调用getView()的时候都去通过findViewById()实例化控件。据测试，使用ViewHolder将提高50%以上的效率

```
public class ViewHolderAdapter extends BaseAdapter{

    private List<String> mData;
    private LayoutInflater mInflater;

    public ViewHolderAdapter(Context context, List<String> mData) {
        this.mData = mData;
        mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return mData.size();
    }

    @Override
    public Object getItem(int position) {
        return mData.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder = null;
        //判断是否有缓存
        if (convertView == null) {
            holder = new ViewHolder();
            //通过LayoutInflater实例化布局
            convertView = mInflater.inflate(R.layout.viewholder_item, null);
            holder.img = (ImageView) convertView.findViewById(R.id.imageView);
            holder.title = (TextView) convertView.findViewById(R.id.textView);
            convertView.setTag(holder);
        } else {
            //通过tag找到缓存的布局
            holder = (ViewHolder) convertView.getTag();
        }
        //设置布局中控件要显示的视图
        holder.img.setBackgroundResource(R.mipmap.ic_launcher);
        holder.title.setText(mData.get(position));
        return convertView;
    }

    public final class ViewHolder {
        public ImageView img;
        public TextView title;
    }
}
```

###4.1.2 设置项目间分割线
>设置有颜色和有厚度的分割线
```
android:dividerHeight="10dp"
android:divider="@android:color/darker_gray"
```
>设置无分割线

```
android:divider="@null"
```

###4.1.3 隐藏ListView滚动条

```
android:scrollbars="none"
```

###4.1.4 取消ListView的Item点击效果

```
android:listSelector="@android:color/transparent"
```

###4.1.5 设置ListView需要显示在第几项
>自然的滑动到第几项
```
listview.setSelection(N);
```
>瞬间的滑动到第几项

```
listview.smoothScrollBy(distance,duration);
listview.smoothScrollByOffset(offset);
listview.smoothScrollToPosition(index);
```

###4.1.6 动态修改ListView
>当数据发生变化时，可以使用notifyDataSetChanged()来刷新ListView，但是必须保证使用这个方法传进Adapter的数据List是同一个List而不能是其他对象
```
mData.add("new");
mAdapter.notifyDataSetChanged();
```

###4.1.7 遍历ListView中的所有Item
```
for (int i = 0; i < mListview.getChildCount();i++){
     View view = mListview.getChildAt(i);
}
```
###4.1.8 处理空ListView
>在开发中，会遇到ListView为空的时候，比如：购物车在没有添加物品时，需要显示该购物车没有任何物品的View，这个时候也就是ListView数据为空的时候，ListView为我们提供好了方法
>
> 在存在ListView的FrameLayout中，添加一个ImageView，作为空ListView时显示：

```
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/listview"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

    <ImageView
        android:id="@+id/empty_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</FrameLayout>
```
>在Activity中实现ListView空数据时显示布局
```
ListView listView = (ListView)findViewById(R.id.listview);
listview.setEmptyView(findViewById(R.id.tv_null));
```
###4.1.9 ListView滑动监听
> onTouchListener：

>* MotionEvent.ACTION_DOWN：触摸时操作
>* MotionEvent.ACTION_MOVE：移动时操作
>* MotionEvent.ACTION_UP：离开时操作

```
mListview.setOnTouchListener(new View.OnTouchListener() {
    @Override
    public boolean onTouch(View view, MotionEvent motionEvent) {
        switch (motionEvent.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //触摸时操作
                break;
            case MotionEvent.ACTION_MOVE:
                //移动时操作
                break;
            case MotionEvent.ACTION_UP:
                //离开时操作
                break;
        }
        return false;
    }
});
```

> onScrollListener和onScroll：

>* OnScrollListener.SCROLL_STATE_IDLE：滑动停止时
>* OnScrollListener.SCROLL_STATE_TOUCH_SCROLL：正在滚动
>* OnScrollListener.SCROLL_STATE_FLING：手指抛动时

```
mListview.setOnScrollListener(new OnScrollListener() {
    @Override
    public void onScrollStateChanged(AbsListView view, int scrollState) {
                       switch (scrollState) {
            case OnScrollListener.SCROLL_STATE_IDLE:
                //滚动停止
                break;
            case OnScrollListener.SCROLL_STATE_TOUCH_SCROLL:
                //正在滚动
                break;
            case OnScrollListener.SCROLL_STATE_FLING:
                //手指抛动时
                break;
        }
    }

    @Override
    public void onScroll(AbsListView view, int firstVisibleItem, int visibleItemCount, int totalItemCount) {
        //滚动的时候一直在调用
    }
});
```
>onScroll参数：

>* firstVisibleItem：当前能看见的第一个Item的ID
>* visibleItemCount：当前能看见的Item总数
>* totalItemCount：整个ListView的Item总数

>利用onScroll方法的参数可以判断滚动到最后一行：

```
if(firstVisibleItem + visibleItemCount == totalItemCount && totalItemCount>0){
     //滚动到最后一行
}
```
>判断上滑和下滑：

```
if(firstVisibleItem > LastVisibleItemPosition){
    //上滑
}else if(firstVisibleItem < LastVisibleItemPosition){
    //下滑
}
LastVisibleItemPosition = firstVisibleItem;
```
>ListView也给我们提供封装好的方法获得当前可视的Item位置等信息：

```
//获取可视区域内最后一个item的id
mListview.getLastVisiblePosition();
//获取可视区域内第一个item的id
mListview.getFirstVisiblePosition();
```
###**4.2 ListView常用拓展**

###4.2.1 具有弹性的ListView
>* 弹性的ListView......见经典代码回顾案例一
###4.2.2 自动显示、隐藏布局的ListView
>* 自动显示、隐藏布局的ListView......见经典代码回顾案例二
###4.2.3 聊天ListView
>* 聊天ListView......见经典代码回顾案例三
###4.2.4 动态改变ListView布局
>* 动态改变ListView布局......见经典代码回顾案例四


----------
##经典代码回顾##
###**案例一：弹性的ListView**
>这个案例测试了好久，跟书本源码一样效果还是没出来，具体原因还不清楚

```
public class TanXingListView extends ListView {

    private int mMaxOverDistance;

    public TanXingListView(Context context, AttributeSet attrs) {
        super(context, attrs);
        DisplayMetrics metrics = context.getResources().getDisplayMetrics();
        float density = metrics.density;
        mMaxOverDistance = (int) (density * mMaxOverDistance);
    }

    @Override
    protected boolean overScrollBy(int deltaX, int deltaY, int scrollX, int scrollY, int scrollRangeX, int scrollRangeY, int maxOverScrollX, int maxOverScrollY, boolean isTouchEvent) {
        return super.overScrollBy(deltaX, deltaY, scrollX, scrollY, scrollRangeX, scrollRangeY, mMaxOverDistance, maxOverScrollY, isTouchEvent);
    }
}
```

----------
###**案例二：自动显示、隐藏布局的ListView**
>由于书本上的案例比较模糊，实现了很久才做出效果，可能与作者的实现方法大同小异，不过条条道路通罗马

```
public class ShowAndHideListViewActivity extends AppCompatActivity {

    private int mTouchSlop;
    private ObjectAnimator mAnimator;
    private Toolbar mToolBar;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_show_and_hide_list_view);
        //添加头部
        mToolBar = (Toolbar) findViewById(R.id.toolbar);
        setSupportActionBar(mToolBar);
        //填充布局
        ListView mListView = (ListView) findViewById(R.id.lv);
        String[] str = new String[]{"1", "2", "3", "4", "5", "6", "7", "8", "9"};
        mListView.setAdapter(new ArrayAdapter<String>(this, R.layout.list_item, R.id.tv, str));
        //添加头布局，防止第一条数据被遮盖
        View header = new View(this);
        header.setLayoutParams(new AbsListView.LayoutParams(AbsListView.LayoutParams.MATCH_PARENT,
                (int) getResources().getDimension(android.support.v7.
                        appcompat.R.dimen.abc_action_bar_default_height_material)));
        mListView.addHeaderView(header);
        //获得TouchSlop
        mTouchSlop = ViewConfiguration.get(this).getScaledTouchSlop();
        //设置监听
        mListView.setOnTouchListener(myTouchListener);
    }

    View.OnTouchListener myTouchListener = new View.OnTouchListener() {

        public boolean mShow;
        public int direction;
        public float mCurrentY;
        public float mFirstY;

        @Override
        public boolean onTouch(View v, MotionEvent event) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_DOWN:
                    mFirstY = event.getY();
                    break;
                case MotionEvent.ACTION_MOVE:
                    mCurrentY = event.getY();
                    if (mCurrentY - mFirstY > mTouchSlop) {
                        direction = 0;//down
                    } else if (mFirstY - mCurrentY > mTouchSlop) {
                        direction = 1;//up
                    }
                    if (direction == 1) {
                        if (mShow) {
                            toolBarAnim(0);//hide
                            mShow = !mShow;
                        }
                    } else if (direction == 0) {
                        if (!mShow) {
                            toolBarAnim(1);//up
                            mShow = !mShow;
                        }
                    }
                    break;
                case MotionEvent.ACTION_UP:

                    break;
            }
            return false;
        }
    };

    private void toolBarAnim(int flag) {
        if (mAnimator != null && mAnimator.isRunning()) {
            mAnimator.cancel();
        }
        if (flag == 0) {
            //up:hide
            mAnimator = ObjectAnimator.ofFloat(mToolBar, "translationY", -mToolBar.getHeight());
        } else {
            //down:show
            mAnimator = ObjectAnimator.ofFloat(mToolBar, "translationY", -mToolBar.getHeight(),0);
        }
        mAnimator.start();
    }

}
```
>布局文件的编写，记得将theme设置为NoActionBar

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"/>

    <ListView
        android:id="@+id/lv"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</RelativeLayout>
```
###**效果图**
![](http://img.blog.csdn.net/20161008220449720)

----------
###**案例三：聊天ListView**
>实现这个效果比较重要的步骤就是左右布局的填充

>左布局：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <ImageView
        android:id="@+id/left_icon"
        android:layout_width="50dp"
        android:layout_height="50dp" />

    <TextView
        android:id="@+id/tv_left"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="30dp" />

</LinearLayout>
```
>右布局：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <TextView
        android:id="@+id/tv_right"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:padding="30dp" />

    <ImageView
        android:id="@+id/right_icon"
        android:layout_width="50dp"
        android:layout_height="50dp" />
</LinearLayout>

```
>聊天的实体类

```
public class ChatItemListViewBean {

    private int type;
    private String text;
    private Bitmap icon;

    public int getType() {
        return type;
    }

    public void setType(int type) {
        this.type = type;
    }

    public String getText() {
        return text;
    }

    public void setText(String text) {
        this.text = text;
    }

    public Bitmap getIcon() {
        return icon;
    }

    public void setIcon(Bitmap icon) {
        this.icon = icon;
    }
}
```
>比较关键的就是Adapter的getItemViewType()方法和getViewTypeCount()方法

```
public class ChatListViewAdapter extends BaseAdapter {

    private List<ChatItemListViewBean> mData;
    private LayoutInflater mInflater;

    public ChatListViewAdapter(Context context, List<ChatItemListViewBean> mData) {
        this.mData = mData;
        mInflater = LayoutInflater.from(context);
    }

    @Override
    public int getCount() {
        return mData.size();
    }

    @Override
    public Object getItem(int position) {
        return mData.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder;
        if (convertView == null) {
            if (getItemViewType(position) == 0) {
                viewHolder = new ViewHolder();
                convertView = mInflater.inflate(R.layout.left_item, null);
                viewHolder.icon = (ImageView) convertView.findViewById(R.id.left_icon);
                viewHolder.text = (TextView) convertView.findViewById(R.id.tv_left);
            } else {
                viewHolder = new ViewHolder();
                convertView = mInflater.inflate(R.layout.right_item, null);
                viewHolder.icon = (ImageView) convertView.findViewById(R.id.right_icon);
                viewHolder.text = (TextView) convertView.findViewById(R.id.tv_right);
            }
            convertView.setTag(viewHolder);
        } else {
            viewHolder = (ViewHolder) convertView.getTag();
        }

        viewHolder.icon.setImageBitmap(mData.get(position).getIcon());
        viewHolder.text.setText(mData.get(position).getText());

        return convertView;
    }

    @Override
    public int getItemViewType(int position) {
        ChatItemListViewBean bean = mData.get(position);
        return bean.getType();
    }

    @Override
    public int getViewTypeCount() {
        return 2;
    }

    public final class ViewHolder {
        public ImageView icon;
        public TextView text;
    }
}
```
>在主Activity中实现我们的效果

```
public class ChatListViewActivity extends AppCompatActivity {

    private ListView mListView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_chat_list_view);

        mListView = (ListView) findViewById(R.id.lv_chat);
        ChatItemListViewBean bean1 = new ChatItemListViewBean();
        bean1.setType(0);
        bean1.setIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        bean1.setText("hello how are you?");

        ChatItemListViewBean bean2 = new ChatItemListViewBean();
        bean2.setType(1);
        bean2.setIcon(BitmapFactory.decodeResource(getResources(), R.mipmap.ic_launcher));
        bean2.setText("find thank you ,and you?");

        List<ChatItemListViewBean> data = new ArrayList<>();
        data.add(bean1);
        data.add(bean2);

        ChatListViewAdapter adapter = new ChatListViewAdapter(this, data);
        mListView.setAdapter(adapter);
    }
}
```

###**效果图**
![](http://img.blog.csdn.net/20161008223127051)

----------
###**案例四：动态改变ListView布局**

```
public class DongTaiListViewActivity extends AppCompatActivity {

    private DongTaiListViewAdapter adapter;
    private List<String> list;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dong_tai_list_view);

        ListView listView = (ListView) findViewById(R.id.lv_dongtai);
        list = new ArrayList<>();
        list.add("item1");
        list.add("item2");
        list.add("item3");
        list.add("item4");
        adapter = new DongTaiListViewAdapter(this, list);
        listView.setAdapter(adapter);
        listView.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                adapter.setCurrentItem(position);
                adapter.notifyDataSetChanged();
            }
        });
    }
}
```

```
public class DongTaiListViewAdapter extends BaseAdapter {

    private List<String> list;
    private Context mContext;
    private int mCurrentItem=0;

    public DongTaiListViewAdapter(Context context, List<String> list) {
        this.list = list;
        this.mContext = context;
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Override
    public Object getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.VERTICAL);
        if (mCurrentItem == position) {
            layout.addView(addFocusView(position));
        } else {
            layout.addView(addNormalView(position));
        }
        return layout;
    }

    private View addFocusView(int i) {
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.mipmap.ic_launcher);
        return iv;
    }

    private View addNormalView(int i) {
        LinearLayout layout = new LinearLayout(mContext);
        layout.setOrientation(LinearLayout.HORIZONTAL);
        ImageView iv = new ImageView(mContext);
        iv.setImageResource(R.mipmap.ic_launcher);
        layout.addView(iv, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT));
        TextView tv = new TextView(mContext);
        tv.setText(list.get(i));
        layout.addView(tv, new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT));
        layout.setGravity(Gravity.CENTER);
        return layout;
    }

    public void setCurrentItem(int position) {
        mCurrentItem = position;
    }
}
```
###**效果图**
![](http://img.blog.csdn.net/20161008225057458)

>经典回顾源码下载

>github：https://github.com/CSDNHensen/QunYingZhuang