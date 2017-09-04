#LeakCanary检测内存泄漏


----------
>本篇文章包括以下内容：

>* [前言](#a)
>* [内存泄漏的简介](#b)
>* [内存溢出的简介](#c)
>* [LeakCanary的配置与使用](#d)
>* [结语](#e)

##**<span id="a">前言</span>**

内存泄漏对于初学者们可能是一个陌生的词语，但是却频频发生于自己的软件上，只不过自己不知道而已。同理，内存溢出也是一个道理。而内存泄漏和内存溢出常常是面试的考题，所以早点掌握是必不可少的


##**<span id="b">内存泄漏的简介</span>**

内存泄漏是指：对象在它有限的生命周期结束时，它们将被垃圾回收，如果在回收时，这个对象还被一系列的引用，导致该对象不会被回收，那么就会导致内存泄漏。随着泄漏的累积，应用将消耗完内存，应用的流畅性就会大大减弱

常见的内存泄漏有：

1. 静态变量引用导致内存泄漏
2. 属性动画未及时关闭导致的内存泄漏

##**<span id="c">内存溢出的简介</span>**

>涉及到内存泄漏，当然也逃不过内存溢出的内容

内存溢出是指：程序要求的内存，超出了系统所能分配的范围，从而发生溢出。Android应用程序的默认最大内存值为16M，如果你使用的内存超过最大值，则会导致应用报错

常见的内存溢出有：

1. 图片过大导致内存溢出

##**<span id="d">LeakCanary的配置与使用</span>**

LeakCanary是square公司开发出来的检测内存泄漏的神器，他可以嵌入到我们的应用中，帮助我们自动检测内存泄漏

###**一、添加依赖**
在项目的gradle中添加对LeakCanary的依赖

```
dependencies {
    //内存泄漏
    debugCompile 'com.squareup.leakcanary:leakcanary-android:1.5'
    releaseCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
    testCompile 'com.squareup.leakcanary:leakcanary-android-no-op:1.5'
}
```

###**二、LeakCanary的使用**

首先，需要创建一个类继承自Application，为了让我们LeakCanary在全局的Activity中能够起作用

```
public class BaseApplication extends Application {
    @Override
    public void onCreate() {
        super.onCreate();
    }
}
```

接着，在Manifests文件中指定该Application

```java
<application
    android:name=".LeakCanary.BaseApplication">
</application>
```
最后，就是在Application中直接配置我们的LeakCanary
```
public class BaseApplication extends Application {

    @Override
    public void onCreate() {
        super.onCreate();
        //配置LeakCanary
        setupLeakCanary();
    }

    /**
     * 配置LeakCanary
     */
    private void setupLeakCanary() {
        if (LeakCanary.isInAnalyzerProcess(this)) {
            return;
        }
        LeakCanary.install(this);
    }
}
```
>到此我们的LeakCanary就已经嵌入成功，就剩测试了
###**三、测试内存泄漏**

我们采用静态变量的内存泄漏来作为我们的测试结果，创建一个类，该类的作用可以保存一个Activity对象到一个ArrayList中，让Activity一直存在

```
public class ActivityPool {

    public static ActivityPool activityPool = new ActivityPool();
    public static ArrayList<Activity> activities = new ArrayList<>();

    public static ActivityPool getActivity() {
        return activityPool;
    }

    public static void addActivity(Activity activity) {
        activities.add(activity);
    }
}
```
做好了内存泄漏类之后，就在我们的主界面中直接使用该类，并将主界面的Activity存放在ArrayList中

```
public class LeakCanaryActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_leakcanary);

        ActivityPool.getActivity().addActivity(this);
    }
}
```

>这个时候，我们就已经模拟了内存泄漏了，接着，就可以运行程序进行测试了

###**四、测试结果与分析**

我们运行程序进入主界面，那么主界面的Activity就被保存起来了，这个时候是没什么事情的，因为主界面的Activity还被引用，并不会发生内存泄漏。接着，我们退出主界面，按道理是：退出主界面Activity，其生命周期就被结束了，该Activity必须被系统回收，但是我们还留了一手，将它保存起来了，那么等待一段时间后，就会在通知栏收到LeakCanary的通知信息

![](http://img.blog.csdn.net/20170211013652212?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这个时候，内存泄漏报告就出来了，点击通知信息，进入内存泄漏报告分析

![](http://img.blog.csdn.net/20170211013747776?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

结果分析得：

第一部分（ActivityPool类的activities变量）由于第二部分（ArrayList中的某个数组）和第三部分（数组中的某个对象） 导致第四部分(LeakCanaryActivity类的实例instance)泄露

>这样我们就可以很容易的定位到内存泄漏的部分

**上面说到为什么要在关闭Activity时，等待一段时间才会出现内存泄漏报告呢？**

原因是：LeakCanary会监听所有Activity的生命周期，并且在Activity的onDestory函数结束后，将该Activity添加到内存泄漏监控队列中。接着，在后台线程中检测这个引用是否被清除，如果没有将会发生GC操作（垃圾回收），如果引用仍然没有清除，将heap内存dump到一个.hprof文件并存放在手机系统里，应用会开启另一个进程来分析该.hprof文件，最后导出泄漏引用链，传回应用程序中，通过推送推送出信息



##**<span id="e">结语</span>**

LeakCanary的使用并不难，重要的是如何排除内存泄漏的问题，可能大部分问题还是需要在实战中才能摸索出来，这里只是举一例子，不过通过这篇之后，相信你对内存泄漏和溢出都有一个大概的了解，对待类似面试题应该是没什么问题了