[toc]

##**前言**

学习过Swift之后，好久没用已经生疏了，现在有项目来了，不得不停下手下的工作，开始学习OC，当然这篇文章会以Java基础和C基础作为支撑，这样学习起来入门很简单，可能这篇文章有点个人主义，用作个人笔记吧

##**OC特点**

* 支持C语法
* 支持面向对象特性
* 兼容性好，可以同时在项目中使用OC、C++，也可以引入C、C++库文件
* OC中没有命名空间机制，也没有包的概念，为了区分不同的类，在类名加前缀
* OC中关键字表示都以@开头，用于区分C和C++的关键字，字符串也以@开头，如@public、@protected、@private、@"Hello World"

##**文件名后缀**
|语言|头文件后缀|主文件后缀|
|-
|c|.h|.c|
|c++|.h|.cpp|
|oc|.h|.m|
|oc与c++|.h|.mm|



##**OC基本数据类型**
基本数据类型：数值型（整型、浮点型），字符型（char）、布尔型（BOOL）、空类型（Void）、Block类型、指针数据类型（class、id类型）、特殊类型（SEL、nil）

① instancetype和id类型

1. instancetype可以作为返回类型，可以返回和方法所在类相同类型的对象，只能作为返回值
2. id可以作为返回类型，返回任意类型的objcetive-c的对象类型，既可以作为返回值也可以作为参数


##**类与属性**

①  类的声明语法
```oc
@interface Person : NSObject{
	属性声明
}
	方法声明
@end
```
特点：

1. @interface：相当于Java中的class
2. 类名后面的冒号：相当于Java中的extends

② 类的实现语法
```oc
@implementation XYZPerson
	方法实现{
		
	}
@end
```
③ 完整例子

