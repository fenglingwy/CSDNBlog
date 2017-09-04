>请点击左上角+号查看目录，文章会长期不定时更新

#面试复习——Android工程师之设计模式


----------

[toc]


##单例模式

1、概念

单例模式是一种对象创建模式，它用于产生一个对象的具体实例，它可以确保系统中的一个类只产生一个实例

2、好处

* 省略创建对象所花费的时间
* 对系统内存的使用频率降低，减轻GC压力，缩短GC停顿时间

3、六种写法

* 饿汉模式

```
public class HungurySingleton {
	private static final HungurySingleton mHungurySingleton = new HungurySingleton();
	private HungurySingleton(){
		
	}
    public static HungurySingleton getHungurySingleton() {
        return mHungurySingleton;
    }
}
```

不足：无法对instance实例做延时加载
优化：懒汉模式

* 懒汉模式

```
public class LazySingleton {
	private static LazySingleton instance;
    private LazySingleton() {

    }
    public static LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }
}
```

不足：在多线程并发时无法保证实例唯一
优化：懒汉线程安全

* 懒汉线程安全

```
public class LazySafetySingleton {
	private static LazySafetySingleton instance;  
    private LazySafetySingleton (){
    
    }
	//方式一
	public static synchronized LazySafetySingleton getInstance() {
		if (instance == null) {  
		 instance = new LazySafetySingleton();  
		}  
		return instance;  
	}
   
	//方式二
	public static LazySafetySingleton getInstance1() {  
		synchronized (LazySafetySingleton.class) {  
		   if(instance == null){
		       instance = new LazySafetySingleton();  
		   }  
		}  
		return instance;  
	}
}
```

不足：性能较低
优化：DCL

* DCL（双检查锁机制）

```
public class DclSingleton {
	private static volatile DclSingleton mInstance = null;
    private DclSingleton() {

    }
    public static DclSingleton getInstance() {
        if (mInstance == null) {
            synchronized (DclSingleton.class) {
                if (mInstance == null) {
                    mInstance = new DclSingleton();
                }
            }
        }
        return mInstance;
    }
}
```

不足：JVM的即时编译器中存在指令重排序的优化
解决：将实例设置为Volatile
优化：静态内部类/枚举

* 静态内部类

```
public class StaticInnerSingleton {
	private StaticInnerSingleton() {
    
    }
    public static StaticInnerSingleton getInstance() {
        return SingletonHolder.sInstance;
    }
    private static class SingletonHolder {
        private static final StaticInnerSingleton sInstance = new StaticInnerSingleton();
    }
}
```

优点：

* 运用类中静态变量的唯一性
* JVM本身同步控制机制（final、static）保证了线程安全
* 没有使用Synchronized，保证了性能的优化
* 静态内部类是私有的，除了外部类其他是无法访问的

* 枚举

```
public enum EnumSingleton {
    INSTANCE;  
    
    public void doSomeThing() {  
	     
    }  
}
```

优点：

* 写法简单/线程安全（限于只有枚举的实例方法）

4、Android中的单例

application、eventBus

##建造者模式

1、概念

建造者模式是较为复杂的创建型模式，它将客户端与包含多个组成部分的复杂对象的创建过程分离

2、使用场景

当构造一个对象需要很多参数的时候，并且参数的个数或者类型不固定的时候

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011541847?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、在Android中应用

AlertDialog、Glide、OkHttp

##适配器模式

1、概念

将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名又称包装类

2、分类

* 类适配器
* 对象适配器

3、类适配器定义

类的适配器模式把适配的类的API转换成为目标类的API

4、类适配器UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011603914?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、对象适配器定义

与类的适配器模式一样，对象的适配器模式把被适配的类的API转换成为目标类的API，与类的适配器模式不同的是，对象的适配器模式不是使用继承关系连接到Adaptee类，而是使用委派关系连接到Adaptee类

6、对象适配器UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011615583?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

7、在Android中应用

BaseAdapter类

##装饰模式

1、概念

动态地给一个对象增加一些额外的职责，就增加对象功能来说，装饰模式比生成子类实现更为灵活

2、使用场景

* 在不影响其他对象的情况下，以动态、透明的方式给单个对象添加职责
* 当不能采用集成的方式对系统进行拓展或者采用继承不利于系统拓展和维护时可以使用装饰模式

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011636482?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、优点

* 对于拓展一个对象的功能，装饰模式比继承更加灵活，不会导致类的个数急剧增加
* 可以通过一种动态的方式来拓展一个对象的功能
* 可以对一个对象进行多次装饰，通过使用不同的具体装饰类以及这些装饰类的排列组合

5、在Android中应用

context类

##外观模式

1、概念

外观模式的主要目的在于让外部减少与子系统内部多个模块的交互，从而让外部能够更简单得使用子系统，它负责把客户端的请求转发给子系统内部的各个的模块进行处理

2、使用场景

* 当你要为一个复杂子系统提供一个简单接口时
* 客户程序与抽象类的实现部分之间存在很大的依赖性
* 当你需要构建一个层次结构的子系统时

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011816105?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、优点

* 由于Facade类封装了各个模块交互的过程，如果今后内部模块调用关系发生了变化，只需要修改Facade实现就可以了
* Facade实现是可以被多个客户端调用的

5、在Android中应用

contextImpl类

##组合模式

1、概念

将对象以树形结构组织起来，以达到“部分-整体”的层次结构，使得客户端对单个对象和组合对象的使用具有一致性

2、使用场景

* 需要表示一个对象整体或部分层次
* 让客户能够忽略不同对象层次的变化

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011844620?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、优点

* 高层模块调用简单
* 节点自由增加

5、在Android中应用

View、ViewGroup

##策略模式

1、概念

定义一系列的算法，把它们一个个封装起来，并且使他们可互相替换，本模式使得算法可独立于使用它的客户而变化

2、使用场景

一个类定义了多种行为，并且这些行为在这个类的方法中以多个条件语句的形式出现，那么可以使用策略模式避免在类中使用大量的条件语句

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011856084?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、优点

* 上下文（Context）和具体策略（AbstractStrange）是松耦合关系
* 策略模式满足“开闭原则”

5、在Android中应用

Volley、属性动画、插值器

##模板方法模式

1、概念

模板方法是通过定义一个算法骨架，而将算法中的步骤延迟到子类，这样子类就可以复写这些步骤的实现来实现特定的算法

2、使用场景

* 多个子类有公有的方法，并且逻辑基本相同时
* 重要、复杂的算法，可以把核心算法设计为模板方法
* 重构时，模板方法模式是一个经常使用的模式

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011907342?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、在Android中应用

Activity和Fragment生命周期、AsyncTask、BaseActivity

##观察者模式

1、概念

定义对象之间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新

2、使用场景

* 一个抽象模型由两个方面，其中一个方面依赖于另一个方面
* 一个对象的改变将导致一个或多个其他对象也发生改变
* 需要在系统中创建一个触发链

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011917867?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、在Android中应用

ListView、RxJava

##责任链模式

1、概念

是一个请求有多个对象来处理，这些对象是一条链，但具体由哪个对象来处理，根据条件判断来确定，如果不能处理会传递给该链中的下一个对象，直到有对象处理它为止

2、使用场景

* 有多个对象可以处理同一个请求，具体哪个对象处理该请求待运行时刻再确定
* 在不明确指定接收者的情况下，向多个对象中的一个提交一个请求
* 可动态指定一组对象处理请求，客户端可以动态创建职责链来处理请求

3、UML图分析

![这里写图片描述](http://img.blog.csdn.net/20170818011927440?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、在Android中应用

try-catch语句、有序广播、事件分发机制

