#Android实战——okhttp3的使用和封装


----------
>本篇文章包括以下内容：

>* okhttp3是什么
>* okhttp3工具类的演示（基于okhttp工具类的封装）
>* okhttp3工具类的封装

#okhttp3是什么


----------


使用作者的项目的原话：Android和Java应用程序的HTTP和HTTP / 2客户端
其项目地址：https://github.com/square/okhttp

##okhttp3的基本使用

1、Get请求
2、Post请求
3、文件上传
4、文件下载

##okhttp3的导入
由于okhttp3里面是依赖于okio进行开发的，所以务必将okio也引入

```
compile 'com.squareup.okhttp3:okhttp:3.2.0'
compile 'com.squareup.okio:okio:1.6.0'
```

#okhttp3工具类的演示

----------

我们根据okhttp3的使用封装了HttpUtils，用起来非常简单，跟第三方平台Api使用一样

##Get演示
这里使用response.body().string()返回响应结果内容
```
/*
 * Get请求 
 * 参数一：请求Url
 * 参数二：请求回调
 */
HttpUtils.doGet("https://www.so.com/", new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {

    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        if(response.isSuccessful()){
            tv.setText(response.body().string());
        }
        //关闭防止内存泄漏
        if(response.body()!=null){
            response.body().close();
        }
    }
});
```

##Post发送键值对演示

```
/*
 * Post请求 
 * 参数一：请求Url
 * 参数二：请求的键值对
 * 参数三：请求回调
 */
Map<String,String> map = new HashMap<>();
map.put("username","Hensen");
map.put("password","123456");

HttpUtils.doPost("http://192.168.31.109:8080/test.php", map, new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {

    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        
    }
});
```

##Post发送JSON数据演示

```
/*
 * Post请求 
 * 参数一：请求Url
 * 参数二：请求的JSON
 * 参数三：请求回调
 */
String json = "{\"name\":\"Hensen\"}";

HttpUtils.doPost("http://192.168.0.8:8080/test.php", json, new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {

    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
```

##上传文件演示

```
/*
 * 上传文件
 * 参数一：请求Url
 * 参数二：保存文件的路径名
 * 参数三：保存文件的文件名
 * 参数四：请求回调
 */
HttpUtils.doFile("http://192.168.0.8:8080/WD/admin.php?a=ajaxuploadimage&&c=Image",
                "/data/data/com.handsome.app4/logoa.jpg", "logoa.jpg", new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {

    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {

    }
});
```

##下载文件演示

```
/*
 * 下载文件
 * 参数一：请求Url
 * 参数二：保存文件的路径名
 * 参数三：保存文件的文件名
 */
HttpUtils.downFile("http://shouji.360tpcdn.com/160804/a05b75b7779f7a4afae83601c195ed2b" +
                "/com.qihoo.haosou_708.apk"
                ,"/data/data/com.handsome.app4/","aqy.apk");
```


#okhttp3工具类的封装

----------
>okhttp3采用构造者模式来实现的，下面是简单的API介绍：

>* Request.Builder  请求构造者
	*  url(String url)：请求的url 
	* post()：默认是Get方式
	* post(RequestBody body)：Post带参数
	* build()：构造请求

>请求参数有三种：

>* RequestBody：普通的请求参数
>* FormBody.Builder：以表单的方式传递键值对的请求参数
>* MultipartBody.Builder：以表单的方式上传文件的请求参数

>执行方法：

>* Call 
	* enqueue(Callback callback)：异步请求
	* execute()：同步请求

##okhttp3单例
创建HttpUtils工具类，由于okhttp3不建议创建多个对象，所以采用饿汉式的单例模式
```
private static OkHttpClient client = null;

private HttpUtils() {}

public static OkHttpClient getInstance() {
    if (client == null) {
        synchronized (HttpUtils.class) {
            if (client == null)
                client = new OkHttpClient();
        }
    }
    return client;
}
```

##Get请求
okhttp3也提供了同步的请求方式，通过call.execute()方法，这里都使用异步来演示
```
/**
 * Get请求
 *
 * @param url
 * @param callback
 */
public static void doGet(String url, Callback callback) {
    Request request = new Request.Builder()
            .url(url)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```

