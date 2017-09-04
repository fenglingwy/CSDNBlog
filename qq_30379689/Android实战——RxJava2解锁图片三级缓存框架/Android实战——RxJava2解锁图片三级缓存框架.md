#RxJava2解锁图片三级缓存框架


----------
>本篇文章包括以下内容

>* 前言
>* 图片三级缓存的介绍
>* 框架结构目录的介绍
>* 构建项目整体框架
>* 实现图片三级缓存
>* 演示效果
>* 源码下载
>* 结语

##**前言**
RxJava2作为如今那么主流的技术，不学习学习都不行了，本篇文章需要你有RxJava2的基础，如果需要对RxJava2学习的同学，可以关注我的博客，查看[Android实战——RxJava2+Retrofit+RxBinding解锁各种新姿势](http://blog.csdn.net/qq_30379689/article/details/68958173)  。项目代码实现模仿Picasso，大伙们可以看下最后的代码效果，那么废话不多说，Hensen老师开车啦

```
RxImageLoader.with(TextImageLoaderActivity.this).load(url).into(iv);
```

##**图片三级缓存的介绍**

图片的三级缓存很多同学可能已经掌握了，很多同学可能也听说过，那么这里就简单的来回顾一下你们学习的三级缓存机制是否正确吧

![这里写图片描述](http://img.blog.csdn.net/20170407120114608?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

首先，这个图就是表示三级缓存机制的所有，其三级分别是（按先后顺序）

1. 内存缓存（一级）：如果内存存在我们的缓存信息，直接用它
2. 文件缓存（二级）：如果内存不存在我们的缓存信息，那么就查看是否有我们的缓存文件，如果有，直接使用它。同时，将其缓存到内存中
3. 网络缓存（三级）：如果文件不存在缓存文件，直接从网络上下载，直接使用它。同时，将其缓存到文件和内存中

那么我们在框架中我们需要做的事情有哪些呢？

1. 内存缓存：采用Google自带的LruCache进行缓存
2. 文件缓存：采用Github上没有被Google收录却被Google认证的DiskLruCache
3. 网络缓存：通过io流的Stream进行文件的读写

如果对于LruCache和DiskCache不懂的同学，可以查看郭霖大神的博客，里面有很详细的讲解，Lru指的是近期最少使用算法


##**框架结构目录的介绍**

框架的结构看似复杂，其实内容不多，实现起来也不难

![这里写图片描述](http://img.blog.csdn.net/20170407121645897?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面对框架结构目录进行介绍，图片上显示了目录结构之间的关系

![这里写图片描述](http://img.blog.csdn.net/20170407121616123?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1. TextImageLoaderActivity：是我们的测试界面
2. ImageBean：使用RxJava的onNext传递的Bean对象
3. CacheObservable：三级缓存的父类
4. DiskCacheObservable：文件缓存Observable
5. MemoryCacheObservable：内存缓存Observable
6. NetworkCacheObservable：网络缓存Observable
7. RequestCreator：对三级缓存进行统一管理
8. RxImageLoader：使用RequestCreator管理类进行缓存机制的调用
9. DiskCacheUtils：文件缓存的工具类

##**构建项目整体框架**

1、准备工作

导入我们需要的依赖库

```
//rxjava
compile "io.reactivex.rxjava2:rxjava:2.0.8"
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
//disklrucache
compile 'com.jakewharton:disklrucache:2.0.2'
```

2、Bean对象的创建

我们以key、value的形式来创建该Bean对象

```
public class ImageBean {
    private String url;
    private Bitmap bitmap;
    public ImageBean(Bitmap bitmap, String url) {
        this.bitmap = bitmap;
        this.url = url;
    }
    public Bitmap getBitmap() {
        return bitmap;
    }
    public void setBitmap(Bitmap bitmap) {
        this.bitmap = bitmap;
    }
    public String getUrl() {
        return url;
    }
    public void setUrl(String url) {
        this.url = url;
    }
}
```

3、缓存类的创建

```
public abstract class CacheObservable {

    /**
     * 获取缓存数据
     * @param url
     * @return
     */
    public Observable<ImageBean> getImage(final String url) {
        return Observable.create(new ObservableOnSubscribe<ImageBean>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<ImageBean> e) throws Exception {
                if (!e.isDisposed()) {
                    ImageBean image = getDataFromCache(url);
                    e.onNext(image);
                    e.onComplete();
                }
            }
        }).subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread());
    }

    /**
     * 取出缓存数据
     * @param url
     * @return
     */
    public abstract ImageBean getDataFromCache(String url);

    /**
     * 缓存数据
     * @param image
     */
    public abstract void putDataToCache(ImageBean image);
}
```

这里我们的三级缓存只要继承至该类，实现存入缓存和取出缓存的操作就可以了

```
public class DiskCacheObservable extends CacheObservable {
    @Override
    public ImageBean getDataFromCache(String url) {
        return null;
    }
    @Override
    public void putDataToCache(final ImageBean image) {
       
    }
}
```

```
public class MemoryCacheObservableextends CacheObservable {
    @Override
    public ImageBean getDataFromCache(String url) {
        return null;
    }
    @Override
    public void putDataToCache(final ImageBean image) {
       
    }
}
```

```
public class NetworkCacheObservable extends CacheObservable {
    @Override
    public ImageBean getDataFromCache(String url) {
        return null;
    }
    @Override
    public void putDataToCache(final ImageBean image) {
       
    }
}
```
这里也是我们最后一步所要实现的逻辑功能，这里我们先把框框搭建好

4、管理缓存类创建

```
public class RequestCreator {
    public MemoryCacheObservable memoryCacheObservable;
    public DiskCacheObservable diskCacheObservable;
    public NetworkCacheObservable networkCacheObservable;
    
    public RequestCreator(Context context) {
        memoryCacheObservable = new MemoryCacheObservable();
        diskCacheObservable = new DiskCacheObservable();
        networkCacheObservable = new NetworkCacheObservable();
    }

    public Observable<ImageBean> getImageFromMemory(String url) {
        return memoryCacheObservable.getImage(url);
    }

    public Observable<ImageBean> getImageFromDisk(String url) {
        return diskCacheObservable.getImage(url);
    }

    public Observable<ImageBean> getImageFromNetwork(String url) {
        return networkCacheObservable.getImage(url);
    }
}
```

5、模拟Picasso源码，使用构造者模式创建我们的RxImageLoader

```
public class RxImageLoader {

    static RxImageLoader singleton;
    private String mUrl;
    private RequestCreator requestCreator;

    //防止用户可以创建该对象
    private RxImageLoader(Builder builder) {
        requestCreator = new RequestCreator(builder.mContext);
    }

    public static RxImageLoader with(Context context) {
        if (singleton == null) {
            synchronized (RxImageLoader.class) {
                if (singleton == null) {
                    singleton = new Builder(context).build();
                }
            }
        }
        return singleton;
    }

    public RxImageLoader load(String url) {
        this.mUrl = url;
        return singleton;
    }

    public void into(final ImageView imageView) {
        Observable
                .concat(
                        requestCreator.getImageFromMemory(mUrl),
                        requestCreator.getImageFromDisk(mUrl),
                        requestCreator.getImageFromNetwork(mUrl)
                )
                .first(new ImageBean(null,mUrl)).toObservable()
                .subscribe(new Observer<ImageBean>() {
                    @Override
                    public void onSubscribe(Disposable d) {

                    }

                    @Override
                    public void onNext(ImageBean imageBean) {
                        if (imageBean.getBitmap() != null) {
                            imageView.setImageBitmap(imageBean.getBitmap());
                        }
                    }

                    @Override
                    public void onError(Throwable e) {
                        e.printStackTrace();
                    }

                    @Override
                    public void onComplete() {
                        Log.e("onComplete", "onComplete");
                    }
                });
    }

    public static class Builder {

        private Context mContext;

        public Builder(Context mContext) {
            this.mContext = mContext;
        }

        public RxImageLoader build() {
            return new RxImageLoader(this);
        }
    }
}
```

上面代码做了哪些事

1. 采用双判空的单例模式
2. 可采用链式编程方式
3. 使用RxJava的concat和first方法
4. concat方法表示将缓存机制Observable进行有序的连接，按顺序读取内存缓存，文件缓存，网络缓存
5. first方法表示判断，如果IamgeBean中的bitmap为空，那么跳过此次连接，例如，requestCreator.getImageFromMemory(mUrl)获取的bitmap为空，那么直接跳过这次concat连接，进行requestCreator.getImageFromDisk(mUrl)操作，直到bitmap不为空则程序继续往下执行，断开concat的连接

6、Activity加载图片

我们简单的使用一个按钮加载图片就好了

```
public class TextImageLoaderActivity extends AppCompatActivity {

    ImageView iv;
    Button bt;
    String url;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_text_image_loader);
        iv = (ImageView) findViewById(R.id.iv);
        bt = (Button) findViewById(R.id.bt);
        url = "http://img2.baa.bitautotech.com/usergroup/editor_pic/2017/3/22/694494c2f3544226ae911bf86b4e2bcc.png";

        bt.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                RxImageLoader.with(TextImageLoaderActivity.this).load(url).into(iv);
            }
        });
    }
}
```

##**实现图片三级缓存**

做好了框架的框框，下面就是对具体的DiskCacheObservable、MemoryCacheObservable、NetworkCacheObservable进行对应的方法实现就可以了

1、内存缓存

内存缓存最简单了，只要放入到LruCache即可

```
public class MemoryCacheObservable extends CacheObservable {

    private int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);
    private int cacheSize = maxMemory / 4;
    private LruCache<String, Bitmap> mLruCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            return bitmap.getRowBytes() * bitmap.getHeight() / 1024;
        }
    };

    @Override
    public ImageBean getDataFromCache(String url) {
        Log.e("getDataFromCache", "getDataFromMemoryCache");
        Bitmap bitmap = mLruCache.get(url);
        return new ImageBean(bitmap, url);
    }

    @Override
    public void putDataToCache(ImageBean image) {
        mLruCache.put(image.getUrl(), image.getBitmap());
    }
}
```


2、文件缓存

文件缓存涉及DiskLruCache的使用、文件下载和文件名用MD5加密

```
public class DiskCacheObservable extends CacheObservable {

    private Context mContext;
    private DiskLruCache mDiskLruCache;
    private final int maxSize = 10 * 1024 * 1024;

    public DiskCacheObservable(Context mContext) {
        this.mContext = mContext;
        initDiskLruCache();
    }

    @Override
    public ImageBean getDataFromCache(String url) {
        Log.e("getDataFromCache","getDataFromDiskCache");
        Bitmap bitmap = getDataFromDiskLruCache(url);
        return new ImageBean(bitmap, url);
    }

    @Override
    public void putDataToCache(final ImageBean image) {
        //由于网络读取需要在子线程中执行
        Observable.create(new ObservableOnSubscribe<ImageBean>() {
            @Override
            public void subscribe(@NonNull ObservableEmitter<ImageBean> e) throws Exception {
                putDataToDiskLruCache(image);
            }
        }).subscribeOn(Schedulers.io()).subscribe();
    }

    public void initDiskLruCache() {
        File cacheDir = DiskCacheUtils.getDiskCacheDir(mContext, "our_cache");
        if (!cacheDir.exists()) {
            cacheDir.mkdirs();
        }
        int versionCode = DiskCacheUtils.getAppVersion(mContext);
        try {
            //这里需要注意参数二：缓存版本号，只要不同版本号，缓存都会被清除，重新使用新的
            mDiskLruCache = DiskLruCache.open(cacheDir, versionCode, 1, maxSize);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取文件缓存
     * @param url
     * @return
     */
    private Bitmap getDataFromDiskLruCache(String url) {
        Bitmap bitmap = null;
        FileDescriptor fileDescriptor = null;
        FileInputStream fileInputStream = null;
        try {
            final String key = DiskCacheUtils.getMD5String(url);
            DiskLruCache.Snapshot snapshot = mDiskLruCache.get(key);
            if (snapshot != null) {
                fileInputStream = (FileInputStream) snapshot.getInputStream(0);
                fileDescriptor = fileInputStream.getFD();
            }
            if (fileDescriptor != null) {
                bitmap = BitmapFactory.decodeStream(fileInputStream);
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (fileInputStream != null) {
                try {
                    fileInputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return bitmap;
    }

    /**
     * 缓存文件数据
     * @param imageBean
     */
    private void putDataToDiskLruCache(ImageBean imageBean) {
        try {
            String key = DiskCacheUtils.getMD5String(imageBean.getUrl());
            DiskLruCache.Editor editor = mDiskLruCache.edit(key);
            if (editor != null) {
                OutputStream outputStream = editor.newOutputStream(0);
                boolean isSuccess = downloadUrlToStream(imageBean.getUrl(), outputStream);
                if (isSuccess) {
                    editor.commit();
                } else {
                    editor.abort();
                }
                mDiskLruCache.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 下载文件
     * @param urlString
     * @param outputStream
     * @return
     */
    private boolean downloadUrlToStream(String urlString, OutputStream outputStream) {
        HttpURLConnection urlConnection = null;
        BufferedOutputStream out = null;
        BufferedInputStream in = null;
        try {
            final URL url = new URL(urlString);
            urlConnection = (HttpURLConnection) url.openConnection();
            in = new BufferedInputStream(urlConnection.getInputStream(), 8 * 1024);
            out = new BufferedOutputStream(outputStream, 8 * 1024);
            int b;
            while ((b = in.read()) != -1) {
                out.write(b);
            }
            return true;
        } catch (final IOException e) {
            e.printStackTrace();
        } finally {
            if (urlConnection != null) {
                urlConnection.disconnect();
            }
            try {
                if (out != null) {
                    out.close();
                }
                if (in != null) {
                    in.close();
                }
            } catch (final IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }

}
```
这里用到一个DiskLruCacheUtils

```
public class DiskCacheUtils {

    /**
     * 获取缓存路径
     * @param context
     * @param uniqueName
     * @return
     */
    public static File getDiskCacheDir(Context context, String uniqueName) {
        String cachePath;
        if (Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
                || !Environment.isExternalStorageRemovable()) {
            cachePath = context.getExternalCacheDir().getPath();
        } else {
            cachePath = context.getCacheDir().getPath();
        }
        return new File(cachePath + File.separator + uniqueName);
    }

    /**
     * 获取App版本号
     * @param context
     * @return
     */
    public static int getAppVersion(Context context) {
        try {
            PackageInfo info = context.getPackageManager().getPackageInfo(context.getPackageName(), 0);
            return info.versionCode;
        } catch (PackageManager.NameNotFoundException e) {
            e.printStackTrace();
        }
        return 1;
    }

    /**
     * 获取加密后的MD5
     * @param key
     * @return
     */
    public static String getMD5String(String key) {
        String cacheKey;
        try {
            final MessageDigest mDigest = MessageDigest.getInstance("MD5");
            mDigest.update(key.getBytes());
            cacheKey = bytesToHexString(mDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            cacheKey = String.valueOf(key.hashCode());
        }
        return cacheKey;
    }

    private static String bytesToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++) {
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if (hex.length() == 1) {
                sb.append('0');
            }
            sb.append(hex);
        }
        return sb.toString();
    }

}
```

3、网络缓存

网络缓存只需要下载文件，不需要实现缓存数据的方法即可

```
public class NetworkCacheObservable extends CacheObservable {
    @Override
    public ImageBean getDataFromCache(String url) {
        Log.e("getDataFromCache", "getDataFromNetworkCache");
        Bitmap bitmap = downloadImage(url);
        return new ImageBean(bitmap, url);
    }

    @Override
    public void putDataToCache(ImageBean image) {

    }

    /**
     * 下载文件
     * @param url
     * @return
     */
    public Bitmap downloadImage(String url) {
        Bitmap bitmap = null;
        InputStream inputStream = null;
        try {
            URL imageUrl = new URL(url);
            URLConnection urlConnection = (HttpURLConnection) imageUrl.openConnection();
            inputStream = urlConnection.getInputStream();
            bitmap = BitmapFactory.decodeStream(inputStream);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            if (inputStream != null) {
                try {
                    inputStream.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return bitmap;
    }
}
```

4、管理缓存

获取三级缓存逻辑实现完之后，就应该对管理我们的缓存，进行对应的缓存操作

```
public class RequestCreator {

    public MemoryCacheObservable memoryCacheObservable;
    public DiskCacheObservable diskCacheObservable;
    public NetworkCacheObservable networkCacheObservable;

    public RequestCreator(Context context) {
        memoryCacheObservable = new MemoryCacheObservable();
        diskCacheObservable = new DiskCacheObservable(context);
        networkCacheObservable = new NetworkCacheObservable();
    }

    public Observable<ImageBean> getImageFromMemory(String url) {
        return memoryCacheObservable.getImage(url)
                .filter(new Predicate<ImageBean>() {
                    @Override
                    public boolean test(@NonNull ImageBean imageBean) throws Exception {
                        Bitmap bitmap = imageBean.getBitmap();
                        return bitmap != null;
                    }
                });
    }

    public Observable<ImageBean> getImageFromDisk(String url) {

        return diskCacheObservable.getImage(url)
                .filter(new Predicate<ImageBean>() {
                    @Override
                    public boolean test(@NonNull ImageBean imageBean) throws Exception {
                        Bitmap bitmap = imageBean.getBitmap();
                        return bitmap != null;
                    }
                }).doOnNext(new Consumer<ImageBean>() {
                    @Override
                    public void accept(@NonNull ImageBean imageBean) throws Exception {
                        //缓存内存
                        memoryCacheObservable.putDataToCache(imageBean);
                    }
                });
    }

    public Observable<ImageBean> getImageFromNetwork(String url) {
        return networkCacheObservable.getImage(url)
                .filter(new Predicate<ImageBean>() {
                    @Override
                    public boolean test(@NonNull ImageBean imageBean) throws Exception {
                        Bitmap bitmap = imageBean.getBitmap();
                        return bitmap != null;
                    }
                })
                .doOnNext(new Consumer<ImageBean>() {
                    @Override
                    public void accept(@NonNull ImageBean imageBean) throws Exception {
                        //缓存文件和内存
                        diskCacheObservable.putDataToCache(imageBean);
                        memoryCacheObservable.putDataToCache(imageBean);
                    }
                });
    }
}
```

##**演示效果**
1、首次运行程序，没有任何缓存，当我们连续点击2次按钮时

![这里写图片描述](http://img.blog.csdn.net/20170407143911597?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到其运行的顺序

1. 第一次点击按钮可以先从内存和文件获取图片，发现没有，再从网络获取图片
2. 第二次点击按钮从内存获取

2、第二次运行程序，有了文件缓存，当我们连续点击2次按钮时

![这里写图片描述](http://img.blog.csdn.net/20170407144318517?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到其运行的顺序

1. 第一次点击按钮先从内存获取，发现没有，再从文件获取图片
2. 第二次点击按钮从内存获取

##**源码下载**

[源码下载](http://download.csdn.net/detail/qq_30379689/9806462)


##**结语**

各位同学可以下载源码进行阅读，最好自己手写一遍，你会更深刻体会到RxJava的好处和掌握图片的三级缓存机制，如果看不懂的同学不要气馁，多看几遍就会了。喜欢我的朋友可以关注我的博客，一定会有你想要学习的知识