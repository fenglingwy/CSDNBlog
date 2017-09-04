#多线程系列之wait、notify、sleep、join、yield、synchronized关键字、ReentrantLock锁


----------
##**前言**
多线程一直是初学者最困惑的地方，每次看到一篇文章，觉得很有难度，就马上叉掉，不看了，我以前也是这样过来的。后来，我发现这样的态度不行，知难而退，永远进步不了。于是，我狠下心来看完别人的博客，尽管很难但还是咬着牙，不懂去查阅资料，到最后弄懂整个过程。虽然花费时间很大，但这就是自学的精髓，别人学不会，而我却学到了。很简单的一个例子，一开始我对自定义View也是很抵触，看到很难的图就不去思考他，故意避开它，然而当我看到自己喜欢的雷达图时，很有兴趣的去查阅资料，不知不觉，自定义View对我已经没有难度了。所以对于多线程我也是0基础，不过我还是咬着牙皮，该学的还是得学。**这里先总结这几个类特点和区别，让大家带着模糊印象来学习这篇文章**

1. Thread是个线程，而且有自己的生命周期
2. 对于线程常用的操作有：wait（等待）、notify（唤醒）、notifyAll、sleep（睡眠）、join（阻塞）、yield（礼让）
3. wait、notify、notifyAll都必须在synchronized中执行，否则会抛出异常
4. synchronized关键字和ReentrantLock锁都是辅助线程同步使用的
5. 初学者常犯的误区：一个对象只有一个锁（正确的）

>本篇文章包含以下内容

