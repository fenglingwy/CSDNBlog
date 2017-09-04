#Dagger2一场老板与员工的故事会


----------

>本篇文章主要包含以下内容

>* 新的公司开张啦：前言
>* 新项目开会讨论：Dagger2的介绍
>* 新员工入职（Hensen）：Dagger2基本使用（@Component、@Inject、@Module、@Named）
>* 新员工入职（Jenny）：Dagger2的模块化开发（@Module、@Singleton）
>* 老板发火了（Boss）：Dagger2的全局单例使用（@dependencies、@Scope）
>* 员工们都辞职了：源码下载、结语


##**新的公司开张啦**

终于到了Hensen老师吹水的时间了，可能各位看惯了文字描述的Dagger2，看久了就会视觉疲劳，因为我也是这样的过来的，所以今天换一种方式，编一个故事会的形式来让观众们换换口味。Dagger2不是一般的难，而是很难，作为新手级别的Hensen老师只能为你们先去Dagger2探探险，一探就是好几天，下面我就把探险的结果告诉大家

终于我们的公司开张啦，新公司命名就叫90后网络科技有限公司吧，下面是人物介绍

1. Boss：老板，大神级别的人物
2. Hensen：元老级员工，负责项目的用户模块
3. Jenny：骨灰级员工，负责项目的网络模块

##**新项目开会讨论**
Boss：今天刚找到一个好项目，嘿嘿，融资了两百万，是时候招人进来把项目做出来啦，我该好好想想项目用什么框架好呢。看最近Dagger2那么火， 不如就它了

1、Dagger2的介绍

Dagger2是一款依赖注入框架，依赖注入框架对MVP架构来说，是最好的解耦工具，不仅为了让你的代码更有模块化，而且更有条理性

2、Dagger2的依赖

项目中使用到的依赖，如果你是Gradle2.2以下的，需要添加apt插件等，这里就不演示了，Gradle2.2是很久之前了，相信很多人都不用了

```
//dagger2
//gradle2.2以下的需要配置依赖插件
compile 'com.google.dagger:dagger:2.10'
annotationProcessor 'com.google.dagger:dagger-compiler:2.10'
```

3、Dagger2的基本原理

