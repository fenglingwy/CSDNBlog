[toc]

##**前言**

最近发现要学习C++来开发NDK，不得不把基础的东西记录下来，否则学的太多会混淆，废话不多说，开始记录我的C++学习之旅吧

##**HelloWord**

1. 导库
2. 命名空间
3. 输出函数

```c
#include <iostream>
//必须带有命名空间才能使用cout等
using namespace std;
int main()
{
    cout << "Hello, world!" << endl;
    return 0;
}
```

##**命名空间**

1、命名空间属性访问和结构体访问

```c
namespace NSP_A{
	int a = 9;
	struct Student{
		char name[20];
		int age;
	};
}

void main(){
	//使用命名空间
	cout << NSP_A::a << endl;

	//使用命名空间中的结构体
	using NSP_A::Student;
	Student t;
}
```

2、命名空间的嵌套

```c
namespace NSP_B{
	//命名空间嵌套
	namespace NSP_C{
		int c = 90;
	}
}

void main(){
	cout << NSP_B::NSP_C::c << endl;
}
```


##**类**

1、类、属性、方法的声明与使用

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdlib.h>
#include <iostream>
#include <stdarg.h>
using namespace std;
#define PI 3.14
class MyCircle{
//属性（共用权限访问修饰符）
private:
	double r;
	double s;
public:
	void setR(double r){
		this->r = r;
	}
	//获取面积
	double getS(){
		return PI * r * r;
	}
};

void main(){
	MyCircle c;
	c.setR(4);

	cout << "圆的面积：" << c.getS() << endl;
	system("pause");
}
```

2、类的大小

类的大小只计算普通属性的大小，其他都不包括在内

```c
class A{
public:
	int i;
	int j;
	int k;
	static int m;
};
class B{
public:
	int i;
	int j;
	int k;
	void myprintf(){
		cout << "打印" << endl;
	}
};
void main(){
	//输出的结果都是12
	cout << sizeof(A) << endl;
	cout << sizeof(B) << endl;
}
```

##**继承**

1、继承基本使用

```c
class Human{
public:
	void say(){
		cout << "说话" << endl;
	}
protected:
	int age;	
};
class Man : public Human{

};

void main(){
	//子类
	Man m1;
	m1.say();
	//将子类赋值给父类的引用或指针
	Human* h_p = &m1;
	h_p->say();
	Human &h1 = m1;
	h1.say();
	//子类对象初始化父类类型的对象
	Human h2 = m1;
	h2.say();
	//子类对象调用父类的成员
	m1.Human::say();
	m1.Human::age = 10;
}
```

2、向父类构造方法传参

```c
class Human{
public:
	Human(char* name, int age){
		this->name = name;
		this->age = age;
	}
protected:
	char* name;
	int age;
};
class Man : public Human{
public:
	//给父类构造函数传参，同时给属性h对象赋值
	Man(char *brother, char *s_name, int s_age, char *h_name, int h_age) : Human(s_name, s_age), h(h_name,h_age){
		this->brother = brother;
	}
private:
	Human h;
};

void main(){
	Man m1("Hensen","HensenBoy",18,"HensenGirl",18);
}
```

3、多继承的实现

```c
class Person{

};

class Citizen{

};

class Student : public Person, public Citizen{

};
```

4、继承间的访问修饰符

```c
基类中        继承方式        子类中
public     ＆ public继承 = > public
public     ＆ protected继承 = > protected
public     ＆ private继承 = > private

protected  ＆ public继承 = > protected
protected  ＆ protected继承 = > protected
protected  ＆ private继承 = > private

private    ＆ public继承 = > 子类无权访问
private    ＆ protected继承 = > 子类无权访问
private    ＆ private继承 = > 子类无权访问
```

5、继承的二义性

virtual：表示虚继承，不同路径继承来的同名成员只有一份拷贝，解决不明确的问题

```c
class A{
public:
	char* name;
};
//这里面加了virtual关键字
class A1 : virtual public A{
	
};
//这里面加了virtual关键字
class A2 : virtual public A{

};
class B : public A1, public A2{

};

