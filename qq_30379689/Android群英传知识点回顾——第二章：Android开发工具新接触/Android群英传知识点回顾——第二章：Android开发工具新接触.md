# Android群英传知识点回顾——第二章：Android开发工具新接触 #
----------
##知识点目录##
>* **2.1 Google生态系统**
	* 2.1.1 Android Studio初体验
	* 2.1.2 Android Studio配置
>* **2.2 Android高级使用技巧**
	* 2.2.1 更新SDK
	* 2.2.2 Android Studio常用界面
	* 2.2.3 导入Android Studio工程
>* **2.3 ADB命令使用技巧**
	* 2.3.1 ADB基础
	* 2.3.2 ADB常用命令
	* 2.3.2 ADB命令来源
>* **2.4 模拟器使用与配置**

----------
##知识点回顾##

###**2.1 Android开发IDE介绍**
>Eclipse缺点：

>* 内存占用高
>* 经常崩溃
>* 开发界面不美观
>* Android作为插件的形式存在于Eclipse
>* ......
###2.1.1 Android Studio初体验
>Android 5.x融合了车载、可穿戴、TV等各个方面的系统
>Android Studio整合了它的云服务、Go语言、车载、可穿戴、TV等各项功能
>使用Android Studio原因——UI设计更加即时，代码提示更加丰富，Lint更加智能等
###2.1.2 Android Studio配置
>Android Studio官网：http://developer.android.com/sdk/installing/studio.html
>AndroidDevTool镜像网站：http://www.android.evtools.cn/

>配置JDK环境变量：

>* JAVA_HOME：C:\xxx\java\jdk1.x（JDK目录）
>* CLASSPATH：%JAVA_HOME%\lib
>* %JAVA_HOME%\lib\tools.jar
>* %JAVE_HOME%\lib\dt.jar
>* Path：%JAVA_HOME%\bin

> Android Studio使用技巧：

>*  Eclipse导出工程到Android Studio使用Export->Generate Gradle build files
>* 通过小扳手进入设置界面，进入Appearance标签
	* 在Theme中设置主题
	* "Override default font"可以改变字体不兼容问题
>* 进入Editor标签
	* 进入Colors&Fonts标签，单击"Save as"按钮，修改字号大小
	* 进入Other标签，勾选"show quick doc on mouse move"，开启显示悬浮提示
	* 进入General标签，在"Code Sensitive Completion"选择None，开启提示不区分大小写
>* 进入Keymap标签
	* 修改快捷键风格
	* 修改快捷键
###**2.2 Android Studio高级使用技巧**
> 配置好了Android Studio就等于配好了一把好枪，但是别忘了我们还得买子弹，这里的子弹自然是开发Android最重要的SDK开发工具
###2.2.1 SDK更新
> SDK Manager配置镜像代理
> 建议保持最新的SDK Build-tools
> SDK API文档资源（Documentation for Android SDK）
> Android源代码资源（Sources for Android SDK）
###2.2.2 Android Studio常用界面
>* Debug窗口：调试、截图、录制屏幕等操作
>* Memory Monitor：监视内存消耗，对CPU使用率的实时监控
>* Android Device Monitor：分析应用性能、优化调试、展示CPU Load信息等
>* 断点调试：断点查看、实时计算变量值、多种调试方法等功能
###2.2.3 导入Android Studio工程
> 解决导入Android Studio卡死的问题

>* 在当前版本Gradle创建一个正常的项目
>* 复制本地项目中的"gradle"文件夹和"build.gradle"文件去替换要导入项目中的这两个文件夹
>* 导入我们所需的Android Studio工程
###**2.3 ADB命令使用技巧**
> 手机和电脑的"脐带"，ADB——Android Debug Bridge
###2.3.1 ADB基础
> ADB位于SDK的platform-tools目录下，在该目录下启动cmd：

>* 输入adb version查看adb版本号
>* 安装对应的手机驱动，如豌豆荚、91、QQ手机助手
>* 进入手机Setting中，开启USB调试模式
>* 输入adb shell，进入Shell
###2.3.2 ADB常用命令
>* 显示系统中全部Android平台：android list targets
>* 安装Apk程序：adb install -r 应用程序.apk
>* 向手机安装Apk程序：adb push D:\Test.apk /system/app/
>* 向手机写入文件：adb push D:\Test.txt /system/app/
>* 从手机获取文件：adb pull /system/temp/ D:\file.txt
>* 查看Log：
	* adb shell
	* shell@t03gchn：/$ logcat | grep "abc"
>* 删除应用：
	* adb remount（重新挂载系统分区，使系统分区重新可写）
	* adb shell
	* cd system/app
	* rm *.apk
>* 查看系统盘符adb shell df
>* 输入所有已经安装的应用adb shell pm list packages -f
>* 模拟按键输入：
	* menu：adb shell input keyevent 82
	* home：adb shell input keyevent 3
	* up：adb shell input keyevent 19
	* down：adb shell input keyevent 20
	* left：adb shell input keyevent 21
	* right：adb shell input keyevent 22
	* enter：adb shell input keyevent 66
	* back：adb shell input keyevent 4
>* 模拟滑动输入：adb shell input touchscreen swipe 200 500  400 500
>* 查看运行状态：adb shell dumpsys
>* 启动一个Activity：adb shell am start -n 包名/包名+类名
>* 录制屏幕：adb shell screenrecord /sdcard/demo.mp4
>* 重新启动：adb reboot
###**2.4 模拟器使用与配置**
> 第三方模拟器Genymotion官网：http://www.genymotion.net/