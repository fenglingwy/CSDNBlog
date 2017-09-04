#Android基础——框架模式MVP在安卓中的实践


----------
>本篇文章包含一下内容：

>* MVP的介绍
>* MVP的实践

##MVP的介绍##
MVP模式（Model View Presenter）可以说是MVC模式（Model View Controller）在Android开发上的一种变种、进化模式
>* Model：数据层，负责处理数据的加载或者存储
>* View：视图层，负责界面数据的展示，与用户进行交互
>* Presenter：中间者，绑定Model层和View层，是Model与View之间的桥梁

MVP的模型关系图：
![](http://img.blog.csdn.net/20161024205559628)

MVP设计执行的基本流程：
>首先视图接受用户输入请求，然后将请求传递给Presenter，Presenter再调用某个模型来处理用户的请求，模型中修改数据后传递更新数据到Presenter中，Presenter再将处理后的结果交给视图进行格式化输出给用户

对MVP认识：
>MVP 是从经典的模式MVC演变而来，它们的基本思想有相通的地方：Controller/Presenter负责逻辑的处理，Model提供数据，View负责视图。作为一种新的模式，MVP与MVC有着一个重大的区别：在MVP中View并不直接使用Model，它们之间的通信是通过Presenter (MVC中的Controller)来进行的，所有的交互都发生在Presenter内部，而在MVC中View会从直接Model中读取数据而不是通过 Controller

MVP优点：
>* 模型与视图完全分离，我们可以修改视图而不影响模型
>* 可以更高效地使用模型，因为所有的交互都发生在一个地方——Presenter内部
>* 我们可以将一个Presenter用于多个视图，而不需要改变Presenter的逻辑。这个特性非常的有用，因为视图的变化总是比模型的变化频繁
>* 如果我们把逻辑放在Presenter中，那么我们就可以脱离用户接口来测试这些逻辑（单元测试）

MVP缺点：
>由于对视图的渲染放在了Presenter中，所以视图和Presenter的交互会过于频繁。还有一点需要明白，如果Presenter过多地渲染了视图，往往会使得它与特定的视图的联系过于紧密。一旦视图需要变更，那么Presenter也需要变更了

----------
##MVP的实践##
 
我们采用ListView来演示我们的MVP模式，目录结构：

![](http://img.blog.csdn.net/20161024210740503)

实体类：包含了学生的名字和图片信息

```
public class Student {

    //学生的名字
    private String name;
    //学生的图片信息
    private int image;

    public Student(String name, int image) {
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
模型类（Model层），模型类分为两个类：
1、模型抽象类：对模型的方法的抽象，方便阅读该模型层有哪些功能，类似于说明书
2、模型实现类：通常是对本地数据库的操作或者是通过网络请求获取网络数据的操作 
我们在Model里面模拟了一个本地数据库，并提供了增删改查的方法

```
/**
 * 作者：许英俊
 * 模型抽象类
 * 对模型层的抽象
 */
public interface IStudentMode {

    /**
     * 查询所有学生
     * @param listener
     */
    void query(onQueryListener listener);

    /**
     * 添加学生
     * @param listener
     */
    void addStudent(onAddStudentListener listener);

    /**
     * 删除学生
     * @param listener
     */
    void deleteStudent(onDeleteStudentListener listener);

    /**
     * 查询学生回调
     */
    interface onQueryListener{
        void onComplete(List<Student> students);
    }

    /**
     * 添加学生回调
     */
    interface onAddStudentListener{
        void onComplete();
    }

    /**
     * 删除学生回调
     */
    interface onDeleteStudentListener{
        void onComplete();
    }
}
```

```
/**
 * 作者：许英俊
 * 模型实现类
 * 对模型层的实现
 */
public class StudentMode implements IStudentMode {

    private static List<Student> list = new ArrayList<>();

    /**
     * 本地模拟数据库
     */
    static {
        list.add(new Student("小龙", R.drawable.man));
        list.add(new Student("小红", R.drawable.woman));
        list.add(new Student("小龙", R.drawable.man));
    }

    /**
     * 查询学生
     * @param listener
     */
    @Override
    public void query(onQueryListener listener) {
        if (listener != null) {
            listener.onComplete(list);
        }
    }

    /**
     * 添加学生
     * @param listener
     */
    @Override
    public void addStudent(onAddStudentListener listener) {
        list.add(new Student("小燕", R.drawable.girl));
        if (listener != null) {
            listener.onComplete();
        }
    }

    /**
     * 删除学生
     * @param listener
     */
    @Override
    public void deleteStudent(onDeleteStudentListener listener) {
        if (list.size() > 0) {
            list.remove(list.size() - 1);
        }
        if (listener != null) {
            listener.onComplete();
        }
    }
}
```
视图类（View层）：对视图的方法的抽象，方便阅读该视图层有哪些功能，类似于说明书

```
/**
 * 作者：许英俊
 * 视图类
 * 对视图方法的抽象
 */
public interface IStudentView {
    /**
     * 展示学生
     * @param list
     */
    void showStudent(List<Student> list);

    /**
     * 刷新学生
     */
    void refreshStudent();
}
```
中间者（Presenter）：绑定Model层和View层，操作Model数据，并在View更新

```
/**
 * 作者：许英俊
 * 中间者
 * 绑定View层和Model层
 */
public class StudentPresenter {

    private IStudentMode studentMode;
    private IStudentView studentView;

    public StudentPresenter(IStudentView studentView) {
        studentMode = new StudentMode();
        this.studentView = studentView;
    }

    /**
     * 通过Model查询学生，在View中展示
     */
    public void queryStudent(){
        studentMode.query(new IStudentMode.onQueryListener() {
            @Override
            public void onComplete(List<Student> students) {
                studentView.showStudent(students);
            }
        });
    }

    /**
     * 通过Model添加学生，在View中更新
     */
    public void addStudent(){
        studentMode.addStudent(new IStudentMode.onAddStudentListener() {
            @Override
            public void onComplete() {
                studentView.refreshStudent();
            }
        });
    }

    /**
     * 通过Model删除学生，在View中更新
     */
    public void deleteStudent(){
        studentMode.deleteStudent(new IStudentMode.onDeleteStudentListener() {
            @Override
            public void onComplete() {
                studentView.refreshStudent();
            }
        });
    }
}
```
视图类：Activity类中，实现我们的视图抽象类，并使用Presenter的方法	

```
/**
 * 作者：许英俊
 * 视图类
 * 对视图抽象类的实现
 */
public class StudentActivity extends AppCompatActivity implements IStudentView, View.OnClickListener {

    private ListView lv;
    private StudentAdapter adapter;
    private Button bt_add, bt_delete;
    private StudentPresenter presenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_student);
        lv = (ListView) findViewById(R.id.lv);
        bt_add = (Button) findViewById(R.id.bt_add);
        bt_delete = (Button) findViewById(R.id.bt_delete);
        bt_add.setOnClickListener(this);
        bt_delete.setOnClickListener(this);

        //中间者类
        presenter = new StudentPresenter(this);
        //查询学生
        presenter.queryStudent();
    }

    /**
     * 展示学生
     * @param list
     */
    @Override
    public void showStudent(List<Student> list) {
        adapter = new StudentAdapter(this, list);
        lv.setAdapter(adapter);
    }

    /**
     * 刷新学生界面
     */
    @Override
    public void refreshStudent() {
        adapter.notifyDataSetChanged();
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            //添加学生
            case R.id.bt_add:
                presenter.addStudent();
                break;
            //删除学生
            case R.id.bt_delete:
                presenter.deleteStudent();
                break;
        }
    }
}
```

##效果图##
![](http://img.blog.csdn.net/20161024213112754)

##源码##

github：https://github.com/AndroidHensen/Design-Mode