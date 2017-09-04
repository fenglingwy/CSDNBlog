#多线程系列之Thread、Runnable、Callable、Future、FutureTask


----------
##**前言**
多线程一直是初学者最抵触的东西，如果你想进阶的话，那必须闯过这道难关，特别是多线程中Thread、Runnable、Callable、Future、FutureTask这几个类往往是初学者容易搞混的。**这里先总结这几个类特点和区别，让大家带着模糊印象来学习这篇文章**

1. Thread、Runnable、Callable：都是线程
2. Thread特点：提供了线程等待、线程睡眠、线程礼让等操作
3. Runnable和Callable特点：都是接口，并提供对应的实现方法
4. Runnable、Callable区别：Runnable无返回值，Callable有返回值
5. Future：提供了对Runnable和Callable任务的执行结果进行取消、查询是否完成、获取结果、设置结果等操作
6. FutureTask：Runnable和Future的结合体，即拥有Future的特性

>本篇文章包含以下内容

>* [Thread和Runnable的关系](#a)
>* [Runnable和Callable的区别](#b)
>* [Future](#c)
>* [FutureTask](#d)

##**<span id="a">Thread和Runnable的关系</span>**
我们对线程的使用，经常有这两种写法
```
new Thread(new Runnable() {
    @Override
    public void run() {
        //子线程操作
    }
}).start();

new Thread(){
    @Override
    public void run() {
        //子线程操作
    }
}.start();
```
这也就是我们要讲的Thread和Runnable的关系，其实它们是一样的，都是线程，最终启动线程后都会执行run()方法里面的内容，具体是怎么一回事，让我们从Thread的源码开始分析

```
class Thread implements Runnable {
	
	private Runnable target;
	
	//构造函数
	public Thread(ThreadGroup group, Runnable target) {
	    init(group, target, "Thread-" + nextThreadNum(), 0);
	}
	
	//继续追踪init()方法
	private void init(ThreadGroup g, Runnable target, String name, long stackSize) {
	    Thread parent = currentThread();
	    if (g == null) {
	        g = parent.getThreadGroup();
	    }
	
	    g.addUnstarted();
	    this.group = g;
	
	    this.target = target;
	    this.priority = parent.getPriority();
	    this.daemon = parent.isDaemon();
	    setName(name);
	
	    init2(parent);
	
	    /* Stash the specified stack size in case the VM cares */
	    this.stackSize = stackSize;
	    tid = nextThreadID();
	}

}
```

可以看到Thread就是实现Runnable的，所以Thread也可以说是Runnable，它俩就像亲生兄弟一样。由于我们创建线程的时候第一步都是new Thread()开始的，所以看到Thread的构造函数，继续追踪之后，你会发现如果传进来一个Runnable的话，会被赋予target的成员变量中，而target就是一个Runnable。接着，调用Thread的start()方法，我们查看start()方法的源码

```
public synchronized void start() {
    if (threadStatus != 0)
        throw new IllegalThreadStateException();

    group.add(this);

    started = false;
    try {
	    //最终调用这个方法
        nativeCreate(this, stackSize, daemon);
        started = true;
    } finally {
        try {
            if (!started) {
                group.threadStartFailed(this);
            }
        } catch (Throwable ignore) {
           
        }
    }
}

//继续追踪nativeCreate()方法
private native static void nativeCreate(Thread t, long stackSize, boolean daemon);
```
我们可以看到start()方法在最后是调用了本地底层的nativeCreate()方法，这个方法我们大胆的猜测，应该是执行Thread里面的run()方法，因为我们开启线程之后，线程总是会执行run()里面的内容，所以这里应该就是调用了Thread的run()方法。我们查看run()方法的源码
```
@Override
public void run() {
    if (target != null) {
        target.run();
    }
}
```
到这里，实际上最终被线程执行的任务是Runnable而非Thread，Thread只是对Runnable的包装。如果target不为空则执行target的run()方法，否则执行当前对象的run()方法

##**<span id="b">Runnable和Callable的区别</span>**
Runnable和Callable同样都是线程，而它们的区别可以从其源码中看出来，下面是Runnable和Callable的源码

```
public interface Runnable {
      
    public abstract void run();
}

@FunctionalInterface
public interface Callable<V> {

    V call() throws Exception;
}
```
可以发现，Callable带有一个泛型的返回值，而Runnable并没有返回值，所以使用Callable可以返回线程的结果，而Runnable不行

##**<span id="c">Future</span>**
Future不是线程，它可以理解为管理线程的人，不过它对线程的管理方法不是很多，这点可以从Future的源码中看出来，下面是Future的源码

```
public interface Future<V> {
    //取消任务
    boolean cancel(boolean mayInterruptIfRunning);
	//该任务是否已经取消
    boolean isCancelled();
	//该任务是否已经完成
    boolean isDone();
	//获取任务的返回结果，该方法会阻塞线程
    V get() throws InterruptedException, ExecutionException;
	//获取任务的返回结果，该方法会阻塞线程
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```
关说不练假把式，我们通过例子来理解，这里的线程我们都使用线程池来执行，如果对线程池不理解的，可以查看我的博客[Android进阶——多线程系列之四大线程池的使用介绍](http://blog.csdn.net/qq_30379689/article/details/53522085)

```
public class ThreadActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_thread);
		
		//线程池
        ExecutorService executor = Executors.newFixedThreadPool(10);

		//第一步：创建线程
        MyRunnable myRunnable = new MyRunnable();
        MyCallable myCallable = new MyCallable();

        try {
	        //第二步：执行线程带有返回值
            Future<Integer> CallableFuture = executor.submit(myCallable);
            //get()方法会使线程阻塞
            System.out.println("CallableFuture获取的结果：" + CallableFuture.get());

            System.out.println("CallableFuture isCancelled（是否已经取消）:" + CallableFuture.isCancelled());
            System.out.println("CallableFuture isDone（是否已经完成）:" + CallableFuture.isDone());

            //第二步：执行线程没有返回值
            Future<?> RunnableFuture = executor.submit(myRunnable);
            System.out.println("RunnableFuture获取的结果：" + RunnableFuture.get());

            System.out.println("RunnableFuture isCancelled（是否已经取消）:" + RunnableFuture.isCancelled());
            System.out.println("RunnableFuture isDone（是否已经完成）:" + RunnableFuture.isDone());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    public class MyRunnable implements Runnable {
        @Override
        public void run() {
            System.out.println("我是Runnable");
        }
    }

    public class MyCallable implements Callable<Integer> {
        @Override
        public Integer call() throws Exception {
            System.out.println("我是Callable");
            return 10;
        }
    }
}
```
第一步：创建线程MyCallable中是实现Callable的，泛型里面填返回值的类型，而Runnable则不用
第二步：通过线程池executor提交线程，返回对应的Future< ?>对象，若有返回值则填返回值类型，若无返回值则填？号

**下面我们通过打印信息来查看Future管理类的执行结果**，这里再一次的证明：Runnable和Callable的区别

```
我是Callable
CallableFuture获取的结果：10
CallableFuture isCancelled（是否已经取消）:false
CallableFutureisDone（是否已经完成）:true
我是Runnable
RunnableFuture获取的结果：null
RunnableFuture isCancelled（是否已经取消）:false
RunnableFuture isDone（是否已经完成）:true
```


##**<span id="d">FutureTask</span>**

FutureTask是Future的实现类，而且不仅是Future又是Runnable，还包装了Callable，它是这两者的合体。从FutureTask的源码可以看出

```
//继续追踪RunnableFuture
public class FutureTask<V> implements RunnableFuture<V> {
	 //构造方法
	 public FutureTask(Callable<V> callable) {
        if (callable == null)
            throw new NullPointerException();
        this.callable = callable;
        this.state = NEW;       
     }
     //构造方法
     public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;      
     }
}
```

```
//RunnableFuture继承Runnable、Future
public interface RunnableFuture<V> extends Runnable, Future<V> {

    void run();
}
```

源码中可以看出构造函数中需要传进去一个Callable或者是Runnable，如果传进去是个Runnable则会被Executors.callable()转换为Callable类型的任务，即FutureTask最终都是执行Callable类型的任务。继续追踪Executors.callable()

```
//Executors.callable()方法
public static <T> Callable<T> callable(Runnable task, T result) {
    if (task == null)
        throw new NullPointerException();
    return new RunnableAdapter<T>(task, result);
}

//继续追踪RunnableAdapter<T>类
private static final class RunnableAdapter<T> implements Callable<T> {
    private final Runnable task;
    private final T result;
    RunnableAdapter(Runnable task, T result) {
        this.task = task;
        this.result = result;
    }
    public T call() {
        task.run();
        return result;
    }
}
```
由于FutureTask实现了Runnable，因此，它既可以通过Thread包装来直接运行，也可以通过给ExecuteService来执行，并且还可以通过get()方法获取执行结果，该函数会阻塞，直到结果返回。下面通过简单的例子演示FutureTask的用法

```
public class FutureTaskActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_future_task);

        //线程池
        ExecutorService executor = Executors.newFixedThreadPool(10);
        //创建线程
        MyCallable callable = new MyCallable();
        //包装线程
        FutureTask<String> futureTask = new FutureTask<String>(callable);
        //执行线程
        executor.submit(futureTask);
        //获取结果
        try {
            System.out.println(futureTask.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }

    public class MyCallable implements Callable<String> {

        @Override
        public String call() throws Exception {
            Thread.sleep(2000);
            return "我是Callable返回值";
        }
    }
}
```
其打印信息是

```
我是Callable返回值
```

>好了，这一章介绍就到这里，对于多线程的开发还需多多实践才能真正理解它的实际意义，更多的多线程学习可以关注我的博客

