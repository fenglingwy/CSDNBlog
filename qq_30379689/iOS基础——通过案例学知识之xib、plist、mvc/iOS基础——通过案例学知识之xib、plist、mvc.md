#iOS基础——通过案例学知识之xib、plist、mvc


----------
透过案例学习xib的使用、plist的使用、mvc在iOS的使用，今天要做的案例效果图

![](http://img.blog.csdn.net/20170301125615708?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**一、xib**

1、xib和nib

xib文件可以被XCode编译成nib文件，xib文件本质上是一个xml文件，而nib文件就是编译后的二进制文件

2、xib和main.storyboard

xib是轻量级的UI布局，main.storyboard是重量级的，用来描述整个软件的多个界面，并且能展示多个界面的跳转关系

3、xib的创建

创建项目中需要的xib文件

![](http://img.blog.csdn.net/20170301125647235?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、xib的使用

进入xib界面，直接通过底部的控件，拖拽控件，组成我们项目中需要的一个Item，接着就是重复遍历一样的出来就可以形成九宫格了

![](http://img.blog.csdn.net/20170301125758820?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、xib的属性设置

我们需要创建一个YellowView继承UIView，来与xib进行关联

```
#import <UIKit/UIKit.h>

@interface YellowView : UIView

@end
```

① 设置class属性

![](http://img.blog.csdn.net/20170301125920166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 设置xib中的View的大小

![](http://img.blog.csdn.net/20170301125931971?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

③ 设置xib中View的圆角属性，边框属性等等

![](http://img.blog.csdn.net/20170301125951995?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、xib初始化

当xib将m文件关联起来之后，默认会在m文件中执行初始化方法

```
-(void)awakeFromNib{
	NSLog(@"xib的初始化方法");
}
```

6、代码获取xib

```
//加载xib文件，并取出lastObject（xib的最后一个布局），第一个也是YellowView，因为只有一个
YellowView *view = [[[NSBundle mainBundle] loadNibNamed:@"YellowView" owner:nil options:nil] lastObject];
//获取宽高
CGFloat w = view.frame.size.width;
CGFloat h = view.frame.size.height;
```


##**二、plist**
1、创建plist文件

plist用来存储设置的地方，也可以存储资源，你可以理解为一个小型的自带数据库一样，不过它的操作可不是跟数据库一样，通过New File可以找到plist文件

2、设置plist数据

对plist进行赋值，设置我们要演示的数据到plist上，name表示名字，icon表示图片资源的名字

![](http://img.blog.csdn.net/20170301130612941?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、获取Plist数据

通过代码获取plist的根属性NSArray，通过遍历，将plist数据转换成模型，并存储在dataArray中
```
//属性
@property (nonatomic,strong) NSArray *dataArray;

-(NSArray *)dataArray{
    //获取plist路径
    NSString *path = [[NSBundle mainBundle] pathForResource:@"data.plist" ofType:nil];
    //拿出plist的属性是一个NSArray，里面装的是NSDictionary
    NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
    //创建可变数组
    NSMutableArray *muta = [NSMutableArray array];
    //遍历数组，拿出NSDictionary
    for (NSDictionary *dict in tempArray) {
        //字典转模型
        AppModel *appModel = [AppModel appModelWithDict:dict];
        //添加到数组
        [muta addObject:appModel];
    }
    //将数组赋值给成员变量dataArray
    _dataArray = muta;
    //返回
    return _dataArray;
}
```

##**三、mvc**

1、框架结构目录

![](http://img.blog.csdn.net/20170301131038994?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、Model

根据案例需求，Model层应该储存的数据

1. name属性
2. icon属性
3. 由于数据源是个字典类型，需要构造方法让字典转换为模型

```
//.h文件
#import <Foundation/Foundation.h>

@interface AppModel : NSObject

@property(nonatomic,copy) NSString *name;
@property(nonatomic,copy) NSString *icon;

-(instancetype) initWithDict:(NSDictionary *) dict;
+(instancetype) appModelWithDict:(NSDictionary *) dict;
@end
//.m文件
#import "AppModel.h"

@implementation AppModel

-(instancetype)initWithDict:(NSDictionary *)dict{
    if(self = [super init]){
        self.name = dict[@"name"];
        self.icon = dict[@"icon"];
    }
    return self;
}

+(instancetype)appModelWithDict:(NSDictionary *)dict{
    return [[self alloc] initWithDict:dict];
}
@end
```
3、View

View层主要描述xib中的View，其应该包含

1. 文本View
2. 图片View
3. 按钮点击事件
4. 点击事件处理
5. 点击动画效果

```
//.h文件
#import <UIKit/UIKit.h>
@class AppModel;

//外部声明，全局属性，对外公布
@interface YellowView : UIView

@property(nonatomic,strong) AppModel *appModel;

@end
//.m文件
#import "YellowView.h"
#import "AppModel.h"

//内部声明，局部属性，不对外公布
@interface YellowView()
@property (weak, nonatomic) IBOutlet UIImageView *imageIcon;
@property (weak, nonatomic) IBOutlet UILabel *name;
@end

@implementation YellowView

//复写set方法
-(void) setAppModel:(AppModel *)appModel{
    //对所有属性进行赋值
    _appModel = appModel;
    _imageIcon.image = [UIImage imageNamed:appModel.icon];
    _name.text = appModel.name;
}

//下载按钮
- (IBAction)downlad:(UIButton *)sender {
    //取出ctrollerView
    UIView *controllerView = self.superview;
    //创建coverView
    UIView *coverView = [[UIView alloc]initWithFrame:controllerView.bounds];
    coverView.alpha = 0;
    coverView.backgroundColor = [UIColor blackColor];
    [controllerView addSubview:coverView];
    //创建Label
    UILabel *label  = [[UILabel alloc]initWithFrame:controllerView.bounds];
    label.text = @"下载中...";
    label.center = CGPointMake(controllerView.frame.size.width/2, controllerView.frame.size.height/2);
    label.textColor = [UIColor whiteColor];
    label.textAlignment = NSTextAlignmentCenter;
    [coverView addSubview:label];
    //执行coverView动画
    [UIView animateWithDuration:1 animations:^{
        coverView.alpha = 0.6;
    }completion:^(BOOL finished) {
        //动画完成后执行延时动画
        [UIView animateWithDuration:1
                              delay:1
                            options:UIViewAnimationOptionCurveEaseInOut
                         animations:^{
                             coverView.alpha = 0;
                         } completion:^(BOOL finished) {
                             //动画结束
                             sender.enabled = NO;
                             [coverView removeFromSuperview];
                         }];
    }];
}

@end

```

4、Cotroller

Controller负责逻辑处理，处理数据加载，和处理数据与View的绑定

1. 读取数据源
2. 循环数据源，添加到xib中的View
3. 为View绑定Model数据

```
//.h文件
#import <UIKit/UIKit.h>
@interface ViewController : UIViewController
@end
//.m文件
#import "ViewController.h"
#import "YellowView.h"
#import "AppModel.h"

@interface ViewController ()
@property (nonatomic,strong) NSArray *dataArray;
@end

@implementation ViewController

//复写getDataArray方法
-(NSArray *)dataArray{
    //获取plist路径
    NSString *path = [[NSBundle mainBundle] pathForResource:@"data.plist" ofType:nil];
    //拿出plist的属性是一个NSArray，里面装的是NSDictionary
    NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
    //创建可变数组
    NSMutableArray *muta = [NSMutableArray array];
    //遍历数组，拿出NSDictionary
    for (NSDictionary *dict in tempArray) {
        //字典转模型
        AppModel *appModel = [AppModel appModelWithDict:dict];
        //添加到数组
        [muta addObject:appModel];
    }
    //将数组赋值给成员变量dataArray
    _dataArray = muta;
    //返回
    return _dataArray;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    //遍历程序成员变量
    for (int i=0; i<self.dataArray.count; i++) {
        //加载xib文件，并取出lastObject
        YellowView *view = [[[NSBundle mainBundle] loadNibNamed:@"YellowView" owner:nil options:nil] lastObject];
        CGFloat w = view.frame.size.width;
        CGFloat h = view.frame.size.height;
        CGFloat margin = 20;
        //设置大小和位置，这里i%2表示2列的x值，i/2表示2行的y值
        view.frame = CGRectMake(i%3* (w+margin)+20, i/3 *(h+margin)+20,w, h);
        //添加到主View中
        [self.view addSubview:view];
        //取出数据
        AppModel *appModel = self.dataArray[i];
        //赋值给view
        view.appModel = appModel;
    }
}

- (void)didReceiveMemoryWarning {
    [super didReceiveMemoryWarning];
    // Dispose of any resources that can be recreated.
}

@end

```

[源码下载](http://download.csdn.net/detail/qq_30379689/9767196)