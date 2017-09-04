>请点击左上角+号查看目录，文章会长期不定时更新

#面试复习——Android工程师之Java基础

----------

[toc]

>时光如梭，一转眼大三第二学期就要完了，为了面试准备，还是乖乖开始复习Java和Android基础吧，我知道复习的东西不能一蹴而就，所以计划打算先从Java开始，然后一天复习三点内容，每天完成目标，就可以开始人民的名义、王者荣耀啦，哈哈哈

##**cmd命令**

当我们编写好一段程序后，会执行下面的cmd命令来运行java程序，以HelloWorld这个类为例

```
$ javac HelloWorld.java
$ java HelloWorld
Hello World
```

1. javac：将java文件源编译成class字节码文件，如果运行指令没出错，就会产生一个HelloWorld.class的文件
2. java：java后面跟着的是java文件中的类名，注意java命令后面是不需要加.class的

##**Java语言的特点**

1. 简单：java语言不使用指针，而是引用，并提供废料收集，让程序员不必为内存管理而担忧
2. 面向对象：java只支持类的单继承、接口之间的多继承、类与接口之间的实现机制（关键字为implements）
3. 健壮：强类型转换、异常处理、垃圾回收机制都是java程序健壮型的保障
4. 可移植：java系统本身具有很强的可移植性，java编译器是用Java实现的，java的运行环境是用ANSI C实现的
5. 多线程：java线程创建通常有两种方式，一种是继承Thread，一种是实现Runnable

##**配置环境变量**

右击"我的电脑"->"属性"->"高级系统设置"，在"高级"选项卡中选择"环境变量"

添加两个新的环境变量和修改一个环境变量

1. 新增变量名：JAVA_HOME，变量值：C:\Program Files (x86)\Java\jdk1.8.0_91             //具体根据你的jdk路径为主
2. 新增变量名：CLASSPATH，变量值：.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar;
3. 修改添加变量名：PATH，变量值：%JAVA_HOME%\bin;%JAVA_HOME%\jre\bin;

>注意：如果使用1.5以上版本的JDK，不用设置CLASSPATH环境变量，也可以正常编译和运行Java程序

##**命名规范**

1. 大小写敏感：所有的命名都区分大小写
2. 类名：每个单词首写字母为大写字母
3. 方法名：所有方法都以小写字母开头
4. 类文件名：类文件名必须和类名相同
5. 程序主入口：所有程序都从public static void main(String[] args)方法开始执行

##**标识符**

类名、方法名、变量名都称为标识符

1. 所有标识符都以字母（a-z或A-Z）、美元符（$）、下划线开头（_）
2. 首字符之后可以是字母、美元符、下划线、数字
3. 关键字不能作为标识符
4. 标识符区分大小写

##**变量类型**

1. 成员变量：在类中，方法体之外定义的变量，存在堆内存中
2. 局部变量：在方法、构造方法、语句块定义的变量，存在栈内存中
3. 静态变量（类变量）：在类中，方法体之外定义的变量，但必须由static关键字修饰，存在静态存储区（方法区）

##**基本数据类型**

1. 四种数字类型：byte（1个字节）、short（2个字节）、int（4个字节）、long（8个字节）
2. 两种浮点型：float（4个字节）、double（8个字节）
3. 一种布尔型：boolean（false和true）
4. 一种字符型：char（2个字节）

##**访问控制修饰符**

|修饰符|当前类|同一包内|子孙类|其他包|
|:
|public|√|√|√|√|
|protected|√|√|√|×|
|default|√|√|×|×|
|private|√|×|×|×|

<br>
简单的说：

1. private：只有在本类才能访问
2. public：任何地方都可以访问
3. protected：在同包内的类及其他包的子类能访问
4. default：默认不写在同包内能访问

需要注意：

1. private：不能修饰类和接口
1. protected：不能修饰类和接口，能修饰方法和成员变量，但不能修饰接口的成员方法和成员变量


##**非访问修饰符**

1、static修饰符

1. 静态变量：无论实例化多个对象，静态变量只有一个，局部变量不能声明static变量
2. 静态方法：静态方法不能使用非静态变量

2、final修饰符

1. final修饰的变量：为常量，值不可变
2. final修饰的对象：值可变，引用不变
3. final修饰的方法：可以继承，但不可以重写
4. final修饰的类：不能够继承

3、abstract修饰符

1. 抽象类：不能用来实例化，一个类不能同时被 abstract 和 final 修饰
2. 抽象方法：不能被声明成 final 和 static，抽象方法必须在抽象类中出现

4、synchronized修饰符

1. synchronized修饰的方法：同一时间只能被一个线程访问

5、transient修饰符

1. transient修饰的变量：该变量在序列化后无法获得访问

6、volatile修饰符

1. volatile修饰的变量：该变量存取的是变量内存地址

##**继承**

1、变量和方法

1. 子类具有父类非private的变量和方法

2、构造方法

1. 子类不可以继承父类的构造方法，可以用super()调用
2. 子类默认在所有构造方法中调用无参父类构造方法，即super()

##**重写**

1. 重写发生在父子类
2. 重写的参数和方法名和返回类型必须相同
3. 重写的访问修饰符大于等于父类
4. 重写抛出的异常小于等于父类
5. 不能重写构造方法
6. 不能重写被final或static或private修饰的方法

##**重载**

