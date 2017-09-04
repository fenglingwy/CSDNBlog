#PHP实战——开发遇到过的错误问题与解决方案汇总


----------


##一、PHPStudy

1、问题：

phpstudy apache 无法启动或者启动后自动关闭，而且80端口或者是自己定义的其他端口没有被占用

解决： 

1. 如果电脑未安装VC9运行库，那肯定是开不启的，请自行下载VC9运行库
2. 打开phpstudy设置，网站目录不能包含有中文名，而且网站目录必须存在，两者满足后即可

2、问题：

在使用phpstudy作为服务器的时候，我们需要js插件上传视频时，发现上传失败

解决：

1. 修改php.ini中的post_max_size（上传大小）
2. 修改php.ini中的upload_max_filesize（上传文件大小）
3. 重启phpstudy（切记）

##二、uploadify.js

1、问题：

使用uploadify上传图片时，显示上传已经成功，服务器也已经存在该图片，但是，上传后接收uploadify成功回调时，报错

![这里写图片描述](http://img.blog.csdn.net/20170511224137870?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
原因：

问题出现在返回的json数据中，前面多了一个空格，也就是我们所说的BOM头，只要使用js代码去掉即可

解决：

![](http://img.blog.csdn.net/20170511224608372?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXFfMzAzNzk2ODk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

