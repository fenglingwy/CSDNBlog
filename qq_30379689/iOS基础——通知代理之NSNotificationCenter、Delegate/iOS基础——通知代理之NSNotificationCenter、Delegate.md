#iOS基础——通知代理之NSNotificationCenter、Delegate


----------

##**前言**
* NSNotificationCenter就像Android的广播接收者，它可以通过发送通知，让监听通知的者收到通知，并执行相应事件，它是一种一对多的事件通知
* Delegate就像Android的监听接口，它可以通过实现Delegate，并实现其方法，通过调用方法即可获取Delegate里面传过来的内容

##**一、NSNotificationCenter**

1、自定义通知

第一步：定义一个方法

```
-(void)getCarName{
    NSLog(@"getCarName");
}
```

第二步：注册通知

```
[[NSNotificationCenter defaultCenter]addObserver:self
                                        selector:@selector(getCarName)
                                            name:@"event1"
                                          object:nil];
```
参数

* addObserver：添加该通知的类
* selector：接收到通知后执行的方法
* name：通知的名字，作为通知的标识
* object：传递的对象

第三步：发送广播

我们通过创建一个按钮的点击事件来发送广播

```
- (IBAction)notifyEvent:(id)sender {
    [[NSNotificationCenter defaultCenter]postNotificationName:@"event1"
                                                       object:nil
                                                     userInfo:nil];
}
```
参数

* postNotificationName：发送广播的名字
* object：传递的对象
* userInfo：传递的参数

第四步：移除通知

注册了通知之后，必须记得移除通知，否则程序在退出时候回报错

```
-(void)dealloc{
    [[NSNotificationCenter defaultCenter]removeObserver:self];
}
```

2、监听键盘通知

系统提供了监听各种控件的通知，这里演示键盘的通知

第一步：注册通知

这里注册键盘将要弹出时和将要隐藏时的通知

```
[[NSNotificationCenter defaultCenter]addObserver:self
                                            selector:@selector(show:)
                                                name:UIKeyboardWillShowNotification
                                              object:nil];
    
[[NSNotificationCenter defaultCenter]addObserver:self
                                        selector:@selector(hide:)
                                            name:UIKeyboardWillHideNotification
                                          object:nil];
```
第二步：实现对应方法

这里会接受一个NSNotification，可以通过NSNotification获取通知传递过来的信息
```
-(void)show:(NSNotification *) notification{
    NSDictionary *userInfo = notification.userInfo;
    NSLog(@"userInfo:%@",userInfo);
}

-(void)hide:(NSNotification *)notification{
   
}
```
当键盘弹出时，这里会打印出键盘的各种信息，可以获取键盘的Frame等信息

```
2017-03-14 01:02:24.857 DelegateDemo[1865:112324] userInfo:{
    UIKeyboardAnimationCurveUserInfoKey = 7;
    UIKeyboardAnimationDurationUserInfoKey = "0.25";
    UIKeyboardBoundsUserInfoKey = "NSRect: {{0, 0}, {375, 258}}";
    UIKeyboardCenterBeginUserInfoKey = "NSPoint: {187.5, 796}";
    UIKeyboardCenterEndUserInfoKey = "NSPoint: {187.5, 538}";
    UIKeyboardFrameBeginUserInfoKey = "NSRect: {{0, 667}, {375, 258}}";
    UIKeyboardFrameEndUserInfoKey = "NSRect: {{0, 409}, {375, 258}}";
    UIKeyboardIsLocalUserInfoKey = 1;
}
```

第三步：移除通知

别忘了移除通知哦

3、解决键盘遮盖文本框问题

通过通知传递过来的参数可以计算出键盘应该偏移的y值，让界面向上移动即可，效果图如下

