# Android群英传知识点回顾——第九章：Android系统信息与安全机制 #

----------

##知识点目录##
>* **9.1 Android系统信息获取**
	*  9.1.1 android.os.Build
	*  9.1.2 SystemProperty
	*  9.1.3 Android系统信息实例
>* **9.2 Android Apk应用信息获取之PackageManager**
	*  9.2.1 PackageManager 
>* **9.3 Android Apk应用信息获取之ActivityManager**
>* **9.4 解析Packages.xml获取系统信息**
>* **9.5 Android安全机制**
	*  9.5.1 Android安全机制简介
	*  9.5.2 Android系统安全隐患
	*  9.5.3 Android Apk反编译
	*  9.5.4 Android Apk加密


----------

##知识点回顾##

###**9.1 Android系统信息获取**
>要获取系统的配置信息，通常可以从以下两个方面获取：

>* android.os.Build
>* SystemProperty

###9.1.1 android.os.Build
> 该类包含系统编译时大量设备、配置信息：

>* Build.BOARD：主板
>* Build.BRAND：Android系统定制商
>* Build.SUPPORTED_ABIS：CPU指令集
>* Build.DEVICE：设备参数
>* Build.DISPLAY：显示屏参数
>* Build.FINGERPRINT：唯一编号
>* Build.SERIAL：硬件序列号
>* Build.ID：修订版本列表
>* Build.MANUFACTURER：硬件制造商
>* Build.MODEL：版本
>* Build.HARDWARE：硬件名
>* Build.PRODUCT：手机产品名
>* Build.TAGS：描述Build的标签
>* Build.TYPE：Builder类型
>* Build.VERSION.CODENAME：当前开发代号
>* Build.VERSION.INCREMENTAL：源码控制版本号
>* Build.VERSION.RELEASE：版本字符串
>* Build.VERSION.SDK_INT：版本号
>* Build.HOST：host值
>* Build.USER：User名
>* Build.TIME：编译时间

###9.1.2 SystemProperty
>SystemProperty包含了许多系统配置属性值和参数：

>* os.version：OS版本
>* os.name：OS名称
>* os.arch：OS架构
>* user.home：Home属性
>* user.name：Name属性
>* user.dir：Dir属性
>* user.timezone：时区
>* path.separator：路径分隔符
>* line.separator：行分隔符
>* file.separator：文件分隔符
>* java.vendor.url：Java vender Url属性
>* java.class.path：Java Class属性
>* java.class.version：Java Class版本
>* java.vendor：Java Vender属性
>* java.version：Java版本
>* java.home：Java Home属性

###9.1.3 Android系统信息实例

>上面列举了那么多，让我们通过代码获取他们的系统信息：

```
String board = Build.BOARD;
String brand = Build.BRAND;

String os_version = System.getProperty("os.version");
String os_name = System.getProperty("os.name");
```

>我们还可以通过命令行模式查看系统信息：

>* 命令行模式进入system/build.prop文件目录，使用cat build.prop命令查看文件信息

>我们还可以通过adb shell的getprop来获取对应的属性值：

>* 进入adb shell，使用getprop ro.build.id获取信息

>还有一个非常重要的目录存储系统信息，那就是/proc目录：

>* 命令行模式进入/proc文件目录，使用cat cpuinfo命令打开cpuinfo文件查看系统信息

###**9.2 Android Apk应用信息获取之PackageManager**
>看了这么多系统信息，应该看Apk应用信息了，在ADB Shell命令中，有两个非常强大的助手，PM和AM，PM主宰着应用的包管理，而AM主宰着应用的活动管理


###9.2.1 PackageManager

>* ActivityInfo
> ActivityInfo封装在了Mainfest文件中的< activity >和< eceiver>之间的所有信息，包括name、icon、label、launchMode等。
>* ServiceInfo
> ServiceInfo与ActivityInfo类似，封装了< service>之间的所有信息。
>* ApplicationInfo
> 它封装了< application>之间的信息，特别的是，ApplicationInfo包含了很多Flag
	* FLAG_SYSTEM表示为系统应用
	* FLAG_EXTERNAL_STORAGE表示为安装在SDcard上的应用
