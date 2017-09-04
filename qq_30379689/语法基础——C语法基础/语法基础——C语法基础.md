[toc]

##**前言**
马上是就到大四的我，面临校招的我倍感压力，而且今年又是安卓的瓶颈期，外面对能力的要求特别的高，但是我看了很多面经之后，发现很多大公司注重的是数据结构与算法，所以我们不得不有两手准备，开始温习数据结构与算法

当然要学习算法和数据结构，那么C语言是必须先学习的，因为大部分算法和数据结构都是以C语言作为使用语言的，所以这篇文章是针对Android程序员的，由于有了Java的基础，所以有一些语法并没有介绍全面

##**常量与变量**

1、常量：C语言常量的声明有两种方式

1. 使用define关键字
2. 使用const关键字

它们的区别是：

1. define是在编译时，会自动将声明的变量替换到程序中
2. const是在运行是，会自动将声明的变量替换到程序中，同时，const可以很容易知道常量的类型

```c
#include<stdio.h>

#define YOU_AGE 23 
const int HIS_AGE = 25;

int main(){

	return 0;
}
```
2、变量

C语言基本数据类型有：short、int、long、char、float、double

```
int a = 10;
char b = 'A';
float c = 25.5;
double d = 25.5;
```
这里可以提前说（char *）这个类型，这个是字符指针，我们可以把它当作Java的String类型

3、自定义变量类型名称

使用typedef关键字可以自定义数据类型的名称


```c
typedef int my_int;

int main(){
	//相当于int b = 10
	my_int b = 10;
	return 0;
}
```

##**流程控制与循环**

1、if、if-else语句

```
int a = 10;
int b = 5;
if(a>b){

}else{

}
```

2、switch语句


```c
switch(表达式){ 
    case 常量表达式1:  语句1;
    case 常量表达式2:  语句2;
    … 
    case 常量表达式n:  语句n;
    default:  语句n+1;
}
```

3、goto语句

goto语句是跳转语句，可以将执行顺序跳转到指定的标签中


```c
//实现1+2+3...+100的和
int i = 1;
loop: if(i<=100){
    sum=sum+i;
    i++;
    goto loop;
}
```

4、for语句


```c
int i,sum;
for( i=1; i<=100; i++ ){  
	sum=sum+i;
}
```

5、while、do-while语句

1. while语句是先判断循环条件之后，再决定执不执行循环体
2. do-while语句是先执行一次循环体，再判断循环条件，决定执不执行循环体


```c
int i=1,sum;
while(i<=100){
	sum=sum+i;
	i++;
}

do{
	sum=sum+i;
	i++;
}while(i<100);
```

##**运算符**

1、算术运算符

<、<=、>、>=、==、!=、赋值运算符（=）

2、逻辑运算符

&&（与）、||（或）、!（非）

3、位运算符

&（位与）、|（位或）、~（位反）、^（异或）、>>（右移）、<<（左移）

4、三目运算符

条件表达式  ?  结果1  :  结果2

5、sizeof运算符

用来计算变量、常量、数据类型所占用存储空间的字节数

6、自增自减运算符
                  
++a，a++，--a，a--

##**输出和输入**

1、输出

```c
//输出字符
char ch = 'a';
putchar(ch);
//输出字符串
puts("hello word");
printf("hello c");
```

2、格式化输出

数据类型|数据格式
--|--|
int|%d
short|%d
long|%ld
float|%f
double|%lf
char|%c
十六进制|%X
八进制|%o
字符串|%s

<br>

```c
//输出整型
printf("number is %d",23);
//输出十六进制
printf("%X",0xFF00FF);
//输出八进制
printf("%o",8);
//输出浮点型
printf("%f",3.14);
//输出字符
printf("%c",'A');
//输出字符串
printf("%s","Hello C");
```
3、输入

```c
//输入字符
char answer = getchar();
printf("%c",answer);
```
4、格式化输入

