#Android进阶——多线程系列之四大线程池的使用介绍

----------
>本篇文章包括以下内容：

>* [前言](#a)
>* [线程池的简介](#b)
>* [四大线程池之老大ThreadPoolExecutor](#c)
* [四大线程池之FixedThreadPool](#d)
* [四大线程池之CachedThreadPool](#e)
* [四大线程池之ScheduledThreadPool](#f)
* [四大线程池之SingleThreadExecutor](#g)


##**<span id="a">前言</span>**
线程池一直是初学者最抵触的东西，由于刚开始学习做项目并不会涉及到线程池的使用，但是不去学习它，心里又好像有个石头一直沉着，一直放心不下，其实是很简单的东西，早晚都要学，不如趁现在吧。由于文章从初学者的角度出发，所以文章显得粗浅，但通俗易懂。废话不多说，开车啦
##**<span id="b">线程池的简介</span>**
线程池简单的说就是管理线程的一个总调度官。它可以存储着多个核心线程和多个非核心线程，也可以派遣核心线程或非核心线程去处理事情，总之它就是线程的老大。当然它也有四个小弟，即是四大线程池

**一、提到线程池，当然也要提到线程池的优点：**

1. 重用线程池中的线程，避免因为线程的创建和销毁所带来的性能开销
2. 能有效控制线程池的最大并发数，避免大量的线程之间因互相抢占系统资源而导致的阻塞现象 
3. 能够对线程简单的管理，并提供定时执行以及指定间隔循环执行等功能

**二、真正的线程池只有一个类：ThreadPoolExecutor，而四大线程池则是不同功能的线程池，其关系如下图**

![](http://img.blog.csdn.net/20161208183713507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**三、线程池采用的是工厂模式，其四大线程池就是由老大打造出来的**

```
ExecutorService fixedThreadExecutor = Executors.newFixedThreadPool(5);
ExecutorService cachedThreadExecutor = Executors.newCachedThreadPool();
ExecutorService scheduledThreadExecutor = Executors.newScheduledThreadPool(5);
ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();
```

##**<span id="c">四大线程池之老大ThreadPoolExecutor</span>**
ThreadPoolExecutor是线程池的真正实现，它的构造方法提供了一系列参数来配置线程池，其构造方法如下

```
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```
**corePoolSize**
线程池的核心线程数，默认情况下，核心线程会在线程池中一直存活，即使它们处于闲置状态。如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务到来时会有超时策略，这个时间间隔由keepAliveTime所指定，当等待时间超过keepAliveTime所指定的时长后，核心线程就会被终止
**maximumPoolSize**
线程池所能容纳的最大线程数，当活动线程数达到这个数值后，后续的新任务将会被阻塞
**keepAliveTime**
非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。当ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true时，keepAliveTime同样会作用于核心线程
**unit**
用于指定keepAliveTime参数的时间单位，常用的有TimeUnit.MILLISECONDS（毫秒），秒，分钟等
**workQueue**
线程池中的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中
**threadFactory**
线程工厂，为线程池提供创建新线程的功能
**handler**
当线程池无法执行新任务时，这可能是由于任务队列已满或者是无法成功执行任务，这个时候ThreadPoolExecutor会调用handler的rejectedExecution方法来通知调用者

>好了，看好了老大之后，我们开始看小弟了

##**<span id="d">四大线程池之FixedThreadPool</span>**
**一、FixedThreadPool的分析**
FixedThreadPool是一种线程数量固定的线程池，当线程处于空闲状态时，它们并不会被回收，除非线程池被关闭了。下面是创建FixedThreadPool的源码
```
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```
其特点是：

1. 线程数量固定
2. 空闲不会被回收
3. 更快的响应速度
4. 无超时机制，无大小限制

**二、FixedThreadPool的使用**
我们创建五个线程的线程池来演示
```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ExecutorService fixedThreadExecutor = Executors.newFixedThreadPool(5);

        for (int i = 0; i < 10; i++) {
            MyRunnable myRunnable = new MyRunnable(i);
            fixedThreadExecutor.execute(myRunnable);
        };
    }

    /**
     * 模拟耗时任务
     */
    class MyRunnable implements Runnable{

        int threadNum;

        MyRunnable(int threadNum){
            this.threadNum = threadNum;
        }

        @Override
        public void run() {
            Log.e("打印信息：","等待中...执行第"+threadNum+"号线程");
            try {
                Thread.currentThread().sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Log.e("打印信息：","完成第"+threadNum+"号线程");
        }
    }
}
```
查看Log信息

```
12-08 08:24:12.320 1918-1953/com.hensen.boke E/打印信息：: 等待中...执行第2号线程
12-08 08:24:12.320 1918-1951/com.hensen.boke E/打印信息：: 等待中...执行第0号线程
12-08 08:24:12.321 1918-1954/com.hensen.boke E/打印信息：: 等待中...执行第3号线程
12-08 08:24:12.322 1918-1952/com.hensen.boke E/打印信息：: 等待中...执行第1号线程
12-08 08:24:12.322 1918-1955/com.hensen.boke E/打印信息：: 等待中...执行第4号线程
12-08 08:24:16.321 1918-1953/com.hensen.boke E/打印信息：: 完成第2号线程
12-08 08:24:16.321 1918-1951/com.hensen.boke E/打印信息：: 完成第0号线程
12-08 08:24:16.321 1918-1954/com.hensen.boke E/打印信息：: 完成第3号线程
12-08 08:24:16.321 1918-1954/com.hensen.boke E/打印信息：: 等待中...执行第5号线程
12-08 08:24:16.322 1918-1953/com.hensen.boke E/打印信息：: 等待中...执行第6号线程
12-08 08:24:16.322 1918-1951/com.hensen.boke E/打印信息：: 等待中...执行第7号线程
12-08 08:24:16.322 1918-1952/com.hensen.boke E/打印信息：: 完成第1号线程
12-08 08:24:16.322 1918-1955/com.hensen.boke E/打印信息：: 完成第4号线程
12-08 08:24:16.322 1918-1952/com.hensen.boke E/打印信息：: 等待中...执行第8号线程
12-08 08:24:16.322 1918-1955/com.hensen.boke E/打印信息：: 等待中...执行第9号线程
12-08 08:24:20.322 1918-1954/com.hensen.boke E/打印信息：: 完成第5号线程
12-08 08:24:20.322 1918-1951/com.hensen.boke E/打印信息：: 完成第7号线程
12-08 08:24:20.322 1918-1953/com.hensen.boke E/打印信息：: 完成第6号线程
12-08 08:24:20.322 1918-1955/com.hensen.boke E/打印信息：: 完成第9号线程
12-08 08:24:20.322 1918-1952/com.hensen.boke E/打印信息：: 完成第8号线程
```

##**<span id="e">四大线程池之CachedThreadPool</span>**
**一、CachedThreadPool的分析**
CachedThreadPool是一种线程数量不定的线程池，它只有非核心线程，并且其最大线程数为Integer.MAX_VALUE，下面是创建CachedThreadPool的源码
```
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```
其特点是：

1. 只有最大线程数为int的最大值
2. 超时时间为60秒
3. 当整个线程池都出去闲置状态超过60秒的时候，会被回收
4. 几乎不占用任何系统资源

**二、CachedThreadPool的使用**

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ExecutorService cachedThreadExecutor = Executors.newCachedThreadPool();

        for (int i = 0; i < 10; i++) {
            MyRunnable myRunnable = new MyRunnable(i);
            cachedThreadExecutor.execute(myRunnable);
        };
    }

    /**
     * 模拟耗时任务
     */
    class MyRunnable implements Runnable{

        int threadNum;

        MyRunnable(int threadNum){
            this.threadNum = threadNum;
        }

        @Override
        public void run() {
            Log.e("打印信息：","等待中...执行第"+threadNum+"号线程");
            try {
                Thread.currentThread().sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Log.e("打印信息：","完成第"+threadNum+"号线程");
        }
    }
}
```

```
12-08 08:44:41.885 8497-8522/? E/打印信息：: 等待中...执行第1号线程
12-08 08:44:41.885 8497-8523/? E/打印信息：: 等待中...执行第2号线程
12-08 08:44:41.886 8497-8525/? E/打印信息：: 等待中...执行第4号线程
12-08 08:44:41.886 8497-8524/? E/打印信息：: 等待中...执行第3号线程
12-08 08:44:41.887 8497-8521/? E/打印信息：: 等待中...执行第0号线程
12-08 08:44:41.888 8497-8526/? E/打印信息：: 等待中...执行第5号线程
12-08 08:44:41.890 8497-8527/? E/打印信息：: 等待中...执行第6号线程
12-08 08:44:41.890 8497-8528/? E/打印信息：: 等待中...执行第7号线程
12-08 08:44:41.892 8497-8530/? E/打印信息：: 等待中...执行第9号线程
12-08 08:44:41.893 8497-8529/? E/打印信息：: 等待中...执行第8号线程
12-08 08:44:45.885 8497-8522/com.hensen.boke E/打印信息：: 完成第1号线程
12-08 08:44:45.885 8497-8523/com.hensen.boke E/打印信息：: 完成第2号线程
12-08 08:44:45.886 8497-8525/com.hensen.boke E/打印信息：: 完成第4号线程
12-08 08:44:45.886 8497-8524/com.hensen.boke E/打印信息：: 完成第3号线程
12-08 08:44:45.887 8497-8521/com.hensen.boke E/打印信息：: 完成第0号线程
12-08 08:44:45.888 8497-8526/com.hensen.boke E/打印信息：: 完成第5号线程
12-08 08:44:45.890 8497-8527/com.hensen.boke E/打印信息：: 完成第6号线程
12-08 08:44:45.890 8497-8528/com.hensen.boke E/打印信息：: 完成第7号线程
12-08 08:44:45.892 8497-8530/com.hensen.boke E/打印信息：: 完成第9号线程
12-08 08:44:45.893 8497-8529/com.hensen.boke E/打印信息：: 完成第8号线程
```

##**<span id="f">四大线程池之ScheduledThreadPool</span>**
**一、ScheduledThreadPool的分析**
ScheduledThreadPool是一种核心线程数量时固定的，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收。下面是创建ScheduledThreadPool的源码

```
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
    return new ScheduledThreadPoolExecutor(corePoolSize);
}

public ScheduledThreadPoolExecutor(int corePoolSize) {
    super(corePoolSize, Integer.MAX_VALUE,
          DEFAULT_KEEPALIVE_MILLIS, MILLISECONDS,
          new DelayedWorkQueue());
}
```
其特点是：

1. 核心数量固定
2. 非核心数量无限制
3. 非核心线程闲置时立即被回收

**二、ScheduledThreadPool的使用**

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ExecutorService scheduledThreadExecutor = Executors.newScheduledThreadPool(5);

        for (int i = 0; i < 10; i++) {
            MyRunnable myRunnable = new MyRunnable(i);
            scheduledThreadExecutor.execute(myRunnable);
        };
    }

    /**
     * 模拟耗时任务
     */
    class MyRunnable implements Runnable{

        int threadNum;

        MyRunnable(int threadNum){
            this.threadNum = threadNum;
        }

        @Override
        public void run() {
            Log.e("打印信息：","等待中...执行第"+threadNum+"号线程");
            try {
                Thread.currentThread().sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Log.e("打印信息：","完成第"+threadNum+"号线程");
        }
    }
}
```

```
12-08 08:54:25.339 15130-15157/? E/打印信息：: 等待中...执行第0号线程
12-08 08:54:25.341 15130-15156/? E/打印信息：: 等待中...执行第1号线程
12-08 08:54:25.347 15130-15158/? E/打印信息：: 等待中...执行第2号线程
12-08 08:54:25.347 15130-15159/? E/打印信息：: 等待中...执行第3号线程
12-08 08:54:25.349 15130-15160/? E/打印信息：: 等待中...执行第4号线程
12-08 08:54:29.340 15130-15157/com.hensen.boke E/打印信息：: 完成第0号线程
12-08 08:54:29.340 15130-15157/com.hensen.boke E/打印信息：: 等待中...执行第5号线程
12-08 08:54:29.346 15130-15156/com.hensen.boke E/打印信息：: 完成第1号线程
12-08 08:54:29.346 15130-15156/com.hensen.boke E/打印信息：: 等待中...执行第6号线程
12-08 08:54:29.347 15130-15158/com.hensen.boke E/打印信息：: 完成第2号线程
12-08 08:54:29.347 15130-15158/com.hensen.boke E/打印信息：: 等待中...执行第7号线程
12-08 08:54:29.348 15130-15159/com.hensen.boke E/打印信息：: 完成第3号线程
12-08 08:54:29.348 15130-15159/com.hensen.boke E/打印信息：: 等待中...执行第8号线程
12-08 08:54:29.349 15130-15160/com.hensen.boke E/打印信息：: 完成第4号线程
12-08 08:54:29.349 15130-15160/com.hensen.boke E/打印信息：: 等待中...执行第9号线程
12-08 08:54:33.347 15130-15157/com.hensen.boke E/打印信息：: 完成第5号线程
12-08 08:54:33.347 15130-15156/com.hensen.boke E/打印信息：: 完成第6号线程
12-08 08:54:33.348 15130-15158/com.hensen.boke E/打印信息：: 完成第7号线程
12-08 08:54:33.348 15130-15159/com.hensen.boke E/打印信息：: 完成第8号线程
12-08 08:54:33.350 15130-15160/com.hensen.boke E/打印信息：: 完成第9号线程
```

##**<span id="g">四大线程池之SingleThreadExecutor</span>**
**一、SingleThreadExecutor的分析**
SingleThreadExecutor是一种只有一个核心线程的线程池，它确保所有的任务都在同一个线程中按顺序执行。下面是创建SingleThreadExecutor的源码

```
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```
其特点是：

1. 只有一个核心线程

**二、SingleThreadExecutor的使用**

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ExecutorService singleThreadExecutor = Executors.newSingleThreadExecutor();

        for (int i = 0; i < 10; i++) {
            MyRunnable myRunnable = new MyRunnable(i);
            singleThreadExecutor.execute(myRunnable);
        };
    }

    /**
     * 模拟耗时任务
     */
    class MyRunnable implements Runnable{

        int threadNum;

        MyRunnable(int threadNum){
            this.threadNum = threadNum;
        }

        @Override
        public void run() {
            Log.e("打印信息：","等待中...执行第"+threadNum+"号线程");
            try {
                Thread.currentThread().sleep(4000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Log.e("打印信息：","完成第"+threadNum+"号线程");
        }
    }
}
```

```
12-08 08:58:16.721 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第0号线程
12-08 08:58:20.722 18100-18136/com.hensen.boke E/打印信息：: 完成第0号线程
12-08 08:58:20.722 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第1号线程
12-08 08:58:24.724 18100-18136/com.hensen.boke E/打印信息：: 完成第1号线程
12-08 08:58:24.724 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第2号线程
12-08 08:58:28.725 18100-18136/com.hensen.boke E/打印信息：: 完成第2号线程
12-08 08:58:28.725 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第3号线程
12-08 08:58:32.726 18100-18136/com.hensen.boke E/打印信息：: 完成第3号线程
12-08 08:58:32.726 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第4号线程
12-08 08:58:36.728 18100-18136/com.hensen.boke E/打印信息：: 完成第4号线程
12-08 08:58:36.728 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第5号线程
12-08 08:58:40.731 18100-18136/com.hensen.boke E/打印信息：: 完成第5号线程
12-08 08:58:40.731 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第6号线程
12-08 08:58:44.733 18100-18136/com.hensen.boke E/打印信息：: 完成第6号线程
12-08 08:58:44.733 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第7号线程
12-08 08:58:48.734 18100-18136/com.hensen.boke E/打印信息：: 完成第7号线程
12-08 08:58:48.734 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第8号线程
12-08 08:58:52.735 18100-18136/com.hensen.boke E/打印信息：: 完成第8号线程
12-08 08:58:52.735 18100-18136/com.hensen.boke E/打印信息：: 等待中...执行第9号线程
12-08 08:58:56.737 18100-18136/com.hensen.boke E/打印信息：: 完成第9号线程
```

>回过头来看，是不是觉得线程池很简单呢，只要配置好了线程池的作用，直接往里面塞线程就行了。

**简单记忆（以线程数量为线索）**

1. FixedThreadPool：核心固定，非核心为固定
2. CachedThreadPool：核心为0，非核心无限制
3. ScheduledThreadPool：核心固定，非核心无限制
4. SingleThreadExecutor：核心为1，非核心为1