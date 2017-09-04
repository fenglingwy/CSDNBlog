#GsonFormat、Gson、fastjson快速搞定JSON数据解析


----------
>本篇文章包括以下内容：

>* GsonFormat的使用
>* Gson框架的使用
>* fastjson框架的使用

如果对JSON数据还不是很明白它的解析步骤的话，可以看我的另一篇[Android基础——JSON数据的全方位解析](http://blog.csdn.net/qq_30379689/article/details/53009112)

##GsonFormat的使用


----------
>GsonFormat是一个Android Studio的插件，输入一段Json格式的数据，会自动生成对应的Bean对象，省去了我们以前手动创建Json对象的时间

一、安装GsonFormat

在Android Studio中，通过File->Settings->Plugins，然后点击Browse repositories...按钮

![](http://img.blog.csdn.net/20161108134224494)

输入GsonFormat右侧进行安装，我这里已经安装过

![](http://img.blog.csdn.net/20161108134353917)

二、使用GsonFormat

我们准备了一段比较简单的Json数据作为我们的测试数据

```
{
	"error_code": 0,
	"reason": "Success",
	"result": {
		"data":[
			{
				"content":"床不在好，有女(你)就行，枕不在久，整完就走，斯是陋室，惟吾色心！！…",
				"hashId":"accd4dea540fe0b2f2205f9234114335",
				"unixtime":1478579630,
				"updatetime":"2016-11-08 12:33:50"
			},
			{
				"content":"老婆问老公：亲爱的你觉得我是灰姑娘吗？老公回答：当然不是了。老婆：那你的意思说是我是白雪公主？老公回答：不，你是黑姑娘。",
				"hashId":"8144c8979f02c539841968ef8046db98",
				"unixtime":1478577230,
				"updatetime":"2016-11-08 11:53:50"
			},
			{
				"content":"睡的正香，老婆把我叫醒说她要去卫生间，我说你上厕所就上呗！她说我是要叫你起来穿衣服的，我说那你上厕所我起来穿衣服干啥？我又不冷！然后就看着老婆把被子抱在身上去上厕所了。",
				"hashId":"76b7f6c0ba6c3455fbd216820ce4a68c",
				"unixtime":1478577230,
				"updatetime":"2016-11-08 11:53:50"
			},
			{
				"content":"老奶奶加了出租车，到了之后车费显示是8块，但是她只给了3块。司机赶紧叫住老奶奶：“老太太，这车费8块块你怎么只付3元呢？”老奶奶回答：“我刚才坐车的时候计价器显示5元块了。那我不就是在给三块就可以了。",
				"hashId":"85a82ecf746981ea3f48e5066eb034e1",
				"unixtime":1478577230,
				"updatetime":"2016-11-08 11:53:50"
			},
			{
				"content":"每次看战争片时总有一些疑惑，在冲锋时前面的士兵一边奔跑着一边狂打枪，可以理解。但总是看到后面的士兵也紧跟着跑开着枪，难道子弹会拐弯，打不到前面的士兵吗？……求解答，为什么打不到队友？",
				"hashId":"d3ddd517e97a6dbeac24c82d4ce34e72",
				"unixtime":1478575431,
				"updatetime":"2016-11-08 11:23:51"
			}
		]
	}
}
```

使用GsonFormat非常简单，首先创建一个Bean对象

```
public class Info {
    
}
```

接着在这个类里面使用alt+shift+s快捷键（就是鼠标右键的快捷键），进入Generate...，就可以找到GsonFormat

![](http://img.blog.csdn.net/20161108134816364)

进入GsonFormat将我们的测试Json数据输入，点击确定，即可完成我们的Bean类的创建

![](http://img.blog.csdn.net/20161108134929850)

查看我们自动生成的类

```
public class Info {

    /**
     * error_code : 0
     * reason : Success
     * result : ......
     */

    private int error_code;
    private String reason;
    private ResultBean result;

    public int getError_code() {
        return error_code;
    }

    public void setError_code(int error_code) {
        this.error_code = error_code;
    }

    public String getReason() {
        return reason;
    }

    public void setReason(String reason) {
        this.reason = reason;
    }

    public ResultBean getResult() {
        return result;
    }

    public void setResult(ResultBean result) {
        this.result = result;
    }

    public static class ResultBean {
        /**
         * content : 床不在好，有女(你)就行，枕不在久，整完就走，斯是陋室，惟吾色心！！…
         * hashId : accd4dea540fe0b2f2205f9234114335
         * unixtime : 1478579630
         * updatetime : 2016-11-08 12:33:50
         */

        private List<DataBean> data;

        public List<DataBean> getData() {
            return data;
        }

        public void setData(List<DataBean> data) {
            this.data = data;
        }

        public static class DataBean {
            private String content;
            private String hashId;
            private int unixtime;
            private String updatetime;

            public String getContent() {
                return content;
            }

            public void setContent(String content) {
                this.content = content;
            }

            public String getHashId() {
                return hashId;
            }

            public void setHashId(String hashId) {
                this.hashId = hashId;
            }

            public int getUnixtime() {
                return unixtime;
            }

            public void setUnixtime(int unixtime) {
                this.unixtime = unixtime;
            }

            public String getUpdatetime() {
                return updatetime;
            }

            public void setUpdatetime(String updatetime) {
                this.updatetime = updatetime;
            }
        }
    }
}
```
>由于我们自动生成的Bean对象没有加上toString()的方法，为了方便后面的演示，我们手动增加toString()的方法，这里就不介绍了

##Gson框架的使用


----------
>  Gson--是一款Google公司的用来解析json数据格式的库

准备工作，导入依赖：

```
compile 'com.google.code.gson:gson:2.8.0'
```

一、Json数据自动生成Bean对象

```
Gson gson = new Gson();
Info info = gson.fromJson(jsonStr,Info.class);
```

二、Bean对象转化为Json数据

```
String jsonSti = new Gson().toJson(info);
```

>这里演示我们刚才Info对象的数据，通过TextView显示出来

```
Gson gson = new Gson();
Info info = gson.fromJson(jsonStr,Info.class);
tv.setText(info.toString());
```

效果图

![](http://img.blog.csdn.net/20161108135647516)

##fastjson框架的使用


----------
>Fastjson--是一款阿里巴巴的用来解析json数据格式的库，据说目前最快

准备工作，导入依赖：

```
compile 'com.alibaba:fastjson:1.2.20'
```
一、Json数据自动生成Bean对象

```
Info info = JSON.parseObject(jsonStr,Info.class);
```

二、Bean对象转化为Json数据

```
String jsonStr = JSON.toJSONString(info);
```

>这里演示我们刚才Info对象的数据，通过TextView显示出来

```
Info info = JSON.parseObject(jsonStr, Info.class);
tv.setText(info.toString());
```

效果图

![](http://img.blog.csdn.net/20161108135647516)
