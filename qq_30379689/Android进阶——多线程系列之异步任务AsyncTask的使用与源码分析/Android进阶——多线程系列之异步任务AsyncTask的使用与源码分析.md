#多线程系列之异步任务AsyncTask的使用与源码分析


----------
##**AsyncTask是什么**
AsyncTask是一种轻量级的异步任务类，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并主线程中更新UI，通过AsyncTask可以更加方便执行后台任务以及在主线程中访问UI，但是AsyncTask并不适合进行特别耗时的后台任务，对于特别耗时的任务来说，建议使用线程池
##**AsyncTask的使用**
我们简单的模拟下载文件的案例来分析，我们创建自己的异步类继承AsyncTask
```
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {

    @Override
    protected void onPreExecute() {
        //异步任务开启之前回调，在主线程中执行
        super.onPreExecute();
    }

    @Override
    protected Long doInBackground(URL... urls) {
        //执行异步任务，在线程池中执行
        long totalSize = 0;
        int i = 0;
        try {
            while (i < 100) {
                Thread.sleep(50);
                i = i + 5;
                publishProgress(i);
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        totalSize = totalSize + i;
        return totalSize;
    }

    @Override
    protected void onProgressUpdate(Integer... progress) {
        //当doInBackground中调用publishProgress时回调,在主线程中执行
        pb_progress.setProgress(progress[0]);
    }

    @Override
    protected void onPostExecute(Long result) {
        //在异步任务执行之后回调，在主线程中执行
        Toast.makeText(MainActivity.this, "下载完成，结果是" + result, Toast.LENGTH_SHORT).show();
    }

    @Override
    protected void onCancelled() {
        //在异步任务被取消时回调
        super.onCancelled();
    }
}
```
###**参数分析**
通过源码，可以看到AsyncTask是个抽象的泛型类

```
public abstract class AsyncTask<Params, Progress, Result>
```
* Params：表示后台任务执行时的参数类型（对应例子中的URL），该参数会传给AysncTask的doInBackground()方法
* Progress：表示后台任务的执行进度的参数类型（对应例子中的Integer），该参数会作为onProgressUpdate()方法的参数
* Result：表示后台任务的返回结果的参数类型（对应例子中的Long），该参数会作为onPostExecute()方法的参数

注意这三个参数都是代表的类型，如果AsyncTask确实不需要传递具体的参数，那么这三个泛型参数可以用Void来代替

###**方法分析**
常用的AsyncTask继承的方法有

* onPreExecute()：异步任务开启之前回调，在主线程中执行
* doInBackground()：执行异步任务，在线程池中执行
* onProgressUpdate()：当doInBackground中调用publishProgress时回调，在主线程中执行
* onPostExecute()：在异步任务执行之后回调，在主线程中执行
* onCancelled()：在异步任务被取消时回调

###**开启异步**
在主Activity中通过点击按钮执行我们创建的异步任务，然后通过execute()方法执行异步任务

```
bt_down.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        try {
            URL url = new URL("http://blog.csdn.net/");
            new DownloadFilesTask().execute(url);
        } catch (MalformedURLException e) {
            e.printStackTrace();
        }
    }
});
```
###**效果图**

