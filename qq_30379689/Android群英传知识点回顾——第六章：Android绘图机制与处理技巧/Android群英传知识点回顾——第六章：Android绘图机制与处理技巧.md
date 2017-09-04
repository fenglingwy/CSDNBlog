# Android群英传知识点回顾——第六章：Android绘图机制与处理技巧 #
----------
##知识点目录##
>* **6.1 屏幕的尺寸信息**
	*  6.1.1 屏幕参数
	*  6.1.2 系统屏幕密度
	*  6.1.3 独立像素密度dp
	*  6.1.4 单位转换
>* **6.2 2D绘图基础**
>* **6.3 Android XML绘图**
	*  6.3.1 Bitmap
	*  6.3.2 Shape
	*  6.3.3 Layer
	*  6.3.4 Selector
>* **6.4 Android绘图技巧**
	*  6.4.1 Canvas
	*  6.4.2 Layer图层
>* **6.5 Android图像处理之色彩特效处理**
	*  6.5.1 色彩矩阵分析
	*  6.5.2 Android颜色矩阵——ColorMatrix
	*  6.5.3 常用图像颜色矩阵处理效果
	*  6.5.4 像素点分析
	*  6.5.5 常用图像像素点处理效果
>* **6.6 Android图像处理之图形特效处理**
	*  6.6.1 Android变形矩阵——Matrix
	*  6.6.2 像素块分析
>* **6.7 Android图像处理之画笔特效处理**
	*  6.7.1 PorterDuffXfermode
	*  6.7.2 Shader
	*  6.7.3 PathEffect
>* **6.8 View之孪生兄弟——SurfaceView**
	*  6.8.1 SurfaceView与View的区别
	*  6.8.2 SurfaceView的使用
	*  6.8.3 SurfaceView实例

----------
##知识点回顾##

>由于这一章比较难理解，所以大部分知识点都是摘抄原文的，如果是没学习过线性代数的同学，那就难上加难了，不过克服它，是你进阶高级工程师的必经之路，文章比较长，请耐心观看

###**6.1 屏幕的尺寸信息**
>无知识点

###6.1.1 屏幕参数
>* 屏幕大小：屏幕对角线的长度，通常用寸来度量
>* 分辨率：手机屏幕的像素点个数
>* PPI：每英寸像素，又称DPI，由对角线像素点除以屏幕大小获得

