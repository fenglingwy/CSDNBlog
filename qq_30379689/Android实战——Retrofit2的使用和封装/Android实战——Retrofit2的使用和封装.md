#Retrofit2的使用和封装


----------
>本篇文章包含以下内容：

>* Retrofit2是什么
>* Retrofit2工具类的演示
>* Retrofit2工具类的封装

##Retrofit2是什么


----------
使用项目的原话：Android和Java中类型安全的HTTP客户端
项目地址：https://github.com/square/retrofit

###Retrofit2的基本使用
1、Get请求
2、Post请求

###Retrofit2的导入
这里Retrofit还需要导入它的Gson依赖库，因为返回的数据就是使用Gson来处理的
```
compile 'com.squareup.retrofit2:retrofit:2.1.0'
compile 'com.squareup.retrofit2:converter-gson:2.1.0'
```

##Retrofit2工具类的演示

----------
演示提供的接口，该接口不能使用post方式，否则会报错

```
http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&&sort=asc&&time=1418816972
```
返回的JSON数据

```
{
	"error_code": 0,
	"reason": "Success",
	"result": {
		"data":[
			{
				"content":"学校论坛上有人问：“为啥明明用了除蟑螂的药，蟑螂却越来越多了。”某个学生回帖：“如果你家人不见了，你不出来找吗？你会不着急么？”",
				"hashId":"8196907ee902f3508b9be6ea59d2191c",
				"unixtime":1478598830,
				"updatetime":"2016-11-08 17:53:50"
			}
		]
	}
}
```

###Get请求演示
get支持普通请求
```java
/*
 * Get请求 
 * 参数已经封装在工具类的Url中
 */
Call<Info> call = RetrofitUtils.getInstance().get();
call.enqueue(new Callback<Info>() {
    @Override
    public void onResponse(Call<Info> call, Response<Info> response) {
        tv.setText(response.body().toString());
    }

    @Override
    public void onFailure(Call<Info> call, Throwable t) {

    }
});
```
get支持键值对参数的请求
```java
//根据接口需求创建键值对
Map<String,String> map = new HashMap<>();
map.put("key","488c65f3230c0280757b50686d1f1cd5");
map.put("sort","asc");
map.put("time","1418816972");

/*
 * Get请求 
 * @param map 为get参数
 */
Call<Info> call = RetrofitUtils.getInstance().get(map);
call.enqueue(new Callback<Info>() {
    @Override
    public void onResponse(Call<Info> call, Response<Info> response) {
        tv.setText(response.body().toString());
    }

    @Override
    public void onFailure(Call<Info> call, Throwable t) {

    }
});
```
###Post请求演示

```java
//根据接口需求创建键值对
Map<String,String> map = new HashMap<>();
map.put("key","488c65f3230c0280757b50686d1f1cd5");
map.put("sort","asc");
map.put("time","1418816972");

/*
 * Post请求 
 * @param map 为Post参数
 */
Call<Info> call = RetrofitUtils.getInstance().post(map);
call.enqueue(new Callback<Info>() {
    @Override
    public void onResponse(Call<Info> call, Response<Info> response) {
        tv.setText(response.body().toString());
    }

    @Override
    public void onFailure(Call<Info> call, Throwable t) {

    }
});
```
>效果图，由于这个接口不能接受post方式，所以运行post会报错

