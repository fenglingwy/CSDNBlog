[toc]

##前提准备

* Android Studio+Cmake+云服务器
* 这里是在阿里云服务器上编译，采用Ubuntu 16.04 64位
* 采用android-ndk-r10e-linux-x86_64.bin编译ffmpeg-2.6.9
* 采用Xshell和Xftp进行服务器的操作

##编译FFmpeg

1、上传ndk和ffmpeg

通过xftp连接服务器，在/usr目录下创建ndk和ffmpeg目录，上传ndk.bin和ffmpeg.zip

![这里写图片描述](http://img.blog.csdn.net/20170904231204634?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170904231156942?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、工具Vim的使用

Ubuntu一般自带vim不用安装，如果输入vim没有安装过的话，使用下面指令进行安装

```
sudo apt-get install vim-gtk
```

vim基本使用

* 命令模式
    * i：进入编辑模式
    * x：删除
    * dd：删除行
* 编辑模式：
    * Esc：退出编辑模式
    * shift+:q!：强制退出
    * shift+zz：保存退出

Vim配置

```
vim /etc/vim/vimrc
```

进入编辑模式，配置以下内容

![这里写图片描述](http://img.blog.csdn.net/20170904231142911?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、解压ndk

进入/usr目录，使用命令给ndk权限

```
chmod 777 -R ndk
```

进入/usr/ndk目录，执行命令解压ndk

```
./android-ndk-r10e-linux-x86_64.bin
```

4、配置ndk环境变量

通过vim进入环境变量

```
vim ~/.bashrc
```
配置以下内容，注意空格

![这里写图片描述](http://img.blog.csdn.net/20170904231234207?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

更新配置信息

```
source ~/.bashrc
```

输入ndk-build有以下的内容，就说明已经配置正确了

![这里写图片描述](http://img.blog.csdn.net/20170904231244873?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、解压ffmpeg

升级apt-get否则找不到zip安装

```
apt-get update
```

安装zip，会顺带安装unzip

```
apt-get install zip
```

解压/usr/ffmpeg目录下的ffmpeg.zip包

```
unzip ffmpeg-2.6.9.zip
```

给ffmpeg这个解压好的目录权限

```
chmod 777 -R ffmpeg-2.6.9
```

6、编译ffmpeg

上传ffmpeg编译脚本到ffmpeg目录

![这里写图片描述](http://img.blog.csdn.net/20170904231302174?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

Shell脚本的内容，主要是指定编译的NDK、指定的平台、编译完的平台和ffmpeg哪些模块需要和不需要，注意空格等都会影响，必须是UTF-8无Bom头的编码，而且必须是Linux可以辨别的文件，所以可以从Linux创建一个sh文件，再传输出来，把内容加上去

```
#!/bin/bash
make clean
export NDK=/usr/ndk/android-ndk-r10e
export SYSROOT=$NDK/platforms/android-9/arch-arm/
export TOOLCHAIN=$NDK/toolchains/arm-linux-androideabi-4.8/prebuilt/linux-x86_64
export CPU=arm
export PREFIX=$(pwd)/android/$CPU
export ADDI_CFLAGS="-marm"

./configure --target-os=linux \
--prefix=$PREFIX --arch=arm \
--disable-doc \
--enable-shared \
--disable-static \
--disable-yasm \
--disable-symver \
--enable-gpl \
--disable-ffmpeg \
--disable-ffplay \
--disable-ffprobe \
--disable-ffserver \
--disable-doc \
--disable-symver \
--cross-prefix=$TOOLCHAIN/bin/arm-linux-androideabi- \
--enable-cross-compile \
--sysroot=$SYSROOT \
--extra-cflags="-Os -fpic $ADDI_CFLAGS" \
--extra-ldflags="$ADDI_LDFLAGS" \
$ADDITIONAL_CONFIGURE_FLAG
make clean
make
make install
```

由于自带的ffmpeg配置文件中，对文件的命名并不是.so结尾的，这样会导致我们Android的使用，所以在ffmpeg目录下的configure中修改以下内容，然后传输到服务器替代掉原先的configure

![这里写图片描述](http://img.blog.csdn.net/20170904231326116?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

configure文本内容

```
#SLIBNAME_WITH_MAJOR='$(SLIBNAME).$(LIBMAJOR)'
#LIB_INSTALL_EXTRA_CMD='$$(RANLIB) "$(LIBDIR)/$(LIBNAME)"'
#SLIB_INSTALL_NAME='$(SLIBNAME_WITH_VERSION)'
#SLIB_INSTALL_LINKS='$(SLIBNAME_WITH_MAJOR) $(SLIBNAME)'
#Hensen
SLIBNAME_WITH_MAJOR='$(SLIBPREF)$(FULLNAME)-$(LIBMAJOR)$(SLIBSUF)'
LIB_INSTALL_EXTRA_CMD='$$(RANLIB)"$(LIBDIR)/$(LIBNAME)"'
SLIB_INSTALL_NAME='$(SLIBNAME_WITH_MAJOR)'
SLIB_INSTALL_LINKS='$(SLIBNAME)'
```

进入ffmpeg目录，执行shell脚本，等几分钟ffmpeg就编译成功了

```
cd ffmpeg-2.6.9
./build_android.sh
```

##测试ffmpeg

1、取出我们编译好的include文件夹和lib文件夹

![这里写图片描述](http://img.blog.csdn.net/20170904231337616?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、创建Android Studio项目

勾选Include C++ support，并选中C++和另外的两个依赖库

3、由于我们编译的是arm版本的，所以选择以下的so库文件放在armeabi目录，将include放在libs目录中

![这里写图片描述](http://img.blog.csdn.net/20170904231351897?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、在app的Gradle的android标签下设置jniLibs目录指向我们工程的libs并增加so库的过滤

```
android {
    compileSdkVersion 25
    buildToolsVersion "25.0.3"
    defaultConfig {
        applicationId "com.handsome.ndkffmpeg"
        minSdkVersion 16
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
        externalNativeBuild {
            cmake {
                cppFlags "-std=c++11 -frtti -fexceptions"
                abiFilters "armeabi"
            }
        }
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    sourceSets.main {
        jniLibs.srcDirs = ['libs']
        jni.srcDirs = []
    }
    externalNativeBuild {
        cmake {
            path "CMakeLists.txt"
        }
    }
}
```

5、在运行的类中按顺序添加加载本地库

```java
public class MainActivity extends AppCompatActivity {

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("avutil-54");
        System.loadLibrary("swresample-1");
        System.loadLibrary("avcodec-56");
        System.loadLibrary("avformat-56");
        System.loadLibrary("swscale-3");
        System.loadLibrary("postproc-53");
        System.loadLibrary("avfilter-5");
        System.loadLibrary("avdevice-56");
        System.loadLibrary("native-lib");
    }

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Example of a call to a native method
        TextView tv = (TextView) findViewById(R.id.sample_text);
        tv.setText(stringFromJNI());
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI();
}
```

6、编写CMake文件

```
cmake_minimum_required(VERSION 3.4.1)

include_directories(./libs/include)
link_directories(./libs/${ANDROID_ABI})

add_library( # Sets the name of the library.
             native-lib

             # Sets the library as a shared library.
             SHARED

             # Provides a relative path to your source file(s).
             src/main/cpp/native-lib.cpp )

target_link_libraries(native-lib
                       avutil-54
                       swresample-1
                       avcodec-56
                       avformat-56
                       swscale-3
                       postproc-53
                       avfilter-5
                       avdevice-56)
```

7、运行程序，如果通过则表示so库加载成功，可以正常使用了

##源码下载
