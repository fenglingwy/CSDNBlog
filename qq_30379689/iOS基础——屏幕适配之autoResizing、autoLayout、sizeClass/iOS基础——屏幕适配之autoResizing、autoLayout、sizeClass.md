#iOS基础——屏幕适配之autoResizing、autoLayout、sizeClass


----------
##**一、autoResizing**
1、autoResizing的出现

在iOS6之前，屏幕为了支持横屏，开始出现autoResizing作为屏幕适配的首选

2、autoResizing缺点

autoResizing只能指定View与父View之间的适配

3、autoResizing的使用

autoResizing使用简单，在点击每个View的右侧设置中有个autoResizing的选项，其中autoresizing左侧图中有六条线，分别是上下左右距离外边框四条，还有内边框里面的两条，其中六条线都是可以选中并会自动变成红色，表示你使用了该autoResizing

![](http://img.blog.csdn.net/20170312211942220?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当屏幕尺寸变化时，勾选中

1. 左边线：表示View距离父View左边距距离不变，其余方向会随屏幕放大而放大
2. 右边线：表示View距离父View右边距距离不变，其余方向会随屏幕放大而放大
3. 上边线：表示View距离父View上边距距离不变，其余方向会随屏幕放大而放大
4. 下边线：表示View距离父View下边距距离不变，其余方向会随屏幕放大而放大
5. 中间横线：表示View的宽度会随屏幕放大而放大
6. 中间竖线：表示View的高度会随屏幕放大而放大

4、autoResizing代码实现

可以通过来autoresizingMask属性设置autoResizing，例如

```
view.autoresizingMask = UIViewAutoresizingFlexibleLeftMargin|UIViewAutoresizingFlexibleRightMargin......
```

对应其他的API有

```objc
typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
    UIViewAutoresizingNone                 = 0, 
    UIViewAutoresizingFlexibleLeftMargin   = 1 << 0,
    UIViewAutoresizingFlexibleWidth        = 1 << 1,
    UIViewAutoresizingFlexibleRightMargin  = 1 << 2,
    UIViewAutoresizingFlexibleTopMargin    = 1 << 3,
    UIViewAutoresizingFlexibleHeight       = 1 << 4,
    UIViewAutoresizingFlexibleBottomMargin = 1 << 5
};
```

##**二、autoLayout**

1、autoLayout的出现

为了解决autoResizing的缺点，在iOS6之后出现了autoLayout，用来支持View与View之间的位置和间距

2、autoLayout的介绍

autoLayout有两个重要的要素：参照物和约束，只要参照物和约束被确定就能确定View的具体位置，即确定了View的x和y的值，还有宽高

3、autoLayout的使用

① 可以通过storyboard中的ViewController设置开启autoLayout

![这里写图片描述](http://img.blog.csdn.net/20170312225202603?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 拖进一个ImageView，可以点击左下角按钮，可以设置autoLayout的属性

![](http://img.blog.csdn.net/20170312225353325?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

③ 或者也可以选择两个View之间，点击右下角按钮，设置两者之间的autoLayout属性

![](http://img.blog.csdn.net/20170312225719002?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

④ 设置后属性后，可以通过右下角按钮最左边一个，让其更新位置

![这里写图片描述](http://img.blog.csdn.net/20170312230221228?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


4、属性介绍

一个属性相当于一个约束，所以项目中用到的autoLayout的约束有多个

① 上面一层则是自身View与其他View（通过下箭头可查看）的边距（需要选中一个View）
② 下面则是两View之间的宽高以及对齐方式（需要选中一个View）

![这里写图片描述](http://img.blog.csdn.net/20170312230638021?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

③ 这一层主要是两View之间的对齐方式，具体可以看蓝色的图标（需要选中两个View）

![](http://img.blog.csdn.net/20170313000915831?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

4、autoLayout代码的使用

每一条约束都是一个NSLayoutConstraint对象，如果例子中有十几个约束，那么就需要十几条代码，很麻烦，基本上没这么用，每条代码都必须实现下面对应的属性

```
NSLayoutConstraint *layout = [NSLayoutConstraint constraintWithItem:(id) 
														  attribute:(NSLayoutAttribute) 
														  relatedBy:NSLayoutRelationEqual 
														     toItem:(id) 
														  attribute:(NSLayoutAttribute) 
														 multiplier:(CGFloat) 
														   constant:(CGFloat)];
[self.view addConstraint:layoutConstraint];
```

5、VFL

为了解决代码实现起来比较长比较多，所以提供了VFL可视化约束语言进行编写代码，其语法如下

```
//表示水平方向上距离两边的边距只有20
[self.view addConstraints:[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-20-[myView]-20-|"
																  options:NSLayoutFormatAlignAllTop               
																  metrics:nil 
																    views:@{"myView":myView}]];
```

6、Mansory第三方的使用

为了减少代码的编写量和可阅读性，可使用第三方库实现代码的约束，其使用先导入依赖库，之后语法如下所示

```java
//表示约束是距离上左右20，高度为40
[myView makeConstraints:^(MASConstraintMaker *make){
	make.top.left.offset(20);
	make.right.offset(-20);
	make.height.equalTo(40);
}];
```

7、autoLayout的计算公式

① 打开storyboard右手边可以看到每个约束的存在

![](http://img.blog.csdn.net/20170312233046533?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 点击每个约束可以在右手边看到每个约束的属性

![这里写图片描述](http://img.blog.csdn.net/20170312233114398?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

③ 其属性的计算公式如下，可以利用修改属性值，改变约束View的位置

```
First Item = Second Item * Multiplier + Constant
```
8、autoLayout在UITableView的使用

* 在UITableViewCell创建约束的时候，最后一个View记得与底部有个边距约束，这样才能让Cell的高度被确定
* 在UITableViewCell填充数据的时候，也可以将约束作为变量进行赋值，这样可以控制Cell中的控件的宽高和边距
* 在UITableView中记得设置rowHeight和给定一个随机预估高度，这样才能让Cell显示出autoLayout后的高度，代码如下

```
self.tableView.estimatedRowHeight = 10;
self.tableView.rowHeight = UITableViewAutomaticDimension;
```



##**三、sizeClass**

1、sizeClass的出现

屏幕大了，尺寸多了，带给开发者的自然是适配方面的工作量和思考，iOS推出可行的解决方案就是sizeClass

2、sizeClass的介绍

sizeClass是对屏幕尺寸进行了抽象，不在拘泥于具体尺寸，因为尺寸一直都在变化，我们如果按照尺寸去做适配，一定会很累的

3、sizeClass的分类

* compact (紧凑-小)
* Any (任意)
* Regular (宽松-大)

4、sizeClass的使用

在storyboard下面的按钮中有w Any、h Any，可以点击选项，设置storyboard里的ViewController的横竖屏在不同尺寸屏幕中的显示

![这里写图片描述](http://img.blog.csdn.net/20170312233753111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

选择好屏幕的分类之后，就可以在storyboard中进行界面的设计了，那么其对应的就是设置好的分类的屏幕大小的界面

