# Android群英传知识点回顾——第十章：Android性能优化 #

----------

##知识点目录##
>* **10.1 布局优化**
	*  10.1.1 Android UI渲染机制
	*  10.1.2 避免Overdraw
	*  10.1.3 优化布局层级
	*  10.1.4 避免嵌套过多无用布局
	*  10.1.5 Hierarchy Viewer
>* **10.2 内存优化**
	*  10.2.1 什么是内存
	*  10.2.2 获取Android系统内存信息
	*  10.2.3 内存回收
	*  10.2.4 内存优化实例
>* **10.3 Lint工具**
>* **10.4 使用Android Studio的Memory Monitor工具**
>* **10.5 使用TraceView工具优化App性能**
	*  10.5.1 生成TraceView日志的两种方法
	*  10.5.2 打开TraceView日志
	*  10.5.3 分析TraceView日志
>* **10.6 使用MAT工具分析App内存状态**
	*  10.6.1 生成HPROF文件
	*  10.6.2 分析HPROF文件
>* **10.7 使用Dumpsys命令分析系统状态**


----------

##知识点回顾##

###**10.1 布局优化**
>无知识点

###10.1.1 Android UI渲染机制
>* 人眼所感觉的流畅画面，需要画面的帧数达到40帧每秒到60帧每秒，最佳fps大概在60fps左右
>* 16ms也就是1000ms中显示60帧画面的单位时间，即1000/60，所以UI渲染时间间隔为16ms
>* 如果不能在16ms内绘制完成，则会出现卡顿的效果
>* 通过开发者选项，选择"Profile GPU Rendering"并选中"On screen as bars"的选项，可以打开UI渲染时间的工具
>* 每一条柱状线包含三部分，蓝色代表测量绘制Display List的时间，红色代表OpenGL渲染Display List的时间，黄色代表CPU等待GPU处理的时间
>* 中间的绿色横线代表绘制间隔时间16ms，所以尽量保持在绿色之下

