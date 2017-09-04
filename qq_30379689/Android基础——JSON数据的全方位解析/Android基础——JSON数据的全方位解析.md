#Android基础——JSON数据的全方位解析


----------
>本篇文章包括以下内容：

>* JSON是什么
>* JSONObject的解析和存储
>* JSONObject的解析和存储（抽象）
>* JSONArray的解析和存储
>* 模拟周边加油站JSON数据实战

##JSON是什么
JSON：JavaScript对象表示法（JavaScript Object Notation）

* JSON是存储和交换文本信息的语法
* JSON是轻量级的文本数据交换格式
* JSON独立于语言和平台
* JSON具有自我描述性，更易理解

类似XML，比XML更小、更快、更易解析

* 没有结束标签
* 更短
* 读写的速度更快
* 使用数组
* 不使用保留字

JSON语法是JavaScript对象表示法语法的子集

* 数据在名称/值对中
* 数据由逗号分割
* 花括号保存对象
* 方括号保存数据

JSON值可以是

* 数字（整数或浮点数）
* 字符串（在双引号中）
* 逻辑值（true或false）
* 数组（在方括号中）
* 对象（在花括号中）
* null


##JSONObject的解析和存储

----------

JSONObject数据是用key-value来存储的，中间使用冒号隔开，外层的花括号表示一个对象
```
{
	"username":"Hensen",
	"qq":"510402535"
}
```
首先创建一个存储数据的Bean类

```
public class User {
    private String username;
    private int qq;

    public User(String username, int qq) {
        this.username = username;
        this.qq = qq;
    }
}
```

下面我们使用将服务器获取的JSON数据放进一个JSON对象中，获取其对象中的值

```
//模拟服务器传来的JSON数据
String str ="{\n" +
            "\t\"username\":\"Hensen\",\n" +
            "\t\"qq\":\"510402535\""+
            "\n}";
JSONObject json = new JSONObject(str);
String username = json.getString("username");
int qq = json.getInt("qq");
//使用对象的形式进行保存
User user = new User(username, qq);
```

##JSONObject的解析和存储（抽象）

----------
我们会碰到在一个JSON中嵌套有其他JSON对象，而这个嵌套的JSON对象中可以抽象出共同的属性，看例子
```
"price":{
	"E90":"5.36",
	"E93":"5.77",
	"E97":"6.25",
	"E0":"5.34"
}
"gastprice":{
	"92#":"5.58",
	"0#车柴":"5.15"
}
```
>这个时候我们就不用创建两个对象了，创建一个对象抽取他们的属性即可

首先创建一个存储数据的Bean类

```
public class FourthLevelObject {
    private String type;
    private String price;

    public FourthLevelObject(String type, String price) {
        this.type = type;
        this.price = price;
    }

    @Override
    public String toString() {
        return "FourthLevelObject{" +
                "type='" + type + '\'' +
                ", price='" + price + '\'' +
                '}';
    }
}
```
然后我们解析并存储

```
JSONObject price = data_json.getJSONObject("price");
JSONObject gastprice = data_json.getJSONObject("gastprice");
Iterator<String> keys_price = price.keys();
while (keys_price.hasNext()) {
    String key = keys_price.next();
    String value = price.getString(key);
    //使用对象的形式进行保存
    FourthLevelObject fourthLevelObject = new FourthLevelObject(key, value);
}
Iterator<String> keys_gastprice = gastprice.keys();
while (keys_gastprice.hasNext()) {
    String key = keys_gastprice.next();
    String value = gastprice.getString(key);
    //使用对象的形式进行保存
    FourthLevelObject fourthLevelObject = new FourthLevelObject(key, value);
}
```


##JSONArray的解析和存储


----------

```
"data":[
			{
				"id":"59797",
				"name":"太洋加油站"
			},
			{
				"id":"11083",
				"name":"中石化麻子岗加油站"
			}
	   ]
```
通过遍历JSONArray，剩下的跟JSONObject一样，
```
JSONArray data = result.getJSONArray("data");
for (int i = 0; i < data.length(); i++) {
	 String id = data_json.getString("id");
     String name = data_json.getString("name");
     //使用对象的形式进行保存
     User user = new User(id, name);
     //如果User为嵌套对象，应该添加到集合中
     //list.add(user);
}
```



##模拟周边加油站JSON数据实战