```c
//输入整型
int a;
scanf("%d",&a);
//输入字符
char c;
scanf("%c",&c);
//输入字符串
char buf[100];
scanf("%s",buf);
```
这里需要注意的是，scanf()函数第二个参数需要一个数据的内存地址，用&符号可以指向一个内存地址，而char[]类型，本身就是个地址，所以不用加&符号

##**数组**

1、一维数组

```c
//创建指定大小一维数组
int arr[10];
//创建自动指定大小一维数组
int arr[] = {5,6,8,9};
```

2、二维数组

```c
int arr[3][4] = {
	{1,2,3,4},
	{5,6,7,8},
	{9,10,11,12}
};
```
3、字符数组

```c
//字符数组的第一种表现形式，其值是可以修改的
char str[10] = {'H','e','l','l','o','/0'};
str[0] = "h";
//字符数组的第二种表现形式，其值是不可以修改的
char *str = "Hello";
//输出字符串
printf("%s",str);
```

这里需要注意字符数组凡是遇到有'/0'出现就判定该字符数组已经到结尾，默认结尾是会自动补上的，否则输出时会出现中文乱码

##**函数**

1、无返回值的函数

```c
void printfStr(){
	printf("Hello World");
}
```

2、有返回值的函数

```c
int add(int a,int b){
	return a+b;
}
```
3、可变参数函数

```c
int sum(int n,...){
	int i;
	int all = 0;
	va_list args;
	va_start(args,n);
	for(i=0;i<n;i++){
		all+=va_arg(args,int);
	}
	va_end(args);
	return all;
}
```

完整代码如下

```c
#include<stdio.h>
#include<stdlib.h>
#include<stdarg.h>
//函数需要声明才能调用
int add(int a,int b);
void printfStr();
int sum(int n,...);

int main(){
	int c = add(2,3);
	int d = sum(1,2,3);
	printfStr();
	return 0;
}
//有返回值的函数
int add(int a,int b){
	return a+b;
}
//无返回值的函数
void printfStr(){
	printf("Hello World");
}
//可变参数函数
int sum(int n,...){
	int i;
	int all = 0;
	va_list args;
	va_start(args,n);
	for(i=0;i<n;i++){
		all+=va_arg(args,int);
	}
	va_end(args);
	return all;
}

```


##**指针**

1、变量指针

指针可以理解为Java里面数组，它指向数据存储的内存地址，默认指针是指向数组的第一个数据

```c
int a = 10;
//定义指针，指向a的存储地址
int *pa = &a;
//获取数据的两种方式，输出都是10
printf("%d",*pa);
printf("%d",pa[0]);
//获取数据的内存地址
printf("%#x",pa);
```

2、无类型指针变量

无类型指针可以指向任何类型的变量

```c
void *str = "hello world";
void *a = 5;

printf("%s",str);
printf("%d",a);
```

3、函数指针

指针指向的是一个内存地址，不仅是变量内存地址，函数内存地址也可以

```c
#include<stdio.h>
#include<stdlib.h>
void say(int a,int b){
	printf("Hello");
}
int main(){
	void(*func_p)(int,int) = &say;
	func_p(0,0);
}
```

4、自定义函数名

使用typedef关键字自定义函数名

```c
#include<stdio.h>
#include<stdlib.h>
void say(int a,int b){
	printf("Hello");
}
typedef void(*Func)(int ,int);
int main(){
	Func method = &say;
	method(0,0);
}
```

5、指针总结

* 指针有类型，地址没有类型
* 地址只是开始的位置，类型表示在什么地方结束
* 空指针默认值为0，但系统不允许访问空指针地址 

##**结构体**

1、结构体

结构体就相当于Java的Bean对象，可以把（struct+结构体名）当作实体对象，进行对象的声明

```c
struct File{
	char *name;
	int size;
};

int main(){
	//第一种赋值方式
	struct File file;
	file.name = "abc.txt";
	file.size = 10;
	//第二种赋值方式
	struct File file = {"abc.txt",10};
}
```

