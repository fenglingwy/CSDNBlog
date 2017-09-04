#BroadcastReceiver普通广播、有序广播、拦截广播、本地广播、Sticky广播、系统广播


----------
>本篇文章包括以下内容：

>* [前言](#a)
>* [BroadcastReceiver的简介](#b)
>* [普通广播（自定义广播）](#c)
* [有序广播](#d)
* [拦截广播](#e)
* [本地广播](#f)
* [Sticky广播](#g)
* [系统广播](#h)
* [部分源码下载](#i)

##**<span id="a">前言</span>**
又是一篇基础总结性的文章来啦，个人强迫症犯了，非得把博客的四大组件模块给补齐了，总结了一下BoradcastReceiver的各种使用，废话不多说，开车啦。**博主建议自己敲一遍代码来理解广播，因为里面有很多细节的东西需要注意，在学习中博主也犯过一些低级错误，不然以后到大项目中，不牢牢掌握好基础，会浪费很多时间在这上面。或者认真阅读本篇文章内容**

##**<span id="b">BroadcastReceiver的简介</span>**
BroadcastReceiver翻译为广播接收者，Broadcast是一种广泛运用在应用程序之间的传输信息的机制，简单的可以理解为传统意义上的电台广播，通俗一点，发布失物招领

广播机制是一个典型的发布—订阅模式，也就是我们所说的观察者模式。广播最大的特点就是发送方并不关心接收方是否接到数据，也不关心接收方是如何处理数据的，通过这样的形式来达到接、收双方的完全解耦合

##**<span id="c">普通广播（自定义广播）</span>**
普通广播是完全异步的，通过Context的sendBroadcast()方法来发送，消息传递效率比较高，但所有receivers（接收器）的执行顺序不确定。缺点是：接收者不能将处理结果传递给下一个接收者，并且无法终止广播Intent的传播，直到没有与之匹配的广播接收器为止。下面以自定义的普通广播进行演示


###**一、创建广播**
创建广播非常简单，只要继承BroadcastReceiver并实现onReceive()方法

```
public class MyBroadcastReceiver extends BroadcastReceiver {
    @Override
    public void onReceive(Context context, Intent intent) {
		Log.e("receive","onReceive");
    }
}
```

###**二、注册广播**
BroadcastReceiver是四大组件之一，所以毫不疑问需要注册，BroadcastReceiver的注册有两种方法：

* 通过manifests配置
* 通过代码动态配置

**方法一：通过manifests配置**
```
<receiver android:name=".BroadcastReceiver.MyBroadcastReceiver">
    <intent-filter>
        <action android:name="com.handsome.hensen" />
    </intent-filter>
</receiver>
```
这里需要加入intent-filter的action中的name属性，表示我们监听的内容。当有广播发送时，需要判断该广播是否和我们监听的内容一致，如果一致则接收

**方法二：通过代码动态配置**

```
//创建广播
MyBroadcastReceiver receiver = new MyBroadcastReceiver();
//注册广播
registerReceiver(receiver, new IntentFilter("com.handsome.hensen"));
```

###**三、反注册广播**
BroadcastReceiver必须遵循生到死的周期，如果你是使用动态注册广播的则需要在Activity的onDestroy的时候反注册广播

```
@Override
protected void onDestroy() {
    unregisterReceiver(receiver);
    super.onDestroy();
}
```

###**四、发送广播**
这里我们以一个按钮来发送广播，通过sendBroadcast()方法发送我们的创建的Intent自定义广播

```
final Intent intent = new Intent();
//广播内容
intent.setAction("com.handsome.hensen");

bt_send.setOnClickListener(new View.OnClickListener() {
   @Override
   public void onClick(View v) {
       sendBroadcast(intent);
   }
});
```

###**五、运行代码**
运行程序后，我们点击发送广播。我们以Log信息来验证发出的广播被我们准确的接收

```
11-25 10:27:43.341 5188-5188/com.handsome.boke2 E/receive: onReceive
```

##**<span id="d">有序广播</span>**

有序广播通过Context.sendOrderedBroadcast()来发送，所有的广播接收器优先级依次执行，广播接收器的优先级通过receiver的intent-filter中的android:priority属性来设置，数值越大优先级越高。

当广播接收器接收到广播后，可以使用setResult()函数来结果传给下一个广播接收器接收，然后通过getResult()函数来取得上个广播接收器接收返回的结果。当广播接收器接收到广播后，也可以用abortBroadcast()函数来让系统拦截下来该广播，并将该广播丢弃，使该广播不再传送到别的广播接收器接收

###**一、创建广播**
我们创建一个类，存放三个有优先级的广播接收者，并在最高级广播中传递结果到下一个广播
```
public class PriorityBroadcastReceiver {

    public static class HighPriority extends BroadcastReceiver {
        //高级广播接收者
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.e("receive", "High");
            //传递结果到下一个广播接收器
            int code = 0;
            String data = "hello";
            Bundle bundle = null;
            setResult(code, data, bundle);
        }
    }

    public static class MidPriority extends BroadcastReceiver {
        //中级广播接收者
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.e("receive", "Mid");
            //获取上一个广播接收器结果
            int code = getResultCode();
            String data = getResultData();
            Log.e("receive", "获取到上一个广播接收器结果：" + "code=" + code + "data=" + data);
        }
    }

    public static class LowPriority extends BroadcastReceiver {
        //低级广播接收者
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.e("receive", "Low");
        }
    }
}
```
要注意的是：

1. 内部类的BroadcastReceiver必须由public static修饰，否则会报错

###**二、注册广播**
这里的注册方式和普通广播是一样的，这里的区别在于priority属性，确定了他们之间的优先级

```
<receiver android:name=".BroadcastReceiver.PriorityBroadcastReceiver$HighPriority">
    <intent-filter android:priority="3000">
        <action android:name="com.handsome.hensen2" />
    </intent-filter>
</receiver>
<receiver android:name=".BroadcastReceiver.PriorityBroadcastReceiver$MidPriority">
    <intent-filter android:priority="2000">
        <action android:name="com.handsome.hensen2" />
    </intent-filter>
</receiver>
<receiver android:name=".BroadcastReceiver.PriorityBroadcastReceiver$LowPriority">
    <intent-filter android:priority="1000">
        <action android:name="com.handsome.hensen2" />
    </intent-filter>
</receiver>
```
要注意的是：

1. BroadcastReceiver类名与内部类的名字之间用$符号隔开，否则会报错

###**三、发送广播**
和之前的不一样的地方，这里是使用sendOrderedBroadcast()发送有序广播
```
final Intent intent = new Intent();
intent.setAction("com.handsome.hensen2");

bt_send.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        sendOrderedBroadcast(intent,null);
    }
});
```
要注意的是：

1. 这里需要发送的是有序广播，否则在接收者中通过setResult()和getResult()方法会报错，因为只有有序广播才能传递结果

###**四、运行代码**
运行程序后，我们点击发送广播。我们以Log信息来验证发出的广播被我们准确的接收，数据被我们准确的传递

```
11-25 11:50:07.207 12777-12777/com.handsome.boke2 E/receive: High
11-25 11:50:07.217 12777-12777/com.handsome.boke2 E/receive: Mid
11-25 11:50:07.218 12777-12777/com.handsome.boke2 E/receive: 获取到上一个广播接收器结果：code=0data=hello
11-25 11:50:07.233 12777-12777/com.handsome.boke2 E/receive: Low
```

##**<span id="e">拦截广播</span>**

上面我们提到过有序广播中可以拦截广播，那么我们在上面程序的基础上修改代码，在HighPriority接收器中加上拦截广播
###**一、创建广播**
通过在BroadcastReceiver中，执行abortBroadcast()方法，广播就不会继续往下传递了
```
public static class HighPriority extends BroadcastReceiver {
        //高级广播接收者
        @Override
        public void onReceive(Context context, Intent intent) {
            Log.e("receive", "High");
            //拦截广播
            abortBroadcast();
            //传递结果到下一个广播接收器
            int code = 0;
            String data = "hello";
            Bundle bundle = null;
            setResult(code, data, bundle);
        }
    }
```

###**二、运行代码**
运行程序后，我们点击发送广播。我们以Log信息来验证我们拦截了广播

```
11-25 12:12:36.405 30867-30867/com.handsome.boke2 E/receive: High
```
可以看到，后面的Mid和Low广播都没有Log信息，说明我们拦截成功了

###**三、有序广播、拦截广播的拓展——终结广播**
现在有这样的一个应用场景，按照上面的程序走，只能在第一个广播中被拦截住了，后面的广播则不执行。如果这个时候我们需要一个不管有没有被拦截都必须执行的广播，我们称为终结广播，那应该怎么办。同样的，发送有序广播也考虑到这一点，通过以下代码来发送广播，并指定我们不管有没有被拦截都必须执行的终结广播

```
bt_send.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        sendOrderedBroadcast(intent, null, new PriorityBroadcastReceiver.LowPriority(),
                new Handler(), 0, null, null);
    }
});
```
运行代码，我们查看Log信息

```
11-25 12:22:33.466 4764-4764/com.handsome.boke2 E/receive: High
11-25 12:22:33.468 4764-4764/com.handsome.boke2 E/receive: Low
```

可以发现，之前只是有High的Log信息，因为是被拦截了，而Log信息多了一条Low，说明我们拦截后，还要执行终结广播

##**<span id="f">本地广播</span>**

在API21的Support v4包中新增本地广播，也就是LocalBroadcastManager。由于之前的广播都是全局的，所有应用程序都可以接收到，这样就会带来安全隐患，所以我们使用LocalBroadcastManager只发送给自己应用内的信息广播，限制在进程内使用

它的用法很简单，只需要把调用context的sendBroadcast、registerReceiver、unregisterReceiver的地方换为LocalBroadcastManager.getInstance(Context context)中对应的函数即可。这里创建广播的过程和普通广播是一样的过程，这里就不过多介绍了

**一、注册Receiver**

```
 //创建广播
receiver = new MyBroadcastReceiver();
//注册本地广播
LocalBroadcastManager.getInstance(ReceiverActivity.this).registerReceiver(receiver,
        new IntentFilter("com.handsome.hensen"));
```
**二、反注册Receiver**

```
LocalBroadcastManager.getInstance(ReceiverActivity.this).unregisterReceiver(receiver);
```

**三、发送异步广播**

```
final Intent intent = new Intent();
intent.setAction("com.handsome.hensen2");
LocalBroadcastManager.getInstance(ReceiverActivity.this).sendBroadcast(intent);
```

**四、发送同步广播**

```
LocalBroadcastManager.getInstance(ReceiverActivity.this).sendBroadcastSync(intent);
```

##**<span id="g">Sticky广播</span>**
sticky广播通过Context.sendStickyBroadcast()函数来发送，用此函数发送的广播会一直滞留，当有匹配此广播的广播接收器被注册后，该广播接收器就会收到此条信息。使用此函数需要发送广播时，需要获得BROADCAST_STICKY权限

```
<uses-permission android:name="android.permission.BROADCAST_STICKY"/>
```
sendStickyBroadcast只保留最后一条广播，并且一直保留下去，这样即使已经有广播接收器处理了该广播，当再有匹配的广播接收器被注册时，此广播仍会被接收。如果你只想处理一遍该广播，可以通过removeStickyBroadcast()函数来实现。这里创建广播的过程和普通广播是一样的过程，这里就不过多介绍了

##**<span id="h">系统广播</span>**
当然系统中也会有很多自带的广播，当符合一定条件时，系统会发送一些定义好的广播，比如：重启、充电、来电电话等等。我们可以通过action属性来监听我们的系统广播

```
<receiver android:name=".BroadcastReceiver.MyBroadcastReceiver">
    <intent-filter>
	    <!--重启设备-->
        <action android:name="android.intent.action.REBOOT" />
    </intent-filter>
</receiver>
```

这里创建广播的过程和普通广播是一样的过程，这里就不过多介绍了。常用的广播action属性有

* **屏幕被关闭之后的广播**：Intent.ACTION_SCREEN_OFF
* **屏幕被打开之后的广播**：Intent.ACTION_SCREEN_ON
* **充电状态，或者电池的电量发生变化**：Intent.ACTION_BATTERY_CHANGED
* **关闭或打开飞行模式时的广播**：Intent.ACTION_AIRPLANE_MODE_CHANGED
* **表示电池电量低**：Intent.ACTION_BATTERY_LOW
* **表示电池电量充足，即电池电量饱满时会发出广播**：Intent.ACTION_BATTERY_OKAY
* **按下照相时的拍照按键(硬件按键)时发出的广播**：Intent.ACTION_CAMERA_BUTTON


##**<span id="i">[部分源码下载](http://download.csdn.net/detail/qq_30379689/9694237)</span>**