void main(){
	B b;	
	//如果程序不加virtual关键字就会导致二义性，系统无法辨识哪个类的name属性，会报错
	b.name = "Hensen";
	//这里通过指定父类显示调用是可以的
	b.A1::name = "Hensen";
	b.A2::name = "Hensen";
}
```

##**多态**

1、多态的基本使用

```c
//Plane.h文件
#pragma once
//普通飞机
class Plane{
public:
	virtual void fly();
	virtual void land();
};
//Plane.cpp文件
#include "Plane.h"
#include <iostream>
using namespace std;
void Plane::fly(){
	cout << "起飞" << endl;
}
void Plane::land(){
	cout << "着陆" << endl;
}
```

```c
//Jet.h文件
#pragma once
#include "Plane.h"
//直升飞机
class Jet : public Plane{
	virtual void fly();
	virtual void land();
};
//Jet.cpp文件
#include "Jet.h"
#include <iostream>
using namespace std;
void Jet::fly(){
	cout << "直升飞机在原地起飞..." << endl;
}
void Jet::land(){
	cout << "直升飞机降落在女神的屋顶..." << endl;
}
```

```c
#include "Plane.h"
#include "Jet.h"
#include "Copter.h"
//业务函数
void bizPlay(Plane& p){
	p.fly();
	p.land();
}
void main(){
	Plane p1;
	bizPlay(p1);
	//直升飞机
	Jet p2;
	bizPlay(p2);
}
```

##**引用**

1、引用的使用

```c
void main(){
	int a = 10;
	//&是C++中的引用，引用：变量的另外一个别名，共用同个地址
	int &b = a;	
	cout << b << endl;
}
```
2、引用与指针的区别

* 不存在空引用，引用必须连接到一块合法的内存。
* 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象。
* 引用必须在创建时被初始化。指针可以在任何时间被初始化。

3、引用与指针写法上的差异

```c
struct Teacher{
	char* name;
	int age;
};
//带有结构体指针的写法
void myprint(Teacher *t){
	cout << t->name << "," << t->age << endl;	
	//(*t).name 
}
//带有结构体引用的写法
void myprint2(Teacher &t){
	cout << t.name << "," << t.age << endl;
	t.age = 21;
}
//指针值交换
void swap_1(int *a, int *b){
	int c = 0;
	c = *a;
	*a = *b;
	*b = c;
}
//引用值交换
void swap_2(int &a, int &b){
	int c = 0;
	c = a;
	a = b;
	b = c;
}
void main(){
	Teacher t;
	t.name = "Hensen";
	t.age = 20;
	//指针的写法
	myprint(&t);
	//引用的写法
	myprint2(t);

	int x = 10;
	int y = 20;
	//指针的写法
	swap_1(&x, &y);
	//引用的写法
	swap_2(x,y);
}
```

4、引用的作用

* 把引用作为参数：C++支持把引用作为参数传给函数，这比传一般的参数更安全
* 把引用作为返回值：可以从C++函数中返回引用，就像返回其他数据类型一样

5、指针引用，代替二级指针

```c
struct Teacher{
	char* name;
	int age;
};
//引用的写法
void getTeacher(Teacher* &p){
	p = (Teacher*)malloc(sizeof(Teacher));
	p->age = 20;
}
//二级指针的写法，原本应该这样写，但是已经被引用的写法代替了
void getTeacher(Teacher **p){
	Teacher *tmp = (Teacher*)malloc(sizeof(Teacher));
	tmp->age = 20;
	*p = tmp;
}

void main(){
	Teacher *t = NULL;
	//传递引用的指针t，相当于二级指针
	getTeacher(&t);
}
```

6、常引用，类似于java中final

```c
//常引用在方法中的引用
void myprint(const int &a){
	cout << a << endl;	
}

void main(){	
	//引用必须要有值，不能为空，下面写法是错误的
	//const int a;
	//int &a = NULL;

	//常引用属性使用一
	int a = 10, b = 9;
	const int &c = a;
	//常引用属性使用二
	const int &d = 70;
}
```

7、引用与指针的大小

```c
struct Teacher{
	char name[20];
	int age;
};

