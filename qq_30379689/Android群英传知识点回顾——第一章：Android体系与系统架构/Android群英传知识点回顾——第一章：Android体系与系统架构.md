# Android群英传知识点回顾——第一章：Android体系与系统架构 #
----------

##知识点目录##
>* **1.1 Google生态系统**
>* **1.2 Android系统架构**
	* 1.2.1 Linux
	* 1.2.2 Dalvik与ART
	* 1.2.3 Framework
	* 1.2.4 Standard libraries
	* 1.2.5 Application
>* **1.3 Android App组件架构**
	* 1.3.1 Android四大组件如何协同工作
	* 1.3.2 应用运行上下文对象
>* **1.4 Android系统源代码目录与系统目录**
	* 1.4.1 Android系统源代码目录
	* 1.4.2 Android系统目录
	* 1.4.3 Android App文件目录

----------
##知识点回顾##

###**1.1 Google生态系统**
> Android底层通过最快的C语言保证效率，上层采用java简单、快速进行开发



###**1.2 Android系统架构**
>Android大致分为了四层：
>
>* Linux内核层
* 库和运行时
* Framework层
* 应用层


###1.2.1 Linux
> Android最低层最核心的部分，包含：
> 
>* 核心服务
>* 硬件驱动
>* 进程管理
>* 安全系统
>* ......

###1.2.2 Dalvik与ART
>Dalvik包含了一整套的Android运行环境虚拟机。它的特点是运行时编译
>ART模式取代了Dalvik，ART采用安装时进行编译，运行时不用编译


###1.2.3 Framework
>  如果你以后要研究Framework的具体流程，基本就是在和它们打交道


###1.2.4 Standard libraries
> 这里包含的是Android中的一些标准库，即开发者在开源环境中可以使用的开发库

###1.2.5 Application
> 无知识点

###**1.3 Android App组件架构**
>Android四大组件：
>
>* Activity
>* BroadCastReciever
>* ContentProvider
>* Service
###1.3.1 Android四大组件如何协同工作
>* Activity：人机交互的界面，负责向用户展示信息和处理结果
>* ContentProvider：获取其他应用的信息
>* Service：后台计算、下载、处理结果信息
>* BroadcastReciever：获取广播信息
>* Intent：信息传递的载体，可进行组件与组件之间通信、传递信息和交换数据
###1.3.2应用运行上下文对象
> Activity、Service、Application都继承自Context
> 
> Android应用程序会在如下所示的几个时间点创建应用上下文Context：
>
>* 创建Application
>* 创建Activity
>* 创建Service
###**1.4 Android系统源代码目录与系统目录**
> 无知识点
###1.4.1 Android系统源代码目录
> Android源代码网站：http://androidxref.com/
> 
> 源代码目录：
> 
>* Makefile
>* bionic（bionic C库） 
>* bootable（启动引导相关代码）
>* build（存放系统编译规则等基础开发包配置）
>* cts （Google兼容性测试标准）
>* dalvik（dalvik虚拟机）
>* development（应用程序开发相关）
>* external（android使用的一些开源的模块）
>* frameworks（Framework框架核心）
>* hardware（厂商硬件适配层HAL代码）
>* out（编译完成后的代码输出目录）
>* packages（应用程序包）
>* prebuilt（x86和arm架构下预编译资源）
>* sdk（sdk及模拟器）
>* system（底层文件系统库、应用及组件）
>* vendor（厂商定制代码）
>
>Makefile机制：
>
>* 定义了一系列规则来指定需要编译的模块文件
>* 描述Android各个组件间的联系并指导它们进行自动化编译
>* 每个最小功能单位目录下，都有一个Makefile文件，通过一级一级向上传递把整个源代码联系起来
###1.4.2 Android系统目录
>通过Linux的ls命令查看系统的根目录：
>
>* /system/app/ ：系统App
>* /system/bin/：Linux自带的组件
>* /system/build.prop：系统的属性信息
>* /system/fonts/：系统字体存放目录
>* /system/framework/：系统核心文件、框架层
>* /system/lib/：共享库（.so）文件
>* /system/media/：系统提示音、系统铃声
>* /system/media/audio/：系统默认的铃声
>* /system/media/alarms/：闹铃提醒
>* /system/media/notification/：短信或提示音
>* /system/media/ringtones/：来电铃声
>* /system/media/ui/：界面音效
>* /system/usr/：用户配置文件
>* /data/app/：用户大部分数据信息
>* /data/data/：App数据信息、文件信息、数据库信息
>* /data/system/：手机各项系统信息
>* /data/misc/：大部分的Wi-Fi、VPN信息
###1.4.3 Android App文件目录
> 关于Eclipse和Android Studio的文件目录区别：
>
>* Android Studio中的Project就相当于Eclipse里面的WorkSpace
>* Android Studio中的Module就相当于Eclipse里面的Project 