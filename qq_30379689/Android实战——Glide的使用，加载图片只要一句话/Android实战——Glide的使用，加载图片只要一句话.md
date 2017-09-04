#Glide的使用，加载图片只要一句话


----------

>本篇文章包括以下内容：

>* [前言](#a)
>* [Glide的简介](#b)
>* [Glide的配置](#c)
>* [Glide的使用](#d)
>* [结语](#e)

##**<span id="a">前言</span>**

用过的人都知道，加载图片哪家强，只有Glide，由于Glide采用链式调用，所以加载图片只要一句话，而且默认还带有淡出效果的动画

```java
Glide.with(context).load(url).thumbnail(0.1f).skipMemoryCache(true).into(imageView);
```

##**<span id="b">Glide的简介</span>**

官方的原话

* Glide是一个快速和有效的开源媒体管理和图像加载Android框架包装媒体解码，内存和磁盘缓存，和资源汇集成一个简单和易于使用的界面

其优点有

1. 使用简单

2. 可配置度高，自适应程度高

3. 支持常见图片格式，jpg、png、gif、webp

4. 支持多种数据源，网络、资源、assets 、File、Uri等

5. 高效缓存策略支持内存和硬盘缓存

6. 生命周期集成根据Activity/Fragment生命周期自动管理请求

7. 高效处理Bitmap

Github

* https://github.com/bumptech/glide

Glide加载网络图片的效果图

![](http://img.blog.csdn.net/20170305005523779?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**<span id="c">Glide的配置</span>**

配置很简单，只要在项目的Gradle添加依赖即可

```
compile 'com.github.bumptech.glide:glide:3.7.0'
compile 'com.android.support:support-v4:19.1.0'
```
当然，如果涉及到网络加载图片，记得增加网络权限

```
<uses-permission android:name="android.permission.INTERNET" />
```

##**<span id="d">Glide的使用</span>**

###**1、初始化**

Glide支持Activity和Fragment的绑定
```java
Glide.with(Context context);
Glide.with(Activity activity);
Glide.with(FragmentActivity activity);
Glide.with(Fragment fragment);
```

将Activity/Fragment作为with()参数的好处是，图片加载会和Activity/Fragment的生命周期保持一致

###**2、加载资源**

Glide支持网络资源、assets资源、Resources资源、File资源、Uri资源、字节数组

```
Glide.with(context).load("http://xxx.jpg").into(imageView);
Glide.with(context).load("file:///xxx.png").into(imageView);
Glide.with(context).load(R.mipmap.ic_launcher).into(imageView);
Glide.with(context).load(file).into(imageView);
Glide.with(context).load(uri).into(imageView);
Glide.with(context).load(byte[]).into(imageView);
```

###**3、加载gif图片**

① 加载静态gif图片

```
Glide.with(this).load(imageUrl).asBitmap().into(imageView);
```

② 加载动态gif图片

```
Glide.with(this).load(imageUrl).asGif().into(imageView);
```

###**4、设置加载中和加载失败的图片**

① 设置加载中图片

```
.placeholder(R.drawable.placeholder) 
```

② 设置加载失败图片

```
.error(R.drawable.error)
```

###**5、设置缩略图支持**

```java
//先加载缩略图 然后在加载全图
Glide.with(this).load(imageUrl).thumbnail(0.1f).into(imageView);
```

###**6、设置加载动画**

① 淡入淡出效果

```
Glide.with(this).load(imageUrl).crossFade().into(imageView);
```

② 无动画

```
Glide.with(this).load(imageUrl).dontAnimate().into(imageView);
```

③ 自定义动画

```
Glide.with(this).load(imageUrl).animate(R.anim.alpha_in).into(imageView);
```

###**7、设置监听回调**

```
Glide.with(this).load(imageUrl).listener(RequestListener listener).into(imageView);
```

###**8、设置加载尺寸**

```java
//指定尺寸
Glide.with(this).load(imageUrl).override(800, 800).into(imageView);
//拉伸截取中间部分显示
Glide.with(this).load(imageUrl).centerCrop().into(imageView);
//等比拉伸填满ImageView
Glide.with(this).load(imageUrl).fitCenter().into(imageView);
```

###**9、设置缓存策略**

① 设置跳过内存缓存

```java
Glide.with(this).load(imageUrl).skipMemoryCache(true).into(imageView);
```

② 设置缓存策略

```java
Glide.with(this).load(imageUrl).diskCacheStrategy(DiskCacheStrategy.ALL).into(imageView);
```

* DiskCacheStrategy.ALL：缓存源资源和转换后的资源
* DiskCacheStrategy.NONE：不作任何磁盘缓存
* DiskCacheStrategy.SOURCE：缓存源资源
* DiskCacheStrategy.RESULT：缓存转换后的资源

③ 清理缓存

```
//清理磁盘缓存 需要在子线程中执行
Glide.get(this).clearDiskCache();
//清理内存缓存  可以在UI主线程中进
Glide.get(this).clearMemory();
```
###**10、BitmapTransformation**

你可能不知道Glide在Github上还有一个库，可以处理图片效果，比如裁剪、圆角、高斯模糊等等

① 引入依赖库

```
compile 'jp.wasabeef:glide-transformations:2.0.1'
```

② 实现高斯模糊

```java
//radius取值1-25，值越大图片越模糊
Glide.with(context).load(url).bitmapTransform(new BlurTransformation(context, radius)).into(imageView);
```


##**<span id="e">结语</span>**

Glide用法真的很舒服，如果你是老手，可以尝试封装GlideUtils，让它使用到你的项目中，好不好用只有在项目中才能发挥出来

