#Android开发细节——开发实战过程中遇到的细节问题与解决方案汇总


----------


>这是我一篇旧文翻成MarkDown格式的文章，原文时间是2016-09-05 18:21，这里都是做项目中遇到的问题，经过查询资料后，把结果告诉大家

##一、获取系统时间的24小时制与12小时制

最近在做项目的时候发生了一点错误，服务器端是24小时制的时间，而本地数据库则是12小时制的时间

1、获取24小时制的时间

```
public static String showDate() {
	SimpleDateFormat sDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
	String date = sDateFormat.format(new Date());
	return date;
}
```
2、获取12小时制的时间

```
public static String showDate() {
    SimpleDateFormat sDateFormat = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    String date = sDateFormat.format(new Date());
    return date;
}
```

两者区别就在于HH和hh，可以看下文档的其他示例

![](http://img.blog.csdn.net/20170607120734198?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##二、ViewPager中的Fragment在切换时不重新加载

最近在做项目的时候发生了一点错误，在ViewPager中的Fragment中切换到第三页重新切回第一页时，会重新加载第一页，显得页面一直在加载，降低了用户体验，因为viewPager会事先加载好当前页的前后两页，也就是到了第三页的时候，第一页已经被销毁了，回到第二页的时候会重新创建

解决方法

```
//默认是1
mViewPager.setOffscreenPageLimit(3);  
```

##三、优化购物车，选中物品时价钱相加减的精确运算

做到商品购物车模块的时候，发现价钱的加减并不能单纯的使用+、-来实现，由于我们的价格都是double类型的，如10.24元，相互加减的时候会出现20.45555555的情况，所以我们需要使用到API中BigDecimal这个类进行包装，然后运算

代码如下：
```
public void selectSingle() {
    //创建BigDecimal对象
    BigDecimal bj1 = new BigDecimal(Double.toString(money1));
    BigDecimal bj2 = new BigDecimal(Double.toString(money2));
    if (selected_Id.contains(shop.get_id())) {//相减
        sum_money = bj1.subtract(bj2).doubleValue();
    } else {//相加
        sum_money = bj1.add(bj2).doubleValue();
    }
}
```
效果如下：可以看到价钱已经是正常的加减了

![这里写图片描述](http://img.blog.csdn.net/20170607121414628?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##四、应用按Home键返回桌面，再次点击应用不能恢复回离开时的Activity

只要该栈中之前的任何一个Activity在manifest文件中定义了启动模式为singleTask，那么再次点击应用时会启动第一个Activity，只要去除之前Activity中singleTask属性就能恢复回离开时的Activity

##五、通过广播监听网络的变化清况

通常在使用软件的时候会出现网路变化的情况，比如Wifi断线导致使用流量上网，这个时候作为我们的软件就必须通知用户在使用流量上网。首先，Manifests中注册网络变化情况的广播

```
<!-- 广播 -->
<receiver android:name=".Receiver.NetReceiver">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
    </intent-filter>
</receiver>
```
接着，创建该Receiver，根据网络的变化情况进行相对应的提醒。这里当用户打开或关闭Wifi和移动数据时，该广播可以收到

```
public class NetReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
        //网络广播接收者
        if (intent.getAction().equals(ConnectivityManager.CONNECTIVITY_ACTION)) {
            ConnectivityManager connectivityManager = (ConnectivityManager)
                    context.getSystemService(Context.CONNECTIVITY_SERVICE);
            NetworkInfo activeInfo = connectivityManager.getActiveNetworkInfo();
            NetworkInfo netInfo = connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_MOBILE);
            NetworkInfo wifiInfo = connectivityManager.getNetworkInfo(ConnectivityManager.TYPE_WIFI);
            if (activeInfo != null) {
                //网络可用
                if (activeInfo.getState().equals(NetworkInfo.State.CONNECTED)) {
                    //判断移动数据
                    if (netInfo.getState().equals(NetworkInfo.State.CONNECTED)) {
                        Toast.makeText(context, "您正在使用移动数据", Toast.LENGTH_SHORT).show();
                    }
                    //判斷Wifi數據
                    if (wifiInfo.getState().equals(NetworkInfo.State.CONNECTED)) {
                        Toast.makeText(context, "您正在使用Wifi数据", Toast.LENGTH_SHORT).show();
                    }
                } else {
                    Toast.makeText(context, "请检查网络是否已联网", Toast.LENGTH_SHORT).show();
                }
            }
        }
    }
}
```

##六、优化Gradle，编译时间从33.8秒降到4.5秒

在项目的gradle中添加以下语句

```
 tasks.whenTaskAdded { task ->
    if (task.name.contains("lint")
            || task.name.equals("clean")
            || task.name.contains("Aidl")
            || task.name.contains("mockableAndroidJar")
            || task.name.contains("UnitTest")
            || task.name.contains("AndroidTest")
            || task.name.contains("Ndk")
            || task.name.contains("Jni")
    ) {
        task.enabled = false;
    }
}
```

##七、Fragment的懒加载基类

在项目中添加该基类，新的Fragment继承该BaseFragment就可以轻松实现Fragment的懒加载

```
public abstract class BaseFragment extends Fragment implements View.OnClickListener {

    private boolean isPrepared;
    private boolean isVisible;

    public abstract View initViews(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState);

    public abstract void initData();

    public abstract void initListener();

    public abstract void processClick(View v);

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (getUserVisibleHint()) {
            isVisible = true;
            lazyLoad();
        } else {
            isVisible = false;
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return initViews(inflater, container, savedInstanceState);
    }

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        isPrepared = true;
        lazyLoad();
    }

    /**
     * 懒加载
     */
    private void lazyLoad() {
        if (!isVisible || !isPrepared) {
            return;
        }
        //加载数据
        initData();
        initListener();
    }


    @Override
    public void onClick(View v) {
        processClick(v);
    }
}
```

##八、优化错误，不让错误导致程序崩溃

在项目中，常常会因为list.get(0)获取没有数据而导致程序崩溃，这个时候应该把程序try-catch起来，让程序报错但不崩溃，比如

```
public Drawable getDrawable(String name) {
    try {
        return mResources.getDrawable(mResources.getIdentifier(name, "drawable", mPkgName));
    } catch (Exception e) {
        e.printStackTrace();
        return null;
    }
}
```

##九、Context的应用场景

![这里写图片描述](http://img.blog.csdn.net/20170607121723742?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##十、解决PhotoView在ViewPager中多点触摸时，报错崩溃的方法

使用PhotoView在ViewPager中展示出多图操作，如果操作过于频繁，那么下面这个错误

```
java.lang.IllegalArgumentException: pointerIndex out of range  
```

其解决方法就是在PhotoView所在的Activity中添加下面的处理即可

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    try {
        return super.dispatchTouchEvent(ev);
    } catch (IllegalArgumentException ex) {
        ex.printStackTrace();
    }
    return false;
}
```

##十一、使用Html.fromhtml显示图片

在项目开发中，后台常常会使用这样的编辑器来编辑图文

![这里写图片描述](http://img.blog.csdn.net/20170607122005474?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



返回在android端是一串html字符串，这个时候单单靠Html.fromhtml是显示不出来图片的，图片将会被小方格替代，下面是我已经处理好的一个类，该类只要Html.fromhtml识别到图片就会回调该类的getDrawable()方法。该类兼容网络图片和服务器图片，兼容以下两种格式

服务器图片url：/upload/2017/05/04/590aaae1b0d4a.png
网络图片url：https://p.ssl.qhimg.com/t0140ba2595bb1b8c4a.png

```
/**
 * @author 许英俊 2017/6/7
 */
public class ImageGetterImpl implements Html.ImageGetter {

    private int width, height;
    private TextView tv;
    private String html;
    private File file;

    public ImageGetterImpl(TextView tv, String html, int width, int height) {
        this.tv = tv;
        this.html = html;
        this.width = width - 80;
        this.height = height;
    }

    @Override
    public Drawable getDrawable(String source) {
        Drawable drawable = null;
        // 区分网络图片和自己服务器图片，如果是服务器图片，则加个根目录
        if (!source.startsWith("http")) {
            source = RequestCenter.ROOT_URL + source;
        }
        //获取图片后缀名作为文件名
        String[] fileName = source.split("/");
        file = new File(Environment.getExternalStorageDirectory(), fileName[fileName.length - 1]);
        // 判断是否以http开头
        if (source.startsWith("http")) {
            // 判断路径是否存在
            if (file.exists()) {
                // 存在即获取drawable
                drawable = Drawable.createFromPath(file.getAbsolutePath());
                // 根据屏幕的宽高比等于图片的宽高比
                height = (width) * drawable.getIntrinsicHeight() / drawable.getIntrinsicWidth();
                drawable.setBounds(0, 0, width, height);
            } else {
                // 不存在即开启异步任务加载网络图片
                AsyncLoadNetworkPic networkPic = new AsyncLoadNetworkPic();
                networkPic.execute(source);
            }
        }
        return drawable;
    }
    
    public class AsyncLoadNetworkPic extends AsyncTask<String, Integer, Void> {
        @Override
        protected Void doInBackground(String... params) {
            // 加载网络图片
            loadNetPic(params);
            return null;
        }
        @Override
        protected void onPostExecute(Void result) {
            super.onPostExecute(result);
            // 当执行完成后再次为其设置一次
            tv.setText(Html.fromHtml(html, new ImageGetterImpl(tv, html, width, height), null));
        }
        /**
         * 加载网络图片
         */
        private void loadNetPic(String... params) {
            String path = params[0];
            InputStream in = null;
            FileOutputStream out = null;
            try {
                URL url = new URL(path);
                HttpURLConnection connUrl = (HttpURLConnection) url.openConnection();
                connUrl.setConnectTimeout(10000);
                connUrl.setRequestMethod("GET");
                if (connUrl.getResponseCode() == 200) {
                    in = connUrl.getInputStream();
                    out = new FileOutputStream(file);
                    byte[] buffer = new byte[1024];
                    int len;
                    while ((len = in.read(buffer)) != -1) {
                        out.write(buffer, 0, len);
                    }
                } else {
                    Log.e("TAG", connUrl.getResponseCode() + "");
                }
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (in != null) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (out != null) {
                    try {
                        out.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```

对应的在安卓端使用三个参数的方法进行解析html字符串，其中DensityUtils两个方法表示获取屏幕的宽和高

```
tv_content.setText(Html.fromHtml(data.getContent(),
                new ImageGetterImpl(tv_content, data.getContent(),
                        DensityUtils.getDisplayWidth(this),
                        DensityUtils.getDisplayHeight(this)), null));
```

在手机的显示效果如下，中间隔那么多行请忽略

![这里写图片描述](http://img.blog.csdn.net/20170607122248367?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)