void main(){
	Teacher t;
	//引用
	Teacher &t1 = t;
	//指针
	Teacher *p = &t;

	//结果是24，引用相当于变量的别名
	cout << sizeof(t1) << endl;
	//结果是4，指针只是存放的地址
	cout << sizeof(p) << endl;
	system("pause");
}
```

##**异常**

1、C++异常处理，会根据抛出的异常数据类型，进入到相应的catch块中

```c
void main(){
	try{
		int age = 300;
		if (age > 200){
			throw 9.8;
		}
	}
	catch (int a){
		cout << "int异常" << endl;
	}
	catch (char* b){
		cout << b << endl;
	}
	catch (...){
		cout << "未知异常" << endl;
	}
}
```

2、向下抛出异常

```c
void mydiv(int a, int b){
	if (b == 0){
		throw "除数为零";
	}
}
void func(){
	try{
		mydiv(8, 0);
	}
	catch (char* a){
		throw a;
	}
}
void main(){
	try{
		func();
	}
	catch (char* a){
		cout << a << endl;
	}
}
```

3、抛出对象

```c
class MyException{
	
};
void mydiv(int a, int b){
	if (b == 0){
		throw MyException();
		//不要抛出异常指针
		//throw new MyException; 		
	}
}
void main(){
	try{
		mydiv(8,0);
	}
	catch (MyException& e2){
		cout << "MyException引用" << endl;
	}
	catch (MyException* e1){
		cout << "MyException指针" << endl;		
		//指针的话需要手动释放内存
		delete e1;
	}
}
```

4、方法抛出异常

```c
void mydiv(int a, int b) throw (char*, int) {
	if (b == 0){
		throw "除数为零";	
	}
}
```

5、标准异常

```c
//需要导入标准异常的依赖库
#include<stdexcept>

class NullPointerException : public exception{
public:
	NullPointerException(char* msg) : exception(msg){

	}
};
void mydiv(int a, int b){
	if (b > 10){
		throw out_of_range("超出范围");		
	}	
	else if (b == NULL){
		throw NullPointerException("为空");
	}
	else if (b == 0){
		throw invalid_argument("参数不合法");
	}
}
void main(){
	try{
		mydiv(8,NULL);
	}
	catch (out_of_range e1){
		cout << e1.what() << endl;
	}
	catch (NullPointerException& e2){
		cout << e2.what() << endl;
	}
	catch (...){
		cout << "未知异常" << endl;
	}
}
```

6、外部类异常

```c
class Err{
public:
	class MyException{
		public:MyException(){

		}
	};
};
void mydiv(int a, int b){
	if (b > 10){
		throw Err::MyException();
	}
}
```

##**IO流**

1、普通文件

```c
#include <iostream>
#include <fstream>
void main(){
	char* fname = "c://dest.txt";
	//输出流
	ofstream fout(fname);
	//创建失败
	if (fout.bad()){ 
		return;
	}
	//写入文件
	fout << "Hensen" << endl;
	//关闭输出流
	fout.close();

	//输入流
	ifstream fin(fname);
	//创建失败
	if (fin.bad()){
		return;
	}
	char ch;
	//读取数据
	while (fin.get(ch)){
		cout << ch;
	}
	//关闭输入流
	fin.close();
}
```

2、二进制文件

```c
void main(){
	char* src = "c://src.jpg";
	char* dest = "c://dest.jpg";

	//输入流
	ifstream fin(src, ios::binary);
	//输出流
	ofstream fout(dest, ios::binary);

	if (fin.bad() || fout.bad()){
		return;
	}
	while (!fin.eof()){
		//读取
		char buff[1024] = {0};
		fin.read(buff,1024);
		//写入
		fout.write(buff,1024);
	}
	//关闭
	fin.close();
	fout.close();
}
```

3、C++对象的持久化

```c
class Person
{
public:
	Person()
	{
	}
	Person(char * name, int age)
	{
		this->name = name;
		this->age = age;
	}
	void print()
	{
		cout << name << "," << age << endl;
	}
private:
	char * name;
	int age;
};
void main()
{
	Person p1("Hensen", 22);
	Person p2("HensenBoy", 18);
	//输出流
	ofstream fout("c://c_obj.data", ios::binary);
	fout.write((char*)(&p1), sizeof(Person));
	fout.write((char*)(&p2), sizeof(Person));
	fout.close();
	//输入流
	ifstream fin("c://c_obj.data", ios::binary);
	Person tmp;
	fin.read((char*)(&tmp), sizeof(Person));
	tmp.print();
	fin.read((char*)(&tmp), sizeof(Person));
	tmp.print();
}
```

##**函数**

1、函数的重载

函数可以传默认值参数，如果调用存在参数，则默认参数会被替代

```c
void myprint(int x, int y = 9, int z = 8){
	cout << x << endl;
}
//重载
void myprint(int x,bool ret){
	cout << x << endl;
}