1. 重载发生在编译时
1. 重载的方法名必须相同
2. 重载的参数或参数个数或参数顺序必须不同
3. 重载的返回类型可以不同
4. 重载的访问修饰符可以不同

##**多态**

1、多态的伪代码表现形式

```
父类 a = new 子类;
a.doSometing();
```

编译器会寻找父类是否有doSometing方法，如果有，则执行子类的doSometing方法，如果没有，则编译报错

##**抽象类**

1. 抽象类不能被实例化
2. 抽象类不一定包含抽象方法，但有抽象方法一定是抽象类
3. 抽象类只是声明方法，不包含方法体
4. 构造方法，类方法不能声明抽象


##**接口**

1. 接口每个方法默认被public abstract修饰
2. 接口每个变量默认被public static final修饰


##**接口与抽象类的区别**

1. 抽象类中的方法可以有方法体，而接口不行
2. 抽象类中的成员变量可以是各种类型，而接口不行
3. 抽象类中可以有静态代码和静态方法，而接口不行
4. 抽象类所体现的是继承关系，接口仅仅实现接口定义的契约

##**集合**

1. Collection接口：提供子类接口继承，List和Set继承自Collection接口
2. List接口：有序、key和value可重复，可以动态增长，查找效率高、插入删除效率低。（实现类：ArrayList、LinkedList、Vector）
3. Set接口：无序、key和value不可重复，查找效率慢、插入删除效率高。（实现类：HashSet、TreeSet）
4. Map接口：无序、key唯一、value可重复。（实现类：HashMap、TreeMap、Hashtable）
5. 注意：TreeSet和TreeMap都是有序的


##**泛型**

泛型是"参数化类型"，即可以确定参数的类型

1、泛型方法定义
```
public static <T> T getMiddle(T... a){
	return a[a.length/2];
}
```
2、泛型类定义
```
public class List<T>{

}
```
3、泛型变量定义
```
public static <T extends Comparable & Serializeble> void printArray(T[] inputArray){
	
}
```
注意：

1. 泛型只支持< T extends X>形式，不支持< T super X>形式
2. 泛型可以有多个限定，这是接口多继承的体现


4、泛型通配符

```
//无限定通配符
public static void getData(List<?> data){

}
//上限通配符
public static void getData(List<? extends Number> data){

}
//下限通配符
public static void getData(List<? super Number> data){

}
```

注意：（口诀：上get下add）

1. 泛型数组是不存在，即List< ?>[]是错误的
2. List< ? extends Person>使用上限通配符后，可以调用其get方法，不可以调用其add方法
3. List< ? super Person>使用下限通配符后，可以调用其add方法，不可以调用其get方法

5、泛型的擦除

① 概念：虚拟机中没有泛型类型的对象，所有对象都是普通类，无论什么时候定义的泛型类型，在虚拟机中都自动转换成一个相应的原始类型（Object）

② 处理：编译器对泛型的处理

1. Code specialization：实例化一个泛型类型或泛型方法都会产生一份新的目标代码（字节码或二进制代码），例如List可能会针对string，integer，float产生三份新的目标代码。这样的方式代码极度臃肿、膨胀，C++就是采用这种方式进行泛型处理
2. Code sharing：对每个泛型只生成唯一的一份目标代码，泛型的所有实例都映射到这份目标代码上，在需要的时候执行类型检查和类型转换。Java就是采用这种方式进行泛型处理

③ 规则：如果泛型存在限定，就用第一个限定替换，如果没有，就用Object替换

④ 例子：
```
//泛型擦除前
public class Person<T>{
	private T first;
	private T Second;
}
//泛型擦除后
public class Person{
	private Object first;
	private Object Second;
}

-----------------------分割线------------------------

//泛型擦除前
public class Person<T extends Comparable & Serializable>{
	private T first;
	private T Second;
}
//泛型擦除后
public class Person{
	private Comparable  first;
	private Comparable  Second;
}
```
⑤ 注意：

1. 虚拟机中没有泛型，只有普通类和普通方法
2. 所有泛型类的类型参数在编译时都会被擦除
3. 泛型类的静态变量是共享的
4. 创建泛型对象时请指明类型，让编译器尽早地做参数检查
5. 不能使用基本数据类型实例化类型参数，比如int，系统会擦除成Object，对应的应该是Integer

##**注解**

1、什么是Annotation（注解）

Java提供的一种元程序中的元素关联任何信息和任何元数据（metadata）的途径和方法

2、什么是metadata

1. 元数据以标签的形式存在于Java代码中
2. 元数据描述的信息是类型安全的
3. 元数据需要编译器之外的工具额外的处理用来生成其他的程序部件
4. 元数据可以只存在于Java源代码级别，也可以存在于编译之后的Class文件内部

3、注解分类

1. 系统内置标准注解
	* @Override：表示重写
	* @Deprecated：表示已过时
	* @SuppressWarnnings：表示抑制警告
2. 元注解
	* @Target：表示注解的修饰范围
	* @Retention：表示注解的代码生存期
	* @Documented：表示注解为程序员的API
	* @Inherited：表示注解可以继承

4、Android Support Annotation

* Nullness注解（@NonNull、@Nullable）：表示指定的空类型或非空类型
* ResourceType注解（@StringRes、@IntegerRes等）：表示指定的变量类型
* Threading注解（@WorkerThread、@UIThread等）：表示指定的线程类型
* OverridingMethods注解（@CallSuper等）：表示调用父类的方法

