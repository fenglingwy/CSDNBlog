#iOS基础——数据存储之沙盒机制、plist、perference、归档、反归档


----------
##**一、沙盒机制**

1、沙盒机制的介绍 

iOS中的沙盒机制（SandBox）是一种安全体系，它规定了应用程序只能在为该应用创建的文件夹内读取文件，不可以访问其他地方的内容，所有的非代码文件都保存在这个地方，比如图片、声音、属性列表和文本文件等

其特点总结如下

1. 每个应用程序都在自己的沙盒内
2. 不能随意跨越自己的沙盒去访问别的应用程序沙盒的内容
3. 应用程序向外请求或接收数据都需要经过权限认证

2、沙盒机制的沙盒目录

![](http://img.blog.csdn.net/20170320155500674?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、沙盒机制的沙盒结构

1. Documents：保存应用运行时生成的需要持久化的数据，iTunes同步设备时会备份该目录，例如游戏应用的存档等重要数据，保存相对重要的数据
2. tmp：保存应用运行时所需要的临时数据，使用完事后再将相应的文件从该目录删除，应用没有运行时，系统也可能会清楚目录下的文件，iTunes同步设备时不会备份该目录，保存不重要的并且大的数据
3. Library/Caches：保存应用运行时生成需要持久化的数据，iTunes同步设备时不会备份该目录，一般存储体积大，不需要备份的非重要数据
4. Library/Preference：保存应用的所有偏好设置，iOS的Settings应用会在该目录中查找应用的设置信息，iTunes同步设备时会备份该目录，该目录由系统管理，无需我们来管理，通常用来存储一些基本的软件配置信息，比如记住密码等

4、沙盒机制的路径

① 获取沙盒路径
```
NSString *path = NSHomeDirectory();
NSLog(@"%@",path);
```

② 获取bundle路经

```
NSString *bundlePath = [NSBundle mainBundle].bundlePath;
NSLog(@"%@",bundlePath);
```

③ 获取Documents路径

```
NSString *documentPath = nil;
//第一种方式
documentPath = [NSHomeDirectory() stringByAppendingString:@"/Documents"];
//第二种方式
documentPath = [NSHomeDirectory() stringByAppendingPathComponent:@"Documents"];
//第三种方式
//Domains：领域，表示在用户的领域中找到Document路经
documentPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
documentPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, NO) lastObject];
```

这里的第三种方式中参数三的YES和NO表示是否使用"~"代替根目录，我们可以通过打印结果看出区别

```
//YES：展开根目录
/Users/handsomexu/Library/Developer/CoreSimulator/Devices/CF6CF69B-8BF6-4394-BBF8-5D2C087038EF/data/Containers/Data/Application/FE1B9DEB-FC89-4398-A08A-108951F2672B/Documents
//NO：不展开根目录
~/Documents
```

④ 获取caches目录

```
NSString *cache = nil;
cache = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
```

⑤ 获取tmp目录

```
NSString *tem = NSTemporaryDirectory();
```

##**二、Plist**

>数据存储方式有三种

>1. Plist
2. Perference
3. 归档和反归档

Plist文件通常用于储存用户设置，也可以存储必要的数据，相当于一个小数据库，但是不能存储对象

1、Plist存储

```
//创建数据，结尾必须要有一个nil参数，否则会报错
NSArray *arr = [NSArray arrayWithObjects:@"hensen",@"java", nil];
//创建路径
NSString *savePath = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject]stringByAppendingPathComponent:@"data.plist"];
//写进文件
[arr writeToFile:savePath atomically:YES];
```
	
2、Plist读取

```
//创建路径
NSString *savePath = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject]stringByAppendingPathComponent:@"data.plist"];
//获取数据
NSArray *getArr = [NSArray arrayWithContentsOfFile:savePath];
NSString *myName = getArr [0];
NSString *myLaugage = getArr [1];
NSLog(@"name:%@,laugage:%@",myName,myLaugage);
```

##**三、Perference**

保存应用的所有偏好设置，该目录由系统管理，无需我们来管理，系统也为我们提供了API，但是不能存储对象

1、设置Perference
```
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
[defaults setInteger:18 forKey:@"age"];
[defaults setObject:@"hensen" forKey:@"name"];
//同步，否则数据不会保存
[defaults synchronize];
```

2、获取Perference

```
    
NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];
NSInteger *age = [defaults integerForKey:@"age"];
NSString *name = [defaults objectForKey:@"name"];
```

##**四、归档**

归档和反归档其实就是序列化和反序列化，为了弥补Plist，Perference的不能存储对象的缺点，归档和反归档遵循NSCoding协议

1、准备工作

① 创建Model

```
#import <Foundation/Foundation.h>

@interface PersonModel : NSObject
@property(strong,nonatomic)NSString *name;
@property(assign,nonatomic)NSInteger age;
@end
```

② 遵循NSCoding协议

```
#import "PersonModel.h"

@interface PersonModel()<NSCoding>

@end
```

③ 实现NSCoding方法

```
//NSCoding的委托方法
-(instancetype)initWithCoder:(NSCoder *)aDecoder{
    if(self = [super init]){
		//存储对象
        self.name = [aDecoder decodeObjectForKey:@"name"];
        self.age = [aDecoder decodeIntegerForKey:@"age"];
    }
    return self;
}
```

2、归档

```
//创建路径
NSString *filePath =[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
//创建文件
NSString *fileName = [filePath stringByAppendingPathComponent:@"con.plist"];
PersonModel *model = [[PersonModel alloc]init];
model.name = @"hensen";
model.age = 18;
//归档
[NSKeyedArchiver archiveRootObject:model toFile:fileName];
```

##**五、反归档**

反归档其实就是和归档做相反的操作，相当于装包和解包一个道理

1、准备工作

① 创建Model、遵循NSCoding协议

同上

② 实现NSCoding方法

```
-(void)encodeWithCoder:(NSCoder *)aCoder{
    [aCoder encodeObject:self.name forKey:@"name"];
    [aCoder encodeInteger:(self.age) forKey:@"age"];
}
```

2、反归档

```
//创建路径
NSString *filePath =[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) lastObject];
//创建文件
NSString *fileName = [filePath stringByAppendingPathComponent:@"con.plist"];
 //反归档
PersonModel *person = [NSKeyedUnarchiver unarchiveObjectWithFile:fileName];
NSLog(@"name:%@,age:%ld",person.name,(long)person.age);
```

[源码下载](http://download.csdn.net/detail/qq_30379689/9787437)