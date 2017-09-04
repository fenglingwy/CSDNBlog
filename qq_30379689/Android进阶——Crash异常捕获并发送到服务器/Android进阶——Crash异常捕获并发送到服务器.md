#Crash异常捕获并发送到服务器


----------
>本篇文章包含以下内容：

>* Crash异常捕获的简单使用
>* Crash异常捕获并发送到服务器

在项目中，我们常常会遇到Crash的现象，也就是程序崩溃的时候，这个时候最常看到的就是这个界面

![](http://img.blog.csdn.net/20161219002033714?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果你的项目已经发布到市场上了，这样的崩溃对于开发人员是看不到的，所以我们得想方法将崩溃信息发送到服务器，交给我们的程序员查看，Google考虑到这一点，也提供了Thread.UncaughtExceptionHandler接口来实现这一问题

##**一、Crash异常捕获的简单使用**

创建Crash异常捕获很简单，主要的步骤有：

1. 创建BaseApplication继承Application并实现Thread.UncaughtExceptionHandler
2. 通过Thread.setDefaultUncaughtExceptionHandler(this)设置默认的异常捕获
3. 最后在manifests中注册创建的BaseApplication
```
public class BaseApplication extends Application implements Thread.UncaughtExceptionHandler {

    @Override
    public void onCreate() {
        super.onCreate();
        //设置异常捕获
        Thread.setDefaultUncaughtExceptionHandler(this);
    }

    @Override
    public void uncaughtException(final Thread thread, final Throwable ex) {
	    //当有异常产生时执行该方法
    }
}
```
我们可以在uncaughtException()方法中输出异常信息，并让它隔两秒杀死自己进程，这样就不会弹出崩溃的弹窗，让它直接退出程序

```
/**
 * 当有异常产生时执行该方法
 * @param thread 当前线程
 * @param ex 异常信息
 */
@Override
public void uncaughtException(final Thread thread, final Throwable ex) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            Log.e("TAG","currentThread:"+Thread.currentThread()+"---thread:"+thread.getId()+"---ex:"+ex.toString());
        }
    }).start();
    SystemClock.sleep(2000);
    android.os.Process.killProcess(android.os.Process.myPid());
}
```
最后一步，别忘了在manifests中注册BaseApplication

```
<application
    android:name=".Base.BaseApplication"
    android:allowBackup="true"
    android:icon="@mipmap/ic_logo"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
```
我们通过运行这个方法，来测试我们的程序
```
private void testError() {
    int a = 10 / 0;
}
```
查看Log信息，验证我们的错误信息

```
12-18 11:40:02.074 7510-7555/com.handsome.ap E/TAG: currentThread:Thread[Thread-325,5,main]---thread:1---ex:
java.lang.RuntimeException: Unable to start activity ComponentInfo{com.handsome.ap/com.handsome.ap.Activity.MainActivity}: 
java.lang.ArithmeticException: divide by zero
```
完整代码

```
public class BaseApplication extends Application implements Thread.UncaughtExceptionHandler {

    @Override
    public void onCreate() {
        super.onCreate();
        //设置异常捕获
        Thread.setDefaultUncaughtExceptionHandler(this);
    }

    /**
     * 当有异常产生时执行该方法
     * @param thread 当前线程
     * @param ex 异常信息
     */
    @Override
    public void uncaughtException(final Thread thread, final Throwable ex) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                Log.e("TAG","currentThread:"+Thread.currentThread()+"---thread:"+thread.getId()+"---ex:"+ex.toString());
            }
        }).start();
        SystemClock.sleep(2000);
        android.os.Process.killProcess(android.os.Process.myPid());
    }
}
```


##**二、Crash异常捕获并发送到服务器**

其实这里就是将上面的简单使用进行封装，在一个类中处理相关的逻辑，主要步骤和上面是一样的

```
public class CrashHandler implements Thread.UncaughtExceptionHandler {

    /**
     * 捕获异常回掉
     *
     * @param thread 当前线程
     * @param ex     异常信息
     */
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        //导出异常信息到SD卡
        dumpExceptionToSDCard(ex);
        //上传异常信息到服务器
        uploadExceptionToServer(ex);
        //延时1秒杀死进程
        SystemClock.sleep(2000);
        Process.killProcess(Process.myPid());
    }
}
```
我们为下面的信息保存先提供一些成员变量

```
//文件夹目录
private static final String PATH = Environment.getExternalStorageDirectory().getPath() + "/crash_log/";
//文件名
private static final String FILE_NAME = "crash";
//文件名后缀
private static final String FILE_NAME_SUFFIX = ".trace";
//上下文
private Context mContext;

//单例模式
private static CrashHandler sInstance = new CrashHandler();
private CrashHandler() {}
public static CrashHandler getInstance() {
    return sInstance;
}
```
提供一个初始化的方法，记得调用Thread.setDefaultUncaughtExceptionHandler(this)这个方法

```
/**
 * 初始化方法
 *
 * @param context
 */
public void init(Context context) {
    //将当前实例设为系统默认的异常处理器
    Thread.setDefaultUncaughtExceptionHandler(this);
    //获取Context，方便内部使用
    mContext = context.getApplicationContext();
}
```
剩下的就是保存异常信息了，这里发送到服务端采用的是Bmob第三方后端云
```
/**
 * 导出异常信息到SD卡
 *
 * @param ex
 */
private void dumpExceptionToSDCard(Throwable ex) {
    if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
        return;
    }
    //创建文件夹
    File dir = new File(PATH);
    if (!dir.exists()) {
        dir.mkdirs();
    }
    //获取当前时间
    long current = System.currentTimeMillis();
    String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(current));
    //以当前时间创建log文件
    File file = new File(PATH + FILE_NAME + time + FILE_NAME_SUFFIX);
    try {
        //输出流操作
        PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
        //导出手机信息和异常信息
        PackageManager pm = mContext.getPackageManager();
        PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);
        pw.println("发生异常时间：" + time);
        pw.println("应用版本：" + pi.versionName);
        pw.println("应用版本号：" + pi.versionCode);
        pw.println("android版本号：" + Build.VERSION.RELEASE);
        pw.println("android版本号API：" + Build.VERSION.SDK_INT);
        pw.println("手机制造商:" + Build.MANUFACTURER);
        pw.println("手机型号：" + Build.MODEL);
        ex.printStackTrace(pw);
        //关闭输出流
        pw.close();
    } catch (Exception e) {

    }
}

/**
 * 上传异常信息到服务器
 *
 * @param ex
 */
private void uploadExceptionToServer(Throwable ex) {
    Error error = new Error(ex.getMessage());
    error.save(new SaveListener<String>() {
        @Override
        public void done(String objectId, BmobException e) {

        }
    });
}
```
在我们的Application中创建该异常捕获

```
CrashHandler crashHandler = CrashHandler.getInstance();
crashHandler.init(this);
```

我们同样按照上面的方法来测试这个异常捕获，运行程序，在文件夹中找到我们创建的目录

![](http://img.blog.csdn.net/20161219010640012?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

找到对应文件

![](http://img.blog.csdn.net/20161219010706778?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

查看对应信息

![](http://img.blog.csdn.net/20161219010718107?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

后台数据库的信息

![](http://img.blog.csdn.net/20161219010739607?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

完整代码

```
public class CrashHandler implements Thread.UncaughtExceptionHandler {

    //文件夹目录
    private static final String PATH = Environment.getExternalStorageDirectory().getPath() + "/crash_log/";
    //文件名
    private static final String FILE_NAME = "crash";
    //文件名后缀
    private static final String FILE_NAME_SUFFIX = ".trace";
    //上下文
    private Context mContext;

    //单例模式
    private static CrashHandler sInstance = new CrashHandler();
    private CrashHandler() {}
    public static CrashHandler getInstance() {
        return sInstance;
    }

    /**
     * 初始化方法
     *
     * @param context
     */
    public void init(Context context) {
        //将当前实例设为系统默认的异常处理器
        Thread.setDefaultUncaughtExceptionHandler(this);
        //获取Context，方便内部使用
        mContext = context.getApplicationContext();
    }

    /**
     * 捕获异常回掉
     *
     * @param thread 当前线程
     * @param ex     异常信息
     */
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        //导出异常信息到SD卡
        dumpExceptionToSDCard(ex);
        //上传异常信息到服务器
        uploadExceptionToServer(ex);
        //延时1秒杀死进程
        SystemClock.sleep(2000);
        Process.killProcess(Process.myPid());
    }

    /**
     * 导出异常信息到SD卡
     *
     * @param ex
     */
    private void dumpExceptionToSDCard(Throwable ex) {
        if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            return;
        }
        //创建文件夹
        File dir = new File(PATH);
        if (!dir.exists()) {
            dir.mkdirs();
        }
        //获取当前时间
        long current = System.currentTimeMillis();
        String time = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(current));
        //以当前时间创建log文件
        File file = new File(PATH + FILE_NAME + time + FILE_NAME_SUFFIX);
        try {
            //输出流操作
            PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
            //导出手机信息和异常信息
            PackageManager pm = mContext.getPackageManager();
            PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);
            pw.println("发生异常时间：" + time);
            pw.println("应用版本：" + pi.versionName);
            pw.println("应用版本号：" + pi.versionCode);
            pw.println("android版本号：" + Build.VERSION.RELEASE);
            pw.println("android版本号API：" + Build.VERSION.SDK_INT);
            pw.println("手机制造商:" + Build.MANUFACTURER);
            pw.println("手机型号：" + Build.MODEL);
            ex.printStackTrace(pw);
            //关闭输出流
            pw.close();
        } catch (Exception e) {

        }
    }

    /**
     * 上传异常信息到服务器
     *
     * @param ex
     */
    private void uploadExceptionToServer(Throwable ex) {
        Error error = new Error(ex.getMessage());
        error.save(new SaveListener<String>() {
            @Override
            public void done(String objectId, BmobException e) {

            }
        });
    }
}
```

[CrashHandler源码下载](http://download.csdn.net/detail/qq_30379689/9714943)

>到这里我们的Crash异常捕获就结束了，很简单的一段代码就可以解决你缺少的项目经验