![](http://img.blog.csdn.net/20161117203422520)

##**AsyncTask源码分析**

源码分析是基于**API24**的源码，我将会按照下面AsyncTask运行的过程来分析

###主分支

 1. 首先，**execute()方法**，开启异步任务
 2. 接着，**onPreExecute()方法**，异步任务开启前
 3. 接着，**doInBackground()方法**，异步任务正在执行
 4. 最后，**onPostExecute()方法**，异步任务完成

###次分支

 1. **onProgressUpdate()方法**，异步任务更新UI
 2. **onCancelled()方法**，异步任务取消

##**主分支部分**

代码开始的地方，是在创建AsyncTask类之后执行的**execute()方法**

```
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params);
}
```
execute()方法会调用executeOnExecutor()方法

```
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
    if (mStatus != Status.PENDING) {
        switch (mStatus) {
            case RUNNING:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task is already running.");
            case FINISHED:
                throw new IllegalStateException("Cannot execute task:"
                        + " the task has already been executed "
                        + "(a task can be executed only once)");
        }
    }

    mStatus = Status.RUNNING;

    onPreExecute();

    mWorker.mParams = params;
    exec.execute(mFuture);

    return this;
}
```
AsyncTask定义了一个mStatus变量，表示异步任务的运行状态，分别是PENDING、RUNNING、FINISHED，当只有PENDING状态时，AsyncTask才会执行，这样也就保证了AsyncTask只会被执行一次

继续往下执行，mStatus会被标记为RUNNING，接着执行，**onPreExecute()**，将参数赋值给mWorker，然后还有execute(mFuture)

这里的mWorker和mFuture究竟是什么，我们往下追踪，来到AsyncTask的构造函数中，可以找到这两个的初始化

```
public AsyncTask() {
    mWorker = new WorkerRunnable<Params, Result>() {
        public Result call() throws Exception {
            mTaskInvoked.set(true);

            Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
            //noinspection unchecked
            Result result = doInBackground(mParams);
            Binder.flushPendingCommands();
            return postResult(result);
        }
    };

    mFuture = new FutureTask<Result>(mWorker) {
        @Override
        protected void done() {
            try {
                postResultIfNotInvoked(get());
            } catch (InterruptedException e) {
                android.util.Log.w(LOG_TAG, e);
            } catch (ExecutionException e) {
                throw new RuntimeException("An error occurred while executing doInBackground()",
                        e.getCause());
            } catch (CancellationException e) {
                postResultIfNotInvoked(null);
            }
        }
    };
}
```
###**1)分析mWorker**

mWorker是一个WorkerRunnable对象，跟踪WorkerRunnable
```
private static abstract class WorkerRunnable<Params, Result> implements Callable<Result> {
    Params[] mParams;
}
```
实际上，WorkerRunnable是AsyncTask的一个抽象内部类，实现了Callable接口

###**2)分析mFuture **

mFuture是一个FutureTask对象，跟踪FutureTask

```
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```
实际上，FutureTask是java.util.concurrent包下的一个类，参数是个callable，并且将它赋值给FutureTask类中的callable

###**3)回过头来看**

回到我们AsyncTask初始化mFuture，这里的参数是mWorker也就不奇怪了，因为mWorker就是一个callable，我们在上面赋值给FutureTask类中的callable就是这个**mWorker**

```
mFuture = new FutureTask<Result>(mWorker)
```

而关于mWorker和mFuture的初始化早在我们Activity中初始化好了，因为构造函数是跟AsyncTask类的创建而执行的

```
new DownloadFilesTask()
```


----------


知道了mWorker和mFuture是什么后，我们回到原来的executeOnExecutor()方法，在这里将mWorker的参数传过去后，就开始用线程池execute这个mFuture

```
mWorker.mParams = params;
exec.execute(mFuture);
```
###**1)分析exec**

exec是通过executeOnExecutor()参数传进来的
```
public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params)
```
也就是我们execute()方法传过来的
```
public final AsyncTask<Params, Progress, Result> execute(Params... params) {
    return executeOnExecutor(sDefaultExecutor, params);
}
```
这里可以看到exec就是这个sDefaultExecutor

###**2)分析sDefaultExecutor**

我们跟踪这个sDefaultExecutor，截取有关它的代码

```
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;

private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```
从SerialExecutor可以发现，exec.execute(mFuture)就是在调用SerialExecutor类的execute(final Runnable r)方法，这里的参数r就是mFuture

继续往下走，SerialExecutor的execute()方法会将r封装成Runnable，并添加到mTasks任务队列中

继续往下走，如果这时候没有正在活动的AsyncTask任务，那么就会调用SerialExecutor的scheduleNext()方法，来执行下一个AsyncTask任务

```
if (mActive == null) {
    scheduleNext();
}
```

继续往下走，通过mTasks.poll()取出，将封装在mTask的Runnable交给mActive，最后真正执行的这个mActive的是THREAD_POOL_EXECUTOR，即执行的这个mActive，也就是包装在Runnable里面的mFuture

```
protected synchronized void scheduleNext() {
    if ((mActive = mTasks.poll()) != null) {
        THREAD_POOL_EXECUTOR.execute(mActive);
    }
}
```
mFuture被执行了，也就会执行它的run()方法

```
public void run() {
    try {
        r.run();
    } finally {
        scheduleNext();
    }
}
```


----------


我们跟踪到mFuture的run()方法中，切换到FutureTask类

```
public void run() {
    if (state != NEW ||
        !U.compareAndSwapObject(this, RUNNER, null, Thread.currentThread()))
        return;
    try {
        Callable<V> c = callable;
        if (c != null && state == NEW) {
            V result;
            boolean ran;
            try {
                result = c.call();
                ran = true;
            } catch (Throwable ex) {
                result = null;
                ran = false;
                setException(ex);
            }
            if (ran)
                set(result);
        }
    } finally {
        // runner must be non-null until state is settled to
        // prevent concurrent calls to run()
        runner = null;
        // state must be re-read after nulling runner to prevent
        // leaked interrupts
        int s = state;
        if (s >= INTERRUPTING)
            handlePossibleCancellationInterrupt(s);
    }
}
```

这一段代码其实就是将之前在mFutrue创建对象时候传进来的mWorker交给c

```
Callable<V> c = callable;
```

然后再调用c的call()方法，也就是mWorker的call()方法

```
result = c.call();
```


----------


代码又重新的定位到了mWorker类的call()方法
```
 mWorker = new WorkerRunnable<Params, Result>() {
    public Result call() throws Exception {
        mTaskInvoked.set(true);

        Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
        //noinspection unchecked
        Result result = doInBackground(mParams);
        Binder.flushPendingCommands();
        return postResult(result);
    }
};
```
可以发现，这里就调用了我们的**doInBackground()**方法，最后还返回postResult()，我们跟踪这个postResult()方法

```
private Result postResult(Result result) {
    @SuppressWarnings("unchecked")
    Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
            new AsyncTaskResult<Result>(this, result));
    message.sendToTarget();
    return result;
}
```
观察代码，首先是getHandler()，它是一个单例，返回sHandler

```
private static Handler getHandler() {
    synchronized (AsyncTask.class) {
        if (sHandler == null) {
            sHandler = new InternalHandler();
        }
        return sHandler;
    }
}
```
也就是说postResult方法会通过sHandler发送一个MESSAGE_POST_RESULT的消息，这个时候我们追踪到sHandler

```
private static InternalHandler sHandler;

private static class InternalHandler extends Handler {
    public InternalHandler() {
        super(Looper.getMainLooper());
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```

可以发现，sHandler收到MESSAGE_POST_PROGRESS消息后会调用result.mTask.finish(result.mData[0])，那么我们还必须知道result是个什么东西

```
AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
```
result是AsyncTask的内部类，实际上就是个实体类，用来存储变量的

```
private static class AsyncTaskResult<Data> {
    final AsyncTask mTask;
    final Data[] mData;

    AsyncTaskResult(AsyncTask task, Data... data) {
        mTask = task;
        mData = data;
    }
}
```

result.mTask也就是AsyncTask，最后调用result.mTask.finish(result.mData[0])，即AsyncTask的finish()方法，我们跟踪到finish()方法

```
private void finish(Result result) {
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    mStatus = Status.FINISHED;
}
```

这里判断AsyncTask是否已经取消，如果不取消就执行我们的**onPostExecute()**，最后将状态设置为FINISHED，整一个AsyncTask的方法都执行完了，我们只需要继承AsyncTask实现其中的方法就可以按分析的顺序往下执行了

##**次分支部分**

在AsyncTask中的finish()方法，我们可以看到onCancelled()方法跟onPostExecute()一起的，只要isCancelled()的值为true，就执行**onCancelled()**方法

```
private void finish(Result result) {
    if (isCancelled()) {
        onCancelled(result);
    } else {
        onPostExecute(result);
    }
    mStatus = Status.FINISHED;
}
```
我们代码跟踪isCancelled()方法

```
public final boolean isCancelled() {
    return mCancelled.get();
}
```
发现是在mCancelled中获取的，那我们就必须知道这个mCancelled是什么，代码跟踪到mCancelled

```
private final AtomicBoolean mCancelled = new AtomicBoolean();
```
mCancelled实际上就是个Boolean对象，那我们搜索它是在哪个时候设置的

```
public final boolean cancel(boolean mayInterruptIfRunning) {
    mCancelled.set(true);
    return mFuture.cancel(mayInterruptIfRunning);
}
```
可以发现，只要我们在AsyncTask类中调用这个方法即可停止异步任务


----------

而onProgressUpdate()方法，是在sHandler中执行，sHandler收到MESSAGE_POST_PROGRESS消息后，执行，我们搜索MESSAGE_POST_PROGRESS在什么时候发送的

```
private static class InternalHandler extends Handler {
   public InternalHandler() {
        super(Looper.getMainLooper());
    }

    @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
    @Override
    public void handleMessage(Message msg) {
        AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
        switch (msg.what) {
            case MESSAGE_POST_RESULT:
                // There is only one result
                result.mTask.finish(result.mData[0]);
                break;
            case MESSAGE_POST_PROGRESS:
                result.mTask.onProgressUpdate(result.mData);
                break;
        }
    }
}
```
可以发现，只要我们在AsyncTask类中调用publishProgress()方法即可执行onProgressUpdate()方法
```
protected final void publishProgress(Progress... values) {
    if (!isCancelled()) {
        getHandler().obtainMessage(MESSAGE_POST_PROGRESS,
                new AsyncTaskResult<Progress>(this, values)).sendToTarget();
    }
}
```
##**结语**
AsyncTask使用与源码分析就到这里结束了，如果是初学者，这可能会是比较难的分析，如果是想往中高级进阶的朋友，了解AsyncTask的原理是必须的。随着时代的发展，流行的异步框架RxJava的出现已经可以说是替代了这个AsyncTask，为了跟进时代的发展，我也得抽取时间来学习了，废话就不说了，还是节省点时间来学习吧
