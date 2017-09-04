#iOS基础——通过案例学知识之LaunchScreen、APPIcon、StatusBar、UIScrollView、UIPageControl

----------
今天要实现的案例效果图

![](http://img.blog.csdn.net/20170305231623299?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**一、LaunchScreen**

1、设置程序的LaunchScreen

在项目配置文件中配置启动页，并且在LaunchScreen.storyboard中进行布局

![](http://img.blog.csdn.net/20170303221728604?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、设置LaunchScreen时间

```
//单位：秒
[NSThread sleepForTimeInterval:1.5f];
```

##**二、APPIcon**

1、命名规则：iOS应用图标是有命名规则的，对应不同的设备都有不同的尺寸

![](http://img.blog.csdn.net/20170303221942765?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其规则是：名字+尺寸+（@nx），比如

* 29x29尺寸就是29x29
* 29x29@2x尺寸就是58x58
* 29x29@3x尺寸就是87x87

2、设置APPCIcon：将这些图片拖住到默认Assets文件夹中的AppIcon中，会自动生成对应的图标适配版本

![](http://img.blog.csdn.net/20170303225731547?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**三、StatusBar**

![](http://img.blog.csdn.net/20170303230212368?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![](http://img.blog.csdn.net/20170303230221244?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1、全局设置，对所有页面生效

① 配置文件中设置

![](http://img.blog.csdn.net/20170303225933063?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 代码设置
```
//设置是否隐藏状态栏
[UIApplication sharedApplication].statusBarHidden = YES;
//设置状态栏颜色，默认是黑色，这里修改为白色
[UIApplication sharedApplication].statusBarStyle = UIStatusBarStyleLightContent;

```

2、局部设置，对一个页面生效

```
//设置是否隐藏状态栏
- (BOOL)prefersStatusBarHidden{
    return true;
}
//设置状态栏颜色，默认是黑色，这里修改为白色
- (UIStatusBarStyle)preferredStatusBarStyle{
    return UIStatusBarStyleLightContent;
}
```

##**四、userInteraction（额外内容）**

UserInteraction指的是用户交互，即是否允许用户对View进行任何操作，每个View都有这个属性，默认为YES

① 配置文件中设置

![](http://img.blog.csdn.net/20170303230408418?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 代码中设置

```
[view setUserInteractionEnable:NO];
```


##**五、subviews（额外内容）**

subviews指的是某个View中的所有子View

1、移除所有subviews

```
[_fatherView.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
```

2、遍历所有subviews

```
//遍历所有subviews
[_fatherView.subviews enumerateObjectsUsingBlock:^(__kindof UIView
* _Nonnull obj,NSUInteger idx,BOOL * _Nonnull stop{
	//obj 表示数组中的对象
	//idx 表示下标
	//*stop YES 表示跳出遍历
}];
```


##**六、UIScrollView与UIPageControl**

1、构建UIScrollView界面

![](http://img.blog.csdn.net/20170305232713507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看见UIScrollView嵌套1个轮播图和几个UIImageView，轮播图是包含UIScrollView和UIPageControl（指示器），这里需要注意的是头部View和底部的View必须和UIScrollView同级，而且在UIScrollView添加之后，这样才能覆盖在UIScrollView上

2、属性声明

```
//UIScrollViewDelegate声明委托
@interface ViewController ()<UIScrollViewDelegate>
//轮播图
@property (weak, nonatomic) IBOutlet UIScrollView *bannerView;
//主界面
@property (weak, nonatomic) IBOutlet UIScrollView *mainScrollView;
//最后一张图片，用于设置UIScrollView的y值
@property (weak, nonatomic) IBOutlet UIImageView *lastImageView;
//指示器
@property (weak, nonatomic) IBOutlet UIPageControl *bannerIndication;
//计时器
@property (nonatomic,strong) NSTimer *timer;
@end
```

3、主界面代码结构

iOS中的委托类似于Android中的监听事件，其步骤也是类似Android的实现
```
#pragma main方法
- (void)viewDidLoad {
    [super viewDidLoad];
    //设置启动界面时间
    [NSThread sleepForTimeInterval:1.5f];
    //设置主页面UI
    [self setUpUI];
}

#pragma 自定义方法
#pragma 设置UI
- (void)setUpUI{
    //设置滚动委托
    _bannerView.delegate = self;
    
    [self setUpMainScrollView];
    [self setUpBannerView];
    [self setUpBannerIndication];
    [self initTimer];
    //设置Timer的优先级
    [self setUpPriority];
}
```

4、实现代码

```
#pragma 自定义方法
#pragma 设置主界面
-(void)setUpMainScrollerView{
    //设置ScollerView滑动x和y的范围，参数1：表示在x轴不能滑动 参数2:表示在y轴滑动到最后一个View的y坐标
    _mainScrollerView.contentSize = CGSizeMake(0, CGRectGetMaxY(_lastImageView.frame));
    //隐藏滚动条
    _mainScrollerView.showsVerticalScrollIndicator = NO;
    _mainScrollerView.showsHorizontalScrollIndicator = NO;
    //设置内边距
    _mainScrollerView.contentInset = UIEdgeInsetsMake(0, 0, 20, 0);
    //设置偏移量，即一开始滑动到坐标（x，y）
    _mainScrollerView.contentOffset = CGPointMake(0, 0);
    //设置弹簧效果
    _mainScrollerView.bounces = YES;
    //设置弹簧效果，在竖直方向和横向方向，前提bounces属性必须为YES
    _mainScrollerView.alwaysBounceVertical = YES;
    _mainScrollerView.alwaysBounceHorizontal = YES;
    //不允许滚动的两种方法
    //_mainScrollerView.userInteractionEnabled = NO;
    //_mainScrollerView.scrollEnabled = NO;
}

#pragma 自定义方法
#pragma 设置轮播图
-(void)setUpBannerView{
    //获取BannerView的size
    CGSize bannerSize = _bannerView.frame.size;
    for (int i=0; i<5; i++) {
        //计算x值
        CGFloat imageX = i * bannerSize.width;
        //初始化轮播图
        UIImageView *imageView = [[UIImageView alloc]initWithFrame:CGRectMake(imageX, 0, bannerSize.width, bannerSize.height)];
        UIImage *image = [UIImage imageNamed:[NSString stringWithFormat:@"img_%02d",i+1]];
        imageView.image = image;
        //添加到BannerView中
        [_bannerView addSubview:imageView];
    }
    //设置ScollerView滑动x和y的范围
    _bannerView.contentSize = CGSizeMake(5 * bannerSize.width, 0);
    //隐藏滚动条
    _bannerView.showsVerticalScrollIndicator = NO;
    _bannerView.showsHorizontalScrollIndicator = NO;
    //设置ScollerView分页效果，默认是以每个View为分页标准
    _bannerView.pagingEnabled = YES;
}

#pragma 自定义方法
#pragma 设置轮播图指示器
-(void)setUpBannerIndication{
    //指示器总数
    _bannerIndication.numberOfPages = 5;
    //当前页指示器颜色
    _bannerIndication.currentPageIndicatorTintColor = [UIColor orangeColor];
    //非当前页指示器颜色
    _bannerIndication.pageIndicatorTintColor = [UIColor whiteColor];
}

#pragma 自定义方法
#pragma 创建计时器
-(void)initTimer{
    _timer = [NSTimer scheduledTimerWithTimeInterval:3
                                              target:self
                                            selector:@selector(startScroll)
                                            userInfo:nil
                                             repeats:YES];
}

#pragma 自定义方法
#pragma 提升Timer的优先级，防止出现Timer不执行
-(void)setUpPriority{
    NSRunLoop *mainRunLoop = [NSRunLoop mainRunLoop];
    [mainRunLoop addTimer:_timer forMode:NSRunLoopCommonModes];
}

#pragma 滚动委托
#pragma 表示当ScrollerView减速时调用，即是松开手的时候调用
-(void)scrollViewDidEndDecelerating:(UIScrollView *)scrollView{
    //输出当前滚动坐标
    NSLog(@"当前坐标为：%@",NSStringFromCGPoint(_bannerView.contentOffset));
    //根据输出当前坐标计算当前指示器的position
    int position = _bannerView.contentOffset.x / _bannerView.frame.size.width;
    //设置当前指示器position
    _bannerIndication.currentPage = position;
}

#pragma 滚动委托
#pragma 当ScrollerView开始拖动时调用
-(void)scrollViewWillBeginDragging:(UIScrollView *)scrollView{
    //停止轮播
    [_timer invalidate];
}

#pragma 滚动委托
#pragma 当ScrollerView停止拖动时调用
-(void)scrollViewDidEndDragging:(UIScrollView *)scrollView willDecelerate:(BOOL)decelerate{
    //开始轮播
    [self initTimer];
}

#pragma 自定义方法
#pragma 开启轮播，是一个修改值的问题
-(void)startScroll{
    //获取contentOffset
    CGPoint offset = _bannerView.contentOffset;
    //获取当前position
    NSInteger position = _bannerIndication.currentPage;
    //最后一张
    if(position == 4){
        position = 0;
        offset = CGPointZero;
    }else{
        //修改contentOffset
        offset.x += _bannerView.frame.size.width;
        //修改position
        position ++;
    }
    //设置值
    _bannerIndication.currentPage = position;
    [_bannerView setContentOffset:offset animated:YES];
}
```
实现思路：

* 主界面
	* 设置ScrollView滚动区域
	* 设置ScrollView其他属性
* 轮播图
	* 根据图片的个数进行横向平铺
	* 设置ScrollView滚动区域
	* 指示器设置颜色和总数
* 监听事件
	* 当轮播图被人为手指开始拖拽时，不应该播放轮播图
	* 当轮播图被人为手指结束拖拽时，继续播放轮播图
	* 当轮播图被人为手指放开时，计算轮播图位置进行属性值修改
* 计时器
	* 每隔段时间执行@selector方法，用于播放轮播图
	* 提升计时器优先级，让其不被其他控件影响执行

[源码下载](http://download.csdn.net/detail/qq_30379689/9771306)