>本篇文章翻新旧文章，采用markdown格式，原文时间2016-09-19 23:43 1253人阅读

[toc]

##下载

官网NDK传送门https://developer.android.com/ndk/downloads/index.html，如果没有用hosts免费进行翻墙是打不开官网的，可以关注我的博客查看文章

##JNI简介

JNI是java语言提供的Java和C/C++相互沟通的机制，Java可以通过JNI调用本地的C/C++代码，本地的C/C++的代码也可以调用java代码。JNI 是本地编程接口，Java和C/C++互相通过的接口。Java通过C/C++使用本地的代码的一个关键性原因在于C/C++代码的高效性。

##NDK简介
NDK是一系列工具的集合，帮助开发者快速开发C（或C++）的动态库，并能自动将so和java应用一起打包成apk。

##NDK目录结构

![这里写图片描述](http://img.blog.csdn.net/20160920105636634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##配置ndk路径

下载好官网的ndk，在File->Project Structure中设置ndk路径

![这里写图片描述](http://img.blog.csdn.net/20160919225138666?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##编写native方法

在MainActivity中定义并调用本地方法

```
public class MainActivity extends AppCompatActivity {  
  
    static{  
        //加载动态库，名字和c文件名相同  
        System.loadLibrary("hello");  
    }  
    //声明本地方法  
    public native String helloWord();  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
    }  
}  
```

##使用javah

使用AS自带的终端生成我们MainActivity的.h文件，进入该我们项目的对应位置，使用javah指令

![这里写图片描述](http://img.blog.csdn.net/20160919231841612?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##创建jni文件夹

创建JNI Floder，并将刚才生成的.h文件剪切到这个文件夹中

![这里写图片描述](http://img.blog.csdn.net/20160919232233679?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![这里写图片描述](http://img.blog.csdn.net/20160919232248176?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##编写jni

创建hello.c文件，来编写我们刚才的本地方法helloWord（以下是hello.c文件的内容），这里实现的方法是使用NDK的实例例子中的代码，导入c库后，方法名用：Java_包名_类名_方法名

```
#include <stdio.h>  
#include <stdlib.h>  
#include <jni.h>  
  
//env:结构体二级指针，该结构体中封装了大量的函数指针，可以帮助程序员实现某些常用功能  
//thiz:本地方法调用者的对象(MainActivity的对象)  
jstring Java_com_handsome_safe_MainActivity_helloWord(JNIEnv* env, jobject thiz){  
  
    char* cstr = "hello Word";  
    //把C字符串转换成java字符串  
    //jstring     (*NewStringUTF)(JNIEnv*, const char*);  
    jstring jstr = (*env)->NewStringUTF(env, cstr);  
    return jstr;  
}  
```

##解决Bug

创建一个内容为空的.c文件，这个是NDK的bug，不然编译会报错

![这里写图片描述](http://img.blog.csdn.net/20160919232724006?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##Gradle配置ndk

采用Gradle配置ndk方式，在项目根目录的gradle对应的位置进行2处设置

![这里写图片描述](http://img.blog.csdn.net/20160919233116539?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

![这里写图片描述](http://img.blog.csdn.net/20160919233247740?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

如果不采用Gradle配置ndk方式，则采用Android.mk方式，在对应的位置进行2处设置

![这里写图片描述](http://img.blog.csdn.net/20170901130240990?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170901130247167?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



##build

build一下项目，你会发现在相应的位置生成了so库

![这里写图片描述](http://img.blog.csdn.net/20160919233456946?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##运行

我们在MainActivity中添加一个按钮，并让它用Toast弹出调用helloWord的方法，查看效果图

![这里写图片描述](http://img.blog.csdn.net/20160919233625557?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

##源码下载

[源码下载](http://download.csdn.net/download/qq_30379689/9634296)：建议使用ImportProject导入