----------
这里以周边加油站数据为例，其解析步骤有

* 分析数据中的成员变量（花括号为对象，方括号为数组，使用List存储数组） 
* 根据分析后的结果，创建对应的对象
* 解析JSON数据、存储JSON数据

>大家可以尝试一下自己写解析，发现哪里不对时，直接运行程序，系统会自动给你提示哪一行解析出错的

```
{
	"resultcode":"200",
	"reason":"Successed!",
	"result":{
		"data":[
			{
				"id":"59797",
				"name":"太洋加油站",
				"area":"514000",
				"areaname":"广东省 梅州市 梅江区",
				"address":"广东省梅州市梅江区环市北路家乐广场附近,路南侧",
				"brandname":"不详",
				"type":"其他",
				"discount":"非打折加油站",
				"exhaust":"国Ⅲ",
				"position":"116.104117014,24.3286227908",
				"lon":"116.11066877213",
				"lat":"24.33427865799",
				"price":{
					"E90":"5.36",
					"E93":"5.77",
					"E97":"6.25",
					"E0":"5.34"
				},
				"gastprice":{
					"92#":"5.58",
					"0#车柴":"5.15"
				},
				"fwlsmc":"",
				"distance":2462
			},
			{
				"id":"11083",
				"name":"中石化麻子岗加油站",
				"area":"516000",
				"areaname":"广东省 梅州市",
				"address":"广东省梅州市205国道与梅松路交叉口东南方向，嘉应大学附近",
				"brandname":"中石化",
				"type":"直营店",
				"discount":"打折加油站",
				"exhaust":"国Ⅲ",
				"position":"116.124168,24.32516",
				"lon":"116.13067098935",
				"lat":"24.331051295968",
				"price":{
					"E90":"5.36",
					"E93":"5.77",
					"E97":"6.25",
					"E0":"5.34"
				},
				"gastprice":{
					"92#":"5.58",
					"95#":"6.05",
					"0#车柴":"5.15"
				},
				"fwlsmc":"银联卡,信用卡支付,加油卡,便利店,93#自助加油,柴油自助加油,97#自助加油,发卡充值网点,银联卡充值,加油卡充值业务",
				"distance":439
			},
			{
				"id":"51175",
				"name":"月梅加油站",
				"area":"514000",
				"areaname":"广东省 梅州市 梅江区",
				"address":"广东省梅州市梅江区月梅路月梅农贸批发市场北,路西侧",
				"brandname":"不详",
				"type":"其他",
				"discount":"非打折加油站",
				"exhaust":"国Ⅲ",
				"position":"116.1250119928,24.3291280115",
				"lon":"116.1315112916",
				"lat":"24.335033948452",
				"price":{
					"E90":"5.36",
					"E93":"5.77",
					"E97":"6.25",
					"E0":"5.34"
				},
				"gastprice":{
					"92#":"5.58",
					"0#车柴":"5.15"
				},
				"fwlsmc":"",
				"distance":465
			},
			{
				"id":"29356",
				"name":"中石化嘉华加油站",
				"area":"514700",
				"areaname":"广东省 梅州市 梅县",
				"address":"广东省梅州市梅江区月梅路与碧桂路交叉路口,路东",
				"brandname":"中石化",
				"type":"直营店",
				"discount":"打折加油站",
				"exhaust":"国Ⅲ",
				"position":"116.1192494629,24.3272616485",
				"lon":"116.132454",
				"lat":"24.339033",
				"price":{
					"E90":"5.36",
					"E93":"5.77",
					"E97":"6.25",
					"E0":"5.34"
				},
				"gastprice":{
					"92#":"5.58",
					"95#":"6.05",
					"0#车柴":"5.15"
				},
				"fwlsmc":"加油卡,便利店,发卡充值网点,卫生间,银联卡充值,加油卡充值业务",
				"distance":804
			},
			{
				"id":"51077",
				"name":"东郊加油站",
				"area":"514000",
				"areaname":"广东省 梅州市 梅江区",
				"address":"广东省梅州市梅江区东山大道金山龙丰卫生站附近",
				"brandname":"不详",
				"type":"其他",
				"discount":"非打折加油站",
				"exhaust":"国Ⅲ",
				"position":"116.1357199618,24.3121215949",
				"lon":"116.14218687436",
				"lat":"24.31822136463",
				"price":{
					"E90":"5.36",
					"E93":"5.77",
					"E97":"6.25",
					"E0":"5.34"
				},
				"gastprice":{
					"92#":"5.58",
					"0#车柴":"5.15"
				},
				"fwlsmc":"",
				"distance":1720
			}
		],
		"pageinfo":{
			"pnums":20,
			"current":1,
			"allpage":1
		}
	},
	"error_code":0
}
```

