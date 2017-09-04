#Android实战——jsoup实现网络爬虫，爬糗事百科主界面


----------

>本篇文章包括以下内容：

>* [前言](#a)
>* [jsoup的简介](#b)
>* [jsoup的配置](#c)
>* [jsoup的使用](#d)
>* [结语](#e)

##**<span id="a">前言</span>**
对于Android初学者想要做项目时，最大的烦恼是什么？毫无疑问是数据源的缺乏，当然可以选择第三方接口提供数据，也可以使用网络爬虫获取数据，这样就不用第三方数据作为支持。本来是打算爬一些购物网站的数据，由于他们的反爬做得好，所以没办法爬到数据，只能爬取糗事百科的数据，或许聪明的你会想到可以高仿个糗事百科作为自己的练手项目，利用jsoup是完全没问题的

jsoup的学习需要结合前端的基础知识，爬取前端的数据，如果你学过JS，那么你可以自己完全不用看文档的情况下，使用该框架，因为其设计与JS的使用几乎相同，废话不多说，开车啦

##**<span id="b">jsoup的简介</span>**
使用项目原话：jsoup是一个Java库来处理实际的HTML。它提供了一个非常方便的API来提取和操纵数据,使用最好的DOM,CSS和jquery-like方法

项目地址：https://github.com/jhy/jsoup
中文文档：http://www.open-open.com/jsoup/

##**<span id="c">jsoup的配置</span>**
jsoup的配置很简单，需要在gradle中添加以下依赖

```
compile 'org.jsoup:jsoup:1.10.2'
```

由于jsoup需要获取网络数据，所以记得添加网络权限

```
<uses-permission android:name="android.permission.INTERNET" />
```

##**<span id="d">jsoup的使用</span>**
**一、获取HTML**

jsoup提供两种网络请求，get和post，使用代码也及其简单，我们首先爬取糗事百科首页的HTML。**注意：由于是网络请求操作，必须放在子线程中运行，否则4.4以上的版本会报错**

① get方式

```
new Thread() {
    @Override
    public void run() {
        super.run();
        try {
            Document doc = Jsoup.connect("http://www.qiushibaike.com/8hr/page/1/").get();
            Log.e("一、HTML內容", doc.toString());
            }
        catch{
        }
    }
}.start();
```

② post方式

```
Document doc = Jsoup.connect("http://www.qiushibaike.com/8hr/page/1/")
  .data("query", "Java")
  .userAgent("Mozilla")
  .cookie("auth", "token")
  .timeout(3000)
  .post();
```
这里对post的参数介绍一下

* connect：设置连接的Url
* data：设置post的键值对数据
* userAgent：设置用户代理（请求头的东西，可以判断你是PC还是Mobile端）
* cookie：设置缓存
* timeout：设置请求超时
* post：发送post请求

>既然已经获取HTML的Document对象了，接下来就是分析Html元素的时候了

**二、获取Html元素**

① 网页端

以糗事百科为例子，我们查看糗事百科首页的数据对应的Html元素是什么，我们可以通过F12，找到对应的Html元素

![](http://img.blog.csdn.net/20170212222700508?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到一个a标签就是文章详情的内容，我们可以通过这个a标签的class="contentHerf"作为唯一标识来获取该链接，获取之后，继续爬取详情页的文章详细内容，所以我们通过爬取的a标签的链接进入该文章的详情页

![](http://img.blog.csdn.net/20170212222950542?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当然也有一些详情页有图片的，我们可以通过图片的的class="thumb"作为唯一标识来爬取图片里面的链接

![](http://img.blog.csdn.net/20170212223317445?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

由于糗事百科采用分页加载的情况，我们需要在爬取完第一张内容后，接着爬取第二章的内容，下面是糗事百科的分页Url的规则，很简单，我们可以通过一个循环就可以了

```
http://www.qiushibaike.com/8hr/page/1/
http://www.qiushibaike.com/8hr/page/2/
http://www.qiushibaike.com/8hr/page/3/
http://www.qiushibaike.com/8hr/page/4/
http://www.qiushibaike.com/8hr/page/5/
```

>好了，分析完网页端之后，就应该在我们的Android端采用代码，将上面的步骤实现出来了

② Android端

通过上面的分析后，可以总结我们需要实现的步骤有：

1. 爬取主页的详情页url
2. 进入详情页爬取内容和图片
3. 循环爬取第二页、第三页...

聪明的你，可能会想到第四步第五步...

4. 封装Bean对象
5. 使用ListView填充内容
6. 爬取日期、作者、评论等内容完善项目

1） 爬取主页的详情页url

爬取主页的url可以通过a标签的class="contentHerf"，我们通过jsoup的属性选择器来实现，这里会用到css知识，jsoup中文文档也有很详细的介绍
```java
Document doc = Jsoup.connect("http://www.qiushibaike.com/8hr/page/1/").get();
Elements els = doc.select("a.contentHerf");
Log.e("一、HTML內容", els.toString());

for (int i = 0; i < els.size(); i++) {
    Element el = els.get(i);
    Log.e("1.标题", el.text());

    String href = el.attr("href");
    Log.e("2.链接", href);
}
```
这里对使用到的对象进行介绍

* Document：相当于一个Html文件
* Elements：相当于一个标签的集合
* Element：相当于一个标签

这里要注意Elements与Element的toString()方法和text()方法

* toString()：打印出来的是标签的Html内容
* text()：打印出来的是标签对应的文本内容

css选择器

* select()：获取符合属性选择器要求的标签内容
* 或getElementById：获取符合ID选择器要求的标签内容
* 或getElementsByTag：获取符合Tag选择器要求的标签内容

2） 进入详情页爬取内容和图片

这段代码也相当简单，这里就不多解释了
```java
Document doc = Jsoup.connect("http://www.qiushibaike.com/8hr/page/1/").get();
Elements els = doc.select("a.contentHerf");
Log.e("一、HTML內容", els.toString());

for (int i = 0; i < els.size(); i++) {
    Element el = els.get(i);
    Log.e("1.标题", el.text());

    String href = el.attr("href");
    Log.e("2.链接", href);
    
	//获取详情页内容
    Document doc_detail = Jsoup.connect("http://www.qiushibaike.com" + href).get();
    Elements els_detail = doc_detail.select(".content");
    Log.e("3.內容", els_detail.text());
    
	//获取图片
    Elements els_pic = doc_detail.select(".thumb img[src$=jpg]");
    if (!els_pic.isEmpty()) {
        String pic = els_pic.attr("src");
        Log.e("4.图片连接", "" + pic);
    } else {
        Log.e("4.图片连接", "无");
    }
}
```
3） 循环爬取第二页、第三页...

这里只需要嵌套一个循环进去就可以了，完整代码如下

```
public class JsoupActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_jsoup);

        new Thread() {
            @Override
            public void run() {
                super.run();
                try {
                    for (int k = 0; k < 5; k++) {
                        Document doc = Jsoup.connect("http://www.qiushibaike.com/8hr/page/" + k + "/").get();
                        Elements els = doc.select("a.contentHerf");
                        Log.e("一、HTML內容", els.toString());

                        for (int i = 0; i < els.size(); i++) {
                            Element el = els.get(i);
                            Log.e("1.标题", el.text());

                            String href = el.attr("href");
                            Log.e("2.链接", href);

                            Document doc_detail = Jsoup.connect("http://www.qiushibaike.com" + href).get();
                            Elements els_detail = doc_detail.select(".content");
                            Log.e("3.內容", els_detail.text());

                            Elements els_pic = doc_detail.select(".thumb img[src$=jpg]");
                            if (!els_pic.isEmpty()) {
                                String pic = els_pic.attr("src");
                                Log.e("4.图片连接", "" + pic);
                            } else {
                                Log.e("4.图片连接", "无");
                            }
                        }
                    }
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }.start();

    }
}
```

4） 当然，我们爬取到的内容之后，毫无疑问就是要封装成对象，通过ArrayList存储起来，这样你的数据源就解决了

```
public class Xiaohua { 
    private String content;
    private String title;
    private String url;
    private String userName;
    private String date;
}
```

5） 后面爬取作者、日期、评论等信息就由你们去练习了，然后界面一仿，项目就出来了

![](http://img.blog.csdn.net/20170212232120166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

**三、爬取结果**

```
02-12 08:16:55.010 18074-18110/com.handsome.boke2 E/1.标题: 小时候有个常去楼主家的阿姨总是把楼主叫成楼主哥哥的名字，终于有一天，楼主忍无可忍，大骂了她一顿：“你这个人是不是白痴啊？”她暴怒了，立马告诉了楼主的爸爸，楼主永远也忘不了哥哥被揍时眼睛里的无辜与绝望...
02-12 08:16:55.011 18074-18110/com.handsome.boke2 E/2.链接: /article/118543240
02-12 08:16:55.329 18074-18110/com.handsome.boke2 E/3.內容: 小时候有个常去楼主家的阿姨总是把楼主叫成楼主哥哥的名字，终于有一天，楼主忍无可忍，大骂了她一顿：“你这个人是不是白痴啊？”她暴怒了，立马告诉了楼主的爸爸，楼主永远也忘不了哥哥被揍时眼睛里的无辜与绝望...
02-12 08:16:55.331 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:55.881 18074-18110/com.handsome.boke2 E/1.标题: 一朋友，给暗恋许久的女神，匿名网购了一大堆的礼物，可把女神高兴的，在朋友圈发说说，这是谁买的。告诉我，我要做他女朋友！ 朋友乐坏了，于是激动的说，是我，是我！ 那女神愣了愣，然后把礼物全部退给了他……
02-12 08:16:55.881 18074-18110/com.handsome.boke2 E/2.链接: /article/118542673
02-12 08:16:56.104 18074-18110/com.handsome.boke2 E/3.內容: 一朋友，给暗恋许久的女神，匿名网购了一大堆的礼物，可把女神高兴的，在朋友圈发说说，这是谁买的。告诉我，我要做他女朋友！ 朋友乐坏了，于是激动的说，是我，是我！ 那女神愣了愣，然后把礼物全部退给了他……
02-12 08:16:56.106 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:56.106 18074-18110/com.handsome.boke2 E/1.标题: LZ在非洲曾经遇到过抢劫，有人拿枪指着我们，愣了一下热血当头没当回事，继续反抗，后来情急之下，他射了一枪，结果发现那是玩具枪，特么的，我们抓起扫把就把那个劫匪揍了一顿。事后想想，又害怕又想笑。
02-12 08:16:56.106 18074-18110/com.handsome.boke2 E/2.链接: /article/118542683
02-12 08:16:56.608 18074-18110/com.handsome.boke2 E/3.內容: LZ在非洲曾经遇到过抢劫，有人拿枪指着我们，愣了一下热血当头没当回事，继续反抗，后来情急之下，他射了一枪，结果发现那是玩具枪，特么的，我们抓起扫把就把那个劫匪揍了一顿。事后想想，又害怕又想笑。
02-12 08:16:56.609 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:56.609 18074-18110/com.handsome.boke2 E/1.标题: 今年换了工作，今天第一天上班，老妈早早起床准备早餐，等我吃完早餐准备出门的时候，老妈塞给我一个红包说，新年第一天上班图吉利。当时急着上班也没有细看就放在口袋里。等上班空闲的时候，掏出红包，发现红包里只有一张纸条，上面写着四个大字:好好工作……
02-12 08:16:56.609 18074-18110/com.handsome.boke2 E/2.链接: /article/118542647
02-12 08:16:57.140 18074-18110/com.handsome.boke2 E/3.內容: 今年换了工作，今天第一天上班，老妈早早起床准备早餐，等我吃完早餐准备出门的时候，老妈塞给我一个红包说，新年第一天上班图吉利。当时急着上班也没有细看就放在口袋里。等上班空闲的时候，掏出红包，发现红包里只有一张纸条，上面写着四个大字:好好工作……
02-12 08:16:57.142 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:57.142 18074-18110/com.handsome.boke2 E/1.标题: 腰疼，趴在床上，让大侄子来给我踩踩后背，踩得我挺舒服，没忍住，放个响屁，小家伙愣了一下，然后狠狠 踹 我 屁 股“让你蹦我！让你蹦我！”。。。。
02-12 08:16:57.142 18074-18110/com.handsome.boke2 E/2.链接: /article/118542708
02-12 08:16:57.379 18074-18110/com.handsome.boke2 E/3.內容: 腰疼，趴在床上，让大侄子来给我踩踩后背，踩得我挺舒服，没忍住，放个响屁，小家伙愣了一下，然后狠狠 踹 我 屁 股“让你蹦我！让你蹦我！”。。。。
02-12 08:16:57.382 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:57.382 18074-18110/com.handsome.boke2 E/1.标题: 闺蜜的妈妈非常迷信，自从闺蜜放假回家陪妈妈去了几次麻将馆后，她妈每次都能赢钱，所以她妈这一个寒假只要去打麻将，都要拉着她去，直到昨天闺蜜开学，她妈妈送她走得时候，眼泪汪汪的对闺蜜说:宝贝，这是我第一次不舍的你走~
02-12 08:16:57.382 18074-18110/com.handsome.boke2 E/2.链接: /article/118542657
02-12 08:16:57.881 18074-18110/com.handsome.boke2 E/3.內容: 闺蜜的妈妈非常迷信，自从闺蜜放假回家陪妈妈去了几次麻将馆后，她妈每次都能赢钱，所以她妈这一个寒假只要去打麻将，都要拉着她去，直到昨天闺蜜开学，她妈妈送她走得时候，眼泪汪汪的对闺蜜说:宝贝，这是我第一次不舍的你走~
02-12 08:16:57.882 18074-18110/com.handsome.boke2 E/4.图片连接: 无
02-12 08:16:57.882 18074-18110/com.handsome.boke2 E/1.标题: 早上起床后发现阳台的地上到处是泡沫水，花盆里也有很多泡沫，而且地上躺着洗衣液的空瓶子，一下便明白了，转头去问熊孩子，熊孩子若无其事的说我只是给花洗洗头而已嘛！
02-12 08:16:57.882 18074-18110/com.handsome.boke2 E/2.链接: /article/118542709
02-12 08:16:58.391 18074-18110/com.handsome.boke2 E/3.內容: 早上起床后发现阳台的地上到处是泡沫水，花盆里也有很多泡沫，而且地上躺着洗衣液的空瓶子，一下便明白了，转头去问熊孩子，熊孩子若无其事的说我只是给花洗洗头而已嘛！
02-12 08:16:58.393 18074-18110/com.handsome.boke2 E/4.图片连接: 无
```


##**<span id="e">结语</span>**

网络爬虫虽然带来了很多数据源的问题，但很多网站都已经通过一些技术实现反爬虫的效果了，所以大家还是以学习jsoup为主，不管是Android端还是Web端jsoup的用处很广泛，所以掌握起来是必须的，听说豆瓣和知乎都可以爬出来哦，想做项目的同学可以去试试哦

[代码下载](http://download.csdn.net/detail/qq_30379689/9753006)