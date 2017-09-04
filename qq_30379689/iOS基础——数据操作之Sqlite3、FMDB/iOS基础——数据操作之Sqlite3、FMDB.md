#iOS基础——数据操作之Sqlite3、FMDB


----------
##**前言**

* iOS数据库操作可分为Sqlite3、CoreData、FMDB第三方库
* sqlite3在不使用的时候需要close，在需要的使用的时候重新open

##**一、Sqlite3**

数据库操作无非就是Sql语句的书写，最常见的就是增删改查，通过Sqlite3实现我们简单的数据存储

1、导入Sqlite3依赖库

在项目的设置文件中找到Link Binary With Libraries，点击左下角加号

![这里写图片描述](http://img.blog.csdn.net/20170315230818003?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

输入我们需要的Sqlite3库，点击添加

![这里写图片描述](http://img.blog.csdn.net/20170315230927270?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、准备工作

导入Sqlite3依赖库

```
#import "ViewController.h"
#import <sqlite3.h>
```

声明变量

```
@interface ViewController ()
@property (assign,nonatomic)sqlite3 *db;
@end
```

在布局中添加四个按钮

![这里写图片描述](http://img.blog.csdn.net/20170315230746304?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、创建数据库、表

```
- (void)viewDidLoad {
    [super viewDidLoad];
    //1、创建数据库
    //获取app的路径，并添加数据库名
    NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject]stringByAppendingPathComponent:@"person.sqlite"];
    NSLog(@"%@",path);
    //打开数据库，如果没有会自动生成，如果有就打开
    int success = sqlite3_open(path.UTF8String, &_db);
    //判断是否执行成功
    if(success == SQLITE_OK){
        NSLog(@"创建数据库成功");
        //2、创建表
        //sql语句
        NSString *sql = [NSString stringWithFormat:@"create table if not exists t_person(id integer primary key autoincrement,name text not null,age integer default 10)"];
        //执行sql语句
        int successExec = sqlite3_exec(_db, sql.UTF8String, NULL, NULL, NULL);
        if(successExec == SQLITE_OK){
            NSLog(@"创建表成功");
        }else{
            NSLog(@"创建表失败");
        }
    }else{
        NSLog(@"创建数据库失败");
    }
}
```



4、sqlite增删改查

```
#pragma 添加数据
- (IBAction)addData:(id)sender {
    //sql语句
    NSString *sql = [NSString stringWithFormat:@"insert into t_person(name,age) values('zhangsan',15)"];
    //执行sql语句
    int successExec = sqlite3_exec(_db, sql.UTF8String, NULL, NULL, NULL);
    if(successExec == SQLITE_OK){
        NSLog(@"插入成功");
    }else{
        NSLog(@"插入失败");
    }
}

#pragma 删除数据
- (IBAction)deleteData:(id)sender {
    //sql语句
    NSString *sql = [NSString stringWithFormat:@"delete from t_person where age>5"];
    //执行sql语句
    int successExec = sqlite3_exec(_db, sql.UTF8String, NULL, NULL, NULL);
    if(successExec == SQLITE_OK){
        NSLog(@"删除成功");
    }else{
        NSLog(@"删除失败");
    }
}

#pragma 更新数据
- (IBAction)updateData:(id)sender {
    //sql语句
    NSString *sql = [NSString stringWithFormat:@"update t_person set name='lisi' where age>5"];
    //执行sql语句
    int successExec = sqlite3_exec(_db, sql.UTF8String, NULL, NULL, NULL);
    if(successExec == SQLITE_OK){
        NSLog(@"更新成功");
    }else{
        NSLog(@"更新失败");
    }
}

#pragma 查询数据库
- (IBAction)queryData:(id)sender {
    //sql语句
    NSString *sql = [NSString stringWithFormat:@"select name,age from t_person"];
    //执行sql语句
    sqlite3_stmt *stmt = nil;
    sqlite3_prepare_v2(_db, sql.UTF8String, -1, &stmt, NULL);
    //读取数据
    while(sqlite3_step(stmt) ==SQLITE_ROW){
        const unsigned char * name = sqlite3_column_text(stmt, 0);
        int age = sqlite3_column_int(stmt, 1);
        NSLog(@"%@", [NSString stringWithFormat:@"姓名：%s，年龄：%d",name,age]);
    }
}
```

查看Log输出

```
2017-03-15 23:12:21.860 DataBaseDemo[1650:109872] 创建数据库成功
2017-03-15 23:12:21.862 DataBaseDemo[1650:109872] 创建表成功
2017-03-15 23:12:22.664 DataBaseDemo[1650:109872] 插入成功
2017-03-15 23:12:23.662 DataBaseDemo[1650:109872] 姓名：zhangsan，年龄：15
2017-03-15 23:12:25.377 DataBaseDemo[1650:109872] 更新成功
2017-03-15 23:12:26.195 DataBaseDemo[1650:109872] 姓名：lisi，年龄：15
2017-03-15 23:12:27.750 DataBaseDemo[1650:109872] 删除成功
```

当然也可以通过打印出来的Path值，打开对应的文件夹，找到我们的数据库，可以通过Navicat Premium查看数据

![这里写图片描述](http://img.blog.csdn.net/20170315231451701?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**二、FMDB**

fmdb是第三方库用来简化sqlite3操作，这里会介绍FMDB的增删改查、FMDB线程安全操作、FMDB事务操作

1、下载fmdb，复制fmdb到工程目录中

![这里写图片描述](http://img.blog.csdn.net/20170316143709898?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、准备工作

导入依赖

```
#import "FMDB.h"
```
声明变量

```
@property (strong,nonatomic) FMDatabase *database;
```

3、创建数据库、表

```
NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject] stringByAppendingPathComponent:@"datafmdb.sqlite"];

FMDatabase *database = [FMDatabase databaseWithPath:path];
self.database = database;
BOOL success =  [database open];
if (success) {
    NSLog(@"创建数据库成功");
    //非查询语句都是用executeUpdate
    BOOL successT=  [self.database executeUpdate:@"create table if not exists t_person(id integer primary key autoincrement,name text not null,age integer default 10);"];
    if (successT) {
        NSLog(@"创建表成功");
    }else{
        NSLog(@"创建表失败");
    }
}else{
    NSLog(@"创建失败");
}
```

4、FMDB增删改查

```
#pragma 添加数据
- (IBAction)addData:(id)sender { 
    NSString *sql = [NSString stringWithFormat:@"insert into t_person(name,age) values('zhangsan',15)"];
    BOOL success =  [self.database executeUpdate:sql];
    if (success) {
        NSLog(@"添加数据成功");
    }else{
        NSLog(@"添加数据失败");
    }
}

#pragma 删除数据
- (IBAction)deleteData:(id)sender {
    NSString *sql = [NSString stringWithFormat:@"delete from t_person where age>5"];
    BOOL success =  [self.database executeUpdate:sql];
    if (success) {
        NSLog(@"删除成功");
    }else{
        NSLog(@"删除失败");
    }
}

#pragma 更新数据
- (IBAction)updateData:(id)sender {
    NSString *sql = [NSString stringWithFormat:@"update t_person set name='lisi' where age>5"];
    BOOL success =  [self.database executeUpdate:sql];
    if (success) {
        NSLog(@"删除成功");
    }else{
        NSLog(@"删除失败");
    }
}

#pragma 查询数据库
- (IBAction)queryData:(id)sender {
    NSString *sql = [NSString stringWithFormat:@"select name,age from t_person"];
    FMResultSet *result = [self.database executeQuery:sql];
    while ([result next]) {
        NSString *name =   [result stringForColumnIndex:0];
        int age =  [result intForColumnIndex:1];
        NSLog(@"%@", [NSString stringWithFormat:@"姓名：%@，年龄：%d",name,age]);
    }
}
```

5、线程安全、block写法

FMDB提供了线程安全的写法，使用FMDatabaseQueue的 inDatabase方法即可对参数db进行数据库操作
```
NSString *path = [[NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES)lastObject] stringByAppendingPathComponent:@"person.sqlite"];
FMDatabaseQueue *queue = [FMDatabaseQueue databaseQueueWithPath:path];
self.queue = queue;
//执行数据库操作
[self.queue inDatabase:^(FMDatabase *db) { 
	//在block 内部的就是线程安全
    BOOL success =  [db open];
    if (success) {
        NSLog(@"创建数据库成功!");
    }else{
        NSLog(@"创建失败!");
    }
}];
```
其他操作也通过block写法进行
```
[self.queue inDatabase:^(FMDatabase *db) {
    NSString *sql = @"SELECT  * FROM t_person;";
    FMResultSet *result = [db executeQuery:sql];
    while ([result next]) {
        
    }
}];
```

6、事务操作

开启事务到结束事务很简单

```
[self.queue inDatabase:^(FMDatabase *db) {
	[db beginTransaction];
	
	//这里放需要事务的代码

	[db commit];
}];
```

可以通过[db rollback]进行数据回滚，或者通过[db inTransaction]判断是否在事务中

[源码下载](http://download.csdn.net/detail/qq_30379689/9783327)