void main(){
	myprint(20);
	//覆盖默认参数的9和8
	myprint(20,10,5);
}
```

2、可变参数函数

```c
void func(int i,...)
{
	//可变参数指针
	va_list args_p;
	//可变参数设置开始位置
	//i表示可变参数的前一位参数，从i开始，后面的就是可变参数
	va_start(args_p,i);
	int a = va_arg(args_p,int);
	char b = va_arg(args_p, char);
	int c = va_arg(args_p, int);
	cout << a << endl;
	cout << b << endl;
	cout << c << endl;
	//可变参数设置结束
	va_end(args_p);
}
void main(){
	//使用
	func(9,20,'b',30);
}
```

可变参数也可以循环读取，但是必须保证所有数大于0使用

```c
void func(int i,...)
{
	va_list args_p;
	//开始
	va_start(args_p,i);
	int value;
	while (1){
		value = va_arg(args_p,int);
		if (value <= 0){
			break;
		}
		cout << value << endl;
	}
	//结束
	va_end(args_p);
}
```

3、静态属性和方法

```c
class Teacher{
public:
	char* name;
	//计数器
	static int total;
public:
	Teacher(char* name){
		this->name = name;		
		cout << "Teacher有参构造函数" << endl;
	}
	void setName(char* name){
		this->name = name;
	}
	char* getName(){
		return this->name;
	}
	//计数，静态函数
	static void count(){
		total++;		
		cout << total << endl;
	}
};
//静态属性初始化赋值
int Teacher::total = 9;
void main(){
	//静态变量的使用
	Teacher::total++;
	cout << Teacher::total << endl;
	//静态方法的使用一
	Teacher::count();
	//静态方法的使用二
	Teacher t1("Hensen");
	t1.count();
}
```

4、常量对象和常函数

```c
class Teacher{
private:
	char* name;
	int age;
public:
	Teacher(char* name,int age){
		this->name = name;
		this->age = age;
		cout << "Teacher有参构造函数" << endl;
	}
	//常函数，当前对象不能被修改，防止数据成员被非法访问
	void myprint() const{
		//不能通过this->name修改值
		cout << this->name << "," << this->age << endl;
	}
	void myprint2(){		
		cout << this->name << "," << this->age << endl;		
	}
};
void main(){
	//普通对象
	Teacher t1("Hensen",20);
	//常对象
	const Teacher t2("rose", 18);
	//普通对象可以调用常函数，也可以调用普通函数
	t1.myprint();
	//t2.myprint2(); 常量对象只能调用常量函数，不能调用非常量函数
	t2.myprint();
}
```

##**虚函数**

1、纯虚函数(类似于抽象类)

* 当一个类具有一个纯虚函数，这个类就是抽象类
* 抽象类不能实例化对象
* 子类继承抽象类，必须要实现纯虚函数，如果没有，子类也是抽象类

```c
class Shape{
public:
	//纯虚函数
	virtual void sayArea() = 0;
	void print(){
		cout << "hi" << endl;
	}
};
class Circle : public Shape{
public:
	Circle(int r){
		this->r = r;
	}
	//必须实现纯虚函数
	void sayArea(){
		cout << "圆的面积：" << (3.14 * r * r) << endl;
	}
private:
	int r;
};
void main(){
	Circle c(10);
}
```

2、接口

```c
//接口（只是逻辑上的划分，语法上跟抽象类的写法没有区别）
class Drawble{
	virtual void draw();
};
```

##**模板函数**

1、模板函数使用，相当于泛型

```c
void myswap(int& a,int& b){
	int tmp = 0;
	tmp = a;
	a = b;
	b = tmp;
}
void myswap(char& a, char& b){
	char tmp = 0;
	tmp = a;
	a = b;
	b = tmp;
}
//可以将上面的函数抽取成模板函数
template <typename T>
void myswap(T& a, T& b){
	T tmp = 0;
	tmp = a;
	a = b;
	b = tmp;
}
void main(){
	int a = 10, b = 20;
	//使用时，根据实际类型，指定模板类型
	myswap<int>(a,b);
}
```

##**模板类**

1、模板类使用

```c
template<class T>
class A{
public:
	A(T a){
		this->a = a;
	}
protected:
	T a;
};
void main(){
	//实例化模板类对象
	A<int> a(6);
}
```

2、普通类继承模板类

```c
class B : public A<int>{
public:
	B(int a,int b) : A<int>(a){
		this->b = b;
	}
private:
	int b;
};
```

3、模板类继承模板类

```c
template <class T>
class C : public A<T>{
public:
	C(T c, T a) : A<T>(a){
		this->c = c;
	}
protected:
	T c;
};
```

##**构造函数**

函数分为三种

* 构造函数
* 析构函数
* 拷贝函数

1、无参构造函数和有参构造函数

```c
class Teacher{
private:
	char *name;
	int age;
public:
	//无参构造函数（无参构造函数会覆盖默认的无参构造函数）
	Teacher(){
		cout << "无参构造函数" << endl;
	}
	//有参构造函数会覆盖默认的构造函数
	Teacher(char *name, int age){
		this->name = name;
		this->age = age;
		cout << "有参构造函数" << endl;
	}	
};
void main(){
	Teacher t1("Hensen",20);
	//另外一种调用方式
	Teacher t2 = Teacher("Hensen",21);
}
```

2、析构函数

```c
class Teacher{
private:
	char *name;
	int age;
public:
	//无参构造函数赋默认值
	Teacher(){
		this->name = (char*)malloc(100);
		strcpy(name,"Hensen");
		age = 20;
		cout << "无参构造函数" << endl;
	}
	//析构函数写法
	//当对象要被系统释放时，析构函数被调用，一般使用于程序的善后工作
	~Teacher(){
		cout << "析构" << endl;
		//释放内存
		free(this->name);
	}
};
void func(){
	Teacher t1;
}
void main(){
	func();
}
```

3、拷贝构造函数

```c
class Teacher{
private:
	char *name;
	int age;
public:
	Teacher(char *name, int age){
		this->name = name;
		this->age = age;
		cout << "有参构造函数" << endl;
	}
	//拷贝构造函数写法
	//默认拷贝构造函数，就是值拷贝
	Teacher(const Teacher &obj){
		this->name = obj.name;
		this->age = obj.age;
		cout << "拷贝构造函数" << endl;
	}
	void myprint(){
		cout << name << "," << age << endl;
	}
};

