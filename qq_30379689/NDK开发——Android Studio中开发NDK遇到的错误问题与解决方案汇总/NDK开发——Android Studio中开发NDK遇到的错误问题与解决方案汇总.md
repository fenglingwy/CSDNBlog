##错误一

在Genymotion中执行程序时，会出现找不到ndk编译的库的错误，可是libs目录下明明存在对应的so库

```
Caused by: java.lang.UnsatisfiedLinkError: dlopen failed: "/data/app/com.handsome.ndkvoice-2/lib/arm/libfmodL.so" has unexpected e_machine: 40
```

解决方案：

1. 由于Genymotion不支持arm架构，所以得用真机测试

##错误二

在Genymotion中执行程序时，会出现找不到ndk编译的库的错误，可是libs目录下明明存在对应的so库

```
Caused by: java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader[DexPathList[[zip file "/data/app/com.handsome.ndkvoice-1/base.apk"],nativeLibraryDirectories=[/data/app/com.handsome.ndkvoice-1/lib/arm, /vendor/lib, /system/lib]]] couldn't find "libfmodL.so"
```

解决方案：

1. 由于你的so库都放在libs目录下，没有放在AndroidStudio的jniLibs目录下，所以找不到对应的so库，这个时候应该在app的Gradle上添加下面代码，让jniLibs目录指向libs目录

```
sourceSets.main {
    jniLibs.srcDirs = ['libs']
    jni.srcDirs = []
}
```

##错误三

在运行FFmpeg项目的时候，会出现以下错误

```
Error:(18) undefined reference to 'avcodec_register_all()'
```

解决方案：

这是由于Google留下的坑，正确写法如下

```
extern "C" {
#include "libavcodec/avcodec.h"
}

extern "C"
JNIEXPORT void JNICALL
Java_com_handsome_ndkffmpeg_FFmpegUtils_video2Yua(JNIEnv *env, jclass jclazz, jstring video_path_,
                                                  jstring yuv_path_) {
    const char *video_path = env->GetStringUTFChars(video_path_, 0);
    const char *yuv_path = env->GetStringUTFChars(yuv_path_, 0);

    //1、注册所有组件
    avcodec_register_all();

}
```