当然，还可以使用typedef关键字将结构体起个别名，使其更像Java的实体对象

```c
struct _File{
	char *name;
	int size;
};

typedef struct _File File;

int main(){
	File file;
	file.name = "abc.txt";
	file.size = 10;
}
```
也有另一种简化的写法，效果是一样的，为了使代码更好看

```c
typedef struct _File{
	char *name;
	int size;
}File;

int main(){
	File file;
	file.name = "abc.txt";
	file.size = 10;
}
```

2、指针结构体

```c
#include<stdio.h>
#include<stdlib.h>
//实体对象
typedef struct{
	char *name;
	int size;
}File，*File_p;
//创建文件
File * createFile(char *name,int size){
	//第一种写法
	File *f = malloc(sizeof(File));
	f->name = name;
	f->size = size;
	return f;
	//第二种写法
	File_p file_p = malloc(sizeof(File_p));
	file_p->name = name;
	file_p->size = size;
	return file_p;
}
//删除文件
void deleteFile(File *file){
	free(file);
}

int main(){
	File *f = createFile("abc.txt",10);
	printf("%s,%d",f->name,f->size);
	deleteFile(f);
}
```

3、嵌套结构体

```c
//第一种写法
struct Teacher{
	char *name;
	int age;
};

struct Student{
	char *name;
	int age;
	//嵌套结构体声明
	struct Teacher t;
};

//第二种写法
struct Student{
	char *name;
	int age;
	//嵌套结构体声明
	struct Teacher{
		char *name;
		int age;
	}t;
};

void getTeacher(){
	struct Student s1;
	s1.age = 10;
	s1.name = "hensen";
	//嵌套结构体赋值
	s1.t.name = "hensen's teacher";
}
```

4、结构体数组

```c
void structArr(){
	struct Teacher ts[] = { {"hensen",21}, {"hensen2",22} };
	//遍历结构体数组
	struct Teacher *t = ts;
	for (; t < ts +2; t++){
		printf("%s,%d\n", t->name, t->age);
	}

	system("pause");
}
```

5、结构体大小（字节对齐）

```c
//大小为16字节，其大小是最宽的基本数据类型的整数倍
//提升读取效率
struct Man{
	int age;
	double weight;
};
```

##**共同体（联合体）**

共同体从字面上可以理解使用共同内存地址的一种结构，即共同体里面的属性所存储的值都是一样的，因为它们的内存地址都指向一个地方（共同体内存地址最大长度 = 所有属性中内存地址长度最大的那个属性）

```c
typedef union _Fmaily{
	char a;
	int b;
}Fmaily;

int main(){
	Fmaily f;
	f.b = 97;
	//输出97的ASCLL值，即a
	printf("%c",f.a);
	//输出97
	printf("%d",f.b);
	return 0;
}
```

##**枚举**

枚举只是用来列举所有情况，枚举默认的值从1开始

```c
enum Day{
	Monday,
	Tuesday,
	Webnesday,
	Thursday,
	Friday,
	Saturday,
	Sunday
};

int main(){
	//枚举的值，必须是括号中的值
	//值为1
	enum Day day = Monday;
}
```

##**文件操作**

1、写文件

```c
//打开一个文件，参数一：文件名，参数二：读写模式（w表示写，r表示读）
FILE * f = fopen("abc.txt","w");
if(f != NULL){
	//写进字符
	fputc('A',f);
	//写进字符串
	fputs("Hello \nWord",f);
	//关闭文件流
	fclose(f);
}
```

2、读文件

通过一串一串的读取

```c
#define _CRT_SECURE_NO_WARNINGS
#include<stdio.h>
#include<stdlib.h>

void main(){
	char *path = "D:\\untitled\\my.py";
	//打开文件
	FILE * file = fopen(path, "r");
	if (file == NULL){
		printf("文件打开失败");
		return;
	}
	//读取文件
	char buff[1024];
	while (fgets(buff, 1024, file)){
		printf("%s",buff);
	}
	//关闭文件流
	fclose(file);
	system("pause");
}
```
通过一个一个字符读取