>* [线程同步之synchronized关键字](#a)
>* [线程同步之ReentrantLock锁](#b)
>* [线程的生命周期的介绍](#c)
>* [线程的等待唤醒机制之wait()、notify()、notifyAll()](#d)
>* [线程的sleep()、join()、yield()](#e)

##**<span id="a">线程同步之synchronized关键字</span>**
马上就过年了，火车抢票又是一年沸沸扬扬的事情，这也就好比我们的多线程抢夺资源是一个道理，下面我们通过火车抢票的案例来理解
```
public class SyncActivity extends AppCompatActivity {

    private int ticket = 10;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sync);

        for (int i = 0; i < 10; i++) {
            new Thread() {
                @Override
                public void run() {
	                //买票
                    sellTicket();
                }
            }.start();
        }
    }

    public void sellTicket() {
        ticket--;
        System.out.println("剩余的票数：" + ticket);
    }
}
```
这里我们通过开启十个线程来购买火车票，不过火车票只有十张，**下面通过打印信息来看一下抢票的情况**

```
剩余的票数：9
剩余的票数：8
剩余的票数：7
剩余的票数：6
剩余的票数：5
剩余的票数：1
剩余的票数：1
剩余的票数：1
剩余的票数：1
剩余的票数：0
```
可以发现，票数出现了误差，这明显就是不行的，这也是因为开启了十个线程，大家都抢着自己的票。上面这种情况是因为其中有四个线程都挤在一起了，然后一起执行了【ticket--;】，接着再一起执行【System.out.println("剩余的票数：" + ticket);】导致的。那么该如何保证大家都是能够自觉排队，井然有序的抢票呢。这个时候就要用到synchronized关键字

**方法一：我们在方法上添加synchronized关键字**
```
public class SyncActivity extends AppCompatActivity {

    private int ticket = 10;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sync);

        for (int i = 0; i < 10; i++) {
            new Thread() {
                @Override
                public void run() {
	                //买票
                    sellTicket();
                }
            }.start();
        }
    }
	
	//添加在这里
    public synchronized void sellTicket() {
        ticket--;
        System.out.println("剩余的票数：" + ticket);
    }
}
```
这样就表示这个方法是同步的，只能由一个个线程来争夺里面的资源，**下面通过打印信息可以验证**

```
剩余的票数：9
剩余的票数：8
剩余的票数：7
剩余的票数：6
剩余的票数：5
剩余的票数：4
剩余的票数：3
剩余的票数：2
剩余的票数：1
剩余的票数：0
```
**方法二：我们在方法内添加synchronized关键字**
```
public class SyncActivity extends AppCompatActivity {

    private int ticket = 10;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_sync);

        for (int i = 0; i < 10; i++) {
            new Thread() {
                @Override
                public void run() {
	                //买票
                    sellTicket();
                }
            }.start();
        }
    }
	
	//添加在这里
	Object lock = new Object();
    public void sellTicket() {
        synchronized(lock){
            ticket--;
            System.out.println("剩余的票数：" + ticket);
        }
    }
}
```
其实，synchronized关键字可以理解为一个锁，而锁就需要被锁的东西，所以synchronized又分为**类锁和对象锁**，即可以锁类又可以锁对象，它们共同的作用就是保证线程的同步。就好比如我们上面中synchronized(lock)，就是对象锁，将Object对象锁起来

**一、类锁和对象锁的概念**
对象锁和类锁在锁的概念上基本上和内置锁是一致的，但是在多线程访问时，两个锁实际是有很大的区别的，对象锁是用于对象实例方法，或者一个对象实例上的，类锁是用于类的静态方法或者一个类的class对象上的。我们知道，类的对象实例可以有很多个，但是每个类只有一个class对象，所以，**结论是：1、不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。2、而且类锁和对象锁互相不干扰。**

**二、对象锁**
类锁创建如下两种方法
```
public class SynchronizedDemo {
    //同步方法，对象锁
    public synchronized void syncMethod() {
        
    }

    //同步块，对象锁
    public void syncThis() {
        synchronized (this) {
           
        }
    }
}
```

**三、类锁**
对象锁创建如下两种方法
```
public class SynchronizedDemo {
    //同步class对象，类锁
    public void syncClassMethod() {
        synchronized (SynchronizedDemo.class) {
            
        }
    }

    //同步静态方法，类锁
    public static synchronized void syncStaticMethod(){

    }
}
```
**四、通过例子理解结论和概念**
根据类锁和对象锁的概念，我们来通过例子验证一下其正确性，这里演示两个对象锁和一个类锁，我们创建一个类

```
public class SynchronizedDemo {
	 private int ticket = 10;
    //同步方法，对象锁
    public synchronized void syncMethod() {
        for (int i = 0; i < 1000; i++) {
            ticket--;
            System.out.println(Thread.currentThread().getName() + "剩余的票数：" + ticket);
        }
    }

    //同步块，对象锁
    public void syncThis() {
        synchronized (this) {
            for (int i = 0; i < 1000; i++) {
                ticket--;
                System.out.println(Thread.currentThread().getName() + "剩余的票数：" + ticket);
            }
        }
    }

    //同步class对象，类锁
    public void syncClassMethod() {
        synchronized (SynchronizedDemo.class) {
            for (int i = 0; i < 50; i++) {
                ticket--;
                System.out.println(Thread.currentThread().getName() + "剩余的票数：" + ticket);
            }
        }
    }
}
```

**情况一：同一个对象，使用两个线程调用不同对象锁**

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_sync);

    final SynchronizedDemo synchronizedDemo = new SynchronizedDemo();

	//线程一
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo.syncMethod();
        }
    }.start();
	//线程二
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo.syncThis();
        }
    }.start();
}
```
>由于使用的是同一个对象的对象锁，所以执行出来的结果是同步的（即先运行线程一，等线程一运行完后运行线程二，ticket有序的减少），这里使用1000比较大的数字是为了一次能看出效果

```
Thread-1611剩余的票数：7
Thread-1611剩余的票数：6
Thread-1611剩余的票数：5
Thread-1611剩余的票数：4
Thread-1611剩余的票数：3
Thread-1611剩余的票数：2
```

**情况二：不同对象，使用两个线程调用同个对象锁**

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_sync);

    final SynchronizedDemo synchronizedDemo1 = new SynchronizedDemo();
    final SynchronizedDemo synchronizedDemo2 = new SynchronizedDemo();

	//线程一
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo1.syncMethod();
        }
    }.start();
	//线程二
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo2.syncMethod();
        }
    }.start();
}
```
>由于是不同对象，所以执行的对象锁都不是不同的，其结果是两个线程互相抢占资源的运行，即ticket偶尔会无序的减少

```
Thread-1667剩余的票数：-1612
Thread-1667剩余的票数：-1613
Thread-1668剩余的票数：-1630
Thread-1668剩余的票数：-1631
Thread-1668剩余的票数：-1632
```

**情况三：同一个对象，使用两个线程调用一个对象锁一个类锁**

