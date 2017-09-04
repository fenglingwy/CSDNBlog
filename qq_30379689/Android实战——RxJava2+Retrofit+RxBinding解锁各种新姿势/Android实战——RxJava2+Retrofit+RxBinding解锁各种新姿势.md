#RxJava2+Retrofit+RxBinding解锁各种新姿势


----------
>**本文已授权微信公众号：鸿洋（hongyangAndroid）原创首发。**

>本篇文章内容包含以下内容

>* 前言
>* RxJava2的基本介绍
>* RxJava2观察者模式的介绍
>* RxJava2观察者模式的使用
>* RxJava2的基本使用
	* 模拟发送验证码
>* RxJava2与Retrofit的使用
	* 模拟用户登陆获取用户数据
	* 合并本地与服务器购物车列表
>* RxJava2与RxBinding的使用
	* 优化搜索请求
	* 优化点击请求
>* 源码下载
>* 结语

##**前言**

作为主流的第三方框架Rx系列，不学习也不行啊，对于初学者来说，可能RxJava看起来很难，用起来更难，但是你要知道，越复杂的东西往往能解决越复杂的问题，有可能你应用在项目中，也许你在面试的时候，就会和初级工程师拉开一大段距离。这门课程需要大家有Retrofit的基础，如果想学习Retrofit的同学可以查看我的博客，废话不多说，Hensen老师开车了。

##**RxJava2的介绍**

用原话就是：RxJava2是一个在Java虚拟机上，使用可观察的序列构成基于事件的，异步的程序库。不理解没关系，可以类比成我们的AsyncTask，这样就好理解多了

RxJava传送门：https://github.com/ReactiveX/RxJava

##**RxJava2观察者模式的介绍**

观察者模式就是RxJava使用的核心点，掌握这个模式，可以理解RxJava更简单，观察者模式简单的说就是"订阅-发布"的模式，举一例子说，当你订阅某家牛奶店的早餐奶（订阅过程），只要牛奶店生产牛奶，便会给你送过去（发布过程）。这里的牛奶店只有一家，但是订阅的人可以很多，这是一种一对多的关系，只要牛奶店发布牛奶，那么订阅的人就会收到牛奶。换做RxJava里面的话，牛奶店就是被观察者（Observable），订阅的人就是观察者（Observer）

##**RxJava2观察者模式的使用**
这里我们举一例子学校点名的例子，首先创建我们所说的观察者和被观察者

```
public interface Observable {
	//订阅
    public void attach(Observer observer);
    //取消订阅
    public void detach(Observer observer);
    //发布
    public void notifyObservers(String message);
}
```

```
public interface Observer {
	//给个名字来分辨不同的观察者
    void setName(String name);
    //观察者的方法
    void say(String message);
}
```
各位可以思考一下，根据上面的介绍，学生和老师，谁是观察者，谁是被观察者，下面就看代码给你分析

```
public class Teather implements Observable {

    private List<Observer> observers = new ArrayList<>();

    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String message) {
        for (Observer observer : observers) {
            observer.say(message);
        }
    }
}
```

```
public class Student implements Observer {

    private String name;

    @Override
    public void setName(String name) {
        this.name = name;
    }

    @Override
    public void say(String message) {
        System.out.println(message + ":" + this.name + "到");
    }
}
```
通过代码可以看到，注意分别实现的不同接口

1、老师是被观察者，他需要实现接口的方法

* 订阅/取消订阅：往集合中存放/移除观察者
* 发布：循环遍历观察者，调用观察者方法

2、学生是观察者，那么我们只需要给他个名字，实现观察者的方法即可

最后，我们就把观察者和被观察者关联起来，LessonStart （上课啦）

```
public class LessonStart {

    public static void main(String[] args){

        Observable teather = new Teather();

        Observer xiaoming = new Student();
        xiaoming.setName("小明");
        Observer xiaohong = new Student();
        xiaohong.setName("小红");

        teather.attach(xiaoming);
        teather.attach(xiaohong);

        teather.notifyObservers("点名啦");
    }
}
```