>* PackageInfo
> PackageInfo与前面三个Info类类似，都是用于封装Manifest文件的相关节点信息，而PageageInfo包含了所有的Activity和Service信息。
>* ResolveInfo
> ResolveInfo包含了< intent>信息的上一级信息，所以它可以返回ActivityInfo、ServiceInfo等包含了< intent>的信息
>* PackageManager中封装的用来获取这些信息的方法：
	* getPackageManager()：通过这个方法可以返回一个PackageManager对象
	* getApplicationInfo()：以ApplicationInfo的形式返回指定包名的ApplicationInfo
	* getApplicationIcon()：返回指定包名的Icon
	* getInstalledApplications()：以ApplicationInfo的形式返回安装的应用
	* getInstalledPackages()：以PackageInfo的形式返回安装的应用
	* queryIntentActivities()：返回指定Intent的ResolveInfo对象、Activity集合
	* queryIntentServices()：返回指定Intent的ResolveInfo对象、Service集合
	* resolveActivity()：返回指定Intent的Activity
	* resolveService()：返回指定Intent的Service

>下面通过一个实际的例子通过PackageManager筛选不同类型的App，利用ApplicationInfo中的FLAG_SYSTEM来判断：

```
app.flags & ApplicationInfo.FLAG_SYSTEM
```

>* 如果当前应用的flags & ApplicationInfo.FLAG_SYSTEM != 0则为系统应用
>* 如果flags & ApplicationInfo.FLAG_SYSTEM <= 0 则为第三方应用
>* 特殊的，当系统应用升级后，也将会成为第三方应用： flags & ApplicationInfo.FLAG_UPDATED_SYSTEM_APP != 0
>* 如果当前应用的flags & ApplicationInfo.FLAG_EXTERNAL_STORAGE != 0 则为安装在SDCard上的应用

>我们封装一个Bean来保存我们所需的字段：

```
public class PMAppInfo {

    private String appLabel;
    private Drawable appIcon;
    private String pkgName;

    public PMAPPInfo(){
    }

    public String getAppLabel() {
        return appLabel;
    }

    public void setAppLabel(String appLabel) {
        this.appLabel = appLabel;
    }

    public Drawable getAppIcon() {
        return appIcon;
    }

    public void setAppIcon(Drawable appIcon) {
        this.appIcon = appIcon;
    }

    public String getPkgName() {
        return pkgName;
    }

    public void setPkgName(String pkgName) {
        this.pkgName = pkgName;
    }
}
```
>接下来，通过上面所说的判断方法来判断各种类型的应用：

