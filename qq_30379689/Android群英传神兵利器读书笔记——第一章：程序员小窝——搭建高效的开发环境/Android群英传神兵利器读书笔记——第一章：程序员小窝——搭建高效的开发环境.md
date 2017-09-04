# Android群英传神兵利器读书笔记——第一章：程序员小窝——搭建高效的开发环境 #
----------
##目录##
>* **1.1 搭建高效的开发环境之操作系统**
>* **1.2 搭建开发环境之高效配置**
	* 基本环境配置
	* 基本开发工具
>* **1.3 搭建程序员的博客平台**
	* 开发者为什么要写作
	* 写作平台
	* 第三方博客平台
	* 自建博客平台
	* 开发论坛
>* **1.4 Geek PPT Persentation**
	* impress.js
	* Strut
	* reveal.js
	* Slides
>* **1.5 开发文档**
	*  Markdown
	*  项目文档生成器


----------

###**1.1 搭建高效的开发环境之操作系统**
>作者力荐用MacBook开发，因为其优点：

>* 优美的外观颜值
>* 集Windows的易用性与Linux的高可开发性于一体
>* 使用Unix系统，它是Linux系统的始祖
>* 大量开源软件和开发工具可以非常容易地用来M开发Mac版
>* 不用担心Windows下的各种电脑病毒和木马、也不用清理磁盘碎片、甚至不用安装各种驱动程序
>* 由于Mac与Android内核都是Unxi\Linux架构，不需要任何驱动程序就可以使用
>* 系统安全性非常高

Mac快捷键一览表：
https://support.apple.com/zh-cn/HT201236

>窗口操作

>* Command＋～：切换同一应用的窗口
>* Command＋tab：切换不同应用的窗口
>* Command＋W：关闭该应用的其中一个窗口
>* Command＋Q：关闭该应用
>* Command＋N：快速创建应用新窗口

>截图

>* Command+Shift+4：自由截图
>* Command+Shift+4+空格键：截取当前窗口

>编辑

>* Command+Left\Right：关标快速移到行首或者行尾
>* Option+Left\Right：按单词进行关标移动
>* Command+Up\Down：在一页的页首和页尾快速切换
>* Command+Delete：快速删除一行


----------


###**1.2 搭建开发环境之高效配置**

 >Fn键
 
 >* 通过在"系统偏好设置-键盘"中选中"将F1、F2等键用作标准功能键"
 >* 这样修改的一个原因就是在很多IDE、编辑器中，Fn键都是一些快捷键