一、分析数据中的成员变量

>在JSON中，只有两种语法，JSONObject（花括号内）和JSONArray（方括号内）

>* JSONObject：可以理解为一个Map集合，通过get获取value
>* JSONArray：可以理解为一个数组，通过循环获取对应的JSONObject

从上面的数据可以发现其中有五个JSON对象，一个JSON数组，从外到里分析

对象1~4：

![](http://img.blog.csdn.net/20161102170920465)

对象5：在最后面

![](http://img.blog.csdn.net/20161102171350593)

二、根据分析后的结果，创建对应的对象（按循序从1~5）
>在JSONObject中，左边是属性，右边是值

>如果右边的值为一个JSONArray，则在对象中使用List< Object>来存储，简单的说就是对象中的List嵌套另一个对象

>记住：花括号用对象，方括号用集合

根据上面的分析，创建第一个对象

```
public class FirstLevelObject {
    private String resultcode;
    private String reason;
    private List<Result> result;
    private String error_code;

    public FirstLevelObject(String resultcode, String reason, List<Result> result, String error_code) {
        this.resultcode = resultcode;
        this.reason = reason;
        this.result = result;
        this.error_code = error_code;
    }

    @Override
    public String toString() {
        return "FirstLevelObject{" +
                "resultcode='" + resultcode + '\'' +
                ", reason='" + reason + '\'' +
                ", result=" + result +
                ", error_code='" + error_code + '\'' +
                '}';
    }
}
```
创建第二个对象

```
public class Result {

    private List<Data> data;
    private List<PageInfo> pageinfo;

    public Result(List<Data> data, List<PageInfo> pageinfo) {
        this.data = data;
        this.pageinfo = pageinfo;
    }

    @Override
    public String toString() {
        return "Result{" +
                "data=" + data +
                ", pageinfo=" + pageinfo +
                '}';
    }
}
```
创建第三个对象

```
public class Data {

    private String id;
    private String name;
    private String area;
    private String areaname;
    private String address;
    private String brandname;
    private String type;
    private String discount;
    private String exhaust;
    private String position;
    private String lon;
    private String lat;

    private List<FourthLevelObject> price;
    private List<FourthLevelObject> gastprice;

    private String fwlsmc;
    private int distance;

    public Data(String id, String name, String area, String areaname, String address
            , String brandname, String type, String discount, String exhaust
            , String position, String lon, String lat, List<FourthLevelObject> price
            , List<FourthLevelObject> gastprice, String fwlsmc, int distance) {
        this.id = id;
        this.name = name;
        this.area = area;
        this.areaname = areaname;
        this.address = address;
        this.brandname = brandname;
        this.type = type;
        this.discount = discount;
        this.exhaust = exhaust;
        this.position = position;
        this.lon = lon;
        this.lat = lat;
        this.price = price;
        this.gastprice = gastprice;
        this.fwlsmc = fwlsmc;
        this.distance = distance;
    }

    @Override
    public String toString() {
        return "Data{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", area='" + area + '\'' +
                ", areaname='" + areaname + '\'' +
                ", address='" + address + '\'' +
                ", brandname='" + brandname + '\'' +
                ", type='" + type + '\'' +
                ", discount='" + discount + '\'' +
                ", exhaust='" + exhaust + '\'' +
                ", position='" + position + '\'' +
                ", lon='" + lon + '\'' +
                ", lat='" + lat + '\'' +
                ", price=" + price +
                ", gastprice=" + gastprice +
                ", fwlsmc='" + fwlsmc + '\'' +
                ", distance=" + distance +
                '}';
    }
}
```
创建第四个对象

```
public class FourthLevelObject {
    private String type;
    private String price;

    public FourthLevelObject(String type, String price) {
        this.type = type;
        this.price = price;
    }

    @Override
    public String toString() {
        return "FourthLevelObject{" +
                "type='" + type + '\'' +
                ", price='" + price + '\'' +
                '}';
    }
}
```
创建第五个对象

```
public class PageInfo {
    private int pnums;
    private int current;
    private int allpage;

    public PageInfo(int pnums, int current, int allpage) {
        this.pnums = pnums;
        this.current = current;
        this.allpage = allpage;
    }

    @Override
    public String toString() {
        return "PageInfo{" +
                "pnums=" + pnums +
                ", current=" + current +
                ", allpage=" + allpage +
                '}';
    }
}
```
三、解析JSON数据、存储JSON数据
>由于数据对象是一层嵌套一层的，这个解析思想跟树的遍历是一个道理，从外层->内层->外层，所以我们在解析内层的时候是需要边解析边存储我们的数据

数据的解析和存储

```
try {
    //模拟服务器传来的JSON数据
    JSONObject json = new JSONObject(jsonString);
    //第一层读取
    String resultcode = json.getString("resultcode");
    String reason = json.getString("reason");
    JSONObject result = json.getJSONObject("result");
    String error_code = json.getString("error_code");
    //第一层List
    List<Result> result_list = new ArrayList<>();

    //第二层读取
    JSONArray data = result.getJSONArray("data");
    JSONObject pageinfo = result.getJSONObject("pageinfo");
    //第二层List
    List<Data> data_list = new ArrayList<>();
    List<PageInfo> pageinfo_list = new ArrayList<>();

    //第三层读取
    for (int i = 0; i < data.length(); i++) {
        //第三层List
        List<FourthLevelObject> price_list = new ArrayList<>();
        List<FourthLevelObject> gastprice_list = new ArrayList<>();

        JSONObject data_json = (JSONObject) data.get(i);
        String id = data_json.getString("id");
        String name = data_json.getString("name");
        String area = data_json.getString("area");
        String areaname = data_json.getString("areaname");
        String address = data_json.getString("address");
        String brandname = data_json.getString("brandname");
        String type = data_json.getString("type");
        String discount = data_json.getString("discount");
        String exhaust = data_json.getString("exhaust");
        String position = data_json.getString("position");
        String lon = data_json.getString("lon");
        String lat = data_json.getString("lat");
        JSONObject price = data_json.getJSONObject("price");
        JSONObject gastprice = data_json.getJSONObject("gastprice");
        String fwlsmc = data_json.getString("fwlsmc");
        int distance = data_json.getInt("distance");

        //第四层读取
        Iterator<String> keys_price = price.keys();
        while (keys_price.hasNext()) {
            String key = keys_price.next();
            String value = price.getString(key);
            //装载第三层List
            FourthLevelObject fourthLevelObject = new FourthLevelObject(key, value);
            price_list.add(fourthLevelObject);
        }
        Iterator<String> keys_gastprice = gastprice.keys();
        while (keys_gastprice.hasNext()) {
            String key = keys_gastprice.next();
            String value = gastprice.getString(key);
            //装载第三层List
            FourthLevelObject fourthLevelObject = new FourthLevelObject(key, value);
            gastprice_list.add(fourthLevelObject);
        }

        //装载第二层List
        Data data1 = new Data(id, name, area, areaname, address, brandname, type
                , discount, exhaust, position, lon, lat, price_list
                , gastprice_list, fwlsmc, distance);
        data_list.add(data1);
    }

    //第五层读取
    int pnums = pageinfo.getInt("pnums");
    int current = pageinfo.getInt("current");
    int allpage = pageinfo.getInt("allpage");
    //装载第五层List
    PageInfo pageInfo = new PageInfo(pnums, current, allpage);
    pageinfo_list.add(pageInfo);

    //装载第一层List
    Result result1 = new Result(data_list, pageinfo_list);
    result_list.add(result1);

    //最后封装我们需要的得到的对象
    firstLevelObject = new FirstLevelObject(resultcode, reason, result_list, error_code);

} catch (JSONException e) {
    e.printStackTrace();
}
```

>由于我们第四个对象是嵌套在第三个对象数组中的，所以在里面再嵌套一层循环

>由于第四个对象是同一性质的属性，所以我们抽象成一个属性为type和price的对象，其key是不确定的，需要自己通过keys遍历来获取value



接着我们输出我们解析的结果

```
tv.setText(firstLevelObject.toString());
```

效果图

![](http://img.blog.csdn.net/20161102174000978)