Teacher func1(Teacher t){
	t.myprint();
	return t;
}

void main(){
	Teacher t1("rose",20);
	func1(t1);

	//这里不会调用拷贝函数的
	//Teacher t1 ;
	//Teacher t2;
	//t1 = t2;
}
```

拷贝构造函数被调用的场景：

* 声明时赋值：Teacher t2 = t1;
* 作为参数传入，实参给形参赋值：上面的写法就是
* 作为函数返回值返回，给变量初始化赋值：Teacher t3 = func1(t1);

4、拷贝函数的问题，浅拷贝（值拷贝）问题

```c
class Teacher{
private:
	char *name;
	int age;
public:
	Teacher(char *name, int age){
		this->name = (char*)malloc(100);
		strcpy(this->name,name);
		this->age = age;
		cout << "有参构造函数" << endl;
	}	
	~Teacher(){
		cout << "析构" << endl;
		//释放内存
		free(this->name);
	}
	void myprint(){
		cout << name << "," << age << endl;
	}
};
void func(){
	Teacher t1("rose", 20);
	Teacher t2 = t1;
}
void main(){
	func();
}
```

这样的使用，会导致t1和t2都调用析构函数，导致同一份内存被释放两次，结果程序会报错

5、解决浅拷贝问题，使用深拷贝

深拷贝很好理解，其实就是将参数进行再一次分配内存，这样的程序就不会出错

```c
class Teacher{
private:
	char *name;
	int age;
public:
	Teacher(char *name, int age){
		int len = strlen(name);
		this->name = (char*)malloc(len+1);
		strcpy(this->name, name);
		this->age = age;
		cout << "有参构造函数" << endl;
	}
	~Teacher(){
		cout << "析构" << endl;
		//释放内存
		free(this->name);
	}
	//深拷贝
	Teacher(const Teacher &obj){
		int len = strlen(obj.name);
		this->name = (char*)malloc(len+1);
		strcpy(this->name,obj.name);
		this->age = obj.age;
	}
	void myprint(){
		cout << name << "," << age << endl;
	}
};
void func(){
	Teacher t1("rose", 20);
	Teacher t2 = t1;
}
void main(){
	func();
}
```

6、构造函数初始化属性写法

```c
class Student{
private:
	int id;
	//属性对象可以直接的初始化
	//Teacher t = Teacher("miss cang");
	Teacher t1;
	Teacher t2;
public:
	//属性对象可以在这里间接的初始化
	Student(int id,char *t1_n, char* t2_n) : t1(t1_n), t2(t2_n){
		this->id = id;
		cout << "Student有参构造函数" << endl;
	}
	void myprint(){
		cout << id << "," << t1.getName() <<"," << t2.getName() << endl;
	}
	~Student(){
		cout << "Student析构函数" << endl;
	}
};
void func(){
	Student s1(10, "miss xu", "mrs li");
	Student s2(20, "miss ya", "mrs wang");
}
void main(){
	func();
}
```

7、析构函数和构造函数的执行顺序

```c
class Human{
public:
	Human(char* name, int age){
		this->name = name;
		this->age = age;
		cout << "Human 构造函数" << endl;
	}
	~Human(){
		cout << "Human 析构函数" << endl;
	}
protected:
	char* name;
	int age;
};
class Man : public Human{
public:
	//给父类构造函数传参，同时给属性对象赋值
	Man(char *brother, char *s_name, int s_age) : Human(s_name, s_age){
		this->brother = brother;
		cout << "Man 构造函数" << endl;
	}
	~Man(){
		cout << "Man 析构函数" << endl;
	}
private:
	char* brother;
};
void func(){
	Man m1("Hensen", "HensenBoy", 18);
}
void main(){
	func();
}
```

输出结果是：父类构造函数先调用，子类的析构函数先调用

```
Human构造函数
Man构造函数
Man析构函数
Human析构函数
```

##**友元函数**

友元函数：使得类的私有属性可以通过友元函数进行访问

1、友元函数

```c
class A{
	//友元函数声明
	friend void modify_i(A *p, int a);
private:
	int i;
public:
	A(int i){
		this->i = i;
	}
	void myprint(){
		cout << i << endl;
	}	
};