![这里写图片描述](http://img.blog.csdn.net/20170410225235307?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里可以简单的理解为多个模块（Module）通过Component直接注入到Activity中，我们可以把多个Module比喻为成为不同的针药水，Component比喻为针管，Container比喻为屁股，我们通过不同针药水混合后放入到针管，然后打到我们的屁股上，这就是Dagger2的基本原理

4、Dagger2的注解

这里提前简单的介绍我们需要用到的注解


1. @Inject：通常在需要依赖的地方使用这个注解，也就是针水打进屁股的过程
2. @Module：Modules类似于我们的模块，也就是成分不同的针药水，提供各种实例和对象
3. @Provides：在Modules中，我们定义的方法是用这个注解，以此来告诉Dagger2我们想要提供哪些实例和对象，也就是成为不同的针药水的厂家
4. @Component：注入器，是@Inject和@Module的桥梁，它的主要作用就是连接这两个部分，也就是我们的针管+针头
5. @Scope：Dagger2使用的作用域，局部还是全局

5、Dagger2三步曲

1. 创建Module，提供使用实例
2. 创建Component，连接Module与Activity之间的桥梁
3. Inject注入，开始在Activity中进行注入

##**新员工入职（Hensen）**

Hensen：老板好啊，今天很高兴能来到90后网络科技有限公司，请问有什么任务吩咐吗？

Boss：来，把Dagger2作为我们新项目的框架，你负责用户模块，今晚你就把它做完

Hensen：好的，老板，我这就去工作了

Hensen习惯了MVC的框架模式，所以他一来就把MVC和Dagger2的框框先搭建了起来

![这里写图片描述](http://img.blog.csdn.net/20170410181855962?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1、主体逻辑：首先是定义几个Controller来作为主要的功能逻辑

```
public class MoneyController {

    private Context mContext;
	//需要传入参数，虽然没什么用，但是为了演示Dagger2而用
    public MoneyController(Context context) {
        this.mContext = context;
    }

    public void payMoney() {
        Log.e("MoneyController", "payMoney");
    }
}
```

```
public class OrderController {

    public void order(){
        Log.e("OrderController","order");
    }
}
```

```
public class UserController {
	//需要传入参数，虽然没什么用，但是为了演示Dagger2而用
    public UserController(String name) {
        Log.e("UserController", "name:" + name);
    }
	//需要传入参数，虽然没什么用，但是为了演示Dagger2而用
    public UserController(String name, int age) {
        Log.e("UserController", "name:" + name + "age:" + age);
    }
}
```

2、Module创建：刚学习完Dagger2的Hensen，知道Dagger2依赖注入的思路，就跟自己去打屁股针是一样道理的，现在需要针药水和针管针头，来将药水打入我们的屁股中，这里的屁股就是Activity，即将Controller打入Activity中

```
//（打针的药水）
@Module
public class UserModule {

    private Context mContext;

    public UserModule(Context context) {
        this.mContext = context;
    }

	//（成分不同的针药水）
    @Provides
    public OrderController providerOrderController(){
	    return new OrderController();
	}

    @Provides
    public MoneyController providerMoneyController() {
        return new MoneyController(mContext);
    }

}
```
3、Conmponet创建：准备好药水，下面准备针管针头

```
//（针管+针头）
@Component(modules = {UserModule.class})
public interface UserComponet {
    void inject(UserActivity userActivity);
}
```

4、Inject的注入：这里都准备好了，**这时候千万要注意记得Rebuild一下项目，它会生成我们需要的代码**，那么就可以开始打针了

```
public class UserActivity extends AppCompatActivity {
	//（打针前准备）
    @Inject
    OrderController orderController;
    @Inject
    MoneyController moneyController;
    
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dagger2);
        
        //（开始打针）
        DaggerUserComponet.builder().userModule(new UserModule(this)).build().inject(this);

        orderController.order();
        moneyController.payMoney();
	}
}
```
运行程序，控制台成功的打印出信息

![这里写图片描述](http://img.blog.csdn.net/20170410192506331?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

5、Named的使用：做到这里，Hensen已经把大体的都做完了，剩下一个UserController 不知道怎么下手，因为他有两个构造方法，Dagger2会去默认会去调用哪个构造方法呢，于是Hensen又开始找资料了，他发现可以这么做，他使用@Named方法，提供两个不同构造方法

```
@Module
public class UserModule {

    private Context mContext;

    public UserModule(Context context) {
        this.mContext = context;
    }

    @Named("debug")
    @Provides
    public UserController providerUserControllerName() {
        return new UserController("lisi");
    }

    @Named("release")
    @Provides
    public UserController providerUserControllerNameAndAge() {
        return new UserController("lisi",15);
    }

    @Provides
    public OrderController providerOrderController(){return new OrderController();}

    @Provides
    public MoneyController providerMoneyController() {
        return new MoneyController(mContext);
    }

}
```

那么在使用的时候的也要相对应相同的名字

```
public class UserActivity extends AppCompatActivity {
	//（打针前准备）
    @Inject
    OrderController orderController;
    @Inject
    MoneyController moneyController;
    
    @Named("debug")
    @Inject
    UserController userControllerDebug;
    @Named("release")
    @Inject
    UserController userControllerRelease;
    
	@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dagger2);
        
        //（开始打针）
        DaggerUserComponet.builder().userModule(new UserModule(this)).build().inject(this);

        orderController.order();
        moneyController.payMoney();
	}
}
```

运行程序，控制台打印出信息

![这里写图片描述](http://img.blog.csdn.net/20170410192711910?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

6、自定义Named的使用：Hensen是个追求完美的人，觉得程序还可以把这个Named再完善一下，于是它查看Named的源码，照抄了一份作为自己的注释

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Debug {
}
```

```
@Qualifier
@Documented
@Retention(RUNTIME)
public @interface Release {
}
```

那么使用起来看上去就很舒服了，效果和上面是一样的，只不过换一种写法

```
@Debug
@Inject
UserController userControllerDebug;
@Release
@Inject
UserController userControllerRelease;
```

这样就 没问题了，Hensen的工作基本上已经完成了，下面就可以给老板交付任务了

##**新员工入职（Jenny）**

Jenny：老板您好，我是Jenny，今天很高心来到公司，我个人比较擅长网络这块，有什么任务可以吩咐给我吗？

Boss：这样啊，那你写个网络模块，让Hensen的用户管理连上网呗，对了最好用OkHttp

Jenny：恩，好的，老板，那我先去工作了

由于Jenny的Dagger学得不好，所以只能去阅读Hensen的代码，然后根据意思模拟出自己的一套Dagger出来

![这里写图片描述](http://img.blog.csdn.net/20170410183549891?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1、主体逻辑：首先是创建负责网络模块的主体逻辑，提供get和post的方法

```
public class HttpController {

    private OkHttpClient mOkHttpClient;

    public HttpController(OkHttpClient okHttpClient){
        this.mOkHttpClient = okHttpClient;
    }

    public OkHttpClient getOkHttpSingleton(){
        return mOkHttpClient;
    }

    public void doGet() {
	    //正常得用到这里传进来的okhttp请求网络
        Log.e("doGet", "doGet");
    }

    public void doPost() {
        Log.e("doPost", "doPost");
    }

}
```
2、Module的创建：这里就是为Hensen准备好的Controller实例，直接把这个Module添加进去就能用

```
@Module
public class HttpModule {

	//需要下面的HttpClient
    @Provides
    public HttpController providerHttpController(OkHttpClient okHttpClient) {
        return new HttpController(okHttpClient);
    }
		
	//提供HttpClient给Controller
    @Provides
    public OkHttpClient providerOkHttpClient(){
        return new OkHttpClient().newBuilder().build();
    }
}
```

一开始不是很懂的Jenny，让Hensen直接用自己的Module，它的意思如下面所示，可以直接把自己Module嵌入到Hensen中去，然而Hensen为了不得罪新同伴，就增加进去了，它知道这样做不好，同时也劝Jenny把Component也写出来

```
@Component(modules = {UserModule.class, HttpModule.class})
public interface UserComponet {
    void inject(UserActivity userActivity);
}
```

3、Componet的创建：让这个模块注入到另一个Activity中，**这里得注意，如果其他Component使用的inject方法中也传递LoginActivity的话，这里就会报错，所以一个Componet能对应多个inject到Activity中，但是不能重复**
```
@Component(modules = {HttpModule.class})
public interface HttpComponet {
	//等着LoginActivity 打针
    void inject(LoginActivity loginActivity);
}
```

4、模块使用：Hensen收到了Jenny做好的网络模块后，直接把它加入LoginActivity中就好啦，**注意记得重新Rebuild一次才会生成对应的DaggerHttpComponet**

```
public class LoginActivity extends AppCompatActivity {

    @Inject
	HttpController httpController;
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        DaggerHttpComponet.builder().httpModule(new HttpModule()).build().inject(this);

		httpController.doPost();
    }
}
```

5、Singleton的使用：Hensen作为老员工，知道OkHttp不能重复创建，让Jenny把它进行单例，Jenny听起来也时很有道理，于是就查资料，如何在Dagger2进行单例，它根据资料查到的方法如下，对Module中提供单例的方法中添加@Singleton字样，然后对使用它的Componet添加@Singleton字样即可，这样就完成了

```
@Module
public class HttpModule {

    @Provides
    public HttpController providerHttpController(OkHttpClient okHttpClient) {
        return new HttpController(okHttpClient);
    }
    @Provides
    @Singleton
    public OkHttpClient providerOkHttpClient(){
        return new OkHttpClient().newBuilder().build();
    }

}
```


```
@Singleton
@Component(modules = {HttpModule.class})
public interface HttpComponet {
    void inject(LoginActivity loginActivity);
}
```
6、但是很快，Hensen又发现了令人头疼的事情，由于UserActivity使用的是UserComponet、LoginActivity使用的是HttpComponet的，它们创建出来的HttpClient是不同对象。只有在同一个Componet中的Activity使用HttpClient才是单例。**这是因为Singleton是跟Componet绑定在一起的，不同的Componet表示的肯定是不同的单例，所以没办法**。Hensen是通过打印地址检查出问题的

```
public class UserActivity extends AppCompatActivity {
    @Inject
    HttpController httpController;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dagger2);

        DaggerUserComponet.builder().userModule(new UserModule(this)).build().inject(this);
        Log.e("okHttpClient2Address",httpController.getOkHttpSingleton()+"");
    }
}
```

```
public class LoginActivity extends AppCompatActivity {
    @Inject
    HttpController httpController;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        DaggerHttpComponet.builder().httpModule(new HttpModule()).build().inject(this);
        Log.e("LoginActivity",""+httpController.getOkHttpSingleton());
    }
}
```


##**老板发火了（Boss）**

Boss：Jenny，你这个单例不能这样子，在不同Componet会出现不同的单例，你得把它放到Application中去，这样才是全局才是只有一个

Jenny：老板，这。。我不会啊，怎么办？

Boss：哎，看来还是我得出场了

老板也开始写出了一套App级别的Dagger出来，并提供okHttp单例出来，大Boss来啦

![这里写图片描述](http://img.blog.csdn.net/20170410185747702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1、Module的创建：老板先从需求开始写，需要什么就给什么，而且是@Singleton单例
```
@Module
public class AppModule {