![](http://img.blog.csdn.net/20170221000410616?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

④ 创建对象

```oc
//第一种创建方式：创建一个可用的对象
Person *p=[Person new];
//new方法的内部会分别调用两个方法来完成2件事情
//1、使用alloc方法来分配存储空间（返回分配的对象）
//2、使用init方法来对对象进行初始化

//1、调用类方法+alloc分配存储空间，返回未经初始化的对象
Person *p1=[person  alloc];
//2、调用对象方法-init进行初始化，返回对象本身
Person *p2=[p1 init];
//第二种创建方式：以上2步简化为
Person *p=[[Person alloc] init];

//默认初始化完毕后，所有成员变量的值都为0
//用完对象要释放内存
[lisi release];
```
究竟new方式和alloc/init方式有什么区别？

1. new和alloc/init在功能上是一致的，分配内存并完成初始化
2. 采用new方式只能采用默认的init方法完成初始化
3. 采用alloc的方式可以用自定义的构造方法完成初始化

⑤ 访问对象属性
```
//调用类的属性，语法：对象->属性名
lisi->age;
lisi->name;
//或者：(*对象名).对象属性
(*lisi).name
```
⑥ 调用方法

```oc
//调用无参方法，语法:[对象名 方法名]
[Person eat]
[Person run]
//调用有参方法，语法:[对象名 方法名:参数]
[Person eat:@"吃虾"]
[Person run:5]
```

⑦ @property和@synthesize

在头文件中声明属性

```
//编译器会自动补出其set和get方法
@property int age;
```

在m文件中实现

```
//编译器会自动生成set和get方法的实现，默认已经自动@synthesize属性的值
@synthesize age;
```

##**构造方法**
① 默认的构造方法
```oc
- (instancetype)init{
	if(slef = [super init]){
		//初始化
	}
	return self;
}
```
② 自定义构造方法

一般构造方法都以init开头，这样并不是复写系统默认的构造方法，而是再添加一个构造方法
```
- (instancetype)initWithDict:(NSDictionary *)dict{
	if(slef = [super init]){
		//初始化并赋值
		self.name = dict[@"name"];
	}
	return self;
}
```

##**函数**

① 减号表示对象方法，语法：-（返回值类型）方法名：（参数类型）参数名

```
@interface Person : NSObject
-(Void) someMethod;
-(Void) someMethodWithValue:(someType)Value;
@end
```
特点：

1. 函数的返回类型和参数类型必须用括号
2. 函数的形参用冒号表示

② 加号表示静态方法（类方法），语法：+（返回值类型）方法名：（参数类型）参数名

```oc
@interface NSString: NSObject
+(id) string;
+(id) stringWithString:(NSString *)astring;
+(id) stringWithFormat:(NSString *)format,...;
@end
```

区别：

1. 对象方法可以直接访问属性，静态方法不能访问属性
2. 静态方法可用类名直接调用方法，而对象方法不行

##**点语法**
声明对象并提供get和set方法

```
#import <Foundation/Foundation.h>

@interface Person : NSObject{
   int _age;
}
- (void)setAge:(int)age;
- (int)age;

@end
```
实现对象

```
#import "Person.h"
  
@implementation Person

- (void)setAge:(int)age{   //set方法
    _age = age;
}
- (int)age{   //get方法
   return _age;
}

@end
```
点语法的使用

```
#import <Foundation/Foundation.h>
#import "Person.h"

int main(int argc, const char * argv[]){
    @autoreleasepool {    
	    Person *person = [[Person alloc] init];

	    person.age = 10;//点语法，等效与[person setAge:10];
	    //这里并不是给person的属性赋值，而是调用person的setAge方法
	     
	    int age = person.age;//等效与int age = [person age];
	    //这里调用person的set方法
	    
	    NSLog(@"age is %i", age);
	    [person release];        
    }
    return 0;
}
```
为什么有些使用_age，有些语法使用self.age？

属性值：_age
点语法：self.age

1. 下划线是一种编程规范，_age是直接拿到age这个属性，不通过getAge方法获取，建议在init里面使用_age
2. self.age相当于调用[self getAge]，系统自动生成的代码(_age = age)，其实也是_age，如果你重写了getAge方法，返回的值就不一样了


##**静态变量**

① OC与Java或C的区别

1. OC中的静态变量：它是私有的全局变量，不能直接通过类名访问，它只作用于它声明所在的.m文件中
2. C或者Java中的静态变量：可以直接通过类名拿到这个变量，在其他类中可以进行修改

② 静态变量全局化

不是说OC中静态变量不能使用在其他类中，可以通过静态方法向外提供该静态变量

在h文件中定义静态变量和向外提供静态变量的方法

```
#import <UIKit/UIKit.h>
//静态变量
static int selectIndex;

@interface TBC_SendPassword : UITabBarController
//提供静态变量
+(int)getSelectIndex;
@end
```

在m文件中覆写该方法，提供静态变量，你可以在m文件中修改对应的selectIndex的值

```
+(int)getSelectIndex{
    return selectIndex;
}
```

在其他类中需要使用该类的静态变量，首先在h文件中导入头文件

```
#import "TBC_SendPassword.h"
```

在m文件中直接通过静态的方法的使用，获取对应的值

```
if([TBC_SendPassword getSelectIndex] != 0){
    return;
}
```

##**Hello World**

相当于Java的main方法

```
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]){
	@autoreleasepool{
		NSLog(@"Hello World");
	}
	return 0;
}
```
特点：

1. Foundation.h是Foundation框架中的头文件，是OC一个基础类库，基本所有OC代码都要引入这个类库
2. @autoreleasepool是OC的关键字，作用是对在关键字后面的程序自动进行内存回收

##**import指令**

* #include指令：单独使用会造成导入重复库文件，从而引起报错
* #import指令：#include增强版，有效处理重复导入问题，不会报错

特点：

1. 导入系统类库用<>，导入自定义类库用””
2. BOOL类型，包含两个值YES和NO，其实布尔类型就是整数1和0

##**NSLog**

```oc
//定义一个字符串，str存放是内存地址
NSString *str = "你好";
NSLog(@"str的地址=%p,str的值=%@",str,str);
//打印OC字符串要用@""
NSLog(@"Hello OC");
//用C语言打印字符串
printf("Hello OC");
```

注意不能用C语言打印OC字符串，也不能用OC语言打印C字符串


##**NSString**

```
//创建字符串
NSString * str = @"Hello";
//字符串长度，OC中英文都占1个字节
[str length]
//创建字符串对象
NSString * str1 = [NSString new]
//复制字符串
NSString * str2 = [[NSString malloc]initWithString:str];
//格式化字符串
NSString * str3 = [NSString  stringWithFormat:@"图片 xx %02d- %02d",1,9];
//字符串拼接
NSString *newString = [NSString stringWithFormat:@"%@%@",str1,str2];
```

##**NSMutableArray**

NSMutableArray是长度可变数组，也就是说当数组前面的元素被删除后，后面的元素会往前移，而NSArray是长度不可变数组

```
//创建NSMutableArray
NSMutableArray *mArray = [NSMutableArray array]; 
//添加元素
[mArray addObject:@"name"];
//移除元素
[mArray removeObject:@"name"];
//移除所有元素
[mArray removeAllObjects];
//取出元素
mArray[2];

```

##**强制类型转换**

```
//字符转int
int intString = [newString intValue];
//int转字符
NSString *stringInt = [NSString stringWithFormat:@"%d",intString];
//字符转float
float floatString = [newString floatValue];
//float转字符
NSString *stringFloat = [NSString stringWithFormat:@"%f",intString];

```


##**NSBundle**

NSBundle是一个目录，其中包含了程序会使用到的资源，相当于项目中的主目录文件夹，NSBundle用得最多的是获取Plist文件，并获取Plist的内容，Plist文件可以简单理解为小数据库类型的文件，它在项目初始化时默认会创建一个info.plist文件，里面会存储系统的默认配置信息
```
//打印沙盒路径
NSLog(NSHomeDirectory());
//创建NSBundle
NSBundle *bundle = [NSBundle mainBundle];
//获取NSBundle里myPic.png文件的路径的两种方法
NSString *path = [mainBundle pathForResource:@"myPic" ofType:@".png"];
NSString *path = [mainBundle pathForResource:@"myPic.png" ofType:nil];
//获取NSBundle里的Plist文件
NSString *plistPath = [mainBundle pathForResource:@"myList.plist" ofType:nil];
//通过Plist路径获取字典
NSDictionary *dict = [NSDictionary dictionaryWithContentsOfFile:plistPath];
//通过Plist路径获取数组
NSArray *array = [NSArray arrayWithContentsOfFile:plistPath];
```

##**kvc**

OC中的KVC操作就和Java中使用反射机制去访问类中private权限的变量一样

1、kvc与对象

① 单个对象赋值：通过kvc的方式，可以省去自动解包和装包操作

```java
//通过kvc方式设置属性
Person *p = [[Person alloc] init];
[p setValue:@"jiangwei" forKey:@"name"];
[p setValue:@22 forKey:@"age"];
//通过kvc方式读取属性
NSString *name = [p valueForKey:@"name"];        
```
② 单个对象对对象赋值：对象中包含另一个对象

```java
Book *book = [[Book alloc] init];
Author *author = [[Author alloc] init];   
//通过kvc方式设置对象
[book setValue:author forKey:@"author"];
```
③ 单个对象对数组赋值：对象中存在数组属性，数组装的是另一个对象

```java 
//author中的issueBook字段是个NSArray
//通过kvc方式获取数组的大小，语法：NSArray.@count
NSNumber *count = [author valueForKeyPath:@"issueBook.@count"];
//通过kvc方式获取书籍价格的总和，语法：NSArray.@sum+字段名
NSNumber *sum = [author valueForKeyPath:@"issueBook.@sum.price"];
//通过kvc方式获取书籍的平均值，语法：NSArray.@avg+字段名
NSNumber *avg = [author valueForKeyPath:@"issueBook.@avg.price"];
//通过kvc方式获取书籍的价格最大值和最小值，语法：NSArray.@max+字段名
NSNumber *max = [author valueForKeyPath:@"issueBook.@max.price"];
NSNumber *min = [author valueForKeyPath:@"issueBook.@min.price"];
```

2、kvc与字典

① 通过获取字典字段，将字典转换为属性
```
self.name = dict[@"name"];
self.icon = dict[@"icon"];
```

② 通过kvc方式进行赋值，自动将字段与属性匹配，前提是字段名和字典名必须相同

```
[self setValuesForKeysWithDictionary:dict];
```


##**日期格式化**

```
//取出当前时间 
NSDate *currentDate = [NSDate date];
//设置时间格式
NSDateFormatter *formatter = [[NSDateFormatter alloc]init];
//时间的格式
formatter.dateFormat = @"yyyy-MM-dd HH:mm:ss";
//获取时间的字符串
NSString *date = [formatter stringFromDate:currentDate];
```