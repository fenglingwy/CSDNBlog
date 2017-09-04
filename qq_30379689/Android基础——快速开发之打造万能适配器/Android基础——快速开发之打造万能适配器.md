#快速开发之打造万能适配器


----------
>本篇文章包含以下内容：

>* 官方推荐的Adapter写法
>* 美化官网推荐Adapter写法
>* 打造万能的ViewHolder
>* 打造万能的Adapter
>* 使用万能的Adapter

#官方推荐的Adapter写法
这里以ListView作演示，对于ListView我们再熟悉不过了，其步骤分为：

* 创建ListView的Bean对象
* 创建ListView的Adapter的ItemView布局
* 创建ListView的Adaoter（**重点）
* 对ListView设置Adapter

##一、创建ListView的Bean对象

这里以学生信息为例

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
##二、创建ListView的Adapter的ItemView布局

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="horizontal"
    android:padding="8dp">

    <ImageView
        android:id="@+id/iv"
        android:layout_width="60dp"
        android:layout_height="60dp" />

    <TextView
        android:id="@+id/tv"
        android:padding="8dp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</LinearLayout>
```
其最终效果为

![](http://img.blog.csdn.net/20161104020002209)

##三、创建ListView的Adaoter（**重点）

这里采用官网的Adapter推荐写法，【你可以发现：Adapter缓存的只是每个ItemView的ViewHolder】

```
public class StudentAdapter2 extends BaseAdapter {

    private LayoutInflater inflater;
    private List<Student> list;

    public StudentAdapter2(Context context, List<Student> list) {
        inflater = LayoutInflater.from(context);
        this.list = list;
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Override
    public Object getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder holder;
        if (convertView == null) {
            convertView = inflater.inflate(R.layout.adapter_student, null);
            holder = new ViewHolder();
            holder.tv = (TextView) convertView.findViewById(R.id.tv);
            holder.iv = (ImageView) convertView.findViewById(R.id.iv);
            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }
        holder.tv.setText(list.get(position).getName());
        holder.iv.setBackgroundResource(list.get(position).getImage());
        return convertView;
    }

    static class ViewHolder {
        TextView tv;
        ImageView iv;
    }
}
```
如果我们需要填充ItemView很多，那么getView()方法里面的代码会变得臃肿，难以阅读和修改，那我们就来美化一下代码吧

##四、对ListView设置Adapter
>容我一个强迫症的人先把这一点写完

```
lv = (ListView) findViewById(R.id.lv_student);

list = new ArrayList<>();
list.add(new Student("小龙", R.drawable.man));
list.add(new Student("小红", R.drawable.woman));
list.add(new Student("小龙", R.drawable.man));

adapter = new StudentAdapter2(this, list);
lv.setAdapter(adapter);
```

#美化官网推荐Adapter写法


----------
这里我们就只看Adapter的代码美化，具体看getView()这个方法里面的内容，前面说了，Adapter缓存的只是ViewHolder，那么我们抽取这个ViewHolder

```
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    if (convertView == null) {
        convertView = inflater.inflate(R.layout.adapter_student, null);
    }
    ViewHolder viewHolder = getViewHolder(convertView);
    viewHolder.iv.setBackgroundResource(list.get(position).getImage());
    viewHolder.tv.setText(list.get(position).getName());
    return convertView;
}

private ViewHolder getViewHolder(View view) {
    ViewHolder viewHolder = (ViewHolder) view.getTag();
    if (viewHolder == null) {
        viewHolder = new ViewHolder(view);
        view.setTag(viewHolder);
    }
    return viewHolder;
}

private class ViewHolder {
    private TextView tv;
    private ImageView iv;

    private ViewHolder(View view) {
        tv = (TextView) view.findViewById(R.id.tv);
        iv = (ImageView) view.findViewById(R.id.iv);
    }
}
```

我们将中间的getView()里的findViewById()和convertView.setTag()抽取出来，如果要添加新的View，只需要在ViewHolder类中添加即可，我们可以看到不管对View添加多少，在getView()方法中只需要一句话，ViewHolder viewHolder = getViewHolder(convertView)

#打造万能的ViewHolder


----------
从上面的代码分析，在我们万能的ViewHolder类需要做什么：

* convertView缓存的是ViewHolder（所以我们需要一个convertView，作为ViewHolder的属性）
* getViewHolder的代码固定的（所以我们需要提供一个getViewHolder()方法将其锁死）
* findViewById每次增加View的时候都要执行（所以我们需要抽象一个方法来getView()）

下面我们创建一个ViewHolder类（ViewHolder简单的理解为View的管理器）

```
public class ViewHolder {

