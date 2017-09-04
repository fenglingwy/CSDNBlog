#Android基础——框架模式MVC在安卓中的实践


----------
>本篇文章包含以下内容：

>* MVC的介绍
>* MVC的实践

##MVC的介绍

MVC (Model View Controller)，是模型（model）视图（view）控制器（controller）的缩写，一种软件设计模式，用于组织代码用一种功能模块和数据模块分离的方法

>* Model：模型层，负责处理数据的加载或者存储
>* View：视图层，负责界面数据的展示，与用户进行交互
>* Controller：控制器层，负责逻辑业务的处理

MVC的模型关系图：

![这里写图片描述](http://img.blog.csdn.net/20170713191619715?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

MVC设计执行的基本流程：
>首先视图接受用户输入请求，然后将请求传递给控制器，控制器再调用某个模型来处理用户的请求，在控制器的控制下，再将处理后的结果交给某个视图进行格式化输出给用户。另外，View是可以直接访问Model来进行数据的处理的

对MVC的认识：
>在MVC里，View是可以直接访问Model的。从而，View里会包含Model信息，不可避免的还要包括一些业务逻辑。 在MVC模型里，更关注的Model的不变，而同时有多个对Model的不同显示，及View。所以，在MVC模型里，Model不依赖于View，但是View是依赖于Model的。不仅如此，因为有一些业务逻辑在View里实现了，导致要更改View也是比较困难的，至少那些业务逻辑是无法重用的

MVC优点：

>* 耦合性低
>* 重用性高
>* 生命周期成本低
>* 部署快
>* 可维护性高
>* 有利软件工程化管理

MVC缺点：

>* 没有明确的定义
>* 不适合小型，中等规模的应用程序
>* 增加系统结构和实现的复杂性
>* 视图与控制器间的过于紧密的连接
>* 视图对模型数据的低效率访问
>* 一般高级的界面工具或构造器不支持模式

----------


##MVC的实践##

明白了MVC是什么之后，我们使用个小案例来深入了解

我们采用ListView来演示我们的MVC模式，目录结构：

![](http://img.blog.csdn.net/20161024113610530)

实体类：包含了书的名字和图片信息

```
/**
 * 作者：许英俊
 * 实体类
 * 对数据对象的封装
 */
public class Book {

    //书名
    private String name;
    //书的图片
    private int image;

    public Book(String name, int image) {
        this.name = name;
        this.image = image;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getImage() {
        return image;
    }

    public void setImage(int image) {
        this.image = image;
    }
}
```
模型类（Model层）：通常是对本地数据库的操作或者是通过网络请求获取网络数据的操作
我们在Model里面模拟了一个本地数据库，并提供了增删改查的方法

```
/**
 * 作者：许英俊
 * 模型层
 * 对数据库的操作
 */
public class BookModel {

    private static List<Book> list = new ArrayList<>();

    /**
     * 模拟本地数据库
     */
    static {
        list.add(new Book("Java从入门到精通", R.drawable.java));
        list.add(new Book("Android从入门到精通", R.drawable.android));
        list.add(new Book("Java从入门到精通", R.drawable.java));
        list.add(new Book("Android从入门到精通", R.drawable.android));
    }

    /**
     * 添加书本
     * @param name
     * @param image
     */
    public void addBook(String name, int image) {
        list.add(new Book(name, image));
    }

    /**
     * 删除书本
     */
    public void deleteBook( ) {
        list.remove(list.size() - 1);
    }

    /**
     * 查询数据库所有书本
     * @return
     */
    public List<Book> query() {
        return list;
    }

}
```
控制器（Controller层）：根据Model层的方法，加上我们的业务逻辑处理，对外提供方法并暴露接口
看delete这个方法，判断List是否为空（业务逻辑），使用mode.deleteBook()（Model层方法），通过listener.onComplete()（暴露接口）

```
/**
 * 作者：许英俊
 * 控制器层
 * 处理业务逻辑，调用模型层的操作，并对外暴露接口
 */
public class BookController {

    private BookModel mode;

    public BookController() {
        mode = new BookModel();
    }

    /**
     * 添加书本
     * @param listener
     */
    public void add(onAddBookListener listener) {
        mode.addBook("JavaWeb从入门到精通", R.drawable.javaweb);
        if (listener != null) {
            listener.onComplete();
        }
    }

    /**
     * 删除书本
     * @param listener
     */
    public void delete(onDeleteBookListener listener) {
        if(mode.query().isEmpty()){
           return;
        }else{
            mode.deleteBook();
        }
        if (listener != null) {
            listener.onComplete();
        }
    }

    /**
     * 查询所有书本
     * @return
     */
    public List<Book> query() {
        return mode.query();
    }
    
    /**
     * 添加成功的回调接口
     */
    public interface onAddBookListener {
        void onComplete();
    }
   
    /**
     * 删除成功的回调接口
     */
    public interface onDeleteBookListener {
        void onComplete();
    }
}
```
视图（View层）：我们操作Controller获取List数据填充到ListView中，同时可以添加书本和删除书本

```
/**
 * 作者：许英俊
 * 视图层
 * 发送输入请求给控制器
 */
public class BookActivity extends AppCompatActivity implements View.OnClickListener {

    private BookController bookController;

    private ListView lv_book;
    private List<Book> list;
    private BookAdapter adapter;
    private Button bt_add, bt_delete;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_book);
        lv_book = (ListView) findViewById(R.id.lv);
        bt_add = (Button) findViewById(R.id.bt_add);
        bt_delete = (Button) findViewById(R.id.bt_delete);
        bt_add.setOnClickListener(this);
        bt_delete.setOnClickListener(this);

        bookController = new BookController();
        list = bookController.query();
        adapter = new BookAdapter(this, list);
        lv_book.setAdapter(adapter);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            //添加书本按钮
            case R.id.bt_add:
                bookController.add(new BookController.onAddBookListener() {
                    @Override
                    public void onComplete() {
                        adapter.notifyDataSetChanged();
                    }
                });
                break;
            //删除书本按钮
            case R.id.bt_delete:
                bookController.delete(new BookController.onDeleteBookListener() {
                    @Override
                    public void onComplete() {
                        adapter.notifyDataSetChanged();
                    }
                });
                break;
        }
    }
}
```


----------


##效果图##
![](http://img.blog.csdn.net/20161024114654611)

##源码##
github：https://github.com/AndroidHensen/Design-Mode