    @Provides
    @Singleton
    public OkHttpClient providerOkHttpClient(){
        return new OkHttpClient().newBuilder().build();
    }
}
```

2、Component的创建：老板也懂得Dagger2，所以毫不犹豫进行第二步

```
@Singleton
@Component(modules = {AppModule.class})
public interface AppComponet {
	//这里由于是放在Application级别的，所以不用inject方法，但是必须要有提供了什么东西就要有对应的方法
	//这里我们提供了OkHttpClient，这句话不写会报错
    OkHttpClient okHttpClient();
}
```

3、Application创建：老板深知只有在Application的变量才会整个程序被实例化一次，所以放在这里的OkHttpClient全局是单例是完全没错的，**注意记得重新Rebuild一次才会生成对应的DaggerAppComponet**

```
public class MyApplication extends Application {

    private AppComponet appComponet;

    @Override
    public void onCreate() {
        super.onCreate();

        appComponet = DaggerAppComponet.create();
    }

    public AppComponet getAppComponet() {
        return appComponet;
    }
}
```

当然，老板怎么可能犯小错误呢，记得在Application中注册该MyApplication 

```
<application
    android:name=".Dagger2.BossModel.App.MyApplication"
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
```

4、Scope的创建：老板亲自写了个Scope，跟我们说，以后就别用Singleton了，用我提供的ActivityScope，这样单例才会生效。**这里是因为Singleton也是个Scope，Scope和Scope之间不能冲突，也就是老板使用了Singleton，我们再使用就会报错，我们必须使用老板提供另外的ActivityScope才不会报错**

```
@Scope
@Retention(RUNTIME)
public @interface ActivityScope {
}
```

5、Jenny也偷偷的把自己提供的OkHttpClient注释掉了，**这里这样做为了后面的程序不报错**

```
@Module
public class HttpModule {

