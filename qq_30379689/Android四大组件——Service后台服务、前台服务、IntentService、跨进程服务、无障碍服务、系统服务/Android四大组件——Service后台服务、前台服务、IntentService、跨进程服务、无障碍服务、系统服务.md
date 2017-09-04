#Service后台服务、前台服务、IntentService、跨进程服务、无障碍服务、系统服务


----------
>本篇文章包括以下内容：

>* [前言](#l)
>* [Service的简介](#a)
>* [后台服务](#b)
	* [不可交互的后台服务](#c)
	* [可交互的后台服务 ](#d)
	* [混合性交互的后台服务 ](#e)
* [前台服务](#f)
* [IntentService](#g)
* [AIDL跨进程服务](#h)
* [AccessibilityService无障碍服务](#i)
* [系统服务](#j)
* [部分源码下载](#k)

##**<span id="l">前言</span>**
作为四大组件之一的Service类，是面试和笔试的必备关卡，我把我所学到的东西总结了一遍，相信你看了之后你会对Service娓娓道来，在以后遇到Service的问题胸有成竹，废话不多说，开车啦

##**<span id="a">Service简介</span>**

Service是Android中实现程序后台运行的解决方案，它非常适用于去执行那些不需要和用户交互而且还要求长期运行的任务。Service默认并不会运行在子线程中，它也不运行在一个独立的进程中，它同样执行在UI线程中，因此，不要在Service中执行耗时的操作，除非你在Service中创建了子线程来完成耗时操作

Service的运行不依赖于任何用户界面，即使程序被切换到后台或者用户打开另一个应用程序，Service仍然能够保持正常运行，这也正是Service的使用场景。当某个应用程序进程被杀掉时，所有依赖于该进程的Service也会停止运行

##**<span id="b">后台服务</span>**
后台服务可交互性主要是体现在不同的启动服务方式，startService()和bindService()。bindService()可以返回一个代理对象，可调用Service中的方法和获取返回结果等操作，而startService()不行

##**<span id="c">不可交互的后台服务</span>**
不可交互的后台服务即是普通的Service，Service的生命周期很简单，分别为onCreate、onStartCommand、onDestroy这三个。当我们startService()的时候，首次创建Service会回调onCreate()方法，然后回调onStartCommand()方法，再次startService()的时候，就只会执行一次onStartCommand()。服务一旦开启后，我们就需要通过stopService()方法或者stopSelf()方法，就能把服务关闭，这时就会回调onDestroy()

###**一、创建服务类**
创建一个服务非常简单，只要继承Service，并实现onBind()方法
```
public class BackGroupService extends Service {

    /**
     * 綁定服务时调用
     *
     * @param intent
     * @return
     */
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.e("Service", "onBind");
        return null;
    }

    /**
     * 服务创建时调用
     */
    @Override
    public void onCreate() {
        Log.e("Service", "onCreate");
        super.onCreate();
    }

    /**
     * 执行startService时调用
     *
     * @param intent
     * @param flags
     * @param startId
     * @return
     */
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        Log.e("Service", "onStartCommand");
        //这里执行耗时操作
        new Thread() {
            @Override
            public void run() {
                while (true){
                    try {
                        Log.e("Service", "doSomething");
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }.start();
        return super.onStartCommand(intent, flags, startId);
    }

    /**
     * 服务被销毁时调用
     */
    @Override
    public void onDestroy() {
        Log.e("Service", "onDestroy");
        super.onDestroy();
    }
}
```
###**二、配置服务**
Service也是四大组件之一，所以必须在manifests中配置
```
<service android:name=".Service.BackGroupService"/>
```

###**三、启动服务和停止服务**
我们通过两个按钮分别演示启动服务和停止服务，通过startService()开启服务，通过stopService()停止服务
```
public class MainActivity extends AppCompatActivity {

    Button bt_open, bt_close;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bt_open = (Button) findViewById(R.id.open);
        bt_close = (Button) findViewById(R.id.close);

        final Intent intent = new Intent(this, BackGroupService.class);

        bt_open.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
	            //启动服务
                startService(intent);
            }
        });

        bt_close.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
	            //停止服务
                stopService(intent);
            }
        });
    }
}
```
当你开启服务后，还有一种方法可以关闭服务，在设置中，通过应用->找到自己应用->停止

![](http://img.blog.csdn.net/20161124133116841)

###**四、运行代码**
运行程序后，我们点击开始服务，然后一段时间后关闭服务。我们以Log信息来验证普通Service的生命周期：onCreate->onStartCommand->onDestroy

```
11-24 00:19:51.483 16407-16407/com.handsome.boke2 E/Service: onCreate
11-24 00:19:51.483 16407-16407/com.handsome.boke2 E/Service: onStartCommand
11-24 00:19:51.485 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:19:53.490 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:19:55.491 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:19:57.491 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:19:58.056 16407-16407/com.handsome.boke2 E/Service: onDestroy
11-24 00:19:59.492 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:20:01.494 16407-16613/com.handsome.boke2 E/Service: doSomething
11-24 00:20:03.495 16407-16613/com.handsome.boke2 E/Service: doSomething
```

其中你会发现我们的子线程进行的耗时操作是一直存在的，而我们Service已经被关闭了，关闭该子线程的方法需要直接通过Home键关闭该应用程序

##**<span id="d">可交互的后台服务</span>**
可交互的后台服务是指前台页面可以调用后台服务的方法，可交互的后台服务实现步骤是和不可交互的后台服务实现步骤是一样的，区别在于启动的方式和获得Service的代理对象
###**一、创建服务类**
和普通Service不同在于这里返回一个代理对象，返回给前台进行获取，即前台可以获取该代理对象执行后台服务的方法
```
public class BackGroupService extends Service {

    /**
     * 綁定服务时调用
     *
     * @param intent
     * @return
     */
    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        Log.e("Service", "onBind");
        //返回代理对象
        return new MyBinder();
    }

    /**
     * 代理类
     */
    class MyBinder extends Binder {
        public void showToast() {
            Log.e("Service", "showToast");
        }

        public void showList() {
            Log.e("Service", "showList");
        }
    }

    /**
     * 解除绑定服务时调用
     * @param intent
     * @return
     */
    @Override
    public boolean onUnbind(Intent intent) {
        Log.e("Service", "onUnbind");
        return super.onUnbind(intent);
    }

    /**
     * 服务创建时调用
     */
    @Override
    public void onCreate() {
        Log.e("Service", "onCreate");
        super.onCreate();
    }

    /**
     * 服务被销毁时调用
     */
    @Override
    public void onDestroy() {
        Log.e("Service", "onDestroy");
        super.onDestroy();
    }
}
```
###**二、配置服务**
```
<service android:name=".Service.BackGroupService"/>
```
###**三、绑定服务和解除绑定服务**
我们通过两个按钮分别演示绑定服务和解除绑定服务，通过bindService()开启服务，通过unbindService()停止服务

```
public class MainActivity extends AppCompatActivity {

    Button bt_open, bt_close;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        bt_open = (Button) findViewById(R.id.open);
        bt_close = (Button) findViewById(R.id.close);

        final Intent intent = new Intent(this, BackGroupService.class);

        bt_open.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                bindService(intent, conn, BIND_AUTO_CREATE);
            }
        });

        bt_close.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                unbindService(conn);
            }
        });


    }

    ServiceConnection conn = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
			//拿到后台服务代理对象
            BackGroupService.MyBinder myBinder = (BackGroupService.MyBinder) service;
            //调用后台服务的方法
            myBinder.showToast();
            myBinder.showList();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
}
```
这里和startService的区别在于多了一个ServiceConnection对象，该对象是用户绑定后台服务后，可获取后台服务代理对象的回调，我们可以通过该回调，拿到后台服务的代理对象，并调用后台服务定义的方法，也就实现了后台服务和前台的交互

###**四、运行代码**
运行程序后，我们点击绑定服务，然后一段时间后解除绑定服务。我们以Log信息来验证Service的生命周期：onCreate->onBind->onUnBind->onDestroy，其中也可以看到我们调用后台服务的方法showToast和showList

```
11-24 00:55:32.775 15408-15408/com.handsome.boke2 E/Service: onCreate
11-24 00:55:32.775 15408-15408/com.handsome.boke2 E/Service: onBind
11-24 00:55:32.796 15408-15408/com.handsome.boke2 E/Service: showToast
11-24 00:55:32.796 15408-15408/com.handsome.boke2 E/Service: showList
11-24 00:55:34.290 15408-15408/com.handsome.boke2 E/Service: onUnbind
11-24 00:55:34.290 15408-15408/com.handsome.boke2 E/Service: onDestroy
```

##**<span id="e">混合性交互的后台服务</span>**

或许你会迷惑，startService和bindService之间有什么关系？其实简单的说两者之间是没有关联的，类似于你亲妈生了个双胞胎一样，只有纯粹的血缘关系。那么问题来了，这两个启动方式是否可以同时使用呢，答案是可以的

将上面两种启动方式结合起来就是混合性交互的后台服务了，即可以单独运行后台服务，也可以运行后台服务中提供的方法，其完整的生命周期是：onCreate->onStartCommand->onBind->onUnBind->onDestroy

##**<span id="f">前台服务</span>**
由于后台服务优先级相对比较低，当系统出现内存不足的情况下，它就有可能会被回收掉，所以前台服务就是来弥补这个缺点的，它可以一直保持运行状态而不被系统回收。例如：墨迹天气在状态栏中的天气预报

![](http://img.blog.csdn.net/20161124155501133)

###**一、创建服务类**
前台服务创建很简单，其实就在Service的基础上创建一个Notification，然后使用Service的startForeground()方法即可启动为前台服务

```
public class ForegroundService extends Service {

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        showNotification();
    }

    /**
     * 启动前台通知
     */
    private void showNotification() {
        //创建通知详细信息
        Notification.Builder mBuilder = new Notification.Builder(this)
                .setSmallIcon(R.mipmap.ic_launcher)
                .setContentTitle("2016年11月24日")
                .setContentText("今天天气阴天，8到14度");
        //创建点击跳转Intent
        Intent intent = new Intent(this, MainActivity.class);
        //创建任务栈Builder
        TaskStackBuilder stackBuilder = TaskStackBuilder.create(this);
        stackBuilder.addParentStack(MainActivity.class);
        stackBuilder.addNextIntent(intent);
        PendingIntent pendingIntent = stackBuilder.getPendingIntent(0,PendingIntent.FLAG_UPDATE_CURRENT);
        //设置跳转Intent到通知中
        mBuilder.setContentIntent(pendingIntent);
        //获取通知服务
        NotificationManager nm = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        //构建通知
        Notification notification = mBuilder.build();
        //显示通知
        nm.notify(0, notification);
        //启动为前台服务
        startForeground(0, notification);
    }
}
```

###**二、配置服务**

```
<service android:name=".Service.ForegroundService"/>
```
###**三、启动前台服务**

```
startService(new Intent(this, ForegroundService.class));
```

###**四、运行代码**
我们可以看到状态栏确实增加了我们这条通知

![](http://img.blog.csdn.net/20161124161145289)

当我们将该程序退出并杀掉的时候，通过设置->应用->选择正在运行中的应用，我们可以发现，我们的程序退出杀掉了，而服务还在进行着

![](http://img.blog.csdn.net/20161124161301095)

##**<span id="g">IntentService</span>**

IntentService是专门用来解决Service中不能执行耗时操作这一问题的，创建一个IntentService也很简单，只要继承IntentService并覆写onHandlerIntent函数，在该函数中就可以执行耗时操作了

```
public class TheIntentService extends IntentService {

    public TheIntentService(String name) {
        super(name);
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        //在这里执行耗时操作
    }
}
```

##**<span id="h">AIDL跨进程服务</span>**

关于AIDL跨进程服务的使用和原理分析，可以见我另一篇博客：[Android基础——初学者必知的AIDL在应用层上的Binder机制](http://blog.csdn.net/qq_30379689/article/details/52253413)

##**<span id="i">AccessibilityService无障碍服务</span>**

关于AccessibilityService无障碍服务的使用和实例，可以见我另一篇博客：[Android进阶——学习AccessibilityService实现微信抢红包插件](http://blog.csdn.net/qq_30379689/article/details/53242953)

##**<span id="j">系统服务</span>**

系统服务提供了很多便捷服务，可以查询Wifi、网络状态、查询电量、查询音量、查询包名、查询Application信息等等等相关多的服务，具体大家可以自信查询文档，这里举例几个常见的服务

###**1. 判断Wifi是否开启**

```
WifiManager wm = (WifiManager) getSystemService(WIFI_SERVICE);
boolean enabled = wm.isWifiEnabled();
```
需要权限

```
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE"/>
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"/>
```
###**2. 获取系统最大音量**

```
AudioManager am = (AudioManager) getSystemService(AUDIO_SERVICE);
int max = am.getStreamMaxVolume(AudioManager.STREAM_SYSTEM);
```
###**3. 获取当前音量**

```
AudioManager am = (AudioManager) getSystemService(AUDIO_SERVICE);
int current = am.getStreamMaxVolume(AudioManager.STREAM_RING);
```

###**4. 判断网络是否有连接**

```
ConnectivityManager cm = (ConnectivityManager) getSystemService(CONNECTIVITY_SERVICE);
NetworkInfo info = cm.getActiveNetworkInfo();
boolean isAvailable = info.isAvailable();
```
需要权限

```
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```


##**<span id="k">部分源码下载</span>**

[源码下载](http://download.csdn.net/detail/qq_30379689/9692631)