//友元函数的实现
void modify_i(A *p, int a){
	p->i = a;
}

void main(){
	A* a = new A(10);
	a->myprint();
	modify_i(a,20);
	a->myprint();
}
```

2、友元类

```c
class A{		
	//友元类声明，表示B这个友元类可以访问A类的任何成员
	friend class B;
private:
	int i;
public:
	A(int i){
		this->i = i;
	}
	void myprint(){
		cout << i << endl;
	}	
};

class B{
private:
	A a;
public:
	void accessAny(){
		a.i = 30;		
	}
};
```

##**类型转换**

C++类型转换分为4种

* static_cast：类型转换，意图明显，增加可阅读性
* const_cast：常量的转换，将常量取出，并可以对常量进行修改
* dynamic_cast：子类类型转为父类类型
* reinterpret_cast：函数指针转型，不具备移植性

1、static_cast的使用

```c
void* func(int type){	
	switch (type){
		case 1:	{
			int i = 9;
			return &i;
		}
		case 2:	{
			int a = 'X';
			return &a;
		}
		default:{
			return NULL;
		}
	}	
}
void func2(char* c_p){
	cout << *c_p << endl;
}
void main(){	
	int i = 8;
	double d = 9.5;
	i = static_cast<int>(d);
	
	//C的写法
	//char* c_p = (char*)func(2);
	//C++的写法
	char* c_p = static_cast<char*>(func(2));
	
	//C的写法
	//func2((char*)(func(2)));
	//C++的写法
	func2(static_cast<char*>(func(2)));
}
```

2、const_cast的使用

```c
void func(const char c[]){
	//C的写法
	//char* c_p = (char*)c;
	//c_p[1] = 'X';
	//C++的写法，提高了可读性
	char* c_p = const_cast<char*>(c);
	c_p[1] = 'Y';
	cout << c << endl;
}
void main(){
	char c[] = "hello";
	func(c);
}
```

3、dynamic_cast的使用

```c
class Person{
public:
	virtual void print(){
		cout << "人" << endl;
	}
};
class Man : public Person{
public:
	void print(){
		cout << "男人" << endl;
	}
	void chasing(){
		cout << "泡妞" << endl;
	}
};
class Woman : public Person{
public:
	void print(){
		cout << "女人" << endl;
	}
	void carebaby(){
		cout << "生孩子" << endl;
	}
};
void func(Person* obj){	
	//C的写法，C只会调用传递过来的对象方法，并不知道转型失败这回事
	//Man* m = (Man*)obj;
	//m->print();
	//C++的写法，dynamic_cast如果转型失败，会返回NULL，保证了代码的安全性
	Man* m = dynamic_cast<Man*>(obj);	
	if (m != NULL){
		m->chasing();
	}
	Woman* w = dynamic_cast<Woman*>(obj);
	if (w != NULL){
		w->carebaby();
	}
}
void main(){
	Woman w1;
	Person *p1 = &w1;
	//输出结果是生孩子
	func(p1);
}
```

4、reinterpret_cast的使用

```c
void func1(){
	cout << "func1" << endl;
}
char* func2(){
	cout << "func2" << endl;
	return "abc";
}
typedef void(*f_p)();
void main(){
	//函数指针数组
	f_p f_array[6];
	f_array[0] = func1;
	//C的写法
	//f_array[1] = (f_p)(func2);
	//C++方式
	f_array[1] = reinterpret_cast<f_p>(func2);
	//调用方法
	f_array[1]();
}
```

##**内存分配**

1、C++ 通过new(delete)动态内存分配

* 使用new的方式：和malloc一样的功能，但是会自动调用构造函数
* 使用delete的方式：和free一样的功能，但是会自动调用析构函数
* 指针的malloc、free可以和new、delete互相掺杂使用，但是malloc、free不会调用构造函数和析构函数

```c
class Teacher{
private:
	char* name;
public:
	Teacher(char* name){
		this->name = name;
		cout << "Teacher有参构造函数" << endl;
	}
	~Teacher(){
		cout << "Teacher析构函数" << endl;
	}
	void setName(char* name){
		this->name = name;
	}
	char* getName(){
		return this->name;
	}
};
void func(){
	Teacher *t1 = new Teacher("jack");
	cout << t1->getName() << endl;
	delete t1;
}
void main(){
	func();
}
```

##**运算符重载**

1、运算符重载的写法一

```c
class Point{
public:
	int x;
	int y;
public:
	Point(int x = 0, int y = 0){
		this->x = x;
		this->y = y;
	}
	void myprint(){
		cout << x << "," << y << endl;
	}
};
//重载+号
Point operator+(Point &p1, Point &p2){
	Point tmp(p1.x + p2.x, p1.y + p2.y);
	return tmp;
}
//重载-号
Point operator-(Point &p1, Point &p2){
	Point tmp(p1.x - p2.x, p1.y - p2.y);
	return tmp;
}