##Post请求：发送键值对

```
/**
 * Post请求发送键值对数据
 *
 * @param url
 * @param mapParams
 * @param callback
 */
public static void doPost(String url, Map<String, String> mapParams, Callback callback) {
    FormBody.Builder builder = new FormBody.Builder();
    for (String key : mapParams.keySet()) {
        builder.add(key, mapParams.get(key));
    }
    Request request = new Request.Builder()
            .url(url)
            .post(builder.build())
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```

##Post请求：发送JSON数据

```
/**
 * Post请求发送JSON数据
 *
 * @param url
 * @param jsonParams
 * @param callback
 */
public static void doPost(String url, String jsonParams, Callback callback) {
    RequestBody body = RequestBody.create(MediaType.parse("application/json; charset=utf-8")
            , jsonParams);
    Request request = new Request.Builder()
            .url(url)
            .post(body)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}
```

##上传文件

```
/**
 * 上传文件
 *
 * @param url
 * @param pathName
 * @param fileName
 * @param callback
 */
public static void doFile(String url, String pathName, String fileName, Callback callback) {
    //判断文件类型
    MediaType MEDIA_TYPE = MediaType.parse(judgeType(pathName));
    //创建文件参数
    MultipartBody.Builder builder = new MultipartBody.Builder()
            .setType(MultipartBody.FORM)
            .addFormDataPart(MEDIA_TYPE.type(), fileName,
                    RequestBody.create(MEDIA_TYPE, new File(pathName)));
    //发出请求参数
    Request request = new Request.Builder()
            .header("Authorization", "Client-ID " + "9199fdef135c122")
            .url(url)
            .post(builder.build())
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(callback);
}

/**
 * 根据文件路径判断MediaType
 *
 * @param path
 * @return
 */
private static String judgeType(String path) {
    FileNameMap fileNameMap = URLConnection.getFileNameMap();
    String contentTypeFor = fileNameMap.getContentTypeFor(path);
    if (contentTypeFor == null) {
        contentTypeFor = "application/octet-stream";
    }
    return contentTypeFor;
}
```

##下载文件

```
/**
 * 下载文件
 * @param url
 * @param fileDir
 * @param fileName
 */
public static void downFile(String url, final String fileDir, final String fileName) {
    Request request = new Request.Builder()
            .url(url)
            .build();
    Call call = getInstance().newCall(request);
    call.enqueue(new Callback() {
        @Override
        public void onFailure(Call call, IOException e) {

        }

        @Override
        public void onResponse(Call call, Response response) throws IOException {
            InputStream is = null;
            byte[] buf = new byte[2048];
            int len = 0;
            FileOutputStream fos = null;
            try {
                is = response.body().byteStream();
                File file = new File(fileDir, fileName);
                fos = new FileOutputStream(file);
                while ((len = is.read(buf)) != -1) {
                    fos.write(buf, 0, len);
                }
                fos.flush();
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                if (is != null) is.close();
                if (fos != null) fos.close();
            }
        }
    });
}
```

如果在下载文件中需要进度的话，可以修改onResponse回调里面的内容就可以获取进度

```
@Override
public void onResponse(Call call, Response response) throws IOException {
    InputStream is = null;
    byte[] buf = new byte[2048];
    int len = 0;
    FileOutputStream fos = null;
    try {
        is = response.body().byteStream();
        File file = new File(fileDir, fileName);
        fos = new FileOutputStream(file);
        //---增加的代码---
        //计算进度
        long totalSize = response.body().contentLength();
        long sum = 0;
        while ((len = is.read(buf)) != -1) {
            sum += len;
            //progress就是进度值
            int progress = (int) (sum * 1.0f/totalSize * 100);
            //---增加的代码---
            fos.write(buf, 0, len);
        }
        fos.flush();
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        if (is != null) is.close();
        if (fos != null) fos.close();
    }
}
```


##工具类的下载：[HttpUtils](http://download.csdn.net/detail/qq_30379689/9670398)