![](http://img.blog.csdn.net/20170314010625395?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

通过对通知的处理，完成对应的动画即可，如果对y轴偏移量计算不懂的，可以在纸上画一下就懂了
```
-(void)show:(NSNotification *) notification{
    NSDictionary *userInfo = notification.userInfo;
    NSLog(@"userInfo:%@",userInfo);
    //获取键盘的frame
    CGRect keyBoardFrame = [userInfo[UIKeyboardFrameBeginUserInfoKey] CGRectValue];
    //获取键盘的高度
    CGFloat height = keyBoardFrame.size.height;
    //获取屏幕的高度
    CGFloat screenHeight = [UIScreen mainScreen].bounds.size.height;
    //计算出View的y抽偏移量
    CGFloat dy = screenHeight - height - self.textField.frame.origin.y - self.textField.frame.size.height;
    //如果被遮盖
    if(dy<0){
        //执行动画，将View的中心往上移动
        [UIView animateWithDuration:1 animations:^{
            self.view.center = CGPointMake(self.view.center.x, dy+screenHeight/2);
        }];
    }
}

-(void)hide:(NSNotification *)notification{
    //执行动画，恢复到屏幕原来的中心
    [UIView animateWithDuration:1 animations:^{
        self.view.center = CGPointMake(self.view.center.x, [UIScreen mainScreen].bounds.size.height/2);
    }];
}

#pragma 代理方法
-(void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event{
    //点击非文本框让文本框取消焦点，即隐藏键盘
    [self.textField resignFirstResponder];
}
```

##**二、Delegate**

1、自定义Delegate

第一步：在carModel.h文件中，导入自己class，定义delegate的protocol文件

```
#import <Foundation/Foundation.h>
//导入自己class
@class CarModel;

//自定义代理
@protocol CarModelDelegate <NSObject>
//1、必须实现的回调方法
@required
-(void)getCarModel:(CarModel *)model;
//2、可选实现的回调方法
@optional
-(void)getCarOtherModel:(CarModel *)model;
@end

@interface CarModel : NSObject
@property (nonatomic,strong)NSString *name;
@property (nonatomic,strong)NSString *nickName;
//声明代理
@property (nonatomic,weak)id<CarModelDelegate> delegate;
//声明方法，模拟按钮点击
-(void)didclickButton;
@end
```

这里提供两个变量用于我们代理传递的参数，同时在代理中提供了两个方法，有必选方法和可选方法，还有一个模拟按钮点击的方法

第二步：在carModel.m文件中，先初始化数据

```
-(instancetype)init{
    if(self = [super init]){
        //初始化数据
        self.name = @"奥迪";
        self.nickName = @"A6";
    }
    return self;
}
```
第三步：实现模拟点击按钮方法，执行代理的方法

```
//模仿按钮点击
-(void)didclickButton{
    //1、执行必选代理回调
    if([self.delegate respondsToSelector:@selector(getCarModel:)]){
        [self.delegate getCarModel:self];
    }
    //2、执行可选代理回调
    if([self.delegate respondsToSelector:@selector(getCarOtherModel:)]){
        [self.delegate getCarOtherModel:self];
    }
}
```

第四步：在ViewController文件中，声明代理，实现代理的方法

```
#import "ViewController.h"
#import "CarModel.h"

@interface ViewController ()<CarModelDelegate>

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];

}

-(void)getCarModel:(CarModel *)model{
    NSLog(@"CarName-->%@",model.name);
}

@end
```
第五步：初始化CarModel，并且设置代理方法

```
- (void)viewDidLoad {
    [super viewDidLoad];

    //初始化并设置代理
    CarModel *model = [[CarModel alloc]init];
    model.delegate = self;
}

```

那么这个时候，只要CarModel按钮一点击，在这里就会回调这个代理方法

第六步：执行模拟按钮点击方法

```
//模拟按钮点击
[model didclickButton];
```
第七步：查看Log打印

```
2017-03-17 13:42:18.449 DelegateDemo[757:14292] CarName-->奥迪
```

[源码下载](http://download.csdn.net/detail/qq_30379689/9784472)