```
@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_sync);

    final SynchronizedDemo synchronizedDemo = new SynchronizedDemo();

    //线程一
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo.syncMethod();
        }
    }.start();
    //线程二
    new Thread() {
        @Override
        public void run() {
            synchronizedDemo.syncClassMethod();
        }
    }.start();
}
```
>由于对象锁和类锁互不干扰，所以也是线程不安全的

```
Thread-1667剩余的票数：-1612
Thread-1667剩余的票数：-1613
Thread-1668剩余的票数：-1630
Thread-1668剩余的票数：-1631
Thread-1668剩余的票数：-1632
```

**这里再温习一下结论：1、不同对象实例的对象锁是互不干扰的，但是每个类只有一个类锁。2、而且类锁和对象锁互相不干扰。**

>不知不觉synchronized介绍了那么多，本可以放单独一篇文章的，不过后面的不多，认真看的人应该有点收获

##**<span id="b">线程同步之ReentrantLock锁</span>**
Java6.0增加了一种新的机制：ReentrantLock。ReentrantLock比synchronized理解简单多了，下面看ReentrantLock的使用

```
public class RenntrantLockActivity extends AppCompatActivity {

    Lock lock;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_renntrant_lock);

        lock = new ReentrantLock();
        doSth();
    }

    public void doSth() {
        lock.lock();
        try {
            //这里执行线程同步操作

        } finally {
            lock.unlock();
        }
    }
}
```
使用ReentrantLock很好理解，就好比我们现实的锁头是一样道理的。使用ReentrantLock的一般组合是lock与unlock成对出现的，需要注意的是，千万不要忘记调用unlock来释放锁，否则可能会引发死锁等问题。如果忘记了在finally块中释放锁，可能会在程序中留下一个定时炸弹，随时都会炸了，而是用synchronized，JVM将确保锁会获得自动释放，这也是为什么Lock没有完全替代掉synchronized的原因

##**<span id="c">线程的生命周期的介绍</span>**

线程也有属于自己的生命周期，这里使用我画的一张图来理解，在下面我们会讲解这个有关生命周期的一些方法的使用