```c
void readFile(){
	int i;
	//读文件
	FILE * f = fopen("abc.txt", "r");
	if (f != NULL){
		char buf[100];
		//清空buf里面的内容
		memset(buf, 0, 100);
		for (i = 0; i<100; i++){
			//读取字符
			char ch = fgetc(f);
			//如果ch不是End of FIle
			if (ch != EOF){
				buf[i] = ch;
			}
			else{
				break;
			}
		}
		printf("%s", buf);
		fclose(f);
	}
	return 0;
}
```
3、文件的加密和解密

加密

```c
void crpypt(char normal_path[],char crypt_path[]){
	//打开文件
	FILE *normal_fp = fopen(normal_path, "r");
	FILE *crypt_fp = fopen(crypt_path, "w");
	int ch;
	while ((ch = fgetc(normal_fp)) != EOF){ //End of File
		//写入（异或运算）
		fputc(ch ^ 9,crypt_fp);
	}
	//关闭
	fclose(crypt_fp);
	fclose(normal_fp);
}
```
解密

```c
void decrpypt(char crypt_path[], char decrypt_path[]){
	//打开文件
	FILE *normal_fp = fopen(crypt_path, "r");
	FILE *crypt_fp = fopen(decrypt_path, "w");
	int ch;
	while ((ch = fgetc(normal_fp)) != EOF){ //End of File
		//写入（异或运算）
		fputc(ch ^ 9, crypt_fp);
	}
	//关闭
	fclose(crypt_fp);
	fclose(normal_fp);
}
```
使用

```c
void main(){
	char *normal_path = "E:\\dongnao\\vip\\ndk\\08_08_C_05\\files\\friends.txt";
	char *crypt_path = "E:\\dongnao\\vip\\ndk\\08_08_C_05\\files\\friends_crypt.txt";
	char *decrypt_path = "E:\\dongnao\\vip\\ndk\\08_08_C_05\\files\\friends_decrypt.txt";

	//加密
	crpypt(normal_path, crypt_path);
	//解密
	decrpypt(crypt_path, decrypt_path);

	getchar();
}
```
4、二进制文件的复制

计算机文件存储在物理上都是二进制，读写文本文件与二进制文件的区别体现在回车换行符中

* 写文本时，每遇到一个'\n'，会将其转换成'\r\n'
* 读文件时，每遇到一个'\r\n'，会将其转换成'\n'

```c
void copyFile(){
	char *path = "D:\\workspaceweb\\CSDNMovie\\assets\\imgs\\bg.jpg";
	char *new_path = "D:\\workspaceweb\\CSDNMovie\\assets\\imgs\\new_bg.jpg";
	FILE * file = fopen(path, "rb");
	FILE * new_file = fopen(new_path, "wb");
	int buff[1024];
	int len = 0;
	while ((len = fread(buff, sizeof(int), 1024, file)) != 0){
		fwrite(buff, sizeof(int), len, new_file);
	}
	fclose(file);
	fclose(new_file);
}
```

5、获取文件的大小

```c
void readFileSize(){
	char *path = "D:\\workspaceweb\\CSDNMovie\\assets\\imgs\\bg.jpg";
	FILE *file = fopen(path, "r");
	//fseek：移动文件流的读写位置
	//0是偏移量，SEEK_END表示文件末尾
	fseek(file, 0, SEEK_END);
	//ftell：取得文件流的读取位置
	long fileSize = ftell(file);
	printf("%d", fileSize);
}
```

##**内存分配**##

1、C语言内存分配：

1. 栈区（自动分配、释放）：windows下，栈内存分配2M（确定的常数），超出了限制，提示stack overflow错误
2. 堆区（手动分配、释放）：操作系统80%内存
3. 全局区或静态区
4. 字符常量区
5. 程序代码区

2、分配内存的两种方法

```
int len = 10;
int *p = calloc(len,sizeof(int));
int *p = malloc(sizeof(int)* len);
```