void main(){
	Point p1(10,20);
	Point p2(20,10);
	Point p3 = p1 + p2;
	//输出结果30，30
	p3.myprint();
}
```

2、运算符重载的写法二

```c
class Point{
public:
	int x;
	int y;
public:
	Point(int x = 0, int y = 0){
		this->x = x;
		this->y = y;
	}
	//成员函数，运算符重载
	Point operator+(Point &p2){
		Point tmp(this->x + p2.x, this->y + p2.y);
		return tmp;
	}
	void myprint(){
		cout << x << "," << y << endl;
	}
};

void main(){
	Point p1(10, 20);
	Point p2(20, 10);
	//运算符的重载，本质还是函数调用
	//p1.operator+(p2)
	Point p3 = p1 + p2;
	p3.myprint();
}
```

3、通过友元函数和运算符重载访问私有变量

```c
class Point{
	friend Point operator+(Point &p1, Point &p2);
private:
	int x;
	int y;
public:
	Point(int x = 0, int y = 0){
		this->x = x;
		this->y = y;
	}	
	void myprint(){
		cout << x << "," << y << endl;
	}
};

Point operator+(Point &p1, Point &p2){
	Point tmp(p1.x + p2.x, p1.y + p2.y);
	return tmp;
}

void main(){
	Point p1(10, 20);
	Point p2(20, 10);
	//运算符的重载，本质还是函数调用
	//p1.operator+(p2)
	Point p3 = p1 + p2;
	p3.myprint();
}
```

##**其他**

1、布尔类型（bool）

布尔类型的值大于0的都为true，布尔类型的值小于等于0的都为false

2、三目运算符

三目运算符可以作为参数进行赋值

```c
int a = 10, b = 20;
((a > b) ? a : b) = 30;
```

3、指针常量与常量指针

* 指针常量（int *const p）：指针的常量，表示不可以修改p的地址，但是可以修改p的内容
* 常量指针（const int *p）：指向常量的指针，表示不可以修改p的内容，但是可以修改p的地址

4、指针函数与函数指针

* 指针函数（int *fun(x,y)）：指向指针的函数，其实可以说是返回指针的函数
* 函数指针（int (*fun)(x,y);funa = fun;funb = fun;）：指向函数的指针


