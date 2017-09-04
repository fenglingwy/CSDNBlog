#多线程系列之Semaphore、CyclicBarrier、CountDownLatch


----------
##**前言**
今天向大家介绍的是多线程开发中的一些辅助类，他们的作用无非就是帮助我们让多个线程按照我们想要的执行顺序来执行。如果我们按照文字来理解Semaphore、CyclicBarrier、CountDownLatch可能会有点难度，如果看完实例再来看文字会恍然大悟。不用担心，今天带领大家用生活例子来理解这三个类，废话不多说，开车啦

>今天无意间看到鸿神的并发系列，觉得例子很不错，所以后面我又修改添加鸿神的几个例子帮助大家理解

##**Semaphore**
Semaphore是一个计数信号量。信号量中维护着一个信号量许可集，线程可以通过调用acquire()来获取信号量的许可。当信号量被许可时，线程可以向下执行，否则线程等待。同时，线程也可以通过release()来释放它的信号量。Semaphore简单的可以理解为一张许可证

###**① 饭堂打饭**
```
public class SemaphoreActivity extends AppCompatActivity {

    //线程数
    private static final int THREAD_SIZE = 5;
    private Semaphore semaphore;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_semaphore);

        semaphore = new Semaphore(3);

        for (int i = 0; i < THREAD_SIZE; i++) {
            new CanteenThread().start();
        }
    }

    private class CanteenThread extends Thread {
        @Override
        public void run() {
            try {
                //买到粮票
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + "号买到粮票");
                //模拟食堂打饭中
                System.out.println(Thread.currentThread().getName() + "号窗口开始打饭");
                Thread.sleep(2000);
                //提交粮票
                semaphore.release();
                System.out.println(Thread.currentThread().getName() + "号提交粮票");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

这里模拟了学生到食堂打饭的过程：买到粮票->打饭->提交粮票。其中semaphore代表食堂只有三张粮票的买卖，也可以理解为只有三个打饭窗口。THREAD_SIZE代表有五个学生线程同时打饭。**下面通过打印信息来查看执行过程**

```
Thread-636号买到粮票
Thread-636号窗口开始打饭
Thread-632号买到粮票
Thread-632号窗口开始打饭
Thread-635号买到粮票
Thread-635号窗口开始打饭

......隔了2秒后

Thread-636号提交粮票
Thread-633号买到粮票
Thread-633号窗口开始打饭
Thread-632号提交粮票
Thread-634号买到粮票
Thread-634号窗口开始打饭
Thread-635号提交粮票
Thread-633号提交粮票
Thread-634号提交粮票
```

>可以看到学生们打饭还是井然有序的，打完就撤，留给下一位。Semaphore就是一张通行证，灵活使用它，你就能指挥线程

###**② 打印机**

```
public class PrinterActivity extends AppCompatActivity {

