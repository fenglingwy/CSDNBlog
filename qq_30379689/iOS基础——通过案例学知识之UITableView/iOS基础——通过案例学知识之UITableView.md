#iOS基础——通过案例学知识之UITableView


----------
##**案例一：英雄联盟展示**

对于UITableView的知识点特别多，因为它是iOS用得最多控件之一，我会尽我最大努力和语言的组织，将所有知识点介绍到位，下面是要实现的效果图

![这里写图片描述](http://img.blog.csdn.net/20170306231253784?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

吐槽

* 与Android对比，可以说跟ListView的实现几乎一样，跟RecyclerView一模一样
* Android写起来似乎比iOS复杂一点，因为iOS大部分都被封装好了，这一点iOS做得好
* 对于iOS的方法的命名只能说又长又臭

知识点包括

* UITableView的UITableViewDataSource
* UITableView的UITableViewDelegate
* UITableView的cell的重用

##**一、准备工作**

1、准备数据源（plist）

![](http://img.blog.csdn.net/20170306172619856?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、布局文件

 ![这里写图片描述](http://img.blog.csdn.net/20170306172033792?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**二、Model读取数据**

分析plist数据的格式，然后创建对应的对象模型，并提供相应的初始化方法，这是mvc中经典的一个步骤

* 特别注意：@property中属性不可以使用weak属性，否则在UITableView复用机制中会被回收，导致画面显示不出来

```
@interface HeroModel : NSObject
@property(nonatomic,strong) NSString *icon;
@property(nonatomic,strong) NSString *intro;
@property(nonatomic,strong) NSString *name;

-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)HeroModelWithDict:(NSDictionary *)dict;
@end
```

在m文件中实现初始化方法，方法中实现字典转换为对象

```
@implementation HeroModel

-(instancetype)initWithDict:(NSDictionary *)dict{
    if(self = [super init]){
        [self setValuesForKeysWithDictionary:dict];
    }
    return self;
}

+(instancetype)HeroModelWithDict:(NSDictionary *)dict{
    return [[self alloc]initWithDict:dict];
}
@end
```

##**三、UITableView**

1、声明委托代理，声明属性

要想UITableView有数据，那么就必须通过它的委托代理方法才能显示UITableView中的数据

```
@interface ViewController ()<UITableViewDataSource>
//存放数据的可变数组
@property (strong, nonatomic) NSMutableArray *dataArray;
//tableview
@property (strong, nonatomic) IBOutlet UITableView *tableView;
@end
```

2、实现属性的转换

毫无疑问是通过懒加载将plist的内容转为模型存进我们声明的可变数组中
```
#pragma 复写get方法
#pragma 懒加载，读取plist文件并转换为模型
- (NSMutableArray *)dataArray{
    if(nil == _dataArray){
        //初始化数组
        _dataArray = [NSMutableArray array];
        //获取plist文件路径
        NSString *path = [[NSBundle mainBundle]pathForResource:@"heros.plist" ofType:nil];
        //读取plist文件内容
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        //遍历plist文件内容，存到可变数组中
        for (NSDictionary * dict in tempArray) {
            HeroModel *heroModel = [HeroModel HeroModelWithDict:dict];
            [_dataArray addObject:heroModel];
        }
    }
    //返回
    return _dataArray;
}
```
3、交付委托

```
//交付委托
_tableView.dataSource = self;
```

4、实现代理的方法

通过实现UITableViewDataSource代理的方法，来显示数据，类似于ListView的Adapter

```
#pragma UITableViewDataSource委托方法
#pragma 返回一共有多少组
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    //默认返回1组
    return 1;
}

#pragma UITableViewDataSource委托方法
#pragma 返回一个组由多少行
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    //返回数据的数量
    return self.dataArray.count;
}

#pragma UITableViewDataSource委托方法
#pragma 返回每一行Item的内容
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    //创建tableview的item
    UITableViewCell *cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:nil];
    //通过indexPath的行属性，取出对应的模型
    HeroModel *heroModel = _dataArray[indexPath.row];
    //设置文本信息
    cell.textLabel.text = heroModel.name;
    //设置图片信息
    UIImage *image = [UIImage imageNamed:heroModel.icon];
    cell.imageView.image = image;
    //设置详细信息文本
    cell.detailTextLabel.text = heroModel.intro;
    //设置最右边的箭头
    cell.accessoryType = UITableViewCellAccessoryDisclosureIndicator;
    //返回
    return cell;
}
```
① UITableView的显示有两种方式，在storyboard中可以设置

1.plain：数据平铺显示，中间没有空隙，数组的头标题有悬浮效果

![](http://img.blog.csdn.net/20170306223847830?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.group：数据分组显示，中间留有空隙

![这里写图片描述](http://img.blog.csdn.net/20170306223933175?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② UITableViewDataSource的委托方法，程序会按以下顺序执行

1. numberOfSectionsInTableView：数据中有呈多少组展示（可选实现，默认返回1组）
2. numberOfRowsInSection：数据中每组有多少行（必须实现，否则会报错）
3. cellForRowAtIndexPath：数据中每行的内容（必须实现，否则会报错）

③ UITableViewCell的四种样式

1.UITableViewCellStyleDefault

![](http://img.blog.csdn.net/20170306224306926?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.UITableViewCellStyleValue1

![这里写图片描述](http://img.blog.csdn.net/20170306224320786?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3.UITableViewCellStyleValue2

![这里写图片描述](http://img.blog.csdn.net/20170306224330902?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4.UITableViewCellStyleSubtitle

![](http://img.blog.csdn.net/20170306224343317?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

④ cell.accessoryType的五种样式，用得不多就不解释了

1. UITableViewCellAccessoryNone
2. UITableViewCellAccessoryDisclosureIndicator
3. UITableViewCellAccessoryDetailDisclosureButton
4. UITableViewCellAccessoryCheckmark
5. UITableViewCellAccessoryDetailButton

5、UITableViewDataSource其他代理方法

这两个代理方法会在group样式上展示得比较清晰
```
#pragma UITableViewDataSource委托方法
#pragma 返回tableview中头部的标题
-(NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
    return @"header";
}

#pragma UITableViewDataSource委托方法
#pragma 返回tableview中底部的标题
-(NSString *)tableView:(UITableView *)tableView titleForFooterInSection:(NSInteger)section{
    return @"footer";
}
```

6、cell的重用

* cell与ListView的Item中是一样的，它也是要对Item进行重用的
* 这里的重用主要是用到重用标识符
* 在cellForRowAtIndexPath方法中改一下cell的创建代码即可

```
//重用标识符，需要用static修饰，避免多次分配内存给NSString
static NSString *identifier = @"h1";
//从缓存池中取出tableview的item
UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
//创建cell
if(nil == cell){
    cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleSubtitle reuseIdentifier:identifier];
}
```

7、tableView的属性介绍

1. _tableView.separatorColor：分割线颜色
2. _tableView.separatorInset：分割线缩进
3. _tableView.separatorStyle：分割线类型
4. _tableView.allowsMultipleSelection：允许选项多选
5. _tableView.tableHeaderView：可以添加一个在头部的View
6. _tableView.tableFooterView：可以添加一个在底部的View
7. _tableView.sectionHeaderHeight：每个组的间隔

8、实现cell的点击事件

① 声明委托与交付委托

cell的点击事件是在UITableViewDelegate的实现方法

```
//声明委托
@interface ViewController ()<UITableViewDataSource,UITableViewDelegate>
//交付委托
_tableView.delegate = self;
```
② 实现点击事件函数
```
#pragma UITableViewDelegate委托方法
#pragma 反选数据时调用
-(void)tableView:(UITableView *)tableView didDeselectRowAtIndexPath:(NSIndexPath *)indexPath{
    
}

#pragma UITableViewDelegate委托方法
#pragma 选择数据时调用
-(void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath{
    
}
```

9、实现cell的编辑模式和cell的增加插入

cell的编辑模式和cell的增加插入也是在UITableViewDelegate的实现方法

```
#pragma UITableViewDelegate委托方法
#pragma 决定哪一行可进入编辑模式
-(BOOL)tableView:(UITableView *)tableView canEditRowAtIndexPath:(NSIndexPath *)indexPath{
    if(indexPath.row == 2){
        return NO;
    }else{
        return YES;
    }
}

#pragma UITableViewDelegate委托方法
#pragma 点击delete和insert的回调函数，该函数同时回开启侧滑删除功能
-(void)tableView:(UITableView *)tableView commitEditingStyle:(UITableViewCellEditingStyle)editingStyle forRowAtIndexPath:(NSIndexPath *)indexPath{
    if(editingStyle == UITableViewCellEditingStyleDelete){
        //删除数组中的数据
        [_dataArray removeObjectAtIndex:indexPath.row];
        //tableview完成删除操作，更新UI
        [_tableView deleteRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationLeft];
    }else if(editingStyle == UITableViewCellEditingStyleInsert){
        //模拟添加数据
        HeroModel *hero = [[HeroModel alloc]init];
        hero.name = @"寒冰射手";
        //添加到数组中
        [_dataArray insertObject:hero atIndex:indexPath.row];
        //tableview完成添加操作，更新UI
        [_tableView insertRowsAtIndexPaths:@[indexPath] withRowAnimation:UITableViewRowAnimationRight];
    }
}

#pragma UITableViewDelegate委托方法
#pragma 决定哪一行开启的编辑模式是插入模式还是删除模式
-(UITableViewCellEditingStyle)tableView:(UITableView *)tableView editingStyleForRowAtIndexPath:(NSIndexPath *)indexPath{
    if(indexPath.row <= 5){
        return UITableViewCellEditingStyleInsert;
    }else{
        return UITableViewCellEditingStyleDelete;
    }
}

#pragma UITableViewDelegate委托方法
#pragma 删除模式下的删除按钮文字显示
-(NSString *)tableView:(UITableView *)tableView titleForDeleteConfirmationButtonForRowAtIndexPath:(NSIndexPath *)indexPath{
    return @"蹦瞎卡拉卡";
}
```

① UITableViewDelegate的委托方法

1. canEditRowAtIndexPath：允许哪一行开启编辑模式
2. commitEditingStyle：点击事件的回调，同时开启侧滑删除
3. editingStyleForRowAtIndexPath：决定哪一行的编辑模式

② indexPath属性

1. indexPath.row：行的索引
2. indexPath.section：组的索引

③ 编辑模式动画，大家看名字应该都可以猜得出

1. UITableViewRowAnimationFade
2. UITableViewRowAnimationRight
3. UITableViewRowAnimationLeft
4. UITableViewRowAnimationTop
5. UITableViewRowAnimationBottom
6. UITableViewRowAnimationNone      
7. UITableViewRowAnimationMiddle      
8. UITableViewRowAnimationAutomatic

④ 最后只要开启编辑模式

```
_tableView.editing = YES;
```



[源码下载](http://download.csdn.net/detail/qq_30379689/9772361)




----------

##**案例二：汽车展示**

这是对上面博客缺少的内容进行补充，实现的效果如图

![](http://img.blog.csdn.net/20170308140101922?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

知识点

* UITableViewController
* 侧边字母导航栏

##**一、准备工作**

1、准备数据源（plist）

![](http://img.blog.csdn.net/20170308140354274?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、布局文件

这次采用UITableViewController作为启动界面，其好处是自动将UITableViewDelegate和UITableViewDataSource自动绑定到对应的m文件中

![](http://img.blog.csdn.net/20170308140606575?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

不过需要注意的是m文件中必须实现UITableViewController

```
@interface ViewController : UITableViewController
@end
```

##**二、Model读取数据**

跟上篇文章一样，分析Model中拥有的属性，这里包括有两个Model，Cars和Car，分别代表多辆车的集合和单辆车的数据

Cars的h文件

```
@interface Cars : NSObject
@property (nonatomic,strong)NSString *title;
@property (nonatomic,strong)NSArray *cars;

-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)CarsModelWithDict:(NSDictionary *)dict;
@end
```
Cars的m文件

和Car有点区别，就是在初始化的时候，将Car的NSArray和转换成模型

```
@implementation Cars
-(instancetype)initWithDict:(NSDictionary *)dict{
    if(self = [super init]){
        [self setValuesForKeysWithDictionary:dict];
        //转换Car
        NSMutableArray *muta = [NSMutableArray array];
        for (NSDictionary *dict in self.cars) {
            Car *car = [Car CarModelWithDict:dict];
            [muta addObject:car];
        }
        self.cars = muta;
    }
    return self;
}
+(instancetype)CarsModelWithDict:(NSDictionary *)dict{
    return [[self alloc]initWithDict:dict];
}
@end
```

Car的h文件

```
@interface Car : NSObject
@property (nonatomic,strong)NSString *icon;
@property (nonatomic,strong)NSString *name;

-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)CarModelWithDict:(NSDictionary *)dict;
@end
```

Car的m文件

```
@implementation Car
-(instancetype)initWithDict:(NSDictionary *)dict{
    if(self = [super init]){
        [self setValuesForKeysWithDictionary:dict];
    }
    return self;
}
+(instancetype)CarModelWithDict:(NSDictionary *)dict{
    return [[self alloc]initWithDict:dict];
}
@end
```

##**三、UITableView**

1、属性声明

```
@interface ViewController ()
//存放数据数组
@property (nonatomic,strong)NSMutableArray *dataArray;
//存放索引数组
@property (nonatomic,strong)NSArray *indexArray;
@end
```

2、plist数据的加载

```
#pragma 复写set方法
#pragma 懒加载，读取plist文件内容
-(NSArray *)dataArray{
    if(nil == _dataArray){
        //初始化数组
        _dataArray = [NSMutableArray array];
        //获取plist文件路径
        NSString *path = [[NSBundle mainBundle]pathForResource:@"cars_total.plist" ofType:nil];
        //读取plist文件内容
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        //遍历plist内容存放到可变数组
        for (NSDictionary  *dict in tempArray) {
            Cars *cars = [Cars CarsModelWithDict:dict];
            [_dataArray addObject:cars];
        }
        //取出索引
        _indexArray = [_dataArray valueForKey:@"title"];
    }
    return _dataArray;
}
```
3、由于UITableView是继承UITableViewController，所以可以直接使用其代理方法

```
#pragma UITableViewDataSource委托方法
#pragma 返回一共有多少组
-(NSInteger)numberOfSectionsInTableView:(UITableView *)tableView{
    return self.dataArray.count;
}

#pragma UITableViewDataSource委托方法
#pragma 返回一组多少行
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    Cars *cars = self.dataArray[section];
    return cars.cars.count;
}

#pragma UITableViewDataSource委托方法
#pragma 返回每一行Item的内容
-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    //cell的重用
    static NSString *identifier = @"car";
    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    if(nil == cell){
        cell = [[UITableViewCell alloc]initWithStyle:UITableViewCellStyleDefault reuseIdentifier:identifier];
    }
    //设置内容
    Cars *cars = self.dataArray[indexPath.section];
    Car *car = cars.cars[indexPath.row];
    UIImage *image = [UIImage imageNamed:car.icon];
    cell.textLabel.text = car.name;
    cell.imageView.image = image;
    return cell;
}

#pragma UITableViewDataSource委托方法
#pragma 返回每一组的头部标题
-(NSString *)tableView:(UITableView *)tableView titleForHeaderInSection:(NSInteger)section{
    return _indexArray[section];
}

#pragma UITableViewDataSource委托方法
#pragma 返回右侧索引值
-(NSArray<NSString *> *)sectionIndexTitlesForTableView:(UITableView *)tableView{
    return _indexArray;
}
```

[源码下载](http://download.csdn.net/detail/qq_30379689/9774085)


----------
##**案例三：设置列表**
这个案例很简单，不用代码，直接使用storyboard就可以了，一般使用在固定的列表中使用

![](http://img.blog.csdn.net/20170309131411371?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

知识点

* 静态Cell

##**一、准备数据**

1、使用UITableViewController，设置内容为静态Cell，而且设置有三节内容

![这里写图片描述](http://img.blog.csdn.net/20170309131614747?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、设置其style为分组

![这里写图片描述](http://img.blog.csdn.net/20170309131728560?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、你会看到左手边会出现三节TableView

![这里写图片描述](http://img.blog.csdn.net/20170309131835897?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、选中section可以设置TableViewCell的数量，设置头部标题

![这里写图片描述](http://img.blog.csdn.net/20170309132042603?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、选中TableViewCell，设置其style，照片，文字，accessory

![这里写图片描述](http://img.blog.csdn.net/20170309132003483?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6、大功告成

----------
##**案例四：应用管理**
这个案例使用自定义Cell来完成，可以说是大多数UITableView的首选

![](http://img.blog.csdn.net/20170309195931156?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

知识点

* 自定义Cell

##**一、准备数据**

![这里写图片描述](http://img.blog.csdn.net/20170309200202218?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里直接使用UITableView和UITableViewCell直接自定义我们的Cell

![](http://img.blog.csdn.net/20170309200318861?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

创建类继承自UITableViewCell

```
@interface TableViewCell : UITableViewCell
@end
```

记得填写UITableViewCell的类和Cell重用标识符

![这里写图片描述](http://img.blog.csdn.net/20170309200551021?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170309200601584?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


##**二、Model读取数据**

根据plist文件有的属性创建我们的h文件

```
#import <Foundation/Foundation.h>

@interface AppModel : NSObject
@property (nonatomic,strong)NSString * size;
@property (nonatomic,strong)NSString * download;
@property (nonatomic,strong)NSString * name;
@property (nonatomic,strong)NSString * icon;
//是否点击了下载按钮
@property(nonatomic,assign,getter=isDownloaded)BOOL *downloaded;

-(instancetype)initWithDict:(NSDictionary *)dict;
+(instancetype)AppModelWithDict:(NSDictionary *)dict;
@end
```

m文件，固定的写法

```
#import "AppModel.h"

@implementation AppModel

-(instancetype)initWithDict:(NSDictionary *)dict{
    if(self = [super init]){
        [self setValuesForKeysWithDictionary:dict];
    }
    return self;
}
+(instancetype)AppModelWithDict:(NSDictionary *)dict{
    return [[self alloc]initWithDict:dict];
}

@end
```

##**三、UITableView**

1、Cell的处理

在Cell的类文件中提供一个数据Model，并通过Model绑定我们的View的值

```
//h文件
#import <UIKit/UIKit.h>
#import "AppModel.h"

@interface TableViewCell : UITableViewCell
@property (strong ,nonatomic)AppModel *appModel;
@end

//m文件
#import "TableViewCell.h"

@interface TableViewCell()
@property (weak, nonatomic) IBOutlet UIImageView *icon;
@property (weak, nonatomic) IBOutlet UILabel *name;
@property (weak, nonatomic) IBOutlet UILabel *download;
@property (weak, nonatomic) IBOutlet UIButton *down;


@end

@implementation TableViewCell

-(void)setAppModel:(AppModel *)appModel{
    _appModel = appModel;
    
    _icon.image = [UIImage imageNamed:_appModel.icon];
    _name.text = appModel.name;
    _download.text = [NSString stringWithFormat:@"大小:%@ | 下载量:%@",appModel.size,appModel.download];

    _down.enabled = !appModel.isDownloaded;
}

- (IBAction)actionDown:(id)sender {
    _down.enabled = NO;
    
    _appModel.downloaded = YES;
}

@end
```

2、UITableView的处理

创建数组属性，读取plist文件内容，重写set方法

```
@interface ViewController ()
@property (nonatomic,strong)NSMutableArray *dataArray;
@end

@implementation ViewController

-(NSMutableArray *)dataArray{
    if(nil == _dataArray){
        _dataArray = [NSMutableArray array];
        NSString *path = [[NSBundle mainBundle]pathForResource:@"apps_full.plist" ofType:nil];
        NSArray *tempArray = [NSArray arrayWithContentsOfFile:path];
        for (NSDictionary *dict in tempArray) {
            AppModel *appmodel = [AppModel AppModelWithDict:dict];
            [_dataArray addObject:appmodel];
        }
    }
    return _dataArray;
}
```

UITableViewDataSource委托的实现方法

```
- (void)viewDidLoad {
    [super viewDidLoad];
}

-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section{
    return self.dataArray.count;
}

-(UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath{
    static NSString *identifier = @"appCell";
    
    TableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:identifier];
    cell.appModel = self.dataArray[indexPath.row];
    
    return cell;
}
@end
```

这样就好了，其实很简单

[源码下载](http://download.csdn.net/detail/qq_30379689/9775931)
##**结语**
学习UITableView应该掌握以下的知识点

* 字典转模型
* 双模型转换
* 决定组数
* 决定行数
* 决定每行内容
* Header标题
* Header自定义View
* Footer标题
* Footer自定义View
* 插入／删除操作
* 点击／非点击操作
* Cell重用
* 自定义Cell