3、动态分配内存的注意点

1. 不能多次释放
2. 释放完之后（指针仍然有值），给指针置NULL，标志释放完成
3. 内存泄露（p重新赋值之后，再free，并没有真正释放内存）

4、内存分配例子

```c
void getMalloc(){
	
	int len = 10;
	int *p = malloc(sizeof(int)* len);
	printf("%#x \n", p);

	int i = 0;
	for (; i < len; i++){
		p[i] = rand() % 100;
	}

	int addLen = 5;
	//realloc方法
	//参数一：原来的指针
	//参数二：重新分配的总长度
	//重新分配内存的两种情况：
	//缩小，缩小的那一部分数据会丢失
	//扩大，有以下三种情况
	//1. 如果当前内存段后面有需要的内存空间，直接扩展这段内存空间，realloc返回原指针
	//2. 如果当前内存段后面的空闲字节不够，那么使用堆中的第一个能够满足这一要求的内存块，将目前的
	//数据复制到新的位置，并将原来的数据释放掉，返回新的内存地址
	//3. 如果申请失败，返回NULL，原来的指针仍然有效
	int *p2 = realloc(p, sizeof(int)*(len + addLen));
	printf("%#x", p2);
	if (p2 == NULL){
		printf("重新分配内存失败");
	}

	i = 0;
	for (; i < (len + addLen); i++){
		p[i] = rand() % 100;
	}

	if (p != NULL){
		free(p);
		p == NULL;
	}
	if (p2 != NULL){
		free(p2);
		p2 == NULL;
	}
	getchar();
}
```

##**预编译处理**##

1、define指令作用

* 定义标识
* 定义常数
* 定义宏函数

2、定义标识

```c
//1、标识支持C++语法
#ifdef __cplusplus
//2.1、防止文件重复引入，老版写法
#ifndef AH
#define AH
#include "B.H"

void printA();

#endif
//2.2、防止文件重复引入，新版写法
#pragma once
#include "B.h"

void printA();
```

3、定义常数

```c
#define MAX 100
```

4、定义宏函数

```c
void com_jni_read(){
	printf("read\n");
}

void com_jni_write(){
	printf("write\n");
}
//NAME是参数
#define jni(NAME) com_jni_##NAME();
//使用的时候
void main(){
	jni(write);
}
```

5、定义多参数的宏函数

```c
#define LOG(LEVEL,FORMAT,...) printf(##LEVEL); printf(##FORMAT,__VA_ARGS__);
#define LOG_I(FORMAT,...) LOG("INFO:",##FORMAT,__VA_ARGS__);
#define LOG_E(FORMAT,...) LOG("ERROR:",##FORMAT,__VA_ARGS__);
#define LOG_W(FORMAT,...) LOG("WARN:",##FORMAT,__VA_ARGS__);
//使用的时候
void main(){
	LOG_E("%s%d","大小：",89);
}
```

##**常用例子**##

1、获取0-100随机数

```
void getRandom(){
	//生成随机种子
	int len = 10;
	int i = 0;
	int *p = malloc(sizeof(int)*len);
	srand((unsigned)time(NULL));
	for (; i < len; i++){
		//生成0-100的随机数
		p[i] = rand() % 100;
		printf("%d\n", p[i]);
	}

	if (p!= NULL){
		free(p);
		p = NULL;
	}

	getchar();
}
```

2、操作cmd

```
void openMspaint(){
	system("mspaint");
	system("pause");
}
```

3、注入Dll

```
__declspec(dllexport) void injectDll(){
	//属性（项目右键）-> 常规（配置类型）-> .dll类型
	//生成（选择栏）-> 生成解决方案
	//使用DLLInject工具
	int *p = 0x2ff9d8;
	*p = 99999;
}
```

基本的C语言基础就已经复习完了，更多可以查询API进行学习，这里提供[C语言函数速查文档下载](http://download.csdn.net/detail/qq_30379689/9634580)