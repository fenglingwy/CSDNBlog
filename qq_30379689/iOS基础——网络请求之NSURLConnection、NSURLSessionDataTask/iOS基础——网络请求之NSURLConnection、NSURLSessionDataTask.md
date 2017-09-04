#iOS基础——网络请求之NSURLConnection、NSURLSessionDataTask


----------
##**前言**

iOS网络请求分为GET请求和POST请求，在iOS中，iOS9之前和iOS9之后用的不是同一个类，iOS9之前的已经属于过时版本

##**iOS9前网络请求**

1、准备工作

① 如果采用iOS9前的网络请求方式需要在info.plist文件中添加以下配置信息

![](http://img.blog.csdn.net/20170322134023882?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

② 请求API

该API只支持get请求，所以post请求会报错
```
http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237
```

③ 返回数据

```
2017-03-22 13:30:10.061 NetWorkDemo[1058:34695] response:<NSHTTPURLResponse: 0x60000003e020> { URL: http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237 } { status code: 200, headers {
    Connection = "keep-alive";
    "Content-Language" = "en-US";
    "Content-Length" = 555;
    "Content-Type" = "text/html;charset=utf-8";
    Date = "Wed, 22 Mar 2017 05:30:09 GMT";
    Server = nginx;
    "Set-Cookie" = "JSESSIONID=FFFAE888B76866A28AF0E8713AF27033; Path=/joke/; HttpOnly";
} }  data:{"error_code": 0,"reason": "Success","result": {"data":[{"content":"晚上回家，听到巷子有哭声，靠近一看，原来是一衣衫不整的女子在哭。问怎么了，小姐答：“我被色狼侵犯了！”我：“没事吧？”小姐答：“他突然从背后抓住我的胸部，然后就把我放了……”我：“那还哭什么呢？”小姐答：“因为……那色狼居然说，真倒霉，竟然抱到个男的。”","hashId":"d234529d89136e07c1e924f589c3de70","unixtime":1490153630,"updatetime":"2017-03-22 11:33:50"}]}}  error:(null)
```

2、get请求

get请求和post请求不一样的地方

* get使用的是NSURLRequest
* post使用的是NSMutableURLRequest 

① 同步get请求
```
-(void)SyncGet{
    //1、网址
    NSString *urlstr = @"http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstr];
    //3、创建网络请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //4、发送同步请求
    NSURLResponse *response = nil;
    NSError *error = nil;
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    NSString *value = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"response:%@  data:%@  error:%@",response,value,error);
}
```
② 异步get请求

```
-(void)AsynGet{
    //1、网址
    NSString *urlstr = @"http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstr];
    //3、创建网络请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //4、发送异步请求
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    [NSURLConnection sendAsynchronousRequest:request queue:queue completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        
        NSString *value = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"response:%@  data:%@  error:%@",response,value,connectionError);
    }];
}
```

3、post请求

① 同步post请求第一种方式

```
-(void)SyncPost{
    //1、网址
    NSString *urlstr = @"http://japi.juhe.cn/joke/content/list.from";
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstr];
    //3、创建网络请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //4、设置post请求
    [request setHTTPMethod:@"POST"];
    //5、设置post请求参数
    NSString *postParams = @"key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    NSData *dataParams = [postParams dataUsingEncoding:NSUTF8StringEncoding];
    [request setHTTPBody:dataParams];
    //6、发送同步请求
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    
    NSURLResponse *response = nil;
    NSError *error = nil;
    NSData *data = [NSURLConnection sendSynchronousRequest:request returningResponse:&response error:&error];
    NSString *value = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    NSLog(@"response:%@  data:%@  error:%@",response,value,error);
}
```

② 同步post请求第二种方式

第二种方式需要实现NSURLConnectionDataDelegate，实现其方法
```
-(void)SyncPost2{
    //1、网址
    NSString *urlstr = @"http://japi.juhe.cn/joke/content/list.from";
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstr];
    //3、创建网络请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //4、设置post请求
    [request setHTTPMethod:@"POST"];
    //5、设置post请求参数
    NSString *postParams = @"key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    NSData *dataParams = [postParams dataUsingEncoding:NSUTF8StringEncoding];
    [request setHTTPBody:dataParams];
    //6、发送同步请求
    [NSURLConnection connectionWithRequest:request delegate:self];
}

//当接收到服务器的响应（连通了服务器）时会调用
-(void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response{
    
}

//当接收到服务器的数据时会调用（可能会被调用多次，每次只传递部分数据）
-(void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data{
    
}

//当服务器的数据加载完毕时就会调用
-(void)connectionDidFinishLoading:(NSURLConnection *)connection{
    
}

//请求错误（失败）的时候调用（请求超时\断网\没有网\，一般指客户端错误）
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error{
    
}
```

③ 异步post请求

```
-(void)AsynPost{
    //1、网址
    NSString *urlstr = @"http://japi.juhe.cn/joke/content/list.from";
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstr];
    //3、创建网络请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //4、设置post请求
    [request setHTTPMethod:@"POST"];
    //5、设置post请求参数
    NSString *postParams = @"key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    NSData *dataParams = [postParams dataUsingEncoding:NSUTF8StringEncoding];
    [request setHTTPBody:dataParams];
    //6、发送异步请求
    NSOperationQueue *queue = [[NSOperationQueue alloc]init];
    [NSURLConnection sendAsynchronousRequest:request queue:queue completionHandler:^(NSURLResponse * _Nullable response, NSData * _Nullable data, NSError * _Nullable connectionError) {
        
        NSString *value = [[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"response:%@  data:%@  error:%@",response,value,connectionError);
    }];
}
```

##**iOS9后网络请求**

iOS9后的网络请求做了修改，统一采用异步的方式进行访问

1、get请求

```
-(void)newGet{
    //1、网址
    NSString *urlstring=@"http://japi.juhe.cn/joke/content/list.from?key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237";
    //如果URL网址里有中文，要先转成utf8
     NSString *urlstr=[urlstring stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstring];
    //3、创建网络请求
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    //4、创建get请求
    NSURLSessionDataTask *dataTask = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        //获取的内容是字符串
        NSString *str=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"%@",str);
    }];
    //5、执行请求
    [dataTask resume];
}
```

2、post请求

```
-(void)newPost{
    //1、网址
    NSString *urlstring=@"http://japi.juhe.cn/joke/content/list.from";
    //如果URL网址里有中文，要先转成utf8
    NSString *urlstr=[urlstring stringByAddingPercentEncodingWithAllowedCharacters:[NSCharacterSet URLQueryAllowedCharacterSet]];
    //2、创建URL
    NSURL *url = [NSURL URLWithString:urlstring];
    //3、创建网络请求
    NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
    //4、设置post请求
    request.HTTPMethod = @"POST";
    //5、设置post请求体
    [request setValue:@"application/json;encoding=utf-8" forHTTPHeaderField:@"Content-Type"];
    //6、设置post请求参数
    NSString *param = [NSString stringWithFormat:@"key=488c65f3230c0280757b50686d1f1cd5&page=1&pagesize=1&sort=asc&time=1418745237"];
    request.HTTPBody = [param dataUsingEncoding:NSUTF8StringEncoding];
    //7、创建post请求
    NSURLSessionDataTask *dataTask = [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        //获取的内容是字符串
        NSString *str=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
        NSLog(@"%@",str);
    }];
    //5、执行请求
    [dataTask resume];
}

```

[源码下载](http://download.csdn.net/detail/qq_30379689/9789691)