```
private List<PMAppInfo> getAppInfo(int flag) {
    // 获取PackageManager对象
    pm = this.getPackageManager();
    // 获取应用信息
    List<ApplicationInfo> listAppcations = pm
            .getInstalledApplications(
                    PackageManager.GET_UNINSTALLED_PACKAGES);
    List<PMAppInfo> appInfos = new ArrayList<PMAppInfo>();
    // 判断应用类型
    switch (flag) {
        case ALL_APP:
            appInfos.clear();
            for (ApplicationInfo app : listAppcations) {
                appInfos.add(makeAppInfo(app));
            }
            break;
        case SYSTEM_APP:
            appInfos.clear();
            for (ApplicationInfo app : listAppcations) {
                if ((app.flags & ApplicationInfo.FLAG_SYSTEM) != 0) {
                    appInfos.add(makeAppInfo(app));
                }
            }
            break;
        case THIRD_APP:
            appInfos.clear();
            for (ApplicationInfo app : listAppcations) {
                if ((app.flags & ApplicationInfo.FLAG_SYSTEM) <= 0) {
                    appInfos.add(makeAppInfo(app));
                } else if ((app.flags &
                        ApplicationInfo.FLAG_UPDATED_SYSTEM_APP) != 0) {
                    appInfos.add(makeAppInfo(app));
                }
            }
            break;
        case SDCARD_APP:
            appInfos.clear();
            for (ApplicationInfo app : listAppcations) {
                if ((app.flags &
                        ApplicationInfo.FLAG_EXTERNAL_STORAGE) != 0) {
                    appInfos.add(makeAppInfo(app));
                }
            }
            break;
        default:
            return null;
    }
    return appInfos;
}

private PMAppInfo makeAppInfo(ApplicationInfo app) {
    PMAppInfo appInfo = new PMAppInfo();
    appInfo.setAppLabel((String) app.loadLabel(pm));
    appInfo.setAppIcon(app.loadIcon(pm));
    appInfo.setPkgName(app.packageName);
    return appInfo;
}
```
###效果图
![](http://img.blog.csdn.net/20161014001833257)


###**9.3 Android Apk应用信息获取之ActivityManager**

>ActivityManager获取应用程序信息封装了不少Bean对象：

>* ActivityManager.MemoryInfo
>MemoryInfo有几个非常重要的字段：availMem（系统可用内存），totalMem（总内存），threshold（低内存的阈值，即区分是否低内存的临界值），lowMemory（是否处于低内存）
>* Debug.MemoryInfo
>这个MemoryInfo用于统计进程下的内存信息
>* RunningAppProcessInfo
>运行进程的信息，存储的字段有：processName（进程名），pid（进程pid），uid（进程uid），pkgList（该进程下的所有包）
>* RunningServiceInfo
>运行的服务信息，在它里面同样包含了一些服务进程信息，同时还有一些其他信息，activeSince（第一次被激活的时间、方式），foreground（服务是否在后台执行）

>下面同样通过例子来看看如何使用ActivityManager，我们封装一个Bean来保存我们所需的字段：

```
public class AMProcessInfo {

    private String pid;
    private String uid;
    private String memorySize;
    private String processName;

    public AMProcessInfo() {
    }

    public String getPid() {
        return pid;
    }

    public void setPid(String pid) {
        this.pid = pid;
    }

    public String getUid() {
        return uid;
    }

    public void setUid(String uid) {
        this.uid = uid;
    }

    public String getMemorySize() {
        return memorySize;
    }

    public void setMemorySize(String memorySize) {
        this.memorySize = memorySize;
    }

    public String getProcessName() {
        return processName;
    }

    public void setProcessName(String processName) {
        this.processName = processName;
    }
}
```

>接下来，调用getRunningAppProcesses方法，返回当前运行的进程信息，并将我们关心的信息保存到Bean中：

```
private List<AMProcessInfo> getRunningProcessInfo() {
        mAmProcessInfoList = new ArrayList<AMProcessInfo>();

        List<ActivityManager.RunningAppProcessInfo> appProcessList =
                mActivityManager.getRunningAppProcesses();

        for (int i = 0; i < appProcessList.size(); i++) {
            ActivityManager.RunningAppProcessInfo info =
                    appProcessList.get(i);
            int pid = info.pid;
            int uid = info.uid;
            String processName = info.processName;
            int[] memoryPid = new int[]{pid};
            Debug.MemoryInfo[] memoryInfo = mActivityManager
                    .getProcessMemoryInfo(memoryPid);
            int memorySize = memoryInfo[0].getTotalPss();

            AMProcessInfo processInfo = new AMProcessInfo();
            processInfo.setPid("" + pid);
            processInfo.setUid("" + uid);
            processInfo.setMemorySize("" + memorySize);
            processInfo.setProcessName(processName);
            mAmProcessInfoList.add(processInfo);
        }
        return mAmProcessInfoList;
    }
```

###效果图

![](http://img.blog.csdn.net/20161014001758413)


###**9.4 解析Packages.xml获取系统信息**
>在系统初始化的时候，PackageManager的底层实现类PackageManagerService会去扫描系统的一些特定目录，并且解析其中的Apk文件，同时，Android把它获取到的应用信息，保存到XML文件中，做成一个应用的花名册，当系统中的APK安装、删除、升级时，它也会被更新

>这个packages.xml位于/data/system/目录下，我们用adb pull命令把他导出来，进行查看分析：

>* < permissions>标签
> permissions标签定义了目前系统所有的权限，并分为两类：系统定义的和Apk定义的
>* < package>标签
> package代表的是一个apk的属性，其中各节点的信息含义大致为
	* name：APK的包名
	* cadePath：APK安装路径，主要在system/app和data/app两种，前者是厂商定制的Apk，后者是用户安装的第三方Apk
	* userid：用户ID
	* version：版本

>* < perms>标签
> 对应Apk的AndroidManifest文件中的< uses-permission>标签，记录Apk的权限信息

###**9.5 Android安全机制**
>无知识点

###9.5.1 Android安全机制简介
>* 第一道防线：代码安全机制——代码混淆proguard 
>proguard可以混淆关键代码、替换命名让破坏者阅读难，同样也可以压缩代码，优化编译后的Java字节码
>* 第二道防线：应用接入权限控制——清单文件权限声明，权限检查机制
>任何App在使用Android受限资源的时候，都需要显示向系统声明所需的权限，只有当一个应用App具有相应的权限，才能申请受限资源的时候，通过权限机制的检查并使用系统的Binder对象完成对系统服务的调用，但是这道防线也有先天性不足，如以下几项：
	* 被授予的权限无法停止
	* 在应用声明App使用权限的时，用户无法针对部分权限进行限制
	* 权限的声明机制与用户的安全理念相关
		Android系统通常按照以下顺序来检查操作者的权限： 
		* 首先，判断permission名称，如果为空则直接返回PERMISSION_DENIED 
		* 其次，判断Uid，如果为0则为root权限，不做权限控制，如果为System Service的Uid则为系统服务，不做权限控制，如果Uid与参数中的请求Uid不同则返回PERMISSION_DENIED 
		* 最后，通过调用packagemanageservice.checkUidPermission()方法来判断该Uid是否具有相应的权限，该方法会去XML的权限列表和系统级的platform.xml中进行查找 
>* 第三道防线：应用签名机制一数字证书 
>Android中所有的App都会有一个数字证书，这就是App的签名，数字证书用于保护App的作者和其App的信任关系，只有拥有相同数字签名的App，才会在升级时被认为是同一App，而且Android系统不会安装没有签名的App
>* 第四道防线：Linux内核层安全机制一一Uid 访问权限控制 
>Animid本质是基于Linux内核开发的，所以Android同样继承了Linux的安全特性，比如文件系统的权限控制是由user，group，other与读(r)，写(w)，执行(x)的不同组合来实现的，同样，Android也实现了这套机制，通常情况下，只有System、root用户才有权限访问到系统文件，而一般用户无法访问
>* 第五道防线：Android虚拟机沙箱机制——沙箱隔流 
>Android的App运行在虚拟机中，因此才有沙箱机制，可以让应用之间相互隔离，通常情况下，不同的应用之间不能互相访问，每个App都有与之对应的Uid，每个App也运行在单独的虚似机中，与其他应用完全隔离，在实现安全机制的基础上，也让应用之间能够互不影响，即时一个应用崩溃，，也不会导致其他应用异常

###9.5.2 Android系统安全隐患

>* 代码漏洞
>* Root风险
>* 安全机制不健全
>* 用户安全意识
>* Android开发原则与安全

>书本列举的几个点相信大家也都比较熟悉，所以不做解释了

###9.5.3 Android Apk反编译

>这里个人的博客有简单的介绍：[Android四大组件——Activity切换效果、杀死进程、杀死所有Activity、安装及反编译](http://blog.csdn.net/qq_30379689/article/details/52165571)

###9.5.4 Android Apk加密
>为了能够对编译好的JavaClass文件进行一些保护，通常会用ProGuard混淆代码：
>
>* ProGuard：用无意义的字母来重命名类、字段、方法和属性
>* 删除无用的类、字段、方法和属性，以及删除没用的注释，最大限度地优化字节码文件

>使用ProGuard也简单，在Android Studio中可以打开build.gradle（Module：app）：

```
buildTypes {
    release {
        minifyEnabled false
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```

>这里的minifyEnabled就是打开ProGuard的开关，proGuardFiles属于配置混淆文件，分为两部分：

>* 使用默认的混淆文件，位于< SDK目录>/tools/proguard/proguard-android.txt目录下
>* 使用自定义混淆文件，可以在项目的App文件夹找到这个文件，在这个文件里可以定义引入的第三方依赖包的混淆规则