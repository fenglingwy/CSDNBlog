# Android群英传知识点回顾——第八章：Activity与Activity调用栈分析 #

----------

##知识点目录##
>* **8.1 Activity**
	*  8.1.1 起源
	*  8.1.2 Activity形态
	*  8.1.3 生命周期
>* **8.2 Activity任务栈简介**
>* **8.3 ActivityManifest启动模式**
	*  8.3.1 standard
	*  8.3.2 singleTop
	*  8.3.3 singleTask
	*  8.3.4 singleInstance
>* **8.4 Intent Flag启动模式**
>* **8.5 清空任务栈**
>* **8.6 Activity任务栈使用**


----------

##知识点回顾##

###**8.1 Activity**
>四大组件中出现频率最高的组件

###8.1.1 起源
>Activity是与用户交互的第一接口，它提供了一个用户完成指令的窗口，系统采用Activity栈的方式来管理Activity

###8.1.2 Activity形态
>* Activity/Running
	Activity处于Activity栈的最顶层，可见，并与用户进行交互
>* Paused
	当Activity失去焦点，被一个新的非全屏的Activity或者一个透明的Activity放置在栈顶时，Activity就转换成了Paused形态，但它只是失去了与用户交互的能力，所有状态信息，成员变量都还保留着，只有在系统内存极地的情况下，才会被系统回收掉
>* Stopped
	如果一个Activity被另一个Activity完全覆盖，那么Activity就会进入stop形态，此时他不再可见，但却依然保持了所有状态信息和成员变量
>* Killed
	当Activity被系统回收或者Activity从来没有创建过，Activity就处于Killed状态
	
###8.1.3 生命周期

>Google经典生命周期图：

![](http://img.blog.csdn.net/20161013180641238)

>由于Android群英传介绍的不详细，我采用Android开发艺术探索这本书介绍生命周期：

> * onCreate：表示Activity正在被创建，这是生命周期的第一个方法，在这个方法可以做一些初始化工作，比如调用setContentView去加载界面布局资源、初始化Activity所需数据等
> * onRestart：表示Activity正在重新启动，一般情况下，当当前Activity从不可见重新变为可见状态时，onRestart就会被调用，这种情形一般是用户行为所导致的，比如用户按Home键切换到桌面或者用户打开一个新的Activity，这是当前的Activity就会暂停，也就是onPause和onStop被执行了，接着用户又回到了这个Activity，就会出现这种情况
> * onStart：表示Activity正在被启动，即将开始，这时Activity已经可见了，但是还没有出现在前台，还无法和用户交互，这个时候其实可以理解为Activity已经显示出来了，但是我们还看不到
> * onResume：表示Activity已经可见了，并且出现在前台并开始活动，要注意这个和onStart的对比，onStart和onResume都表示Activity已经可见，但是onStart的时候Activity还在后台，onResume的时候Activity才显示在前台
> * onPause：表示Activity正在停止，正常情况下，紧接着onStop就会被调用，在特殊情况下，如果这个时候快速地再回到当前Activity，那么onResume会被调用，笔者的理解是，这种情况属于极端情况，用户操作很难重现这一场景，此时可以做一些存储数据、停止动画等工作，但是注意不能太耗时，因为这会影响到新Activity显示，onPause必须先执行完，新Activity的onResume才会执行
> * onStop：表示Activity即将停止，可以做一些稍微重量级的回收工作，同样不能太耗时
> * onDestroy：表示Activity即将被销毁，这是Activity生命周期最后一个回调，在这里，我们可以做一些回收工作和最终的资源释放

![](http://img.blog.csdn.net/20161013182511091)

>除生命周期外，还有Activity的状态会保存起来：

> * 如果你长时间处于stopped状态而且此时系统需要更多内存或者系统内存极为紧张时，系统会回收你的Activity，而此时系统为了补偿你，会将Activity状态通过onSaveInstanceState()方法保存到Bundle对象中，当你需要重新创建这些Activity的时候，保存的Bundle对象就会传递到Activity的onRestoreInstanceState()方法与onCreate()方法中，即可调用该方法恢复被销毁时的状态

###**8.2 Android任务栈简介**
>* 当一个App启动时，如果当前环境下不存在该App的任务栈，那么系统就会创建一个任务栈，此后，这个App所启动的Activity都将在这个任务栈中被管理，这个栈也被称为一个Task，即表示若干个Activity的集合，他们组成在一起形成一个Task，特别要注意的是，一个Task中的Activity可以来自不同的App，同一个App的Activity也可能不在一个Task中

>* 栈的结构是后进先出的线性表，通过在AndroidManifest文件中的属性android:launchMode来设置或者是通过Intent的flag来设置的


###**8.3 AndroidManifest启动模式 **
>Manifest提供了四种启动模式：

>* standard
>* singleTop
>* singleTask
>* singleInstance

>对于这四种启动模式可以简单的在我另一篇博客理解一下：[四种启动模式](http://blog.csdn.net/qq_30379689/article/details/52164966)

###**8.4 Intent Flag启动模式 **

>* Intent.FLAG-ACTIVITY-NEW-TASK：使用一个新的Task来启动一个Activity，但启动的每个Aetivity都将在一个新的Task中，该Flag通常使用在从service中启动的actiity场景，由于在Service中并不存在Activity栈，所以使用该Flag来创建一个新的Activity栈，并创建新的Activity实例
>* FLAG-ACTIVITY-SINGLE-TOP：使用singletop模式来启动一个Activity，与指定android:launchMode="singleTop"效果相同
>* FLAG-ACTIVITY-CLEAR-Top：使用SingleTask模式来启动一个Activity，与指定android:launchMode="singleTask"效果相同
>* FLAG-ACTIVITY-NO-HISTORY：使用这种模式启动Acuvity，当该Activity启动其他AcuVity后，该Activity就消失了，不会保留在Activity栈中，例如A-B，B中以这种模式启动C，C再启动D，则当前Activity栈为ABD 

###**8.5 清空任务栈**
>系统同样提供了清空任务栈的方法让我们将一个Task全部清除，通常可以在AndroidManifest的&lt;activity&gt;标签中使用下面属性来清理：

>* clearTaskOnLaunch：每次返回Activity的时候，都将该Activity上的所有Activity都清除，通过这个属性，可以让这个Task每次初始化的时候，都只有一个Activity
>* finishTaskOnLaunch：这个属性和clearTaskOnLaunch有点类似，只不过clearTaskOnLaunch作用在别人身上，而finishTaskOnLaunch作用在自己身上，通过这个属性，当离开这个Activity所处的Task，那么用户再返回的时候，该Activity会被finish掉
>* alwaysRetainTaskState：Task的一道免死金牌，如果将Activity这个属性设置为true，那么该Activity所在的Task将不接受任何清除命令，一直保持当前Task的状态

###**8.6 Activity任务栈使用**
>无知识点