    @Provides
    public HttpController providerHttpController(OkHttpClient okHttpClient) {
        return new HttpController(okHttpClient);
    }

//	  注释了这里，不再由我提供，老板亲自提供
//    @Provides
//    @Singleton
//    public OkHttpClient providerOkHttpClient(){
//        return new OkHttpClient().newBuilder().build();
//    }

}
```

6、dependencies的使用：老板写好了之后，我们就要开始使用老板的，Hensen作为熟悉Dagger2的人，当然是先用为妙，下面是Hensen修改的内容，**这里需要注意要事先Rebuild一下老板的模块，才能使用.appComponet(myApplication.getAppComponet())**

```
//@Singleton
@ActivityScope
@Component(modules = {UserModule.class, HttpModule.class},dependencies = AppComponet.class)
public interface UserComponet {
    void inject(UserActivity userActivity);
}
```

```
public class UserActivity extends AppCompatActivity {
    @Inject
    OkHttpClient bossOkHttpClient;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_dagger2);
        //老板的话
        MyApplication myApplication = (MyApplication) getApplication();
        DaggerUserComponet.builder()
                .userModule(new UserModule(this))
                .appComponet(myApplication.getAppComponet())
                .build().inject(this);
		//为了验证在不同的Component是否为单例，打印出地址
        Log.e("bossOkHttpClient",""+bossOkHttpClient);
}
```

7、不会Dagger2的Jenny也开始模仿Hensen的写法，开始修改自己的代码，下面是Jenny修改的代码

```
//@Singleton
@ActivityScope
@Component(modules = {HttpModule.class},dependencies = AppComponet.class)
public interface HttpComponet {

    void inject(LoginActivity loginActivity);
}
```

8、Hensen顺便也把Jenny写的嵌入到自己的Activity中

```
public class LoginActivity extends AppCompatActivity {
    @Inject
    OkHttpClient bossOkHttpClient;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        MyApplication myApplication = (MyApplication) getApplication();
        DaggerHttpComponet.builder()
                .httpModule(new HttpModule())
                .appComponet(myApplication.getAppComponet())
                .build().inject(this);

        Log.e("bossOkHttpClient",""+bossOkHttpClient);
    }
}
```

9、最后Hensen通过打印信息发现，老板的确实是单例啊，终于解决了不同Component中不会出现不同的单例对象了

![这里写图片描述](http://img.blog.csdn.net/20170410200058039?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



##**员工们都辞职了**

Hensen：老板我要辞职了，天天加班，工资给得又少，工作量又那么多，而且这个Dagger2太烦了，搞得我头皮发麻
Jenny：老板，我不会Dagger2，我也要辞职


1、[源码下载](http://download.csdn.net/detail/qq_30379689/9809863)

2、结语

学习完Dagger2真的是一脸懵逼，如果没有一定的思维能力还是挺难的，最后员工都忍受不了Dagger2这么复杂的东西，直接辞职了，哈哈哈哈，所以说选择框架很重要，如果各位没有心思研究新技术，还是先放着Dagger2让它自由去吧，缺一个也不缺，还是去学习性价比比较高框架吧

回顾本节课内容：

1. @Module、@Component、@Inject基本使用（Everyone）
2. @Named创建不同的参数的实例（Hensen）
3. @Singleton完成单个Component的实例（Jenny）
4. @Singleton和@Scope在不同Component中保持同一个实例（Boss）