![](http://img.blog.csdn.net/20161108203135922)

##Retrofit2工具类的封装


----------
Retrofit的请求是以REST请求方式发送请求的，所以工具的封装需要做两件事

* 对REST请求的API进行封装
* Retrofit自身的封装

###一、数据准备
这里需要对我们需要解析的数据进行Bean对象的封装

```
public class Info {

    @Override
    public String toString() {
        return "Info{" +
                "error_code=" + error_code +
                ", reason='" + reason + '\'' +
                ", result=" + result +
                '}';
    }

    private int error_code;
    private String reason;
    private ResultBean result;

    public static class ResultBean {
        @Override
        public String toString() {
            return "ResultBean{" +
                    "data=" + data +
                    '}';
        }

        private List<DataBean> data;

        public static class DataBean {
            private String content;
            private String hashId;
            private int unixtime;
            private String updatetime;

            @Override
            public String toString() {
                return "DataBean{" +
                        "content='" + content + '\'' +
                        ", hashId='" + hashId + '\'' +
                        ", unixtime=" + unixtime +
                        ", updatetime='" + updatetime + '\'' +
                        '}';
            }
        }
    }
}
```

###二、对REST请求的API进行封装
Retrofit使用注解的方式来声明GET请求、POST请求、请求参数、请求头等进行的网络访问，下面是各个注解的表示的意思

* Get请求相关
	* @Get：发送Get请求
	* @Query：Get请求参数
	* @QueryMap：Get请求Map参数

* Post请求相关
	* @Post：发送Post请求
	* @FormUrlEncoded：采用表单的方式，一般与@Post共用
	* @Field：Post请求参数
	* @FieldMap：Post请求Map参数

* Header请求相关
	* @Headers：发送Header信息
	* @Header：Header信息参数
	* @HeaderMap：Header信息的Map参数

* @Path：访问路径，最终访问BaseUrl+@Path里面的内容

>理解完意思之后，编写REST的API，其实就是请求接口，具体看下面的代码
```
public interface IRetrofitServer {

    String getUrl = "list.from";
    String postUrl = "list.from";

    /**
     * 传递参数的Get请求
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @GET(getUrl)
    Call<Info> get(@Query("key") String key, @Query("sort") String sort, @Query("time") String time);

    /**
     * 封装好Url的Get的请求
     * @return
     */
    @GET(getUrl + "?key=488c65f3230c0280757b50686d1f1cd5&&sort=asc&&time=1418816972")
    Call<Info> get();

    /**
     * 传递Map键值对的Get请求
     * @param params
     * @return
     */
    @GET(getUrl)
    Call<Info> get(@QueryMap Map<String, String> params);

    /**
     * 传递参数的Post请求
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @FormUrlEncoded
    @POST(postUrl)
    Call<Info> post(@Field("key") String key, @Field("sort") String sort, @Field("time") String time);

    /**
     * 传递Map键值对的Post请求
     * @param map
     * @return
     */
    @FormUrlEncoded
    @POST(postUrl)
    Call<Info> post(@FieldMap Map<String, String> map);

    /**
     * 传递Map键值对和Header的Post请求
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @Headers({"os:Android", "version:2.0"})
    @FormUrlEncoded
    @POST(postUrl)
    Call<Info> postWithHeader(@Field("key") String key, @Field("sort") String sort, @Field("time") String time);

    /**
     * 传递Map键值对和Header的Post请求
     * @param os
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @FormUrlEncoded
    @POST(postUrl)
    Call<Info> postWithHeader(@Header("os") String os, @Field("key") String key, @Field("sort") String sort, @Field("time") String time);

    /**
     * 传递Map键值对和Header的Post请求
     * @param map
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @FormUrlEncoded
    @POST(postUrl)
    Call<Info> postWithHeader(@HeaderMap Map<String, String> map, @Field("key") String key, @Field("sort") String sort, @Field("time") String time);

    /**
     * 传递访问路径和键值对的Post请求
     * @param path
     * @param key
     * @param sort
     * @param time
     * @return
     */
    @FormUrlEncoded
    @POST("{path}")
    Call<Info> post(@Path("path") String path, @Field("key") String key, @Field("sort") String sort, @Field("time") String time);
}
```

###三、Retrofit自身的封装
Retrofit和okHttp一样，采用构造者模式创建，采用单例模式防止使用多个对象
```
private static final String baseUrl = "http://japi.juhe.cn/joke/content/";
private static Retrofit retrofit = null;
private static IRetrofitServer iServer;

public static IRetrofitServer getInstance() {
    if (retrofit == null) {
        synchronized (RetrofitUtils.class) {
            if (retrofit == null) {
                retrofit = new Retrofit.Builder()
                        .baseUrl(baseUrl)
                        .addConverterFactory(GsonConverterFactory.create())
                        .build();
                iServer = retrofit.create(IRetrofitServer.class);
            }
        }
    }
    return iServer;
}
```
上面代码做了三件事

* 绑定请求URL
* 采用GSON来处理返回的JSON数据
* 创建并返回REST请求API接口iServer

>下面就可以直接使用工具类拿到这个iServer，调用提供的接口方法

```
Call<Info> call = RetrofitUtils.getInstance().get();
call.enqueue(...);
```




##工具类下载：[RetrofitUtils](http://download.csdn.net/detail/qq_30379689/9676791)