代码很简单，我们模拟了一个老师和小明同学和小红同学，老师已经知道看到两个人来了，那么可以开始点名了，下面通过Log打印出信息

```
点名啦:小明到
点名啦:小红到
```



##**RxJava2的基本使用**

首先我先贴出我们后面所用到的第三方依赖库，以免后面忘记说了，大家对号入座

```
//retrofit
compile 'com.squareup.retrofit2:retrofit:2.2.0'
compile 'com.squareup.retrofit2:converter-gson:2.2.0'
//rx+retrofit
compile 'com.squareup.retrofit2:adapter-rxjava2:2.2.0'
//rxjava
compile "io.reactivex.rxjava2:rxjava:2.0.8"
compile 'io.reactivex.rxjava2:rxandroid:2.0.1'
//rxbinding
compile 'com.jakewharton.rxbinding2:rxbinding:2.0.0'
compile 'com.jakewharton.rxbinding2:rxbinding-support-v4:2.0.0'
compile 'com.jakewharton.rxbinding2:rxbinding-appcompat-v7:2.0.0'
compile 'com.jakewharton.rxbinding2:rxbinding-design:2.0.0'
compile 'com.jakewharton.rxbinding2:rxbinding-recyclerview-v7:2.0.0'
```
其次还需要添加联网权限

```
<uses-permission android:name="android.permission.INTERNET" />
```

最后我们回到正题，看完上面的例子，我们可以知道RxJava就是这种订阅和发布的模式，换成我们的RxJava代码应该是怎么样的？当然也是通过被观察者订阅观察者啦

```
//拿到被观察者
Observable<String> observable = getObservable();
//拿到观察者
Observer<String> observer = getObserver();
//订阅
observable.subscribe(observer);
```

我们具体被观察者和观察者的实现，当然是创建出来啦

```
public Observable<String> getObservable() {
       return Observable.create(new ObservableOnSubscribe<String>() {
           @Override
           public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
               e.onNext("俊俊俊很帅");
               e.onNext("你值得拥有");
               e.onNext("取消关注");
               e.onNext("但还是要保持微笑");
               e.onComplete();
           }
       });
}
```

onNext方法就是我们的发布过程，我们看其观察者的创建就知道怎么回事了

```
public Observer<String> getObserver() {
    return new Observer<String>() {

        Disposable disposable = null;

        @Override
        public void onSubscribe(Disposable d) {
            disposable = d;
        }

        @Override
        public void onNext(String s) {
            Log.e("onNext", s);
            if (s.equals("取消关注")) {
                //断开订阅
                disposable.dispose();
            }
        }

        @Override
        public void onError(Throwable e) {

        }

        @Override
        public void onComplete() {
            Log.e("onComplete", "onComplete");
        }
    };
}
```

我们可以发现，观察者的创建实现的方法，在被观察者中是对应起来的，也就是说，我们发布了什么，就可以在观察者中收到订阅信息，那么我们就可以在代码中编写我们的逻辑了，这样基本上已经使用好了RxJava了，通过Log打印出信息

1、人类就喜欢酷炫，炫耀，当然RxJava也少不了人类这种心理，就是链式编程，下面这段代码可以完美替代上面的所有代码

```
//创建被观察者
Observable.create(new ObservableOnSubscribe<String>() {
    @Override
    //默认在主线程里执行该方法
    public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
        e.onNext("俊俊俊很帅");
        e.onNext("你值得拥有");
        e.onNext("取消关注");
        e.onNext("但还是要保持微笑");
        e.onComplete();
    }
})
//将被观察者切换到子线程
.subscribeOn(Schedulers.io())
//将观察者切换到主线程
.observeOn(AndroidSchedulers.mainThread())
//创建观察者并订阅
.subscribe(new Observer<String>() {
    @Override
    public void onSubscribe(Disposable d) {

    }

    @Override
    public void onNext(String s) {
        Log.e("onNext", s);
    }

    @Override
    public void onError(Throwable e) {

    }

    @Override
    public void onComplete() {

    }
});
```
这里我多写了两个方法，也就是.subscribeOn(Schedulers.io())和.observeOn(AndroidSchedulers.mainThread())，这里就是RxJava的好处之一，他可以手动切换线程，这两个方法在这里表示被观察者创建实现的方法都放在io线程也就是子线程，因为在被观察者中通常会调用网络数据请求，那么网络请求必须在子线程运行，当网络请求收到后，则发布出去，在观察者中通过TextView等控件展示在界面上，那么UI的更新必须在主线程进行，也就是我们上面的代码mainThread。如果你不深造RxJava，基本上这两个方法已经成了固定的写法，这也是很多初学者忘记添加上去的方法

