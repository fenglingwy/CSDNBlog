##Mac下使用Hexo搭建个人博客##


----------


**Hexo介绍**
>* 利用原作者的一句话：A fast，simple&powerful blog framework，powered by Node.js
>* Hexo是基于Node.js的博客平台，Hexo是生成静态的Html文件，部署到各个托管平台完成发布，其官网地址：https://hexo.io/zh-cn/

![](http://img.blog.csdn.net/20161018235740157)

----------

**环境准备**

注意：如果没翻墙的话，可能下载的速度会慢些，个人使用smartHost无压力搭建或者百度使用ss纸飞机，关于如何使用smartHost链接可以在我的博客找

>安装Mac包管理器Homebrew，通过终端输入一下指令
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
>安装另一个包管理器Homebrew Cask，该包管理器会自动配置好环境变量，使用刚才安装的包管理器
```
 brew install caskroom/cask/brew-cask
```
>通过包管理器，安装Node.js
```
brew install node
```
>安装Git
```
brew cask install git
```
>查看安装的版本，检验是否安装正确
```
node -v

npm -v

git --version
```


----------

**安装Hexo**

>安装Hexo，执行以下指令
```
npm install -g hexo-cli
```
>创建一个目录，用于放置你的Hexo博客，然后通过终端进入你创建的目录，初始化站点

```
hexo init hexo
```
>初始化站点后，系统会提示执行npm install以完成所有的配置

```
npm install
```
>上面操作完成后，即完成Hexo的安装和初始化，接下来就可以测试了

----------
**测试Hexo**

>在整个配置完成后，打开终端，输入以下指令即可运行本地服务测试
```
hexo server
```
![](http://img.blog.csdn.net/20161018235414482)

>输入http://localhost:4000/即可打开默认页面


----------

**hexo文件夹目录结构**
>* source：博客资源文件夹
>* source/_drafts：草稿文件夹
>* source/_posts：文章文件夹
>* themes：存放主题的文件夹
>* themes/landscape：默认的主题
>* _config.yml：全局配置文件


----------
**部署到Github Pages**

>博客发布到Github Pages上，供外网访问，当然你也可以部署到其他服务器上，Github Pages服务的使用步骤：

>* 开通Github账号，例如我的账号名CSDNHensen，这个后面会用到
>* 创建一个repository，名称开头必须和账号名一样，然后以.github.io结尾

![](http://img.blog.csdn.net/20161018232005782)

>* 修改配置文件：_config.yml，整个站点的配置都在这里，打开_config.yml文件
>* 找到下面的语句，然后修改你的信息，repository在github仓库中可以复制

![](http://img.blog.csdn.net/20161018232243654)

>* 如果要修改主题，只需要将主题文件clone到themes目录下，并修改_config.yml配置文件即可

```
theme:landscape
```
>例如进入hexo的themes目录，执行如下指令

```
git clone git@github.com:wuchong/jacman.git
```
>然后修改配置文件的theme参数为jacman即可，关于更多主题https://github.com/hexojs/hexo/wiki/Themes


----------
**新建博客**
>站点准备完毕，接下来就是创建博客了，发布博客有两种方式，一种通过命令生成一个博客样板，另一种是直接把Markdown文档拿来使用

>* 命令行生成

```
hexo new HelloWord 
```
>新生成的文章都会保存在/source/_posts目录下，打开生成的模板，内容如下

```
title:HelloWord
date:20116-10-18 22:21:02
tags:
---
```
>这是生成文档的默认格式，在分割线下面，就可以按照正常的Markdown格式进行写作

>* 文档拷贝：也可以将Markdown拷贝到/source/_posts目录下

![](http://img.blog.csdn.net/20161018233346232)


----------
**生成博客**
>新增了博客文章后，需要提交到服务器上，输入以下指令完成将博客生成静态Html文件

```
hexo generate
```
![](http://img.blog.csdn.net/20161018233710362)


----------
**发布博客到服务器**

>安装hexo-deployer-git工具，输入指令

```
npm install hexo-deployer-git --save
```
>提交到服务器上，提交过程中还需要输入github账号和密码

```
hexo deploy
```

![](http://img.blog.csdn.net/20161018233944568)

>这个时候你就可以打开你的github，进入repository的setting页面，往下滑，可以看到会显示出你的个人主页网址

![](http://img.blog.csdn.net/20161018234301697)

----------
**快捷命令**
>Hexo常用命令
```
hexo new "postName" //新建文章
hexo new page "pageName" //新建页面
hexo generate //生成静态页面至public目录
hexo server //开启预览访问端口（默认端口4000，ctrl+c关闭server）
hexo deploy //将.deploy目录部署到服务器
```
>起始这些命令都有快捷命令

```
hexo n == hexo new 
hexo g == hexo generate
hexo s == hexo server
hexo d == hexo deploy
```

>所有操作就到这里结束了，谢谢观看