![](http://img.blog.csdn.net/20161021141943608)

>Trackpad触控板

>* 手势进行缩放、旋转；页面、工作区直接进行切换；显示桌面和多任务调度
>* 通过在"系统偏好设置-触控板"中设置
>* Win10也改进了触控板，增加了类似Mac的手势功能

![](http://img.blog.csdn.net/20161021142002262)

>Dock

>* Dock设置为"自动显示和隐藏"，让桌面更简洁
>* Win10也有类似Mac的自动隐藏功能

![](http://img.blog.csdn.net/20161021142016624)

----------


###**基本开发工具**

**Homebrew**：Mac下的包管理工具（http://brew.sh/index.html）

>Homebrew的安装：只需要在终端输入

```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
>通过Homebrew安装Node.js，会自动配置好环境变量：

```
brew install node
```


----------


**Homebrew镜像**
>* 由于国外的软件，在国内下载会比较慢
>* 镜像地址：http://ban.ninja/

![](http://img.blog.csdn.net/20161021142113732)

----------


**Homebrew Cask**：是Homebrew的孪生兄弟

>它们的优点都是可以直接在终端快速完成App的下载和安装，并配置好各种环境变量
>
>Homebrew和Homebrew Cask的区别：

>* Homebrew 是直接下载源码解压，然后执行./configure指令和make install指令，统一安装在/usr/local/bin/目录下
>* Homebrew Cask下载已经编译好的应用包（.dmg或者.pkg文件），解压后放到统一的目录——/opt/homebrew-cask/Caskroom

>安装Homebrew Cask：

```
brew install caskroom/cask/brew-cask
```

>通过Homebrew Cask可以获得各种开发软件：

>* brew cask install evernote
>* brew cask install skype
>* brew cask install mou
>* brew cask install virtualbox
>* brew cask install iterm2
>* ......

>通过Homebrew Cask可以搜索我们需要的App：

```
brew cask search android
```

>如果Homebrew Cask没有收录你想下载的App，那么你可以直接在其项目中提交pull request

>另外，还可以查看App的相关信息：

```
brew-cask info node
```
>或者通过uninstall指令卸载App：

```
brew cask uninstall node
```
>甚至你可以新建一个Shell脚本，输入所有你想要安装的App，从而创建一个一键自动安装所有App的脚本


----------

**iTerm2终端工具**：iTerm2才是Mac下最好用的终端工具

![](http://img.blog.csdn.net/20161021144401510)

>安装iTerm2：

```
brew cask install iterm2
```
>* 相比Mac原生的终端工具，iTerm2提供了更多的功能，例如强大的快捷键支持、指令历史记录（Command+Shift+H）、自动补全提示（Command+；）强大的搜索功能和粘贴复制功能，等等
>* iTerm2提供了对整个终端工具的全面配置权限，你可以随心所欲地设置iTerm2的各种颜色、透明度，打造一个完全适合你自己开发风格的终端工具
>* http://iterm2colorschemes.com/这个网站收集了大量的配色文件，根据自己的喜好，下载相应的xxx.itermcolors文件，双击进行安装，完成配置
>* 设置iTerm2的配色也非常简单，只需要打开preferences，选择profiles-color标签即可导入主题颜色

![](http://img.blog.csdn.net/20161021144427668)

----------
**Zsh与oh-my-zsh**
>* 什么是Shell？从语义上讲，Shell是个壳，就是包裹内核的壳，用户是不能直接与内核通信的，但是内核提供了一个能够与你通信的对象，这个对象就是Shell
>* Zsh就是帮助用户使用Shell的工具

>显示目前系统中存在的所有Shell：

```
cat /etc/shells
```

![](http://img.blog.csdn.net/20161021144456198)

>* 由于Zsh配置难度很大，所以一般用户很难使用，幸亏有个叫RobbyRussell的程序员开发出了一个项目——oh-my-zsh，简化Zsh的配置，也保留了强大的功能

>由于Mac系统自带了Zsh，切换到Zsh需要使用下面指令：

```
chsh -s /bin/zsh
```
>切完成后就可以直接在oh-my-zsh的官网寻找需要的功能答案，不需要百度和Google，官网首页就有http://ohmyz.sh/

>* 安装oh-my-zsh

```
$ sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
>安装后，打开配置文件——.zshrc，进行配置

```
~ open.zshrc
```
>* 设置环境变量，例如Android SDK的环境变量：

```
export PATH=${PATH}:/Users/xuyisheng/Library/Android/sdk/platform-tools
export PATH=${PATH}:/Users/xuyisheng/Library/Android/sdk/tools
```
>* 通过alias别名设置别名，可以简化复杂的命令

```
alias cls=clear
```
>这样配置后，可以在终端输入cls就能执行clear所执行的清屏命令

```
alias -s html=subl
```
>这样配置后，可以在终端输入test.html等html文件，即可自动用Sublime打开该文件

>另外还有很多别名，如git等操作，可以看官网介绍https://github.com/robbyrussell/oh-my-zsh/wiki/Plugin:git

>* 设置主题：oh-my-zsh的主题设置是它的另一个非常强大的功能，在~/oh-my-zsh/themes目录下，保存了各种主题的配置文件，读者可以根据自己的喜好，设置不同的主题

>官网https://github.com/robbyrussell/oh-my-zsh/wiki/Themes，要修改主题也很简单，只需要修改配置文件中的ZSH_THEME参数即可

>* 插件：在~/.oh-my-zsh/plugins目录下，保存了各种插件，几乎你想要的功能，在官网都有https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins，只需在配置文件中找到plugins参数，并在后面的括号中增加相应的插件名即可

> Zsh有很多强大的功能，在终端进入一个目录时，一般要使用cd指令，但在Zsh中直接输入目录名即可，而且输入d指令，可以查看历史跳转过的路径，选择前面的数字，即可再次跳转


----------

**终端使用技巧**

>* 快速定位：通过Alt+鼠标点击可以将光标快速定位到鼠标点击的地方，另外，通过Control+A和Control+E快捷键，快速将光标移动到开头和结尾处
>* 搜索指令：使用Control+R可以搜索输入的历史指令。系统会进行模糊匹配，找到匹配的历史指令
>* Find：该指令的基本使用格式find[paht][options][experssion]，例如在当前目录下查".txt"结尾文件，指令为

```
find . -name '*.txt'
```
>* Grep：这个指令后面会讲解到，用于过滤筛选结果的


----------
**Alfred2搜索利器**
>在Mac系统中，系统给我们提供了一个强大的搜索工具——Spotlight

![](http://img.blog.csdn.net/20161021144542183)

>* 点击菜单栏上的那个小放大镜即可启动
>* Spotlight能做的，Alfred2都能做，Spotlight不能做的，Alfred2也能做，Alfred2不能做的，你可以编程让它做
>* 可以使用Homebrew安装Alfred2
>* 通常情况下，Alfred2用来替代Spotlight功能的，因此一般将Alfred2的快捷键设置为Option+Space
>* 打开Alfred2的Preferences界面，选择Features选项卡，如图


![](http://img.blog.csdn.net/20161021144618652)

>在Features选项卡下，Alfred2列举了它的一些基本功能，例如全文件检索，如图

![](http://img.blog.csdn.net/20161021144643703)

>通过直接输入open、find、in、tags关键字，直接启动打开、寻找并打开文件目录、在文件中检索、通过tag检索等功能，例如直接输入指令

![](http://img.blog.csdn.net/20161021151429946)

>* 直接回车就可以打开这个文件，find直接打开文件所在目录、in可以直接搜索文件的内容、tags可以根据tag来进行检索
>* 不光是本机，Alfred2同样可以在Web上进行搜索，如图

![](http://img.blog.csdn.net/20161021151445100)

>Alfred2中可以自定义一些搜索，点击右下角"Add Custom Search"按钮，在Search URL中输入http://s.taobao.com/search?q={query}，如图

![](http://img.blog.csdn.net/20161021151516930)

>这样设置后就可以在Alfred2中输入tao关键字就可以直接调用淘宝搜索了，如图

![](http://img.blog.csdn.net/20161021151534680)

>类似地，你可以完全定义自动的搜索入口，只需将相应的搜索URL中的搜索内容换成{query}即可

>Alfred2还提供了强大的系统功能支持，如图

![](http://img.blog.csdn.net/20161021151546149)

>常见的锁屏只需要输入lock即可锁屏

![](http://img.blog.csdn.net/20161021151557399)

>* 相关的Log out、睡眠、清空垃圾箱、关机、退出程序等系统操作，都可以通过Alfred2

>Alfred2提供了强大的Workflows功能（需要购买Alfred2的powerpack），通过点击Preferences界面的Workflows选项卡，可以打开Workflows，由于个人没有购买这里就不展示了


>* 在Workflows中，可以自定义各种高级的功能入口，丰富到几乎都可以通过Alfred2来实现，这里添加一个简单的Workflows——Top Workflows，安装了这个Workflow2之后，调出Alfred2，直接输入top

>* 这时列表中会自动显示目前的进程状态，类似直接在终端中执行的top指令，选中相应的进程，或者输入kill就可以直接结束掉这个进程，整个进程都不需要打开终端

>* 这只是个非常简单的，还有更多的Workflows请看官网：http://alfredworkflow.com/

>* Alfred2安装Workflows，官网：https://github.com/zenorocha/alfred-workflows

>* 这些Workflows网站上，收集了数以万计的Workflows，例如直接搜索快递单号信息、列出今日知乎精华帖、检索新闻、天气信息等等，甚至还可以在Alfred2直接发送微博，Facebook、或者进制转换等功能

>在Windows系统中虽然没有Alfred2，但是有一个很强的搜索利器——Everything，大部分可以替代Alfred2的搜索功能


----------
**Sublime Text**
>* SublimeText的安装和配置可以看我另一篇博客，其内容大同小异：[SublimeText3和插件的安装](http://blog.csdn.net/qq_30379689/article/details/52645441)
>* 使用Homebrew安装Sublime，如果是手动安装则需要手动配置环境变量
>* Multi Cursor Editor：按住Command+点击要编辑的地方可增加光标，从而进行多行同时编辑，按住Option+按住鼠标拖动，即可实现纵向多光标编辑
>* Goto anything：Command+P可以打开该指令，通过该指令可以查找打开的所有文件，当打开文件为代码时，在Goto anything中输入@符号，可以查看代码大纲结构

----------
**Bartender**
>Bartender作用非常简单，帮你管理Mac的菜单栏，其地址为：https://www.macbartender.com/

![](http://img.blog.csdn.net/20161021151812949)


----------
**反编译工具**
>* Jadx：项目主页https://github.com/skylot/jadx
>* 通过如下进行下载和编译

```
git clone https://github.com/skylot/jadx.git
//编译
gradle build
```
>如果已经配置好了gradle的环境变量，那么直接执行build指令即可，等Jadx编译完毕，进入其build/jadx/bin/目录，执行以下操作：

```
./jadx -d out ~/Downloads/test.apk
```
>执行完毕后，在bin目录下就会生成out目录，里面就是反编译出的文件

>这个反编译工具优势在于可以一次性完成资源和代码的反编译，同时GUI界面支持强大的搜索能力


----------
**其他常用工具**
>Git：分布式版本管理工具

```
brew cask install git
```
>Java：使用Homebrew安装能自动配置好环境变量

```
brew cask install java
```
>Android Studio：安卓开发IDE

```
brew cask install android-studio
```
>Parallels Desktop：Mac中虚拟机
>1Password：Mac下的密码管理软件
>Tree：查看文档的目录结构

```
brew install tree
```

>强化Finder：常用的Finder强化工具，主要有Pathfinder和XtraFinder两种


----------
###**1.3 搭建程序员的博客平台**

>在笔者看来，开发者在刚开始写作的时候，建议选择第三方平台，一来可以只关心写作的内容，培养好的写作习惯，二来可以利用它们庞大的用户群，快速提高自己的技术影响力（前提是要有高质量的文章）


----------
**自建博客平台**
>* WordPress是一种使用PHP开发的博客平台，用户可以在支持PHP和MySQL数据库的服务器上架设博客平台
>* Jekyll、Octopress：Jekyll和Octopress同样出自于Ruby，它们的共同特点是可以通过命令行快速生成静态网页，再利用Github Pages这个纯天然的托管平台，几乎几分钟就能搭建好自己的博客
	* Jekyll和Octopress的官网分别是：http://jekyll.bootcss.com/和http://octopress.org/
>* Ghost：前面两种都是基于Ruby的，Ghost则是基于Node.js的，Ghost本身就具有发布文章的功能，类似于轻量级的WordPress，其官网：https://ghost.org/中文官网：http://www.ghostchina.com/
	* 通过Ghost的后台发布系统，用户可以很方便地发布文档，Ghost后台编辑同样适用于Markdown等格式
>* Hexo：好戏在后头，对于Hexo搭建博客，可以看我的个人博客：[Mac下使用Hexo搭建个人博客](http://blog.csdn.net/qq_30379689/article/details/52854003)


----------
**Gitbook**
>除了博客这种平台以外，开发者还可以通过Gitbook创建自己的文集，Gitbook正是这样一个分常好的本地、在线文库制作工具，网址：https://www.gitbook.com/
>Gitbook安装使用非常简单，在官网上下载相应的Gitbook editor或者使用在线版本即可


----------
**开发者论坛**
>NodeBB：基于Node.js的论坛系统，网址https://community.nodebb.org/
>笔者已经在公司搭建了自己的开发者论坛，地址如下：http://bbs.inside.hujiang.com/

![](http://img.blog.csdn.net/20161021152502052)


----------
###**1.4 Geek PPT Presentation**
>通常情况下，Microsoft的PPT是做Presentation的首选工具，同样，Mac下也有一款几乎同样功能的工具——Keynote，它与PPT功能基本类似，而且兼容PPT格式


----------
**impress.js**
>impress.js（https://impress.github.io/impress.js/#/bored）是一个专门用于创建Presentation的JavaScript库
>生成后的页面完全可以使用键盘方向键来进行控制，在它的官网上，作者给出了impress.js的详细信息，https://github.com/impress/impress.js
>如果不熟悉前端的朋友可以直接把Demo拿过去修改，也可以很酷炫，Demo地址https://github.com/impress/impress.js/wiki/Examples-and-demos


----------
**Strut**
>Strut（http://strut.io）实际上是基于impress.js开发的一款编辑器，它给原本没有编辑器支持的impress.js提供了可视化的编辑界面，大大降低了impress.js使用难度


----------
**reveal.js**
>与impress.js类似，reveal.js是一个基于Html5和JavaScript的Presentation展示框架，其官网：https://github.com/hakimel/reveal.js
>显示效果与impress.js基本一致，但reveal.js更贴心的是，他制作了自己的在线编辑器，地址： http://slides.com/


----------
**Slides**
>Slides（http://slides.com/）类似于一款在线的PPT制作工具


----------
###**1.5 开发文档**
>Markdown是一种标记性语言，通过使用简单的语法来实现统一的文字格式，出来的格式很整齐，如果不懂MarkDown的语法，网上有很多资料可供学习


----------
**MarkDown编辑器**
>推荐常用的编辑器有以下几种：

>* 作业部落：在线MarkDown编辑器，包括Web在线版和PC版，同时还能在多端同步
>* CSDN博客：在线 MarkDown编辑器
>* Macdown：Mac编辑器，基于Mou这个经典的Markdown工具改进而来
>* Typora：这款Markdown编辑器与前面所有的编辑器最大的区别就是他没有文本预览界面

>大部分的Markdown编辑器都提供了类似富文本编辑器的工具栏

![](http://img.blog.csdn.net/20161021152938292)

>如果用户忘记了某些格式的标识符，可以通过工具栏进行找到对应格式的标识符，Markdown提供了简洁、高效的文档标记语法，被广泛运用于各种开源项目的README文档、说明文档等，同时Markdown语法还兼容HTML语法


----------
**项目文档生成器**
>帮助开发者展示项目文档的工具：MkDocs，该工具的项目地址为http://www.mkdocs.org/
>该工具生成的界面，左边是项目的文档结构，右边则是对应的文档说明，简洁明了，一目了然
>通过这个工具可以清晰地展现项目文档，这个项目仅仅是通过Markdown文件就可以生成，同时还可以设置不同的主题和风格，适用于开发者进行文档管理
>类似的工具还有Raneto Docs（http://raneto.com），这些工具基本上都是一个原理，Markdown的优势可见一斑。