2、久而久之，人类喜欢简洁，喜欢定制服务，巧了，RxJava也给你满足了，下面这段代码中，实现的方法跟上面的实现方法是对应起来的，大家看参数就知道哪个对应哪个了，你可以通过new Consumer，不需要实现的方法你可以不写，看上去更简洁，这里我为了方便大家看，都new出来了，Consumer就是消费者的意思，可以理解为消费了onNext等等等事件

```
Observable<String> observable = getObservable();
observable.subscribe(new Consumer<String>() {
    @Override
    public void accept(@NonNull String s) throws Exception {
        Log.e("accept", s);
    }
}, new Consumer<Throwable>() {
    @Override
    public void accept(@NonNull Throwable throwable) throws Exception {

    }
}, new Action() {
    @Override
    public void run() throws Exception {

    }
}, new Consumer<Disposable>() {
    @Override
    public void accept(@NonNull Disposable disposable) throws Exception {

    }
});
```
哦，对了，我们还忘记打印Log信息，不能否认了我很帅这个事实

```
04-03 01:32:48.445 13512-13512/com.handsome.boke2 E/onNext: 俊俊俊很帅
04-03 01:32:48.446 13512-13512/com.handsome.boke2 E/onNext: 你值得拥有
04-03 01:32:48.446 13512-13512/com.handsome.boke2 E/onNext: 取消关注
```
当然你觉得只要夸奖我一个帅就行了，那么你也可以通过下面这几种方法发送给观察者

```
public Observable<String> getObservable() {
    //1、可发送对应的方法
    return Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(@NonNull ObservableEmitter<String> e) throws Exception {
            e.onNext("俊俊俊很帅");
            e.onNext("你值得拥有");
            e.onNext("取消关注");
            e.onNext("但还是要保持微笑");
            e.onComplete();
        }
    });
    //2、发送多个数据
    return Observable.just("俊俊俊很帅","你值得拥有","取消关注","但还是要保持微笑");
    //3、发送数组
    return Observable.fromArray("俊俊俊很帅","你值得拥有","取消关注","但还是要保持微笑");
    //4、发送一个数据
    return Observable.fromCallable(new Callable<String>() {
        @Override
        public String call() throws Exception {
            return "俊俊俊很帅";
        }
    });
}
```
##**模拟发送验证码**

这里的案例使用我们平时最简单的需求，看效果图就知道（图片会卡，效果大家脑补）