![](http://img.blog.csdn.net/20161014145945579)

###10.1.2 避免Overdraw
>* Overdraw，过度绘制会浪费很多的CPU、GPU资源
>* Android系统在开发者选项中提供了这样一个检测工具，"Enable GPU Overdraw"
>* 通过这个工具可以查看当前区域中绘制次数，从而尽量优化绘图层次，尽量增大蓝色的区域，减少红色的区域

![](http://img.blog.csdn.net/20161014150316333)

###10.1.3 优化布局层级
>* Android中，系统对View进行测量、布局和绘制时，都是通过对View数的遍历来进行操作的
>* 如果View树太高，就会严重影响测量、布局和绘制的速度，因此适当减少View树的高度，Google建议View树高度不宜超过10层
>* 现在版本的Android，Google已经使用RelativeLayout来替代LinearLayout作为默认的根布局，，其原因就是降低LinearLayout嵌套所产生布局树高度

###10.1.4 避免嵌套过多无用布局
>* 使用< include>标签重用Layout
>* 使用< ViewStub>实现View的延迟加载

>< include>标签重用，首先编写common_ui.xml文件：
```
<?xml version="1.0" encoding="utf-8"?>
<TextView 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="0dp"
    android:layout_height="0dp"
    android:text="this is a common ui"
    android:textSize="30sp"></TextView>
```
>接着在主布局中引用：

```
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <include
        layout="@layout/common_ui"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</RelativeLayout>
```
>这样，common_ui就被引用了，其中include标签里面属性会覆盖common_ui里面的属性

>使用< ViewStub>实现View的延迟加载，< ViewStub>是个轻量级的组件，它不仅不可视，而且大小为0，首先编写not_often_use.xml文件

```
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="not often use"
        android:textSize="30sp" />
</RelativeLayout>
```

>接着在主布局中引用：

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ViewStub
        android:id="@+id/not_often_use"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout="@layout/not_often_use" />
</RelativeLayout>
```
>这个时候是在布局中见不到这个ViewStub的布局的，有两种方法可以来重新显示这个View：

>* VISBLE

```
ViewStub mViewStub = (ViewStub) findViewById(R.id.not_often_use);
mViewStub.setVisibility(View.VISIBLE);
```

>* inflate

```
ViewStub mViewStub = (ViewStub) findViewById(R.id.not_often_use);
View inflateView = mViewStub.inflate();
```
>这两种方法唯一区别：

>* inflate()方法可以返回引用布局，从而可以通过View.findViewById()方法找到相对应控件

> < ViewStub>标签与设置View.GONE有什么区别：

> * 它们的共同点就是在初始化时不会显示，但是< ViewStub>标签只会在显示时，才会去渲染整个布局，而View.GONE在初始化布局树的时候就已经添加在布局树上了，相比之下，< ViewStub>标签的布局具有更高的效率

###10.1.5 Hierarchy Viewer
>* 通常情况下，Hierarchy Viewer无法在真机上使用，只能在模拟器上使用
>* Google大神提供了一个开源软件View Server，让普通手机也能使用Hierarchy Viewer，软件网址http://github.com/romainguy/ViewServer
>* Hierarchy Viewer位于sdk\tools目录下，打开hierarchyviewer.bat启动程序
>* 关于Hierarchy Viewer的使用可查看下面的博客：[Hierarchy Viewer](http://blog.csdn.net/xyz_lmn/article/details/14222975)

###**10.2 内存优化**
>无知识点

###10.2.1 什么是内存
>由于Android的沙箱机制，每个应用所分配的内存大小是有限度的，内存太低就会触发LMK：Low Memory Killer机制，我们所说的内存是指手机的RAM，它包括以下几个部分：

>* 寄存器(Registers)
>速度最快的存储场所，因为寄存器位于处理器内部，在程序中无法控制
>* 栈（Stack）
>存放基本类型的数据和对象的引用，但对像本身不存放在栈中，而是存放在堆中
>* 堆（Heap）
>堆内存用来存放由new创建的对象和数组，在堆中分配的内存，由Java虚拟机的自动垃圾回收（GC）来管理
>* 静态存储区域(Static Field)
>静态存储区域就是指在固定的位置存放应用程序运行时一直存在的数据，Java在内存中专门划分了一个静态存储区域来管理一些特殊的数据变量如静态的数据变量
>* 常量池(Constant Pool)
>JVM虚拟机必须为每个被装载的类型维护一个常量池，常量池就是该类型所用到常量的一个有序集合，包括直接常量(基本类型，String)和对其他类型、字段和方法的符号引用

>这些概念中最容易搞错的是堆和栈的区分：

>* 栈：当定义一个变量时，栈会为该变量分配内存空间，当该变量作用域结束后，这部分内存控件会被用作新的空间进行分配
>* 堆：使用new的方式创建一个变量，那么就会在堆中为这个对象分配内存控件，即使该对象的作用域结束，这部分内存也不会立即被回收，而是等待系统的GC进行回收

>我们可以通过代码分析Heap中的内存状态：

```
ActivityManager manager = (ActivityManager) getSystemService(Context.ACTIVITY_SERVICE);
int heapSize = manager.getLargeMemoryClass();
```


###10.2.2 获取Android系统内存信息
>* Process Stats：系统内存监视服务，通过"Setting-Developer-options-Process Stats"来开启这个功能，也可以使用Dumpsys命令来获取这些信息：

```
adb shell dumpsys procstats
```

>* Meminfo：内存监视工具，通过"Settings-Apps-Running"中打开这个界面，也可以使用Dumpsys命令来获取这些信息：

```
adb shell dumpsys meminfo
```

###10.2.3 内存回收
>* Java对于C、C++这类语言最大的优势就是不用手动管理系统资源，Java创建了垃圾收集器线程来自动进行资源的管理
>* Java的GC是系统自动进行的，但何时进行却是开发者无法控制的，即使调用System.gc()方法，也只是建议系统进行GC，系统不一定会采取你的建议
>* 尽管再强大的算法，也难免存在部分对象忘记回收的想象，这就是造成内存泄露的原因

###10.2.4 内存优化实例
>* Bitmap优化：Bitmap是造成内存占用过高或OOM的最大威胁
	* 使用适当分辨率和大小的图片
	* 及时回收内存
	* 使用图片缓存
>* 代码优化：任何Java类占用大约500字节的内存空间，创建一个类的实例会消耗大概15字节的内存
	* 对常量使用static修饰符
	* 使用静态方法，静态方法会比普通方法提高15%左右的访问速度
	* 减少不必要的成员变量 ,这点在Android Lint工具上已经集成检测了，如果一个变量可以定义为局部变量，则会建议你不要定义为成员变量
	* 减少不必要的对象，使用基础类型会比使用对象更加节省资源, 同时更应该避免频繁创建短作用域的变量
	* 尽量不要使枚举、少使用迭代器
	* 对Cursor、Receiver、Sensor、Fiie等对象，要非常注意对它们的创建，回收与注册、解注册
	* 避免使用IOC框架，IOC通常使用注解、反射来进行实现，虽然现在Java对反射的效率已经进行了很好的优化，但大量使用反射依然会带来性能的下降
	* 使用RenderScript、OpenGL来进行非常复杂的绘图操作
	* 使用SurfaceView来替代View进行大量、频繁的绘图操作
	* 尽量使用视图缓存，而不是每次都执行inflaler()方法解析视图

###**10.3 Lint工具**
>Android Lint工具是Android Studio中集成的一个Android代码提示工具

###**10.4 使用Android Studio的Memory Monitor工具**

>Memory Monitor工具是Android Studio自带的一个内存监视工具，通过Android Studio右下角"Memory Monitor"打开"Memory Monitor"工具

![](http://img.blog.csdn.net/20161014180126277)

###**10.5 使用TraceView工具优化App性能**
>TraceView是一个Android下的可视化性能调查工具，用来分析TraceView日志

###10.5.1 生成TraceView日志的两种方法
>* 通过代码生成精准范围的TraceView日志：
	* 通过调用Debug.startMethodTracing()方法开启监听，调用Debug.stopMethodTracing()方法结束监听，通常会在onCreate时开启监听，在onDestroy时结束监听
	* TraceView日志将会保存到"/sdcard/dmtrace.trace"目录下，因此需要在权限中增加：
```
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
>当然除了默认输出日志名，还可以自定义路径和日志名，最后通过ADB命令，将日志文件导出到本地：
```
adb pull /sdcard/trace_log.trace/local/LOG/
```

>* 通过Android Device Monitor生成TraceView日志：
	* 打开Android Device Monitor工具，选择要调试的进程，点击工具栏中的"start method profiling"按钮，打开，再次点击"start method profiling"按钮即可结束
	* TraceView提供两种监听方式：
		* 整体监听：跟踪每个方法执行的全部过程，这种方式资源消耗比较大
		* 抽样监听：按照指定的频率来进行抽样调查，这种方式需要执行较长时间来获取较精准的样本数据

![](http://img.blog.csdn.net/20161014180558126)

![](http://img.blog.csdn.net/20161014180612439)

###10.5.2 打开TraceView
>* 可以使用SDK中的"sdk\tools\traceview.bat"工具打开
>* 可以在Android Device Monitor工具下，的File菜单选择Open File打开

![](http://img.blog.csdn.net/20161014180716833)


###10.5.3 分析TraceView
>* 时间轴区域：显示了不同线程在不同时间段内的执行情况
	* 在时间轴中，每一行都代表了一个独立的线程
	* 使用鼠标滚轮可以放大时间轴
	* 不同色块代表不同的执行方法，色块的长度，代表了方法所执行的时间
>* Profile区域：显示你选择的色块所代表的方法在该色块所处的时间段内的性能分析
	* Incl CPU Time：某方法占用CPU的时间
	* Excl CPU Time：某方法本身（不包含子方法）占用CPU的时间
	* Incl Real Time：某方法真正执行的时间
	* Excl Real Time：某方法本身（不包含子方法）真正执行的时间
	* Calls+RecurCalls：调用次数+递归回调的次数

>每个时间都包含两列，一个是实际的时间，另一个是百分比，如果占用时间长且Calls+RecurCalls次数少，那么就可以列为怀疑对象	

###**10.6 使用MAT工具分析App内存状态**
>MAT工具是一个分析内存的强力助手

###10.6.1 生成HPROF文件

>打开Android Device Monitor工具，选择要监听的线程，并点击菜单栏中的"Update Heap"按钮

![](http://img.blog.csdn.net/20161014180923411)

>在Heap标签中点击"Cause GC"按钮，就会显示当前内存状态

![](http://img.blog.csdn.net/20161014181040069)

>这里有一 个判断当前是否存在内存泄漏的小技巧：当我们不停地点击"Cause GC"按钮的时，如果"data object"一栏中的"Total size"有明显变化，就代表可能存在内存泄漏

>上面是手动查看Heap状态，下面点击菜单栏的"Dump HPROF File"按钮

![](http://img.blog.csdn.net/20161014181559467)

>系统会生成一个.hprof文件，默认名为包名.hprof，不过还不能直接使用MAT工具查看，还需要进行格式转换，在SDK目录的platform-tools目录下，使用hprof-conv工具帮助转换，命令如下：

```
D:\sdk\platform-tools>hprof-conv F:\Heap\com.handsome.heap.hprof heap.hprof
```
>格式命令："hprof-conv infile outfile"生成heap.hprof文件利用MAT工具就可以进行内存分析

###10.6.2 分析HPROF文件

>打开MAT工具，选择"Open a Heap Dump"选项，进入分析：

>* Histogram
	* Histogram直方图，用于显示内存中每个对象的数量、大小和名称
	* 在选择对象上单击鼠标右键，在弹出的快捷菜单中选择"List objects-with incoming references"选项查看具体的对象
>* Dominator Tree
	* Dominator Tree支配树会将内存中的对象按照大小进行排序，并显示对象之间的引用结构
	* 对象已经按照"Retained Heap"进行排序了，即按照对象及其持有的引用的内存总和进行排序，通过分析内存占用大的对象找出内存消耗的原因

###**10.7 使用Dumpsys命令分析系统状态**
>* Dumpsys命令的功能非常强大，可使用的参数配置也非常多，Dumpsys官方信息：https://source.android.com/devices/input/dumpsys.html
>* 使用Dumpsys命令，只需要输入"adb shell dumpsys+参数"即可
	* adb shell dumpsys activity：查看Activity栈的详细信息
	*  adb shell dumpsys meminfo：查看内存信息
	*  adb shell dumpsys battery：查看电池信息
	*  adb shell dumpsys package：查看包信息
	*  adb shell dumpsys wifi：显示Wi-Fi信息
	*  adb shell dumpsys alarm：显示alarm信息
	*  adb shell dumpsys procstats：显示内存信息
