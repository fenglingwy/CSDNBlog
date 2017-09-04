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