![](http://img.blog.csdn.net/20161227010007105?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**<span id="d">线程的等待唤醒机制之wait()、notify()、notifyAll()</span>**
一开始我们也提到了wait、notify、notifyAll都必须在synchronized中执行，否则会抛出异常。所以下面以一个简单的例子来介绍线程的等待唤醒机制

```
public class WaitAndNotifyActivity extends AppCompatActivity {

    private static Object lockObject = new Object();

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_wait_and_notify);

        System.out.println("主线程运行");
        //创建子线程
        Thread thread = new WaitThread();
        thread.start();

        long start = System.currentTimeMillis();
        synchronized (lockObject) {
            try {
                System.out.println("主线程等待");
                lockObject.wait();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("主线程继续 --> 等待的时间：" + (System.currentTimeMillis() - start));
        }
    }

    class WaitThread extends Thread {
        @Override
        public void run() {
            synchronized (lockObject) {
                try {
                    //子线程等待了2秒钟后唤醒lockObject锁
                    Thread.sleep(2000);
                    lockObject.notifyAll();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
可以看到，我们使用的是同一个对象的锁，和同一个对象执行的wait()和notify()才会保证了我们的线程同步。当主线程执行到wait()方法时，代表主线程等待，让出使用权让子线程执行，这个时候主线程等待这一事件会被加进到【等待唤醒的队列】中。然后子线程则是两秒钟后执行notify()方法唤醒等待【唤醒队列中】的第一个线程，这里指的是主线程。而notifyAll()方法则是唤醒整个【唤醒队列中】的所有线程，这里就不多加演示了

下面采用一道经典的Java多线程面试题来让大家练习熟悉熟悉：子线程循环10次，接着主线程循环15次，接着又回到子线程循环10次，接着再回到主线程又循环15次，如此循环50次

```
//子线程
new Thread() {
    @Override
    public void run() {
        for (int i = 0; i < 50; i++) {
            for (int j = 0; j < 10; j++) {
                System.out.println("子循环循环第" + (j + 1) + "次");
            }
            System.out.println("--> 子线程循环了" + (i + 1) + "次");
        }
    }
}.start();
//主线程
for (int i = 0; i < 50; i++) {
    for (int j = 0; j < 15; j++) {
        System.out.println("主循环循环第" + (j + 1) + "次");
    }
    System.out.println("--> 主线程循环了" + (i + 1) + "次");
}
```

首先是主要思路的搭建，现在的问题就是如何让子线程和主线程有序的执行呢，那肯定是我们的等待唤醒机制

```
//子线程
new Thread() {
    @Override
    public void run() {
        for (int i = 0; i < 50; i++) {
            synchronized (lock){

                for (int j = 0; j < 10; j++) {
                    System.out.println("子循环循环第" + (j + 1) + "次");
                }
                //唤醒
                lock.notify();
                //等待
                try {
                    lock.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}.start();
//主线程
for (int i = 0; i < 50; i++) {
    synchronized (lock){
        //等待
        try {
            lock.wait();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        for (int j = 0; j < 15; j++) {
            System.out.println("主循环循环第" + (j + 1) + "次");
        }
        //唤醒
        lock.notify();
    }
}
```
不管是主线程先运行还是子线程运行，两个线程只能同时进入synchronized (lock)一个锁中。由于是子线程先运行：1、当主线程先进入synchronized (lock)锁时，它就必须是等待，而子线程开始运行输出，输出后就唤醒主线程。2、当子线程先运行的话，那就直接输出，然后等待主线程的运行输出

##**<span id="e">线程的sleep()、join()、yield()</span>**
**一、sleep()**
sleep()作用是让线程休息指定的时间，时间一到就继续运行，它的使用很简单

```
try {
    Thread.sleep(2000);
} catch (InterruptedException e) {
    e.printStackTrace();
}
```

**二、join()**
join()作用是让指定的线程先执行完再执行其他线程，而且会阻塞主线程，它的使用也很简单

```
public class JoinActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_join);

        //启动线程一
        try {
            MyThread myThread1 = new MyThread("线程一");
            myThread1.start();
            myThread1.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("主线程需要等待");

        //启动线程二
        try {
            MyThread myThread2 = new MyThread("线程二");
            myThread2.start();
            myThread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        System.out.println("主线程继续执行");
    }

    class MyThread extends Thread {

        public MyThread(String name) {
            super(name);
        }

        @Override
        public void run() {
            System.out.println(getName() + "在运行");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```
这里就不解释了，看打印信息，你就能发现它的作用了

```
线程一在运行
主线程需要等待
线程二在运行
主线程继续执行
```
**三、yield()**
yield()的作用是指定线程先礼让一下别的线程的先执行，就好比公交车只有一个座位，谁礼让了谁就坐上去。特别注意的是：**yield()会礼让给相同优先级的或者是优先级更高的线程执行，不过yield()这个方法只是把线程的执行状态打回准备就绪状态，所以执行了该方法后，有可能马上又开始运行，有可能等待很长时间**

```
public class YieldActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_yield);

        MyThread myThread1 = new MyThread("线程一");
        MyThread myThread2 = new MyThread("线程二");

        myThread1.start();
        myThread2.start();
    }

    class MyThread extends Thread {

        public MyThread(String name) {
            super(name);
        }

        @Override
        public synchronized void run() {
            for (int i = 0; i < 100; i++) {
                System.out.println(getName() + "在运行，i的值为：" + i + " 优先级为：" + getPriority());
                if (i == 2) {
                    System.out.println(getName() + "礼让");
                    Thread.yield();
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }
}
```
这里我们通过Thread.sleep()的方式，让线程强行延迟一秒回到准备就绪状态，这样在打印信息上就能看到我们想要的结果了

```
线程二在运行，i的值为：0 优先级为：5
线程二在运行，i的值为：1 优先级为：5
线程二在运行，i的值为：2 优先级为：5
线程二礼让
线程一在运行，i的值为：0 优先级为：5
线程一在运行，i的值为：1 优先级为：5
线程一在运行，i的值为：2 优先级为：5
线程一礼让
线程二在运行，i的值为：3 优先级为：5
线程二在运行，i的值为：4 优先级为：5
线程二在运行，i的值为：5 优先级为：5
线程二在运行，i的值为：6 优先级为：5
......
```

>好了，关于线程的介绍就这么多，可能知识点有点多，我自己也学习了好几天来掌握线程，这里的分享我都是测试过的。学习一遍才知道原来是这么一回事，没学习之前看别人的文章还是懂的，当自己码一遍的时候会发现写不出来，原因是没有真正理解线程。现在理解了线程之后，写出来会根据它的作用和思路来写，根本不用记代码