    private SparseArray<View> views;
    private View convertView;

    public ViewHolder(Context context, ViewGroup parent, int layoutId) {
        views = new SparseArray<>();
        convertView = LayoutInflater.from(context).inflate(layoutId, parent, false);
        convertView.setTag(this);
    }

    public static ViewHolder getViewHolder(Context context, View convertView
            , ViewGroup parent, int layoutId) {
        if (convertView == null) {
            return new ViewHolder(context, parent, layoutId);
        }
        return (ViewHolder) convertView.getTag();
    }

    public <T extends View> T getView(int viewId) {
        View view = views.get(viewId);
        if (view == null) {
            view = convertView.findViewById(viewId);
            views.put(viewId, view);
        }
        return (T) view;
    }

    public View getConvertView() {
        return convertView;
    }
}
```
ViewHolder类做了两件事情：

* getViewHolder()拿到这个ViewHolder对象
* 通过viewHolder.getView()方法来对View进行填充数据

#打造万能的Adapter


----------
万能Adapter很简单，就是在类里面用泛型T表示传进来的Bean对象，剩下的就是调用ViewHolder的事情

我们知道ViewHolder只是做了两件事情，那么我们就可以在Adapter中，调用这两件事情

```
public abstract class Adapter<T> extends BaseAdapter {

    protected LayoutInflater inflater;
    protected Context context;
    protected List<T> list;
    protected int layoutId;

    public Adapter(Context context, List<T> list, int layoutId) {
        inflater = LayoutInflater.from(context);
        this.context = context;
        this.list = list;
        this.layoutId = layoutId;
    }

    @Override
    public int getCount() {
        return list.size();
    }

    @Override
    public Object getItem(int position) {
        return list.get(position);
    }

    @Override
    public long getItemId(int position) {
        return position;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        ViewHolder viewHolder = getViewHolder(context, convertView, parent);
        //第二件事
        TextView tv = viewHolder.getView(R.id.tv);
        ImageView iv = viewHolder.getView(R.id.iv);
        tv.setText(((Student)list.get(position)).getName());
        iv.setBackgroundResource(((Student)list.get(position)).getImage());
        return viewHolder.getConvertView();
    }

    //第一件事
    private ViewHolder getViewHolder(Context context, View convertView
            , ViewGroup parent) {
        return ViewHolder.getViewHolder(context, convertView, parent, layoutId);
    }
}
```
我们看到getView()里的代码还是很多，不美观，根据面向对象的思想，我们可以将其抽取为一个抽象方法，让我们的前台去填充这个View

```
@Override
public View getView(int position, View convertView, ViewGroup parent) {
    ViewHolder viewHolder = getViewHolder(context, convertView, parent);
    convert(viewHolder, list.get(position));
    return viewHolder.getConvertView();
}

public abstract void convert(ViewHolder viewHolder, T item);

private ViewHolder getViewHolder(Context context, View convertView
        , ViewGroup parent) {
    return ViewHolder.getViewHolder(context, convertView, parent, layoutId);
}
```
这个时候getView()里面的代码就只剩下一句话了

#使用万能的Adapter


----------
传统的使用：

```
adapter = new StudentAdapter(this, list);
lv.setAdapter(adapter);
```
万能Adapter的使用：

```
lv.setAdapter(new Adapter<Student>(this, list, R.layout.adapter_student) {
    @Override
    public void convert(ViewHolder viewHolder, Student item) {
        TextView tv = viewHolder.getView(R.id.tv);
        ImageView iv = viewHolder.getView(R.id.iv);
        tv.setText(item.getName());
        iv.setBackgroundResource(item.getImage());
    }
});
```
#总结：

* 两者比较有好有坏，传统的前台代码简洁，而万能适配器的代码臃肿，不过万能适配器可以适配各种ListView和GridView
* 如果不明白的话，将万能适配器代码调用，一层一层的往回拼凑，最后执行的代码顺序还是和官网推荐的一样，只不过他用泛型T来让所有对象都适用

[源码下载：由于我的Module东西多，这里我只抽取了使用到的文件](http://download.csdn.net/detail/qq_30379689/9672954)