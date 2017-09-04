##**一、xcode常用快捷键**

1、常用操作

command+c：复制 
command+v：粘贴 
command+r：运行程序 
command+/：注释 

2、工具栏

command+0：左侧工具栏展示/隐藏
option+command+0：右侧工具栏展示/隐藏

3、文件操作

option+command+n：快速创建目录文件夹
shift+command+o：全局搜索头文件
command+control+上下箭头：切换头文件和.m文件 
command+control+左右箭头：退回和前进历史位置

4、编辑

option+左右箭头：跳过单词移动
control+e：跳到句头
control+a：跳到句尾
option+command+[：整行或多行上移
option+command+]：整行或多行下移
command+[：整行或多行左缩进
command+]：整行或多行右缩进

5、调试

F6单步调试、F7跳入、F8继续

##**二、控件入门**

1、main.storyboard介绍

main.storyboard中的ViewController就相当于一个界面，程序启动时，加载的是带有箭头指向的那个，你也可以拖拽箭头修改默认启动的界面，也可以通过is initial ViewController属性来设置

2、ViewController介绍

ViewController可以绑定一个UIViewController类型的类，需要设置customClass，ViewController类默认有两个方法

![](http://img.blog.csdn.net/20170225135549209?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

特点：

1. viewDidLoad：当视图控制器的根视图加载完成之后触发该方法
2. didReceiveMemoryWarning：当收到内存警告时触发

3、main.storyboard与ViewController控件的绑定

控件的使用必须在主文件中申明，如果是按钮等控件则必须在主文件中实现点击事件，可以在分屏模式下，在Main.storyboard使用control+鼠标左键拖拽控件到对应代码处，Xcode会人性化的生成灰色点表示

![](http://img.blog.csdn.net/20170223190806878?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、main.storyboard其他属性

在main.storyboard的ViewCtroller里面点击图中黄色按钮可以在右边属性栏设置开发屏幕的尺寸等属性

![](http://img.blog.csdn.net/20170224122239306?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当有程序运行时，还可以在开发界面的底部点击该按钮，可进入View的层次化视图

![](http://img.blog.csdn.net/20170224130115961?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以按住左键进行拖拽旋转

![](http://img.blog.csdn.net/20170224130138433?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**三、UIView**

UIView是所有UI控件的基类，它可以使用的属性，其他UI控件都可以使用

1、创建UIView

```
//初始化UIView
UIView *mView = [[UIView alloc] init];
//UIView的位置
mView.frame = CGRectMake(0, 0, 100, 100);
//UIView的背景色
mView.backgroundColor = [UIColor redColor];
//添加子view
[self.view addSubview：mView];
```

2、创建动画

```
//开启动画
[UIView beginAnimations：nil context：nil];
//设置动画延时
[UIView setAnimationDelay：2];
//设置动画时间
[UIView setAnimationDuration：5];
//修改mView的x值
CGRect newFlame = mView.frame;
newFlame.origin.x = 200;
mView.frame = newFlame;
//提交动画
[UIView commitAnimations];
```

3、创建动画第二种方式

```
//修改mView的x值
CGRect newFlame = mView.frame;
newFlame.origin.x = 200;
//动画设置
[UIView animateWithDuration：5 animations：^{
	mView.frame = newFlame;
}];
```

4、位置、长宽属性

```
mView.frame.origin.x;//View的x值
mView.frame.origin.y;//View的y值
mView.frame.size.width;//View的宽度
mView.frame.size.height;//View的高度
```

>可以借助修改它们的属性完成动画缩放、移动的动画效果

5、transform属性

可以利用transform属性来完成缩放、移动、旋转等组合效果

```
//1.只执行一次的动画，不会接着上次执行完成后的位置，而是回到起点重新执行动画
//旋转，参数中的角度和数字是不对等的，建议使用M_PI(180°)、M_PI_2、M_PI_4
mView.transform = CGAffineTransformMakeRotation(M_PI_2);
//缩放，在View的基础上放大2倍
mView.transform = CGAffineTransformMakeScale(2, 2);
//平移，在View的基础上再平移100，100
mView.transform = CGAffineTransformMakeTranslation(100, 100);
//2.执行一次可以接着原来的位置继续执行动画
mView.transform = CGAffineTransformRotate(mView.transform, M_PI_2);
mView.transform = CGAffineTransformScale(mView.transform, 2, 2);
mView.transform = CGAffineTransformTranslate(mView.transform, 100, 100);
```

6、添加与删除子View

删除View的时候，如果删除的是父View，那么它的子View全部也会被删除

```
//添加子View
[self.view addSubview：mView];
//subviews是所有子View的数组
UIView view = self.view.subviews[0];
//删除子View
[view removeFromSuperview];
//删除不包含UIButton的所有子View
for(UIView *view in self.view.subviews){
	if([view isKindOfClass：[UIButton class]]){
		continue;
	}
	[view removeFromSuperview]
}
```

7、获取指定Tag的控件

获取前，需要在控件上对Tag进行赋值，如果Tag跟自己(即父View)一样，优先获取自己

```
//获取Tag为1的控件
UIView *view = [self.view viewWithTag：1];
```

##**四、UITextField**

1、设置UITextField弹出的键盘模式

可以在文本框属性设置界面中选择keyboardtype进行设置

2、获取UITextField的文本内容

```
//获取UITextField的文本内容
int time = [self.textField.text intValue];
```

3、隐藏键盘

```
//关闭键盘的两种方式
//1.让输入框失去焦点
[self.textField resignFirstResponder];
//2.让view取消编辑状态
[self.view endEditing：YES];
```
4、点击文本框以外的地方收起键盘

```
-(void)touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event{
    [self.view endEditing：YES];
}
```

5、带icon的文本框

![](http://img.blog.csdn.net/20170308170856092?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
```
//这里使用UIView是为了让UIImageView有个内边距
UIView *leftView = [[UIView alloc]initWithFrame:CGRectMake(0, 0, 30, 30)];
//右边图标
UIImage *leftImg = [UIImage imageNamed:@"user"];
UIImageView *leftImgView = [[UIImageView alloc]initWithFrame:CGRectMake(7, 7, 16, 16)];
leftImgView.image = leftImg;
//添加到View中
[leftView addSubview:leftImgView];
//左边icon
_username.placeholder = @"请输入用户名";
_username.leftView = leftView;
//左边icon总是显示，如不设置该属性默认不显示
_username.leftViewMode = UITextFieldViewModeAlways;
//右边清除按钮
_username.clearButtonMode = UITextFieldViewModeWhileEditing;
```
UITextFieldView默认高度是30，rightView和leftView是一样道理

##**五、UILabel**

1、获取和设置UILabel的内容

```
//获取UILabel的文本内容
NSString *text = self.label.text;
//设置UILabel的值
[self.label setText：@"label two"];
```

##**六、UIColor**


1、创建UIColor的两种方式
```
//第一种方式
UIColor *color = [UIColor redColor];
//第二种方式
UIColor *color = [UIColor colorWithRed：0 green：0 blue：255 alpha：1];
```

##**七、UIButton**

1、代码生成UIButton

```
//初始化UIButton
UIButton *bt = [UIButton buttonWithType：UIButtonTypeCustom];
//初始化一张照片
UIImage *img = [UIImage imageNamed：@"icon.jpg"];
//设置正常状态下的标题
[bt setTitle：@"click" forState：UIControlStateNormal];
//设置正常状态下的图片
[bt setImage：img forState：UIControlStateNormal];
//设置正常状态下的位置大小
bt.frame = CGRectMake(0, 0, 20, 20);
//设置正常状态下的标题的颜色
[bt setTitleColor：[UIColor redColor] forState：UIControlStateNormal];
//添加到View中
[self.view addSubview：bt];
```

>当然也可以设置点击状态下的属性，那么就形成了点击效果，同Android的Selector


2、代码实现UIButton的点击事件

先申明一个方法，并且该方法没有返回IBAction类型，所以不用控件拖拽绑定

```
- (void)doString{
NSLog(@"doString");
};
```

接着上面的代码，为UIButton手动添加点击事件，使用@selector

```
[bt addTarget：self action：@selector(doString) forControlEvents：UIControlEventTouchUpInside];
```

3、Tag属性

当我们将很多按钮绑定在一个方法里面时，我们可以通过UIButton的Tag属性判断点击的是哪个按钮(Switch语句)，从而进行不同的操作

![](http://img.blog.csdn.net/20170224131209173?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**八、UIImageView**

UIImageView不仅支持图片加载，而且还支持数组图片进行帧动画的播放，在布局中创建UIImageView并绑定到代码中

```
@property (weak, nonatomic) IBOutlet UIImageView *animImg;
```

通过点击事件播放动画效果

```
- (IBAction)startAnim：(UIButton *)sender {
	//如果动画正在播放，则不操作
	if(self.animImg.isAnimating)return;
	//创建NSMutableArray 
	NSMutableArray *arr = [NSMutableArray array];
	//添加图片数据，图片存储位置在Assets.xcassets中，命名为water001-water040
	for (int i=1; i<=40; i++) {
		NSString *imgName = [NSString stringWithFormat：@"water%03d",i ];
		UIImage *img = [UIImage imageNamed：imgName];
		[arr addObject：img];
	}
	//设置UIImageView的图片数组
	self.animImg.animationImages = arr;
	//设置UIImageView的动画时间
	self.animImg.animationDuration = 4;
	//设置UIImageView的动画播放次数
	self.animImg.animationRepeatCount = 1;
	//开启动画
	[self.animImg startAnimating];
}
```

下面是动画效果图

![](http://img.blog.csdn.net/20170225212430251?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

如果在界面上需要播放很多组动画，就会导致iOS内存的增加，下面采用另一种方法进行优化

1. 将图片资源的存储位置放在xcode的xproject主目录中，也就是NSBundle的位置
2. 通过NSBundle来获取图片路径
3. 通过路径获取UIImage，通过路径的方法可以使控件变为弱类型
4. 通过延时线程将内存清空

```
- (IBAction)startAnim：(UIButton *)sender {
	//如果动画正在播放，则不操作
	if(self.animImg.isAnimating)return;
	//创建NSMutableArray 
	NSMutableArray *arr = [NSMutableArray array];
	//添加图片数据，图片存储位置在主目录中
	for (int i=1; i<=40; i++) {
		NSString *imgName = [NSString stringWithFormat：@"water%03d.jpg",i ];
		//通过Bundle的方式获取UIImage
		NSBundle *bundle = [NSBundle mainBundle];
		NSString *imgPath = [bundle pathForResource：imgName ofType：nil];
		UIImage *image = [UIImage imageWithContentsOfFile：imgPath];
		[arr addObject：image];
	}
	self.animImg.animationImages = arr;
	self.animImg.animationDuration = 4;
	self.animImg.animationRepeatCount = 1;
	[self.animImg startAnimating];
	//开启延时线程，并回调clean函数
	[self performSelector：@selector(clean) withObject：nil afterDelay：4];
}

- (void)clean{
	self.animImg.animationImages = nil;
}
```

两种方法的比较

* imageNamed：自动对图片做缓存 
	* 优点：第一次加载之后，后面会使用直接读取缓存
	* 缺点：会导致内存的飙升，因为图片类型都为强类型
* imageWithContentsOfFile：不会做缓存，需要从NSBundle路径里面取出 
	* 优点：不做缓存
	* 缺点：加载比较慢