    //线程数
    private static final int THREAD_SIZE = 5;
    private Semaphore semaphore;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_printer);

        semaphore = new Semaphore(1);

        for (int i = 0; i < THREAD_SIZE; i++) {
            new PrinterThread().start();
        }
    }

    private class PrinterThread extends Thread {
        @Override
        public void run() {
            try {
                semaphore.acquire();
                System.out.println(Thread.currentThread().getName() + "进入打印");
                Thread.sleep(1000);
                System.out.println(Thread.currentThread().getName() + "打印中...");
                System.out.println(Thread.currentThread().getName() + "退出打印");
                semaphore.release();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
这里模拟五个人抢占一部打印机的场景。**下面通过打印信息来查看执行过程**

```
Thread-231进入打印
Thread-231打印中...
Thread-231退出打印
Thread-229进入打印
Thread-229打印中...
Thread-229退出打印
Thread-228进入打印
Thread-228打印中...
Thread-228退出打印
Thread-230进入打印
Thread-230打印中...
Thread-230退出打印
Thread-227进入打印
Thread-227打印中...
Thread-227退出打印
```


##**CyclicBarrier**

CyclicBarrier是一个同步辅助类，可以让正在运行中的线程与其他线程在某一公共时刻进行同步。CyclicBarrier简单理解为闸门，当我们达到某一目标时，闸门即可打开。

###**① 短跑比赛**

```
public class MatchActivity extends AppCompatActivity {

    //线程数
    private static final int THREAD_SIZE = 5;
    private CyclicBarrier cb;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_match);

        cb = new CyclicBarrier(THREAD_SIZE, new Runnable() {
            @Override
            public void run() {
                //裁判
                System.out.println("参赛者" + cb.getParties() + "个全部准备完毕 --> 各就各位，预备跑");
            }
        });

        for (int i = 0; i < THREAD_SIZE; i++) {
            new AthleteThread().start();
        }

        System.out.println("主线程不用等待，继续执行");
    }

    private class AthleteThread extends Thread {
        @Override
        public void run() {
            try {
                //运动员
                System.out.println(Thread.currentThread().getName() + "号选手准备好了");
                cb.await();
                System.out.println(Thread.currentThread().getName() + "跑，跑，跑");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```
这里模拟短跑比赛的过程：全部运动员准备->裁判一声令下->全部运动员开跑。其中THREAD_SIZE代表五名运动员，CyclicBarrier代表裁判。**下面通过打印信息来查看这场比赛**

```
Thread-682号选手准备好了
Thread-683号选手准备好了
Thread-684号选手准备好了
主线程不用等待，继续执行
Thread-686号选手准备好了
Thread-685号选手准备好了
参赛者5个全部准备完毕 --> 各就各位，预备跑
Thread-685跑，跑，跑
Thread-682跑，跑，跑
Thread-683跑，跑，跑
Thread-684跑，跑，跑
Thread-686跑，跑，跑
```

>可以看到全部运动员们是在准备好之后开始起跑，这也就是我们所说的线程同步。而且它的特点是主线程不用等待，继续执行。

###**② 门禁系统**

```
public class DoorActivity extends AppCompatActivity {

    //线程数
    private static final int THREAD_SIZE = 10;
    private CyclicBarrier cb;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_door);

        cb = new CyclicBarrier(THREAD_SIZE, new Runnable() {
            @Override
            public void run() {
                System.out.println("人到齐了 --> 开门");
            }
        });

        for (int i = 0; i < THREAD_SIZE; i++) {
            new PeopleThread().start();
        }

        System.out.println("主线程不用等待，继续执行");
    }

    private class PeopleThread extends Thread {
        @Override
        public void run() {
            try {
                System.out.println(Thread.currentThread().getName() + "已刷卡 --> 等待开门回家");
                cb.await();
                System.out.println(Thread.currentThread().getName() + "放学回家");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```
这里模拟十个学生放学回家的场景。**下面通过打印信息来查看执行过程**

```
Thread-268已刷卡 --> 等待开门回家
Thread-267已刷卡 --> 等待开门回家
主线程不用等待，继续执行
Thread-271已刷卡 --> 等待开门回家
Thread-272已刷卡 --> 等待开门回家
Thread-269已刷卡 --> 等待开门回家
Thread-270已刷卡 --> 等待开门回家
Thread-274已刷卡 --> 等待开门回家
Thread-273已刷卡 --> 等待开门回家
Thread-275已刷卡 --> 等待开门回家
Thread-276已刷卡 --> 等待开门回家
人到齐了 --> 开门
Thread-276放学回家
Thread-268放学回家
Thread-267放学回家
Thread-271放学回家
Thread-272放学回家
Thread-269放学回家
Thread-270放学回家
Thread-274放学回家
Thread-273放学回家
Thread-275放学回家
```

##**CountDownLatch**

CountDownLatch也是一个同步辅助类，它可以设置一个或多个线程同时等待，直到条件被满足后，继续执行。

###**① 面试**

```
public class CountDownLatchActivity extends AppCompatActivity {

    //线程数
    private static final int THREAD_SIZE = 5;
    private CountDownLatch cdl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_count_down_latch);

        try {
            cdl = new CountDownLatch(THREAD_SIZE);

            for (int i = 0; i < THREAD_SIZE; i++) {
                new IntervieweeThread().start();
            }

            //开始面试
            System.out.println("主线程需要等待");
            cdl.await();
            System.out.println("主线程继续执行 --> 面试会议结束了");

        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    private class IntervieweeThread extends Thread {
        @Override
        public void run() {
            System.out.println(Thread.currentThread().getName() + "号面试者，开始面试");
            cdl.countDown();
            System.out.println(Thread.currentThread().getName() + "面试中...面试成功");
        }
    }
}
```

这里模拟面试的过程：面试者进来->面试完成->面试者进来->面试完成的循环过程。其中THREAD_SIZE代表五名面试者，CountDownLatch代表面试这场会议。**下面通过打印信息来查看面试过程**

```
Thread-757号面试者，开始面试
Thread-757面试中...面试成功
Thread-758号面试者，开始面试
Thread-758面试中...面试成功
Thread-759号面试者，开始面试
Thread-759面试中...面试成功
主线程需要等待
Thread-760号面试者，开始面试
Thread-760面试中...面试成功
Thread-761号面试者，开始面试
Thread-761面试中...面试成功
主线程继续执行 --> 面试会议结束了
```
>可以看到全部面试者是一个一个执行的，它的特点是可以让主线程进入等待状态，直到我们规定的五名面试者完成后才继续执行。

###**② 家人团圆饭**

```
public class FamilyActivity extends AppCompatActivity {

    private CountDownLatch cdl;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_family);

        cdl = new CountDownLatch(3);

        new Thread() {
            public void run() {
                fatherToRes();
                cdl.countDown();
            }
        }.start();

        new Thread() {
            public void run() {
                motherToRes();
                cdl.countDown();
            }
        }.start();

        new Thread() {
            public void run() {
                meToRes();
                cdl.countDown();
            }
        }.start();

        try {
            cdl.await();
            togetherToEat();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public static void fatherToRes() {
        System.out.println("爸爸步行去饭店需要3小时。");
    }

    public static void motherToRes() {
        System.out.println("妈妈挤公交去饭店需要2小时。");
    }

    public static void meToRes() {
        System.out.println("我乘地铁去饭店需要1小时。");
    }

    public static void togetherToEat() {
        System.out.println("一家人到齐了，开始吃饭");
    }
}
```
这里模拟一家三口回家吃团圆饭的场景，**下面通过打印信息来查看执行过程**
```
妈妈挤公交去饭店需要2小时。
爸爸步行去饭店需要3小时。
我乘地铁去饭店需要1小时。
一家人到齐了，开始吃饭
```

##**CyclicBarrier和CountDownLatch区别**
CyclicBarrier和CountDownLatch看起来很相似，但还是有一些不同点：

1. CountDownLatch的作用是允许1个或N个线程等待其他线程完成执行（刚才就是允许主线程等待），而CyclicBarrier则是允许N个线程相互等待。
2. CountDownLatch的计数器无法被重置，CyclicBarrier的计数器可以被重置后使用。

>今天到这里就结束了，关于多线程系列的其他内容，可以关注我的博客，近期会更新