![这里写图片描述](http://img.blog.csdn.net/20170403145436628?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这里是整个代码的实现思路，我会在代码下面注释一下需要注意的点，代码我就直接贴出来，有句话说得好，成为大神，就必须先学会阅读别人的代码，哈哈哈，我的代码阅读起来应该没问题的吧

```
public class ButtonEnableActivity extends AppCompatActivity implements View.OnClickListener {
    private Button button;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_button_enable);
        button = (Button) findViewById(R.id.button);
        button.setOnClickListener(this);
    }
    @Override
    public void onClick(View v) {
        final long count = 3;
        Observable.interval(0, 1, TimeUnit.SECONDS)
                .take(count + 1)
                .map(new Function<Long, Long>() {
                    @Override
                    public Long apply(@NonNull Long aLong) throws Exception {
                        return count - aLong;
                    }
                })
                .observeOn(AndroidSchedulers.mainThread())
                .doOnSubscribe(new Consumer<Disposable>() {
                    @Override
                    public void accept(@NonNull Disposable disposable) throws Exception {
                        button.setEnabled(false);
                        button.setTextColor(Color.BLACK);
                    }
                })
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onSubscribe(Disposable d) {}
                    @Override
                    public void onNext(Long aLong) {
                        button.setText("剩余" + aLong + "秒");
                    }
                    @Override
                    public void onError(Throwable e) {}
                    @Override
                    public void onComplete() {
                        button.setEnabled(true);
                        button.setTextColor(Color.RED);
                        button.setText("发送验证码");
                    }
                });
    }
}
```

1、操作符

* 像这种interval、take、map、observeOn、doOnSubscribe、subscribe都是属于RxJava的操作符，简单的说就是实现某个方法，里面的功能都被包装起来了
* RxJava支持的操作符很多，很多操作符用起来都简单，但是组合起来就很复杂，功能很强大，具体分类如图所示

![这里写图片描述](http://img.blog.csdn.net/20170403142323692?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2、操作符介绍

* interval：延时几秒，每隔几秒开始执行
* take：超过多少秒停止执行
* map：类型转换，由于是倒计时，案例需要将倒计时的数字反过来
* observeOn：在主线程运行
* doOnSubscribe：在执行的过程中
* subscribe：订阅

##**RxJava2与Retrofit的使用**

RxJava与Retrofit的使用，更像我们的AsyncTask，通过网络获取数据然后通过Handler更新UI

##**模拟用户登陆获取用户数据**

人类总是喜欢看图说话，巧了，我给你提供了，我只能拿出我过硬的美工技术给你们画图了

![这里写图片描述](http://img.blog.csdn.net/20170403135258447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

① Bean对象

```
public class UserParam {
    private String param1;
    private String param2;
    public UserParam(String param1, String param2) {
        this.param1 = param1;
        this.param2 = param2;
    }
    public String getParam1() {
        return param1;
    }
    public void setParam1(String param1) {
        this.param1 = param1;
    }
    public String getParam2() {
        return param2;
    }
    public void setParam2(String param2) {
        this.param2 = param2;
    }
    @Override
    public String toString() {
        return new Gson().toJson(this);
    }
}
```
* 这里我们采用的是httpbin的一个post接口，各位可以在它的网站试一下，这里的NetBean是通过请求返回的数据，进行GsonFormat生成的
```
public class NetBean {
    private FormBean form;
    public FormBean getForm() {
        return form;
    }
    public void setForm(FormBean form) {
        this.form = form;
    }
    public static class FormBean {
        private String username;
        private String password;
        public String getPassword() {
            return password;
        }
        public void setPassword(String password) {
            this.password = password;
        }
        public String getUsername() {
            return username;
        }
        public void setUsername(String username) {
            this.username = username;
        }
    }
}
```

```
public class UserBean {
    private String username;
    private String passwrod;
    public UserBean(String passwrod, String username) {
        this.passwrod = passwrod;
        this.username = username;
    }
    public String getPasswrod() {
        return passwrod;
    }
    public void setPasswrod(String passwrod) {
        this.passwrod = passwrod;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    @Override
    public String toString() {
        return new Gson().toJson(this);
    }
}
```
② ApiService

* 这里返回Observable对象，也就是我们RxJava的被观察者
```
public interface ApiService {
    @FormUrlEncoded
    @POST("/post")
    Observable<NetBean> getUserInfo(@Field("username")String username, @Field("password")String password);
}
```
③ RxJava+Retrofit的实现

```
public class RxLoginActivity extends AppCompatActivity {

    ApiService apiService;
    TextView tv_text;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_rx_login);
        tv_text = (TextView) findViewById(R.id.tv_text);

        //构建Retrofit
        apiService = new Retrofit.Builder()
                .baseUrl("https://httpbin.org/")
                //rx与Gson混用
                .addConverterFactory(GsonConverterFactory.create())
                //rx与retrofit混用
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .build()
                .create(ApiService.class);

        //构建RxJava
        UserParam param = new UserParam("hensen", "123456");
        //发送param参数
        Observable.just(param)
                .flatMap(new Function<UserParam, ObservableSource<NetBean>>() {
                    @Override
                    public ObservableSource<NetBean> apply(@NonNull UserParam userParam) throws Exception {
                        //第一步：发送网络请求，获取NetBean
                        return apiService.getUserInfo(userParam.getParam1(), userParam.getParam2());
                    }
                })
                .flatMap(new Function<NetBean, ObservableSource<UserBean>>() {
                    @Override
                    public ObservableSource<UserBean> apply(@NonNull NetBean netBean) throws Exception {
                        UserBean user = new UserBean(netBean.getForm().getUsername(), netBean.getForm().getPassword());
                        //第二步：转换netBean数据为我们需要的UserBean类型
                        return Observable.just(user);
                    }
                })
                //将被观察者放在子线程，将观察者放在主线程
                .subscribeOn(Schedulers.io()).observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<UserBean>() {
                    @Override
                    public void accept(@NonNull UserBean userBean) throws Exception {
                        //第三步：接收第二步发送过来的数据，进行UI更新
                        tv_text.setText("用户名：" + userBean.getUsername() + "--密码：" + userBean.getPasswrod());
                    }
                });
    }
}
```
1、Retrofit

* RxJava与Retrofit一起使用必须在Retrofit上加上这句话addCallAdapterFactory(RxJava2CallAdapterFactory.create())

2、RxJava

* 我们通过just方式发送数据
* flatMap方法是用于数据格式转换的方法，其后面的参数UserParam与ObservableSource< NetBean>，参数一表示原数据，参数二表示转换的数据，那么就是通过发送网络参数，转换成网络返回的数据，调用Retrofit


##**合并本地与服务器购物车列表**

这个案例其实就是用户添加购物车的时候，首先会在本地存储一份，然后发现如果没有网络，那么没办法提交到服务器上，只能等下一次有网络的时候采用本地数据库和服务器数据的合并来实现上传到服务器，这里我们就贴RxJava与Retrofit的代码，不贴其他代码了，废话不多说，上图

![这里写图片描述](http://img.blog.csdn.net/20170403142936695?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
public class CartMegerActivity extends AppCompatActivity implements View.OnClickListener {
    private Button button;
    private ApiService apiService;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_cart_meger);
        apiService = new Retrofit.Builder()
                .baseUrl("http://httpbin.org/")
                .addCallAdapterFactory(RxJava2CallAdapterFactory.create())
                .addConverterFactory(GsonConverterFactory.create())
                .build()
                .create(ApiService.class);
        button = (Button) findViewById(R.id.button);
        button.setOnClickListener(this);
    }
    @Override
    public void onClick(View v) {
        Observable.merge(getDataForLocal(), getDataForNet()).subscribe(new Observer<List<String>>() {
            @Override
            public void onSubscribe(Disposable d) {}
            @Override
            public void onNext(List<String> strings) {
                for (String str : strings){
                    System.out.println(str);
                }
            }
            @Override
            public void onError(Throwable e) {}
            @Override
            public void onComplete() {
                System.out.println("onComplete");
            }
        });
    }
    private Observable<List<String>> getDataForLocal() {
        List<String> list = new ArrayList<>();
        list.add("购物车物品一");
        list.add("购物车物品二");
        return Observable.just(list);
    }
    private Observable<List<String>> getDataForNet() {
        return Observable.just("购物车物品三").flatMap(new Function<String, ObservableSource<NetBean>>() {
            @Override
            public ObservableSource<NetBean> apply(@NonNull String s) throws Exception {
                return apiService.getCartList(s);
            }
        }).flatMap(new Function<NetBean, ObservableSource<List<String>>>() {
            @Override
            public ObservableSource<List<String>> apply(@NonNull NetBean netBean) throws Exception {
                String shop = netBean.get_$Args257().getShopName();
                List<String> list = new ArrayList<>();
                list.add(shop);
                return Observable.just(list);
            }
        }).subscribeOn(Schedulers.io());
    }
}
```

这里使用到merge的操作符，其表示意思就是将两个ObservableSource合并为一个ObservableSource，最后的打印结果是

```
04-03 02:37:28.840 5615-5615/com.handsome.boke2 I/System.out: 购物车物品一
04-03 02:37:28.840 5615-5615/com.handsome.boke2 I/System.out: 购物车物品二
04-03 02:37:39.501 5615-6337/com.handsome.boke2 I/System.out: 购物车物品三
04-03 02:37:39.501 5615-6337/com.handsome.boke2 I/System.out: onComplete
```

##**RxJava2与RxBinding的使用**

RxBinding的使用也是为了让界面看起来更简洁，剩去了传统的findViewById和setOnClickListener的方法，不用任何声明，只要添加依赖就可以直接使用了

##**优化搜索请求**

这里的案例是说当我们在EditText打字实时搜索的时候，可能用户会打字很会快，那么我们就没有必要一直发送网络请求，请求搜索结果，我们可以通过当用户打字停止后的延时500毫秒再发送搜索请求

```
public class TextChangeActivity extends AppCompatActivity {
   private  EditText edittext;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_text_change);
        edittext = (EditText) findViewById(R.id.edittext);

        RxTextView.textChanges(edittext)
                //当你敲完字之后停下来的半秒就会执行下面语句
                .debounce(500, TimeUnit.MILLISECONDS)
                //下面这两个都是数据转换
                //flatMap：当同时多个网络请求访问的时候，前面的网络数据会覆盖后面的网络数据
                //switchMap：当同时多个网络请求访问的时候，会以最后一个发送请求为准，前面网路数据会被最后一个覆盖
                .switchMap(new Function<CharSequence, ObservableSource<List<String>>>() {
                    @Override
                    public ObservableSource<List<String>> apply(@NonNull CharSequence charSequence) throws Exception {
                        //网络操作，获取我们需要的数据
                        List<String> list = new ArrayList<String>();
                        list.add("2017年款最新帅哥俊俊俊");
                        list.add("找不到2017年比俊俊俊更帅的人");
                        return Observable.just(list);
                    }
                })
                //网络请求是在子线程的
                .subscribeOn(Schedulers.io())
                //界面更新在主线程
                .observeOn(AndroidSchedulers.mainThread())
                .subscribe(new Consumer<List<String>>() {
                    @Override
                    public void accept(@NonNull List<String> strings) throws Exception {
                        //界面更新，这里用打印替代
                        System.out.println(strings.toString());
                    }
                });
    }
}
```

操作符

* RxTextView.textChanges(edittext)：Rxbinding用法
* debounce：表示延时多少秒后执行
* switchMap：也是数据转换，与flatMap的区别在注释中解释很清楚了

演示效果图

![这里写图片描述](http://img.blog.csdn.net/20170403144413749?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

##**优化点击请求**

这个案例很简单，当用户一直点击一个按钮的时候，我们不应该一直调用访问网络请求，而是	1秒内，只执行一次网络请求

```
public class ButtonClickActivity extends AppCompatActivity {
    private Button button;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_button_click);
        button = (Button) findViewById(R.id.button);
        RxView.clicks(button).throttleFirst(1, TimeUnit.SECONDS)
                .subscribe(new Observer<Object>() {
                    @Override
                    public void onSubscribe(Disposable d) {}
                    @Override
                    public void onNext(Object o) {
                        System.out.println("俊俊俊点击了按钮");
                    }
                    @Override
                    public void onError(Throwable e) {}
                    @Override
                    public void onComplete() {}
                });
    }
}
```

##**源码下载**

[源码下载](http://download.csdn.net/detail/qq_30379689/9802660)

##**结语**

听说认真看的同学都会学的很多哦，对于RxJava的操作符还是建议大家通过官网的wiki去深造，只有当RxJava用在项目中的时候，你才会体会它的好处，同时也让你与初级工程师有了一定的距离，像很多RxBus、RxPermission、RxAndroid，很多人会疑问要不要去学习它，毫无疑问是必须学习的，技术是不断更新，只有当你的技术跟上时代的时候，你才有可能和大神聊的津津有味，以上纯属瞎掰，当然有时间我也会去学习Rx系列，如果喜欢我的朋友可以关注我的博客，或者加群大家一起学习吧