###6.1.2 系统屏幕密度
![](http://img.blog.csdn.net/20161009175259931)

###6.1.3 独立像素密度dp
>Android系统使用mdpi即密度值为160的屏幕作为标准，在这个屏幕上1px = 1dp

>ldpi：mdpi：hdpi：xhdpi：xxhdpi=3：4：6：8：12

###6.1.4 单位转换
>这里加0.5f的巧用目的是：四舍五入

>dp == dip

```
/**
 * dp、sp转换成px的工具类
 * 
 */
public class DisplayUtils {
    /**
     * 将px值转换为dpi或者dp值，保持尺寸大小不变
     *
     * @param content
     * @param pxValus
     * 
     * @return
     */
    public static int px2dip(Context content, float pxValus) {
        final float scale = content.getResources().getDisplayMetrics().density;
        return (int) (pxValus / scale + 0.5f);
    }

    /**
     * 将dip和dp值转换为px值,保证尺寸大小不变
     *
     * @param content
     * @param dipValus
     * 
     * @return
     */
    public static int dip2px(Context content, float dipValus) {
        final float scale = content.getResources().getDisplayMetrics().density;
        return (int) (dipValus / scale + 0.5f);
    }

    /**
     * 将px值转换为sp值,保证文字大小不变
     *
     * @param content
     * @param pxValus
     * @return
     */
    public static int px2sp(Context content, float pxValus) {
        final float fontScale = content.getResources().getDisplayMetrics().scaledDensity;
        return (int) (pxValus / fontScale + 0.5f);
    }

    /**
     * 将sp值转换为px值,保证文字大小不变
     *
     * @param content
     * @param spValus
     * @return
     */
    public static int sp2px(Context content, float spValus) {
        final float fontScale = content.getResources().getDisplayMetrics().scaledDensity;
        return (int) (spValus / fontScale + 0.5f);
    }
}
```
>系统提供了TypedValue类帮助我们转换

```
/**
 * dp2px
 */
protected int dp2px(int dp){
    return (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP,dp,getResources().getDisplayMetrics());
}

/**
 * sp2px
 */
protected int sp2px(int sp){
    return (int)TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_SP,sp,getResources().getDisplayMetrics());
}
```

###**6.2 2D绘图基础**
>Paint作为一个重要的元素有以下方法：

>* setAntiAlias()：设置画笔的锯齿效果
>* setColor()：设置画笔的颜色
>* setARGB()：设置画笔的A、R、G、B值
>* setAlpha()：设置画笔的Alpha值
>* setTextSize()：设置字体的尺寸
>* setStyle()：设置画笔的风格（空心或实心）
>* setStrokeWidth()：设置空心边框的宽度

>设置Paint的Style可以画出空心或者实心的矩形：

>* paint.setStyle(Paint.Style.STROKE)：空心效果
>* paint.setStyle(Paint.Style.FILL)：实心效果

![](http://img.blog.csdn.net/20161009211152431)

>系统通过提供的Canvas对象来提供绘图方法：

>* canvas.drawPoint(x, y, paint)：绘制点
>* canvas.drawLine(startX, startY ,endX, endY, paint)：绘制直线
>* canvas.drawLines(new float[]{startX1, startY1, endX1, endY1,......,startXn, startYn, endXn, endYn}, paint)：绘制多条直线
>* canvas.drawRect(left, top, right, bottom, paint)：绘制矩形
>* canvas.drawRoundRect(left, top, right, bottom, radiusX, radiusY, paint)：绘制圆角矩形
>* canvas.drawCircle(circleX, circleY, radius, paint)：绘制圆
>* canvas.drawOval(left, top, right, bottom, paint)：绘制椭圆
>* canvas.drawText(text, startX, startY, paint)：绘制文本
>* canvas.drawPosText(text, new float[]{X1,Y1,X2,Y2,......Xn,Yn}, paint)：在指定位置绘制文本
>* Path path = new Path();
>  path.moveTo(50, 50);
>  path.lineTo(100, 100);
>  path.lineTo(100, 300);
>  path.lineTo(300, 50); 
>  canvas.drawPath(path, paint)：绘制路径

![](http://img.blog.csdn.net/20161009211736439)

>* paint.setStyle(Paint.Style.STROKE);
>drawArc(left, top, right,bottom, startAngle, sweepAngle, true, paint)：绘制扇形
>* paint.setStyle(Paint.Style.STROKE);
>drawArc(left, top, right,bottom, startAngle, sweepAngle, false, paint)：绘制弧形
>* paint.setStyle(Paint.Style.FILL);
>drawArc(left, top, right,bottom, startAngle, sweepAngle, true, paint)：绘制实心扇形
>* paint.setStyle(Paint.Style.FILL);
>drawArc(left, top, right,bottom, startAngle, sweepAngle, false, paint)：绘制实心弧形

![](http://img.blog.csdn.net/20161009212550478)

###**6.3 Android XML绘图**
>无知识点

###6.3.1 bitmap
>通过这样在XML中使用Bitmap就可以将图片直接转换成了Bitmap在程序中使用

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@mipmap/ic_launcher">
</bitmap>
```

###6.3.2 Shape

>通过Shape可以在XML中绘制各种形状

```
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    //默认为rectangle
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners   //当shape="rectangle"时使用
	    //半径，会被后面的单个半径属性覆盖，默认为1dp
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient   //渐变
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:useLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size   //指定大小，一般在imageview配合scaleType属性使用
        android:width="integer"
        android:height="integer" />
    <solid   //填充颜色
        android:color="color" />
    <stroke   //指定边框
        android:width="integer"
        android:color="color"
        //虚线宽度
        android:dashWidth="integer"
        //虚线间隔宽度
        android:dashGap="integer" />
</shape>
```
>下面通过渐变来实现的阴影效果

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">

    <gradient
        android:angle="45"
        android:endColor="#805FBBFF"
        android:startColor="#FF5DA2FF" />

    <padding
        android:bottom="7dp"
        android:left="7dp"
        android:right="7dp"
        android:top="7dp" />

    <corners android:radius="8dp" />

</shape>
```

![](http://img.blog.csdn.net/20161009214710097)

###6.3.3 Layer

>通过Layer会产生图片依次叠加的效果

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--图片1-->
    <item android:drawable="@mipmap/ic_launcher"/>

    <!--图片2-->
    <item
        android:bottom="10dp"
        android:top="10dp"
        android:right="10dp"
        android:left="10dp"
        android:drawable="@mipmap/ic_launcher"
        />

</layer-list>
```

###6.3.4 Selector
>Selector的作用在于帮助开发者实现静态绘图中的事件反馈，通过给不同的事件设置不同的图像，从而在程序中根据用户输入，返回不同的效果

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 默认时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" />
    <!-- 没有焦点时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_window_focused="false" />
    <!-- 非触摸模式下获得焦点并单击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_pressed="true" android:state_window_focused="true" />
    <!-- 触摸模式下单击时的背景图片-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="false" android:state_pressed="true" />
    <!--选中时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_selected="true" />
    <!--获得焦点时的图片背景-->
    <item android:drawable="@mipmap/ic_launcher" android:state_focused="true" />
</selector>
```
>下面这个例子实现了一个具有点击反馈效果的、圆角举证Selector

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:state_pressed="true">
        <shape android:shape="rectangle">
            <!--填充的颜色-->
            <solid android:color="#33444444" />
            <!--设置按钮的四个角为弧形-->
            <!--android:radius 弧形的半径-->
            <corners android:radius="5dp" />
            <!--padding：Button里面的文字与Button边界的间隔-->
            <padding android:bottom="10dp" android:left="10dp" android:right="10dp" android:top="10dp" />
        </shape>
    </item>

    <item>
        <shape android:shape="rectangle">
            <!--填充的颜色-->
            <solid android:color="#FFFFFF" />
            <!--设置按钮的四个角为弧形-->
            <!--android:radius 弧形的半径-->
            <corners android:radius="5dp" />
            <!--padding：Button里面的文字与Button边界的间隔-->
            <padding android:bottom="10dp" android:left="10dp" android:right="10dp" android:top="10dp" />
        </shape>
    </item>
</selector>
```

![](http://img.blog.csdn.net/20161009214634988)


###**6.4 Android绘图技巧**
>无知识点

###6.4.1 Canvas
>Canvas提供了以下几种非常有用的方法：

>* Canvas.save()：保存画布，将之前所有已绘制图像保存起来，让后续的操作就好像在一个新的图层上操作一样
>* Canvas.restore()：在save()之后绘制的所有图像与save()之前的图像进行合并，可以理解为Photoshop中的合并图层操作
>* Canvas.translate()：画布平移
>* Canvas.roate()：画布翻转

>通过一个实例——仪表盘，来理解这几个方法，将仪表盘分为以下几个元素：

>* 仪表盘：外面的大圆盘
>* 刻度线：包含四个长的刻度线和其他短的刻度线
>* 刻度值：包含长刻度线对应的大的刻度值和其他小的刻度值
>* 指针：中间的指针，一粗一细两根指针

>仪表盘......见经典代码案例一

###6.4.2 Layer图层
> 一张复杂的画可以由很多个图层叠加起来，形成一个复杂的图像，使用saveLayer()方法来创建一个图层，图层同样是基于栈的结构进行管理的

![](http://img.blog.csdn.net/20161009235149012)

>Android通过调用saveLayer()方法，saveLayerAlpha()方法将一个图层入栈，使用restore()方法、restoreToCount()方法将一个图层出栈，仿照API Demos里面的一个实例来使用Layer

```
@Override
protected void onDraw(Canvas canvas) {
    canvas.drawColor(Color.WHITE);
    mPaint.setColor(Color.BLUE);
    canvas.drawCircle(150, 150, 100, mPaint);

    canvas.saveLayerAlpha(0, 0,400,400,127,LAYER_TYPE_NONE);
    mPaint.setColor(Color.RED);
    canvas.drawCircle(200, 200, 100, mPaint);
    canvas.restore();
}
```
>* 当透明度为127时，即半透明
>* 当透明度为255时，即完全不透明
>* 当透明度为0时，即完全透明

![](http://img.blog.csdn.net/20161009235422310)

###**6.5 Android图像处理之色彩特效处理**
>Bitmap图片都是由点阵和颜色值组成的，所谓点阵就是一个包含像素的矩阵，每一个元素对应着图片的一个像素。而颜色值——ARGB，分别对应透明度、红、绿、蓝这四个通道分量，它们共同决定了每个像素点显示的颜色

###6.5.1 色彩矩阵分析
>在色彩处理中，我们通常用三个角度描述一张图片：

>* 色调：物体传播的颜色
>* 饱和度：颜色的纯度，从0（灰）-100%（饱和）来进行描述
>* 亮度：颜色的相对明暗程度

>而在Android中，系统会使用一个颜色矩阵——ColorMatrix,来处理这些色彩的效果，Android中的颜色矩阵是4X5的数字矩阵，他用来对颜色色彩进行处理，而对于每一个像素点，都有一个颜色分量矩阵来保存ARGB值

![](http://img.blog.csdn.net/20161010000215738)

>根据前面对矩阵A、C的定义，通过矩阵乘法运算法则，可以得到：

![](http://img.blog.csdn.net/20161010000422642)

>矩阵运算的乘法计算过程如下：

![](http://img.blog.csdn.net/20161010000548768)

>我们观察颜色矩阵A

![](http://img.blog.csdn.net/20161010000721503)

>从这个公式可以发现

>* 第一行的abcde用来决定新的颜色值R——红色
>* 第二行的fghij用来决定新的颜色值G——绿色
>* 第三行的kimno用来决定新的颜色值B——蓝色
>* 第四行的pqrst用来决定新的颜色值A——透明度
>* 矩阵A中的第五列——ejot值分别用来决定每个分量中的offset，即偏移量

>通过一个小例子来讲解：
>首先重新看一下矩阵变换计算公式，以R分量为例，计算过程如下：

```
R1 = a * R + b* G + c*B+d *A + e
```
>如果让a = 1,b,c,d,e都等于0，那么计算的结果为R1 = R，因此我们可以构建一个矩阵

![](http://img.blog.csdn.net/20161010001715961)

>如果把这个矩阵公式带入R1 = AC，那么根据矩阵的乘法运算法则，可以得到R1 = R。因此，这个矩阵通常是用来作为初始的颜色矩阵来使用，他不会对原有颜色进行任何变化

>那么当我们要变换颜色值的时候，通常有两种方法。一个是直接改变颜色的offset，即偏移量的值来修改颜色的分量。另一种方法直接改变对应RGBA值的系数来调整颜色分量的值

>从前面的分析中，可以知道要修改R1的值，只要将第五列的值进行修改即可。即改变颜色的偏移量，其它值保存初始矩阵的值，如图：

![](http://img.blog.csdn.net/20161010001658757)

>在上面这个矩阵中，我们修改了R、G所对应的颜色偏移量，那么最后的处理结果就是图像的红色、绿色分别增加了100。而我们知道，红色混合绿色会得到黄色，所以最终的颜色处理结果就是让整个图片的色调偏黄色

> 如果修改颜色分量中的某个系数值，而其他只依然保存初始矩阵的值，如图：

![](http://img.blog.csdn.net/20161010002001697)

>在上面这个矩阵中，改变了G分量所对应的系数g，这样在矩形运算后G分量会变成以前的两倍，最终效果就是图像的色调更加偏绿

>下面通过实例看看如何通过矩阵改变图像的色调、饱和度和亮度：

>* 色调：setRotate(int axis, float degree)，第一个参数分别使用0、1、2代表Red、Green、Blue三种颜色，第二参数需要处理的值

```
ColorMatrix hueMatrix = new ColorMatrix();
hueMatrix.setRotate(0, hue);
hueMatrix.setRotate(1, hue);
hueMatrix.setRotate(2, hue);
```

>* 饱和度：setSaturation(float sat)，参数代表设置饱和度的值

```
ColorMatrix saturationMatrix = new ColorMatrix();
saturationMatrix.setSaturation(saturation);
```

>* 亮度：setScale(float rscale,float gscale,float bscale,float ascale)，参数分别是红、绿、蓝、透明度的亮度大小值

```
ColorMatrix lumMatrix = new ColorMatrix();
lumMatrix.setScale(lum, lum, lum, 1);
```

>除了单独使用上面三种方式来进行颜色效果处理之外，还提供了postConcat()方法来将矩阵的作用效果混合，从而叠加处理效果

```
ColorMatrix imageMatrix = new ColorMatrix();
imageMatrix.postConcat(hueMatrix);
imageMatrix.postConcat(saturationMatrix);
imageMatrix.postConcat(lumMatrix);
```

>通过SeekBar调节色调、饱和度、亮度......见经典代码回顾案例二

###6.5.2 Android颜色矩阵——ColorMatrix

> 模拟4x5的颜色矩阵......见经典代码回顾案例三

###6.5.3 常用图像颜色矩阵处理效果

>*  灰色效果

![](http://img.blog.csdn.net/20161010132515579)

>*  图像反转

![](http://img.blog.csdn.net/20161010132551424)

>*  怀旧效果

![](http://img.blog.csdn.net/20161010132602674)

>*  去色效果

![](http://img.blog.csdn.net/20161010132612174)

>*  高饱和度

![](http://img.blog.csdn.net/20161010132622706)

###6.5.4 像素点分析
>在Android中，系统系统提供了Bitmap.getPixels()方法来帮我们提取整个Bitmap中的像素点，并保存在一个数组中：

```
bitmap.getPixels(pixels, offset, stride, x, y, width, height);
```
>这几个参数的具体含义如下：

>* pixels：接收位图颜色值的数组
>* offset：写入到pixels[]中的第一个像素索引值
>* stride：pixels[]中的行间距
>* x：从位图中读取的第一个像素的ｘ坐标值
>* y：从位图中读取的第一个像素的y坐标值
>* width：从每一行中读取的像素宽度
>* height：读取的行数

>通常使用如下代码：

```
bitmap.getPixels(oldPx, 0, bitmap.getWidth(), 0, 0, width, height);
```

>接下来获取每个像素具体的ARGB值：

```
color = oldPx[i];
r = Color.red(color);
g = Color.green(color);
b = Color.blue(color);
a = Color.alpha(color);
```
>接下来就是修改像素值，产生新的像素值

```
r1 = (int) (0.393 * r + 0.769 * g + 0.189 * b);
g1 = (int) (0.349 * r + 0.686 * g + 0.168 * b);
b1 = (int) (0.272 * r + 0.534 * g + 0.131 * b);

newPx[i] = Color.argb(a, r1, b1, g1);
```
>最后使用我们的新像素值

```
bmp.setPixels(newPx, 0, width, 0, 0, width, height);
```

###6.5.5 常用图像像素点处理效果

>底片效果、老照片效果、浮雕效果......见经典代码回顾案例四

###**6.6 Android图像处理之图形特效处理**
>无知识点

###6.6.1 Android变形矩阵——Matrix
>对于图形变换，系统提供了3x3的举证来处理：

![](http://img.blog.csdn.net/20161010135125477)

>与颜色矩阵一样，计算方法通过矩阵乘法：

```
X1 = a x X +b x Y +c;
Y1 = d x X +e x Y +f;
1 = g x X +h x Y + i;
```

>与颜色矩阵一样，也有一个初始矩阵：

![](http://img.blog.csdn.net/20161010135142727)

>图像的变形处理包含以下四类基本变换：

>* Translate：平移变换
>* Rotate：旋转变换
>* Scale：缩放变换
>* Skew：错切变换

![](http://img.blog.csdn.net/20161010135410932)

>平移变换：即对每个像素点都进行平移变换，通过计算可以发现如下平移公式：

```
X = X0 + △X;
Y = Y0 + △Y;
```

![](http://img.blog.csdn.net/20161010135650233)

>旋转变换：通过以下三步骤完成以任意点为旋转中心的旋转变换

>* 将坐标原点平移到O点
>* 使用前面讲的以坐标原点为中心的旋转方法进行旋转变换
>* 将坐标原点还原

![](http://img.blog.csdn.net/20161010135954652)

>缩放变换：缩放变换的效果计算公式如下

```
x = K1 X x0;
y = K2 X y0;
```

![](http://img.blog.csdn.net/20161010184756796)

>错切变换：错切变换的效果计算公式如下

```
x = x0 + k1 + y0
y = k2 x x0 + y0
```

>了解四种图形变换矩阵，可以通过setValues()方法将一个一维数组转换为图形变换矩阵：

```
private float [] mImageMatrix = new float[9];
Matrix matrix = new Matrix();
matrix.setValues(mImageMatrix);
canvas.drawBitmap(mBitmmap,matrix,null);
```

>Android中Matrix类也帮我们封装好了几个操作方法：

>* matrix.setRotate()：旋转变换
>* matrix.setTranslate()：平移变换
>* matrix.setScale()：缩放变换
>* matrix.setSkew()：错切变换
>* pre()和post()：提供矩阵的前乘和后乘运算

>举个例子说明前乘和后乘的不同运算方式：

>* 先平移到（300， 100）
>* 再旋转45度
>* 最后平移到（200, 200）

>如果使用后乘运算，代码如下：

```
matrix.setRotate(45);
matrix.postTranslate(200, 200);
```
>如果使用前乘运算，代码如下：

```
matrix.setTranslate(200, 200);
matrix.perRotate(45);
```

###6.6.2 像素块分析
>drawBitmapMesh()与操纵像素点来改变色彩的原理类似，只不过是把图像分成了一个个的小块，然后通过改变每一个图像块来修改整个图像：

```
canvas.drawBitmapMesh(Bitmap bitmap,int meshWidth,int meshHeight,float [] verts,
	int vertOffset,int [] colors,int colorOffset,Paint paint);
```
>参数分析：

>* bitmap：将要扭曲的图像
>* meshWidth：需要的横向网格数目
>* meshHeight：需要的纵向网格数目
>* verts：网格交叉点的坐标数组
>* vertOffset：verts数组中开始跳过的(X,Y)坐标对的数目

>飘动的旗子......见经典代码回顾案例五

###**6.7 Android图像处理之画笔特效处理**
>之前绘图的时候也提到过画笔的一些方法，这里就不再介绍
###6.7.1 PorterDuffXfermode
>PorterDuffXfermod设置的是两个图层交集区域的显示方式，dst是先画的图形，而src是后画的图形

![](http://img.blog.csdn.net/20161010191950044)

>以一个圆角图片为例子：

```
mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test1);
mOut = Bitmap.createBitmap(mBitmap.getWidth(), mBitmap.getHeight(), Bitmap.Config.ARGB_8888);
Canvas canvas = new Canvas(mOut);
mPaint = new Paint();
mPaint.setAntiAlias(true);
canvas.drawRoundRect(0, 0, mBitmap.getWidth(), mBitmap.getHeight(), 80, 80, mPaint);
mPaint.setXfermode(new PorterDuffXfermode(PorterDuff.Mode.SRC_IN));
canvas.drawBitmap(mBitmap,0,0,mPaint);
```

>效果图，由于图片过大，只能看出一边角

![](http://img.blog.csdn.net/20161010193506235)

>刮刮卡效果......见经典代码回顾案例六

###6.7.2 Shader
>Shader又被称为着色器。渲染器，它可以实现渲染，渐变等效果，Android中的Shader包括以下几种：
>
>* BitmapShader：位图Shader
>* LinearGradient：线性Shader
>* RadialGradient：光束Shader
>* SweepGradient：梯形Shader
>* ComposeShader：混合Shader

>其中BitmapShader有三种模式可以选择：

>* CLAMP拉伸：拉伸的是图片最后的那一个像素，不断重复
>* REPEAT重复：横向，纵向不断重复
>* MIRROR镜像：横向不断翻转重复，纵向不断翻转重复

>下面看下例子说明，圆形图片：

```
mBitmap = BitmapFactory.decodeResource(getResources(),R.drawable.nice);
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.CLAMP,Shader.TileMode.CLAMP);
mPaint = new Paint();
mPaint.setShader(mBitmapShader);
canvas.drawCircle(500,250,200,mPaint);
```

>效果图

![](http://img.blog.csdn.net/20161010200702628)

>下面把TileMode改为REPEAT：

```
mBitmap = BitmapFactory.decodeResource(getResources(),R.mipmap.ic_launcher);
mBitmapShader = new BitmapShader(mBitmap, Shader.TileMode.REPEAT,Shader.TileMode.REPEAT);
mPaint = new Paint();
mPaint.setShader(mBitmapShader);
canvas.drawCircle(500,250,200,mPaint);
```

![](http://img.blog.csdn.net/20161010200850966)

>使用LinearGradient：

```
mPaint = new Paint();
mPaint.setShader(new LinearGradient(0,0,400,400, Color.BLUE,Color.YELLOW, Shader.TileMode.REPEAT));
canvas.drawRect(0,0,400,400,mPaint);
```

>效果图

![](http://img.blog.csdn.net/20161010201003708)

>结合前面的PorterDuffXfermode和刚学的LinearGradient，制作出倒影的效果图片

>倒影图片效果......见经典代码回顾案例七

###6.7.3 PathEffect

>先上一张直观的图：

![](http://img.blog.csdn.net/20161010232528339)

>Android提供的几种绘制PathEffect方式：

>* 没效果
>* CornerPathEffect：拐弯角变得圆滑
>* DiscretePathEffect：线段上会产生许多杂点
>* DashPathEffect：绘制虚线，用一个数据来设置各个点之间的间隔
>* PathDashPathEffect：绘制虚线，可以使用方形点虚线和圆形点虚线
>* ComposePathEffect：可任意组合两种路径（PathEffect）的特性

>我们通过一个实例来认识这些效果：

```
public class PathEffectView extends View{

    private Path mPath;
    private PathEffect [] mEffect = new PathEffect[6];
    private Paint mPaint;

    /**
     * 构造方法
     * @param context
     * @param attrs
     */
    public PathEffectView(Context context, AttributeSet attrs) {
        super(context, attrs);

        init();
    }

    /**
     * 初始化
     */
    private void init() {
        mPaint = new Paint();
        mPath = new Path();
        mPath.moveTo(0,0);
        for (int i = 0; i<= 30;i++){
            mPath.lineTo(i*35,(float)(Math.random()*100));
        }
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        mEffect[0] = null;
        mEffect[1] = new CornerPathEffect(30);
        mEffect[2] = new DiscretePathEffect(3.0F,5.0F);
        mEffect[3] = new DashPathEffect(new float[]{20,10,5,10},0);
        Path path = new Path();
        path.addRect(0,0,8,8,Path.Direction.CCW);
        mEffect[4]= new PathDashPathEffect(path,12,0,PathDashPathEffect.Style.ROTATE);
        mEffect[5] = new ComposePathEffect(mEffect[3],mEffect[1]);
        for (int i = 0; i<mEffect.length;i++){
            mPaint.setPathEffect(mEffect[i]);
            canvas.drawPath(mPath,mPaint);
            canvas.translate(0,200);
        }
    }
}
```
>每绘制一个Path，就将画布平移，从而让各种PathEffect依次绘制出来

###效果图
![](http://img.blog.csdn.net/20161010234134439)

###6.8 View之孪生兄弟——SurfaceView
>无知识点

###6.8.1 SurfaceView与View的区别
>View的绘制刷新间隔时间为16ms，如果在16ms内完成你所需要执行的所有操作，那么在用户视觉上，就不会产生卡顿的感觉，否则，就会出现卡顿，所以可以考虑使用SurfaceView来替代View的绘制

>通常在Log会看到这样的提示：

```
Skipped 47 frames! The application may be doing too much work on its main thread
```

>SurfaceView与View的主要区别：

>* View主要适用于主动更新的情况下，而surfaceVicw主要适用于被动更新，例如频繁刷新
>* View在主线程中对画面进行刷新，而surfaceView通常会通过一 个子线程来进行页面的刷新
>* View在绘制时没有使用双缓冲机制，而surfaceVicw在底层实现机制中就已经实现了双缓冲机制

>总结一句话就是，如果你的自定义View需要频繁刷新，或者刷新数据处理量比较大，就可以考虑使用SurfaceView替代View

###6.8.2 SurfaceView的使用
>SurfaceView使用步骤：

>* 创建SurfaceView继承自SurfaceView，并实现两个接口——SurfaceHolder.Callback和Runnable
>* 初始化SurfacHolder对象，并注册SurfaceHolder的回调方法
>* 通过SurfacHolder对象lockCanvas()方法获得Canvas对象进行绘制，并通过unlockCanvasAndPost(mCanvas)方法对画布内容进行提交

>整个使用SurfaceView的模板代码：

```
public class SurfaView extends SurfaceView implements SurfaceHolder.Callback, Runnable {

    //SurfaceHolder
    private SurfaceHolder mHolder;
    //用于绘制的Canvas
    private Canvas mCanvas;
    //子线程标志位
    private boolean mIsDrawing;

    /**
     * 构造方法
     *
     * @param context
     * @param attrs
     */
    public SurfaView(Context context, AttributeSet attrs) {
        super(context, attrs);

        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        setKeepScreenOn(true);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {

    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
        } catch (Exception e) {

        } finally {
            if (mCanvas != null) {
                //提交
                mHolder.unlockCanvasAndPost(mCanvas);
            }
        }
    }
}
```
>唯一注意的是，在绘制中将mHolder.unlockCanvasAndPost(mCanvas)方法放到finally代码块中，保证每次都能将内容提交

###6.8.3 SurfaceView实例
>正弦曲线......见经典代码回顾案例八
>绘图板......见经典代码回顾案例九

---------

##经典代码回顾##

##案例一：仪表盘

```
public class Clock extends View {

    private int mHeight, mWidth;

    public Clock(Context context) {
        super(context);
    }

    public Clock(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public Clock(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        // 获取宽高参数
        mWidth = getMeasuredWidth();
        mHeight = getMeasuredHeight();
        // 画外圆
        Paint paintCircle = new Paint();
        paintCircle.setStyle(Paint.Style.STROKE);
        paintCircle.setAntiAlias(true);
        paintCircle.setStrokeWidth(5);
        canvas.drawCircle(mWidth / 2,
                mHeight / 2, mWidth / 2, paintCircle);
        // 画刻度
        Paint painDegree = new Paint();
        paintCircle.setStrokeWidth(3);
        for (int i = 0; i < 24; i++) {
            // 区分整点与非整点
            if (i == 0 || i == 6 || i == 12 || i == 18) {
                painDegree.setStrokeWidth(5);
                painDegree.setTextSize(30);
                canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
                        mWidth / 2, mHeight / 2 - mWidth / 2 + 60,
                        painDegree);
                String degree = String.valueOf(i);
                canvas.drawText(degree,
                        mWidth / 2 - painDegree.measureText(degree) / 2,
                        mHeight / 2 - mWidth / 2 + 90,
                        painDegree);
            } else {
                painDegree.setStrokeWidth(3);
                painDegree.setTextSize(15);
                canvas.drawLine(mWidth / 2, mHeight / 2 - mWidth / 2,
                        mWidth / 2, mHeight / 2 - mWidth / 2 + 30,
                        painDegree);
                String degree = String.valueOf(i);
                canvas.drawText(degree,
                        mWidth / 2 - painDegree.measureText(degree) / 2,
                        mHeight / 2 - mWidth / 2 + 60,
                        painDegree);
            }
            // 通过旋转画布简化坐标运算
            canvas.rotate(15, mWidth / 2, mHeight / 2);
        }
        // 画圆心
        Paint paintPointer = new Paint();
        paintPointer.setStrokeWidth(30);
        canvas.drawPoint(mWidth / 2, mHeight / 2, paintPointer);
        // 画指针
        Paint paintHour = new Paint();
        paintHour.setStrokeWidth(20);
        Paint paintMinute = new Paint();
        paintMinute.setStrokeWidth(10);
        canvas.save();
        canvas.translate(mWidth / 2, mHeight / 2);
        canvas.drawLine(0, 0, 100, 100, paintHour);
        canvas.drawLine(0, 0, 100, 200, paintMinute);
        canvas.restore();
    }
}
```

##效果图
![](http://img.blog.csdn.net/20161009234351603)


----------


##案例二：通过SeekBar调节色调、饱和度、亮度

>布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/imageview"
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp"
        android:layout_marginTop="24dp" />

    <SeekBar
        android:id="@+id/seekbarHue"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/imageview" />

    <SeekBar
        android:id="@+id/seekbarSaturation"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/seekbarHue" />

    <SeekBar
        android:id="@+id/seekbatLum"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/seekbarSaturation" />

</RelativeLayout>
```
>Activity文件

```
public class PrimaryColor extends Activity implements SeekBar.OnSeekBarChangeListener {

    private static int MAX_VALUE = 255;
    private static int MID_VALUE = 127;
    private ImageView mImageView;
    private SeekBar mSeekbarhue, mSeekbarSaturation, mSeekbarLum;
    private float mHue, mStauration, mLum;
    private Bitmap bitmap;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.primary_color);
        bitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test3);
        mImageView = (ImageView) findViewById(R.id.imageview);
        mSeekbarhue = (SeekBar) findViewById(R.id.seekbarHue);
        mSeekbarSaturation = (SeekBar) findViewById(R.id.seekbarSaturation);
        mSeekbarLum = (SeekBar) findViewById(R.id.seekbatLum);
        mSeekbarhue.setOnSeekBarChangeListener(this);
        mSeekbarSaturation.setOnSeekBarChangeListener(this);
        mSeekbarLum.setOnSeekBarChangeListener(this);
        mSeekbarhue.setMax(MAX_VALUE);
        mSeekbarSaturation.setMax(MAX_VALUE);
        mSeekbarLum.setMax(MAX_VALUE);
        mSeekbarhue.setProgress(MID_VALUE);
        mSeekbarSaturation.setProgress(MID_VALUE);
        mSeekbarLum.setProgress(MID_VALUE);
        mImageView.setImageBitmap(bitmap);
    }

    @Override
    public void onProgressChanged(SeekBar seekBar,
                                  int progress, boolean fromUser) {
        switch (seekBar.getId()) {
            case R.id.seekbarHue:
                mHue = (progress - MID_VALUE) * 1.0F / MID_VALUE * 180;
                break;
            case R.id.seekbarSaturation:
                mStauration = progress * 1.0F / MID_VALUE;
                break;
            case R.id.seekbatLum:
                mLum = progress * 1.0F / MID_VALUE;
                break;
        }
        mImageView.setImageBitmap(ImageHelper.handleImageEffect(
                bitmap, mHue, mStauration, mLum));
    }

    @Override
    public void onStartTrackingTouch(SeekBar seekBar) {
    }

    @Override
    public void onStopTrackingTouch(SeekBar seekBar) {
    }
}

```

```
public class ImageHelper {

    public static Bitmap handleImageEffect(Bitmap bm,
                                           float hue,
                                           float saturation,
                                           float lum) {
        Bitmap bmp = Bitmap.createBitmap(
                bm.getWidth(), bm.getHeight(), Bitmap.Config.ARGB_8888);
        Canvas canvas = new Canvas(bmp);
        Paint paint = new Paint();

        ColorMatrix hueMatrix = new ColorMatrix();
        hueMatrix.setRotate(0, hue);
        hueMatrix.setRotate(1, hue);
        hueMatrix.setRotate(2, hue);

        ColorMatrix saturationMatrix = new ColorMatrix();
        saturationMatrix.setSaturation(saturation);

        ColorMatrix lumMatrix = new ColorMatrix();
        lumMatrix.setScale(lum, lum, lum, 1);

        ColorMatrix imageMatrix = new ColorMatrix();
        imageMatrix.postConcat(hueMatrix);
        imageMatrix.postConcat(saturationMatrix);
        imageMatrix.postConcat(lumMatrix);

        paint.setColorFilter(new ColorMatrixColorFilter(imageMatrix));
        canvas.drawBitmap(bm, 0, 0, paint);
        return bmp;
    }

}
```

##效果图

![](http://img.blog.csdn.net/20161010130422424)


----------


##案例三：模拟4x5的颜色矩阵

>布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/imageview"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="2" />

    <GridLayout
        android:id="@+id/group"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="3"
        android:columnCount="5"
        android:rowCount="4">

    </GridLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal">

        <Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:onClick="btnChange"
            android:text="Change" />

        <Button
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:onClick="btnReset"
            android:text="Reset" />
    </LinearLayout>

</LinearLayout>
```

>Activity文件

```
public class ColorMatrix extends Activity {

    private ImageView mImageView;
    private GridLayout mGroup;
    private Bitmap bitmap;
    private int mEtWidth, mEtHeight;
    private EditText[] mEts = new EditText[20];
    private float[] mColorMatrix = new float[20];

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.color_matrix);
        bitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test1);
        mImageView = (ImageView) findViewById(R.id.imageview);
        mGroup = (GridLayout) findViewById(R.id.group);
        mImageView.setImageBitmap(bitmap);

        mGroup.post(new Runnable() {
            @Override
            public void run() {
                // 获取宽高信息
                mEtWidth = mGroup.getWidth() / 5;
                mEtHeight = mGroup.getHeight() / 4;
                addEts();
                initMatrix();
            }
        });
    }

    // 获取矩阵值
    private void getMatrix() {
        for (int i = 0; i < 20; i++) {
            mColorMatrix[i] = Float.valueOf(
                    mEts[i].getText().toString());
        }
    }

    // 将矩阵值设置到图像
    private void setImageMatrix() {
        Bitmap bmp = Bitmap.createBitmap(
                bitmap.getWidth(),
                bitmap.getHeight(),
                Bitmap.Config.ARGB_8888);
        android.graphics.ColorMatrix colorMatrix =
                new android.graphics.ColorMatrix();
        colorMatrix.set(mColorMatrix);

        Canvas canvas = new Canvas(bmp);
        Paint paint = new Paint();
        paint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
        canvas.drawBitmap(bitmap, 0, 0, paint);
        mImageView.setImageBitmap(bmp);
    }

    // 作用矩阵效果
    public void btnChange(View view) {
        getMatrix();
        setImageMatrix();
    }

    // 重置矩阵效果
    public void btnReset(View view) {
        initMatrix();
        getMatrix();
        setImageMatrix();
    }

    // 添加EditText
    private void addEts() {
        for (int i = 0; i < 20; i++) {
            EditText editText = new EditText(ColorMatrix.this);
            mEts[i] = editText;
            mGroup.addView(editText, mEtWidth, mEtHeight);
        }
    }

    // 初始化颜色矩阵为初始状态
    private void initMatrix() {
        for (int i = 0; i < 20; i++) {
            if (i % 6 == 0) {
                mEts[i].setText(String.valueOf(1));
            } else {
                mEts[i].setText(String.valueOf(0));
            }
        }
    }
}
```
>关键点：将一个颜色矩阵传入画笔，然后画出原始的图在新建的图上面

```
paint.setColorFilter(new ColorMatrixColorFilter(colorMatrix));
```

##效果图
![](http://img.blog.csdn.net/20161010131715538)


----------


##案例四：底片效果、老照片效果、浮雕效果
>布局文件

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <ImageView
            android:id="@+id/imageview1"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />

        <ImageView
            android:id="@+id/imageview2"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />
    </LinearLayout>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <ImageView
            android:id="@+id/imageview3"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />

        <ImageView
            android:id="@+id/imageview4"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />
    </LinearLayout>

</LinearLayout>
```
>Activity文件

```
public class PixelsEffect extends Activity {

    private ImageView imageView1, imageView2, imageView3, imageView4;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.pixels_effect);
        Bitmap bitmap = BitmapFactory.decodeResource(getResources(), R.drawable.test2);
        imageView1 = (ImageView) findViewById(R.id.imageview1);
        imageView2 = (ImageView) findViewById(R.id.imageview2);
        imageView3 = (ImageView) findViewById(R.id.imageview3);
        imageView4 = (ImageView) findViewById(R.id.imageview4);

        imageView1.setImageBitmap(bitmap);
        imageView2.setImageBitmap(ImageHelper.handleImageNegative(bitmap));
        imageView3.setImageBitmap(ImageHelper.handleImagePixelsOldPhoto(bitmap));
        imageView4.setImageBitmap(ImageHelper.handleImagePixelsRelief(bitmap));
    }
}
```
>工具类

```
public class ImageHelper {

    public static Bitmap handleImageNegative(Bitmap bm) {
        int width = bm.getWidth();
        int height = bm.getHeight();
        int color;
        int r, g, b, a;

        Bitmap bmp = Bitmap.createBitmap(width, height
                , Bitmap.Config.ARGB_8888);

        int[] oldPx = new int[width * height];
        int[] newPx = new int[width * height];
        bm.getPixels(oldPx, 0, width, 0, 0, width, height);

        for (int i = 0; i < width * height; i++) {
            color = oldPx[i];
            r = Color.red(color);
            g = Color.green(color);
            b = Color.blue(color);
            a = Color.alpha(color);

            r = 255 - r;
            g = 255 - g;
            b = 255 - b;

            if (r > 255) {
                r = 255;
            } else if (r < 0) {
                r = 0;
            }
            if (g > 255) {
                g = 255;
            } else if (g < 0) {
                g = 0;
            }
            if (b > 255) {
                b = 255;
            } else if (b < 0) {
                b = 0;
            }
            newPx[i] = Color.argb(a, r, g, b);
        }
        bmp.setPixels(newPx, 0, width, 0, 0, width, height);
        return bmp;
    }

    public static Bitmap handleImagePixelsOldPhoto(Bitmap bm) {
        Bitmap bmp = Bitmap.createBitmap(bm.getWidth(), bm.getHeight(),
                Bitmap.Config.ARGB_8888);
        int width = bm.getWidth();
        int height = bm.getHeight();
        int color = 0;
        int r, g, b, a, r1, g1, b1;

        int[] oldPx = new int[width * height];
        int[] newPx = new int[width * height];

        bm.getPixels(oldPx, 0, bm.getWidth(), 0, 0, width, height);
        for (int i = 0; i < width * height; i++) {
            color = oldPx[i];
            a = Color.alpha(color);
            r = Color.red(color);
            g = Color.green(color);
            b = Color.blue(color);

            r1 = (int) (0.393 * r + 0.769 * g + 0.189 * b);
            g1 = (int) (0.349 * r + 0.686 * g + 0.168 * b);
            b1 = (int) (0.272 * r + 0.534 * g + 0.131 * b);

            if (r1 > 255) {
                r1 = 255;
            }
            if (g1 > 255) {
                g1 = 255;
            }
            if (b1 > 255) {
                b1 = 255;
            }

            newPx[i] = Color.argb(a, r1, g1, b1);
        }
        bmp.setPixels(newPx, 0, width, 0, 0, width, height);
        return bmp;
    }

    public static Bitmap handleImagePixelsRelief(Bitmap bm) {
        Bitmap bmp = Bitmap.createBitmap(bm.getWidth(), bm.getHeight(),
                Bitmap.Config.ARGB_8888);
        int width = bm.getWidth();
        int height = bm.getHeight();
        int color = 0, colorBefore = 0;
        int a, r, g, b;
        int r1, g1, b1;

        int[] oldPx = new int[width * height];
        int[] newPx = new int[width * height];

        bm.getPixels(oldPx, 0, bm.getWidth(), 0, 0, width, height);
        for (int i = 1; i < width * height; i++) {
            colorBefore = oldPx[i - 1];
            a = Color.alpha(colorBefore);
            r = Color.red(colorBefore);
            g = Color.green(colorBefore);
            b = Color.blue(colorBefore);

            color = oldPx[i];
            r1 = Color.red(color);
            g1 = Color.green(color);
            b1 = Color.blue(color);

            r = (r - r1 + 127);
            g = (g - g1 + 127);
            b = (b - b1 + 127);
            if (r > 255) {
                r = 255;
            }
            if (g > 255) {
                g = 255;
            }
            if (b > 255) {
                b = 255;
            }
            newPx[i] = Color.argb(a, r, g, b);
        }
        bmp.setPixels(newPx, 0, width, 0, 0, width, height);
        return bmp;
    }
}
```

##效果图
![](http://img.blog.csdn.net/20161010134240692)


----------


##案例五：飘动的旗子

>本人也是懵逼，没有搞懂这个例子

```
public class FlagBitmapMeshView extends View {

    private final int WIDTH = 200;
    private final int HEIGHT = 200;
    private int COUNT = (WIDTH + 1) * (HEIGHT + 1);
    private float[] verts = new float[COUNT * 2];
    private float[] orig = new float[COUNT * 2];
    private Bitmap bitmap;
    private float A;
    private float k = 1;

    public FlagBitmapMeshView(Context context) {
        super(context);
        initView(context);
    }

    public FlagBitmapMeshView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView(context);
    }

    public FlagBitmapMeshView(Context context, AttributeSet attrs,
                              int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initView(context);
    }

    private void initView(Context context) {
        setFocusable(true);
        bitmap = BitmapFactory.decodeResource(context.getResources(),
                R.drawable.test);
        float bitmapWidth = bitmap.getWidth();
        float bitmapHeight = bitmap.getHeight();
        int index = 0;
        for (int y = 0; y <= HEIGHT; y++) {
            float fy = bitmapHeight * y / HEIGHT;
            for (int x = 0; x <= WIDTH; x++) {
                float fx = bitmapWidth * x / WIDTH;
                orig[index * 2 + 0] = verts[index * 2 + 0] = fx;
                orig[index * 2 + 1] = verts[index * 2 + 1] = fy + 100;
                index += 1;
            }
        }
        A = 50;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        flagWave();
        k += 0.1F;
        canvas.drawBitmapMesh(bitmap, WIDTH, HEIGHT,
                verts, 0, null, 0, null);
        invalidate();
    }

    private void flagWave() {
        for (int j = 0; j <= HEIGHT; j++) {
            for (int i = 0; i <= WIDTH; i++) {
                verts[(j * (WIDTH + 1) + i) * 2 + 0] += 0;
                float offsetY =
                        (float) Math.sin((float) i / WIDTH * 2 * Math.PI +
                                Math.PI * k);
                verts[(j * (WIDTH + 1) + i) * 2 + 1] =
                        orig[(j * WIDTH + i) * 2 + 1] + offsetY * A;
            }
        }
    }
}
```

###效果图
![](http://img.blog.csdn.net/20161010192909914)


----------


##案例六：刮刮卡效果

```
public class XfermodeView extends View {

    private Bitmap mBgBitmap, mFgBitmap;
    private Paint mPaint;
    private Canvas mCanvas;
    private Path mPath;

    public XfermodeView(Context context) {
        super(context);
        init();
    }

    public XfermodeView(Context context, AttributeSet attrs) {
        super(context, attrs);
        init();
    }

    public XfermodeView(Context context, AttributeSet attrs,
                        int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        init();
    }

    private void init() {
        mPaint = new Paint();
        mPaint.setAlpha(0);
        mPaint.setXfermode(
                new PorterDuffXfermode(PorterDuff.Mode.DST_IN));
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
        mPaint.setStrokeWidth(50);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        mPath = new Path();
        mBgBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test);
        mFgBitmap = Bitmap.createBitmap(mBgBitmap.getWidth(),
                mBgBitmap.getHeight(), Bitmap.Config.ARGB_8888);
        mCanvas = new Canvas(mFgBitmap);
        mCanvas.drawColor(Color.GRAY);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPath.reset();
                mPath.moveTo(event.getX(), event.getY());
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(event.getX(), event.getY());
                break;
        }
        mCanvas.drawPath(mPath, mPaint);
        invalidate();
        return true;
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawBitmap(mBgBitmap, 0, 0, null);
        canvas.drawBitmap(mFgBitmap, 0, 0, null);
    }
}
```

###效果图

![](http://img.blog.csdn.net/20161010195617684)


----------


##案例七：倒影图片效果

```
public class ReflectView extends View {
    private Bitmap mSrcBitmap, mRefBitmap;
    private Paint mPaint;
    private PorterDuffXfermode mXfermode;

    public ReflectView(Context context) {
        super(context);
        initRes(context);
    }

    public ReflectView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initRes(context);
    }

    public ReflectView(Context context, AttributeSet attrs,
                       int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initRes(context);
    }

    private void initRes(Context context) {
        mSrcBitmap = BitmapFactory.decodeResource(getResources(),
                R.drawable.test);
        Matrix matrix = new Matrix();
        matrix.setScale(1F, -1F);
        mRefBitmap = Bitmap.createBitmap(mSrcBitmap, 0, 0,
                mSrcBitmap.getWidth(), mSrcBitmap.getHeight(), matrix, true);

        mPaint = new Paint();
        mPaint.setShader(new LinearGradient(0, mSrcBitmap.getHeight(), 0,
                mSrcBitmap.getHeight() + mSrcBitmap.getHeight() / 4,
                0XDD000000, 0X10000000, Shader.TileMode.CLAMP));
        mXfermode = new PorterDuffXfermode(PorterDuff.Mode.DST_IN);
    }

    @Override
    protected void onDraw(Canvas canvas) {
        canvas.drawColor(Color.BLACK);
        canvas.drawBitmap(mSrcBitmap, 0, 0, null);
        canvas.drawBitmap(mRefBitmap, 0, mSrcBitmap.getHeight(), null);
        mPaint.setXfermode(mXfermode);
        // 绘制渐变效果矩形
        canvas.drawRect(0, mSrcBitmap.getHeight(),
                mRefBitmap.getWidth(), mSrcBitmap.getHeight() * 2, mPaint);
        mPaint.setXfermode(null);
    }
}
```

###效果图
![](http://img.blog.csdn.net/20161010201738414)

##案例八：正弦曲线

```
public class SinView extends SurfaceView
        implements SurfaceHolder.Callback, Runnable {

    private SurfaceHolder mHolder;
    private Canvas mCanvas;
    private boolean mIsDrawing;
    private int x = 0;
    private int y = 0;
    private Path mPath;
    private Paint mPaint;

    public SinView(Context context) {
        super(context);
        initView();
    }

    public SinView(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public SinView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        initView();
    }

    private void initView() {
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);
        mPath = new Path();
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(10);
        mPaint.setStrokeCap(Paint.Cap.ROUND);
        mPaint.setStrokeJoin(Paint.Join.ROUND);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        mPath.moveTo(0, 400);
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder,
                               int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        while (mIsDrawing) {
            draw();
            x += 1;
            y = (int) (100*Math.sin(x * 2 * Math.PI / 180) + 400);
            mPath.lineTo(x, y);
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            // SurfaceView背景
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {
        } finally {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }
}
```
###效果图
![](http://img.blog.csdn.net/20161011002648523)

##案例九：绘图板

```
public class SimpleDraw extends SurfaceView
        implements SurfaceHolder.Callback, Runnable {

    private SurfaceHolder mHolder;
    private Canvas mCanvas;
    private boolean mIsDrawing;
    private Path mPath;
    private Paint mPaint;

    public SimpleDraw(Context context) {
        super(context);
        initView();
    }

    public SimpleDraw(Context context, AttributeSet attrs) {
        super(context, attrs);
        initView();
    }

    public SimpleDraw(Context context, AttributeSet attrs,
                      int defStyle) {
        super(context, attrs, defStyle);
        initView();
    }

    private void initView() {
        mHolder = getHolder();
        mHolder.addCallback(this);
        setFocusable(true);
        setFocusableInTouchMode(true);
        this.setKeepScreenOn(true);
        mPath = new Path();
        mPaint = new Paint();
        mPaint.setColor(Color.RED);
        mPaint.setStyle(Paint.Style.STROKE);
        mPaint.setStrokeWidth(20);
    }

    @Override
    public void surfaceCreated(SurfaceHolder holder) {
        mIsDrawing = true;
        new Thread(this).start();
    }

    @Override
    public void surfaceChanged(SurfaceHolder holder,
                               int format, int width, int height) {
    }

    @Override
    public void surfaceDestroyed(SurfaceHolder holder) {
        mIsDrawing = false;
    }

    @Override
    public void run() {
        long start = System.currentTimeMillis();
        while (mIsDrawing) {
            draw();
        }
        long end = System.currentTimeMillis();
        // 50 - 100
        if (end - start < 100) {
            try {
                Thread.sleep(100 - (end - start));
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    private void draw() {
        try {
            mCanvas = mHolder.lockCanvas();
            mCanvas.drawColor(Color.WHITE);
            mCanvas.drawPath(mPath, mPaint);
        } catch (Exception e) {
        } finally {
            if (mCanvas != null)
                mHolder.unlockCanvasAndPost(mCanvas);
        }
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        int y = (int) event.getY();
        switch (event.getAction()) {
            case MotionEvent.ACTION_DOWN:
                mPath.moveTo(x, y);
                break;
            case MotionEvent.ACTION_MOVE:
                mPath.lineTo(x, y);
                break;
            case MotionEvent.ACTION_UP:
                break;
        }
        return true;
    }
}
```

###效果图
![](http://img.blog.csdn.net/20161011002954258)

>经典回顾源码下载

>github：https://github.com/CSDNHensen/QunYingZhuang