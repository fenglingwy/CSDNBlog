#Android群英传神兵利器读书笔记——第三章：Android Studio奇技淫巧


----------
>**这篇文章篇幅较长，可以使用版权声明下面的目录，找到感兴趣的进行阅读**

##目录##
>* **3.1 Android Studio使用初探**
	* Project面板
	* Stucture面板
	* Android Monitor
	* Keymap
	* Tip of the Day
	* 快速查找
	* Search Action
	* 演示模式
>* **3.2 Android Studio使用进阶**
	* 操作与导航
	* 快速重构
	* 代码模板
	* 内置模板
	* 自定义代码注释模板
	* 代码分析
	* 在Android Studio中进行版本管理
>* **3.3 Android Studio新功能**
	* 项目模板
	* ThemeEditor
	* Image Asset&&Vector Asset
	* Android Monitor
	* Instant Run
	* Productivity Guide
>* **3.4 Android Studio插件**
	* Ignore
	* 自动生成代码类插件
	* 主题插件
>* **3.5 Android Studio资源网站**
	*  Android Studio中文社区
	*  Android Studio问答社区

----------
##**3.1 Android Studio使用初探**
本人感觉这章对刚使用Android Studio的初学者来说很有用，里面很多技巧需要自己慢慢摸索，建议养成使用快捷键的习惯，久而久之，会给你的编程带来极大的效率。本章作者主要以Mac的快捷键来介绍的，文章最后会给出快捷键附录

##Project面板

在Android Studio最左边可以找到Project标签，这里是开发者管理项目的地方

![](http://img.blog.csdn.net/20161112231134380)

Project标签下有几个选项卡，点击右边的箭头，可以打开切换菜单

![](http://img.blog.csdn.net/20161112231220849)

Project标签展示的是整个项目的目录结构，完全按照文件系统的目录结构来进行展示，不过Android工程选项卡是开发中使用最多的

![](http://img.blog.csdn.net/20161112231410724)

Android选项卡不是按照文件目录结构对项目进行的整理，而是按照module来进行的整理。每个module不论是主项目还是库项目都是一个独立的文件夹，另外所有的Gradle脚本都在一个单独的目录中——Gradle Scrpts


----------
##Stucture面板
Stucture面板在Eclipse时代就已经是标配了，Android Studio同样也进行了集成

![](http://img.blog.csdn.net/20161112231737636)

与Eclipse一样，Stucture标签不仅可以显示代码结构，也可以显示其成员变量、静态常量、方法等信息，而在Android Studio中不仅是代码，XML布局、脚本也可以显示其Stucture信息


----------
##Android Monitor
这个面板应该是开发者使用的非常多的一个面板，这里会显示Debug程序的Log信息，在设置中可以对Logcat所打印的Log根据其种类设置成不同的颜色

![](http://img.blog.csdn.net/20161112232046796)


----------
##Keymap
Android Studio可以设置各种类型的快捷键，在Setting中找到Keymap标签，在下拉菜单中可以选择各种内置的快捷键类型，本文中所有的快捷键都指的是默认的Android Studio快捷键

![](http://img.blog.csdn.net/20161112232259072)


----------
##Tip of the Day
在Android Studio菜单栏的Help标签下，选择Tip of the Day选项，可以打开Android Studio的Tips提示

![](http://img.blog.csdn.net/20161112232416957)

这里面会随机显示一条Android Studio的使用提示，Tip of the Day默认是在启动时显示的，但是很多开发者都不会让它启动时显示，实际上这里才是Android Studio的技巧集萃，里面都是非常实用的使用技巧，每天抽一点时间，简单看下这个Tips，用不了多久这些带给你的时间收益，绝对远大于你看这些Tips的时间成本

![](http://img.blog.csdn.net/20161112232447161)

出了这里的Tips，IntelliJ IDEA的官方网站也应该是开发者经常关注的地方，特别是它的功能介绍，地址：https://www.jetbrains.com/idea/whatsnew/


----------
##快速查找
Android Studio自带的强大全局快捷搜索，只需要双击"shift"键即可，在这个Search EveryWhere中，你只需要输入要查找的内容（可以是模糊查询，有关键字即可），下面就可以实时显示查找出的结果。当勾选上面的复选框——Include non-project items后，还可以搜索非项目中的内容，例如引用的jar包中的内容

![](http://img.blog.csdn.net/20161112233141497)


----------
##Search Action
Android Studio快捷键众多，因此Android Studio提供了一个类似搜索指令的入口，通过快捷键"Command+Shift+A"可以快速调出这个搜索入口，例如要查找打开最近的工程这样一个指令，可以直接输入"Open Recent"，按下回车键后就可以直接使用这条指令

![](http://img.blog.csdn.net/20161112233406110)

再例如查看方法调用栈的快捷键，如果一时无法想到，可以通过输入hier找到该指令及其快捷键

![](http://img.blog.csdn.net/20161112233505483)


----------
##演示模式
Android Studio为开发者提供了极为方便的演示模式，打开菜单栏的View选项，在最下面找到几种演示模式，通过几种模式可以在连接投影仪时非常方便地全屏显示代码区域

![](http://img.blog.csdn.net/20161112233639062)


----------
##**3.2 Android Studio使用进阶**
##操作与导航
##单词选择
在Android Studio中，通过键盘操作来选择单词是编辑代码时最常用的操作，通过"Option+←"快捷键来实现按单词的光标移动，Android Studio也提供了安装驼峰命名法来实现光标移动的设置，在设置中开启"Use CamelHumps words"即可使用该功能，开启后，再通过"Option+←"就可以按照驼峰来移动光标

![](http://img.blog.csdn.net/20161112234410129)


----------
##显示最近操作、修改
在Android Studio中，使用"Command+E"和"Command+Shift+E"快捷键，以快速显示最近文件操作和文件修改

使用"Command+E"快捷键显示最近浏览过的文件

![](http://img.blog.csdn.net/20161112235138792)

使用"Command+Shift+E"快捷键显示了最近编辑过的文件，与此同时使用"Control+Tab"快捷键进行各个界面的切换

![](http://img.blog.csdn.net/20161112235401680)


----------
##操作记录
当开发者在浏览代码时，通常会进行代码的跳转，而当想回到之前浏览过的地方时就比较麻烦了，而Android Studio保存了每个操作的历史，通过快捷键"Command+Option+Left\Right"来进行访问位置的导航


----------
##移动行
整体移动某行是很常用的方法，在Android Studio中通过"Option+Shift+方向键上\方向键下"就可以实现某一行的上下移动


----------
##查找调用
在开发中，查找一个方法在何处被调用过或者查找一个ID在哪里被引用过是经常性操作，例如要查找initViews()的调用处，只要单击鼠标右键，选择"Find Usages"即可，当然你也可以使用"Option+F7"进行快速查找

![](http://img.blog.csdn.net/20161113001019238)


----------
##快速方法操作
在不同的方法间进行跳转是开发者了解程序架构的必备技能，在Android Studio中，开发者可以通过按住"Command"键，并点击方法名的方式进入方法，查看方法详情，你也可以通过直接使用"Command+B"快捷键进入一个方法


----------
##查找参数定义与文档
通过快捷键"Command+P"可以快速查看该方法的参数定义

![](http://img.blog.csdn.net/20161113001409343)

使用快捷键"F1"查看API文档

![](http://img.blog.csdn.net/20161113001446182)

如果你想像使用Eclipse一样，当鼠标放上去的时候就显示文档的提示，那么可以在设置中进行设置，Editor-General-Show quick documention on mouse move

![](http://img.blog.csdn.net/20161113001614777)


----------
##快速行操作

通过快捷键"Command+Shift+Up\Down"，可以迅速地将一行移动到上面一行或者下面一行，而不需要通过剪切来进行两行的交换

通过快捷键"Command+BackSpace"删除一行

通过快捷键"Command+D"迅速复制上一行的代码，同时将光标停留在变量名的地方


----------
##快速断点

条件断点和普通断点一样，直接在左边的编辑面板上点击就能生成，而要给一个普通断点增加条件功能，只需要普通断点上单击鼠标右键，在弹出菜单的Condition中填入断点条件即可

![](http://img.blog.csdn.net/20161113002201161)

在调试时，开发者可能会临时增加一些断点，也就是说，开发者实际上只想让这个断点执行一次，下次就不想在这个地方继续执行断点了，通过快捷键"Command+Option+Shift+F8"，即可作为临时断点

![](http://img.blog.csdn.net/20161113002401817)

临时断点与普通断点区别就在于临时断点上有一个数字"1"，当临时断点执行一次后就会自动消失


----------
##异常断点
举个例子，程序中最常见的Crash莫过于NullPointerException，如何在程序中出现NullPointerException的地方都打上断点呢？其实根本不需要这么做，开发者只要打开Run-View breakpoints界面，点击右上角的"+"，选择Java Exception Breakpoints，并输入要监听的异常即可

![](http://img.blog.csdn.net/20161113002738445)

笔者在这里选择监听NullPointerException，那么在程序时不需要设置任何断点，只要App因为NullPointerException异常而导致崩溃，系统就会在对应的地方自动断点并暂停

![](http://img.blog.csdn.net/20161113002905493)


----------
##日志断点
开发者经常会遇到这样的情况，整个工程的代码已经写完了，突然出现了一个bug需要加一行Log进行调试，因为这一行Log要把整个工程都编译一遍，这是非常痛苦的事，而实际上，Android Studio已经提供了针对这个问题的解决方案，那就是日志断点

例如下面这个例子，开发者需要在每次循环中打出一句Log，但是又不想增加一行Log

![](http://img.blog.csdn.net/20161113003143995)

此时可以使用日志断点来增加Log而不需要修改代码，首先需要打上一个普通断点，然后在断点单击鼠标右键，选择suspend属性为false，并在下面的Log evaluated expression中写入日志信息即可，这样设置后，在程序运行时就不用重新编译，而且会在断点处打出你需要的日志信息

![这里写图片描述](http://img.blog.csdn.net/20161113003336636)


----------
##多重选择
当代码的上下文有很多相同的代码，而开发者又需要同时对这些代码块进行操作时，就可以使用多重选择功能，例如，只要将光标放在第一个int处，使用快捷键"Control+G"就可以选中第一个int，再次按一次快捷键"Control+G"就可以选中第二个int

![](http://img.blog.csdn.net/20161113131614480)

除了通过相似性进行多重选择，Android Studio还提供了通过列进行多重选择的方式，只需要按住"Option"键并拖动即可

 ![](http://img.blog.csdn.net/20161113131713450)

除了上面两种类似的操作，Android Studio也支持多光标的操作方式，通过快捷键"Option+Shift+鼠标点击"就可以增加一个新的编辑光标

![](http://img.blog.csdn.net/20161113131840077)


----------
##快速完成

通过快捷键"Command+Shift+Enter"，在很多地方可以让Android Studio快速完成某些操作，例如方法体大括号的添加、行尾分号的添加、自动格式化该行等操作


----------
##代码提示

Android Studio提供了非常强大的智能提示功能，使用快捷键"Control+Space"就可以在代码的任何地方调出代码提示，在使用代码提示时，有一点需要注意的是，当显示出候选的提示后，通过Enter键可以完成提示的输入，另外通过Tab键同样可以完成提示的输入，区别是它会将后面已经输入的提示全部删除，而Enter键会保留后面的输入

![](http://img.blog.csdn.net/20161113132349392)

除了使用智能提示之外，在Android Studio中，还提供了快捷键"Control+Shift+Space"以显示更加智能的代码提示

代码提示不仅可以用于代码编写的过程中，在程序出现错误时，也可以借助快速完成快捷键"Option+Enter"获取代码修改提示，例如笔者使用快捷模板logi，产生一条日志信息，这时在TAG变量上使用快捷键"Option+Enter"，选择"Create constant field 'TAG'"即可

![](http://img.blog.csdn.net/20161113132749690)


----------
##调试中计算变量的值
在调试过程中，只要按住Alt键，点击代码中的表达式，即可显示表达式的值


----------
##设置变量命名代码风格
根据Google的代码风格指南，类的成员变量通常要以m开头，而静态成员变量通常要以s开头，因此可以在设置中设置变量的命名规则

![](http://img.blog.csdn.net/20161113213424049)

在Field的Name prefix中设置m，在Static field的Name prefix中设置s，这样在输入一个变量的名字时，就可以自动补全m或者s

![](http://img.blog.csdn.net/20161113213530300)


----------
##查看大纲
当项目很大的时候，通过使用快捷键"Command+F12"，可以调出大纲界面，即显示方法和成员变量列表

![](http://img.blog.csdn.net/20161113213639567)

通过输入方法名，可以快速定位到方法，同时它还支持模糊查询，查询方法的一部分关键字也能进行筛选

![](http://img.blog.csdn.net/20161113213726922)


----------
##书签
在接手老项目的代码或者在调试代码时，往往需要分析代码的思想，经常需要记录一些关键的代码、方法，这时候使用书签来记录就是最好的方式，类似在Chrome中添加书签，通过快捷键F3可以将一处代码添加到书签或者从书签中删除

![](http://img.blog.csdn.net/20161113213941523)

添加到书签的代码，在行数旁边会有一个小钩，同时在Favorites标签中，可以找到相应的Bookmarks

![](http://img.blog.csdn.net/20161113214031236)

另外通过快捷键"Command+F3"可以调出书签面板、显示所有书签

![](http://img.blog.csdn.net/20161113214102322)


----------
##附加调试
开发者一定遇到过当项目很大时，编译一次需要很长时间，而这时候又需要调试程序的情况。那么除了直接使用Debug运行程序以外，还可以使用attach to debugger的方式。

在ADB连接手机的情况下，点击attach to debugger按钮并选择要调试的程序（只能调试Debug签名的App），即进入可调式模式，不需要通过Debug运行程序


----------
##其他操作技巧

* 代码折叠："Command+-"和"Command++"，可以对一段代码进行折叠和展开
* 在文件系统中打开文件：选中文件单击鼠标右键，选择Reveal in Finder同样可以在文件系统中打开文件

![](http://img.blog.csdn.net/20161113214529706)

* 预览方法定义：开发者在调试代码的时候，如果想查看某个方法的定义，但又不想跳转到方法所在的类，那么就可以使用快捷键"Command+Y"在当前页面上对指定方法进行预览

![](http://img.blog.csdn.net/20161113214707423)

* 拆分窗口：通常情况下，在编辑界面只有一个界面，通过窗口拆分，可以同时展示更多的界面，在菜单栏中选择Window->Editor Tabs->Split vertical\horizontal，这样就可以在整个编辑区域显示多个编辑界面
* 相关文件：对于Activity来说，通常都有与之对应的XML布局文件，这些布局文件作为Activity的相关文件会被标记在类的最前面，点击这个标记就可以关联到相应的XML文件

![](http://img.blog.csdn.net/20161113215110600)

* 查找快捷键：在下拉框中，Android Studio内置了各个平台的快捷键模板，通过切换可以找到相应的平台，找到需要查找的快捷键，记住其名称，再切换回自己系统的快捷键，通过名称找到对应的快捷键即可，而且在旁边的输入框中，Android Studio提供了通过输入按键进行快捷键查找的方式，非常方便

![](http://img.blog.csdn.net/20161113215509226)


----------
##快速重构
##重构入口
当选择一个代码片段准备重构时，Android Studio提供一个快捷的重构入口

![](http://img.blog.csdn.net/20161113220228918)

通过快捷键"Control+T"可以打开这个重构的入口，或者通过单击鼠标右键，选择"Refactor"调出这个界面

![](http://img.blog.csdn.net/20161113220238212)


----------
##Sorround With
在开发中，开发者经常要对某行代码进行重构，例如增加判空的if条件，或者是增加try catch，那么可以使用快捷键"Command+Option+T"来进行操作，当执行这个快捷键之后，选择相应的Surround类型，就可以快速将该Surround类型作用到选择的代码上

![](http://img.blog.csdn.net/20161113230319957)


----------
##快速提示
通过快捷键"Option+Enter"可以迅速调出快速提示，例如当一行代码写完，还差一个分号时，通过快捷键"Option+Enter"快速提示，Android Studio可以快速帮你补全分号、换行，并格式化该行代码。再例如，你可以先写一个还未生成的方法，通过快捷键"Option+Enter"快速提示来让Android Studio帮你生成这个方法

![](http://img.blog.csdn.net/20161113230552813)

再例如，开发者有时候会在代码中写一些if...else if...这样的条件判断语句，但是在重构的时候，你很可能想把它换成switch语句，那么通过Android Studio的快速提示，这样的转换就是完全智能的，只要在if上使用"Option+Enter"快速提示即可

![](http://img.blog.csdn.net/20161113230732769)


----------
##快速国际化
在项目中进行国际化，是通过建立不同语言的strings.xml文件来实现的，在Android Studio中提供了translation editor帮助开发者快速创建国际化文件

要使用这个功能，开发者只需要打开string.xml文件，打开右上角的提示"Open editor"，即可打开translation editor，在translation editor中，选择左上角的"地球"图标即可打开资源国际化选择器

![](http://img.blog.csdn.net/20161113231402055)

选择相应的语言，即可在目录下产生该语言对应的资源文件

![](http://img.blog.csdn.net/20161113231503790)


----------
##Extract的妙用
Extract在重构代码时是非常有用的，例如将一段重复的代码抽出来作为一个方法

 ![](http://img.blog.csdn.net/20161113231558186)

通过Extract Method，可以将一个代码段抽出作为一个方法，并且可以设置该方法的访问类型

![](http://img.blog.csdn.net/20161113231652734)

在Extract还可以抽取XML文件中的属性作为Style，供其他View复用，那么直接在这个View的XML布局代码中，执行Extract-Style

![](http://img.blog.csdn.net/20161113231836104)

在弹出的界面中设置抽取的Style的名字和要抽取的属性即可

![](http://img.blog.csdn.net/20161113231906503)

Extract不仅可以抽取Style，还可以抽取布局Layout，使用方法基本一致，这里就不再演示了，在代码中，Extract可以提取各种变量、参数、常量


----------
##Stucturally Search
Stucturally Search是Android Studio中一个非常重要的功能，通过Find Action方法，可以快速打开该功能

![](http://img.blog.csdn.net/20161113232129465)

Stucturally Search界面

![](http://img.blog.csdn.net/20161113232202450)

在编辑区域，开发者可以编辑各种要搜索的代码，而最关键的是，可以使用\$xxx\$标志进行匹配搜索，通过这样的搜索就能找到initViews的方法在哪个地方使用

![](http://img.blog.csdn.net/20161113232351857)


----------
##代码模板
##内置模板
Android Studio跟Eclipse一样，内置了很多代码的快速输入模板，例如Eclipse的——syso，Android Studio同样有很多这样的代码模板，在代码编写过程中，只需要使用快捷键"Command+J"就可以调出这些代码模板

![](http://img.blog.csdn.net/20161113232551437)

例如"fori"代表快捷输入for循环，"ifn"代表快捷输入"if null"，等等，当然你还可以增加自己的代码模块

![](http://img.blog.csdn.net/20161113232700906)

在设置中找到Live Templates标签，即可找到所有的代码模板，这里以Log的快捷板为例

![](http://img.blog.csdn.net/20161113232800298)


----------
##后缀模板
前面提到使用"Command+J"调出内置代码模板，同样也给出了一些非常常用的类提供了通过后缀的方式来调出代码模板，例如要给一个List写一个遍历语句，其实并不需要通过内置模板来实现，直接在List后面跟上".for"，即可快速打开foreach遍历语句

![](http://img.blog.csdn.net/20161113233008112)

另外，还可以使用".cast"来快速生成类型转换模板

![](http://img.blog.csdn.net/20161113233043345)


----------
##自定义代码注释模板
##方法注释
在Android Studio中，系统给开发者提供了默认的方法注释模板在方法名上一行输"/**"，再按Enter键确认，即可获取方法的注释代码

![](http://img.blog.csdn.net/20161113233212456)

但和Android一样，Android Studio也提供了强大的自定义功能，首先需要打开设置，选择Live Templates，接下来点击右边栏的加号，选择增加一个Template Group，并在该Group下新增一个Template

![](http://img.blog.csdn.net/20161113233350147)

选中"ma"自定义注释模板，在下方编辑区域中进行注释代码的编辑

![](http://img.blog.csdn.net/20161113233717521)

其中使用\$符号包裹的即为变量，可以通过右边的按钮"Edit variable"来进行修改
![](http://img.blog.csdn.net/20161113233447101)

这里给变量date提供了date()函数的赋值，即获得当前系统时间，并动态赋值给变量，最后，点击下方的change连接，选择在何时对该注释进行生效

![](http://img.blog.csdn.net/20161113233535754)

一般来说，选择Declaration即可，表名在申明时即生效，通过这样的配置后，在方法前输入"ma"即可弹出该模板，按Enter键后确认输入

![](http://img.blog.csdn.net/20161113234026890)


----------
##文件、类注释

当系统生成一个类、接口等文件时，系统会默认生成一些代码和注释

![](http://img.blog.csdn.net/20161113234127641)

和方法注释一样，开发者对这些注释同样可以完全自定义，首先，进入设置界面，选择"File and Code Templates"即可打开代码注释模板界面

![](http://img.blog.csdn.net/20161113234252956)

接下来，选择Include标签，这里的模板，类似于在布局文件中被Include进来的布局，即一些通用模板，例如笔者配置的两个模板

![](http://img.blog.csdn.net/20161113234400164)

![](http://img.blog.csdn.net/20161113234409754)

有了这两个相同模板，开发者就可以组合这些模板来创建新的完整的类、文件模板。例如在File标签新创建一个模板文件，命名MyClass并设置代码模板

![](http://img.blog.csdn.net/20161113234526555)

使用起来也非常简单，只需要单击鼠标右键选择New的时候，选择自定义的模板代码即可

![](http://img.blog.csdn.net/20161113234624977)

选择相应的模板后，生成的代码

![](http://img.blog.csdn.net/20161113234649023)

有了这个实例，大家还可以创建更多的模板，例如笔者创建的MyActivity模板

 ![](http://img.blog.csdn.net/20161113234724634)

生成的代码

![](http://img.blog.csdn.net/20161113234747197)

举一反三，笔者在这里再举一个单例模板

![](http://img.blog.csdn.net/20161113234810806)

生成的代码

![](http://img.blog.csdn.net/20161113234836791)


----------
##代码分析

在Android Studio中，Google还提供了很多代码分析工具，这些工具都集中在Android Studio的Analyze菜单中

![](http://img.blog.csdn.net/20161113234943480)


----------
##Inspect Code&&Code Cleanup
通过Inspect Code功能，可以让IDE分析整个工程，类似于Android的Lint分析

![](http://img.blog.csdn.net/20161113235302699)

可见，Inspect Code不仅提供了Lint的检测功能，还提供了一些其他的代码静态分析结果，同时给出了大致的修改意见，你也可以选择Code Cleanup功能来进行自动的代码修复，这两个功能可以在Analyze菜单中找到

![](http://img.blog.csdn.net/20161113235455871)


----------
##Dependencies
在Analyze菜单中，有几个Dependencies选项，通过这几个选项，可以快速分析项目的Dependencies依赖

![](http://img.blog.csdn.net/20161113235621973)


----------
##Analyze Data flow
这个功能用的不是很多，但是在某写情况下，对于熟悉旧的代码非常有帮助，它可以追踪数据流，了解该数据变量的来龙去脉，可以通过Dataflow from local variable的结果和Dataflow to local variable的结果显示出来

![](http://img.blog.csdn.net/20161114000031686)


----------
##方法调用栈
在Android Studio中通过快捷键"Control+Option+H"可以快速找到该方法的调用栈

![](http://img.blog.csdn.net/20161114000045655)


----------
##在Android Studio中进行版本管理

除了使用Android Studio自带的终端进行Git操作，Android Studio还提供了对Git的直接支持，在任意一个界面上单击鼠标右键即可弹出相应的Git操作

![](http://img.blog.csdn.net/20161114000207249)

类似于Source Tree的图形界面

![](http://img.blog.csdn.net/20161114000233827)

同时，Android Studio本地也有一套自己的版本记录系统，在任意文件处单击鼠标右键选择Local History-show history即可

![](http://img.blog.csdn.net/20161114000331374)

在这里可以看到开发者对该文件的操作版本记录

对Git的设置，可以在设置里面搜索Git即可

![](http://img.blog.csdn.net/20161114000413469)

一旦该文件被纳入Git版本管理，文件的颜色会变成对应状态的颜色

![](http://img.blog.csdn.net/20161114000455688)

红色表示未被纳入的新文件，绿色表示已经Add到暂存区的文件，在主界面的VCS菜单选项中，几乎包含了所有的Git操作

![](http://img.blog.csdn.net/20161114000610783)

Android Studio也内置了Github的支持，选择VCS-Import into Version Control-Share Project on Github，即可一键将项目上传到Github

![](http://img.blog.csdn.net/20161114000733894)


----------
##**3.3 Android Studio新功能**
##项目模板
Android Studio在创建Android项目的时候，会让开发者选择自带的项目模板

![](http://img.blog.csdn.net/20161114000829224)

开发者可以根据系统自带的模板，在Android Studio安装目录的~/plugins/android/lib/templates目录下创建自定义模板


----------
##ThemeEditor
在新版的Android Studio中，当打开一个主题文件时，系统会提示开发者在editor进行编辑

![](http://img.blog.csdn.net/20161114120458926)

这个editor就是Android Studio的新功能——Theme Editor，打开后的界面如图

![](http://img.blog.csdn.net/20161114120534660)

这里Android Studio对主题设置进行了可视化编辑，修改设置马上就能看到显示效果


----------
##Image Asset&&Vector Asset
Image Asset和Vector Asset是Android Studio中新增的功能，可以帮助开发者快速的创建不同分辨率的图像和SVG文件

要使用这个功能，可以在res资源目录下单击鼠标右键，选择New

![](http://img.blog.csdn.net/20161114120750552)

单击Image Asset，选择相应的图片并命名，点击next即可自动生成所有分辨率的图片，同时Image Asset还提供了很多图片的处理选项，开发者可以根据自己的需要设置

![](http://img.blog.csdn.net/20161114120851896)

如果选择Vector Asset，则会弹出下面的图

![](http://img.blog.csdn.net/20161114120921896)

如果开发者选择Material Icon，点击Icon图标，就会调出Android Studio的内置SVG图片

![](http://img.blog.csdn.net/20161114121014511)

开发者可以在Android Studio提供的大量SVG图片中选择自己需要的图片，点击OK后，即可生成相应的SVG XML文件

另外开发者也可以选择加载本地的SVG图片

![](http://img.blog.csdn.net/20161114121147685)

点击Next后，即可生成相应的SVG XML文件

![](http://img.blog.csdn.net/20161114121218998)


----------
##Android Monitor
Android Monitor类似于Eclipse上用的DDMS工具，但是Android Monitor的功能更加强大

![](http://img.blog.csdn.net/20161114121311898)

该工具提供了Logcat、Memory、CPU、GPU、NetWork的实时分析工具，可以让开发者在开发过程中了解App的运行情况


----------
##Instant Run
该功能可以说是Android Studio最引人瞩目的一个新功能，开启该功能后，Android Studio将以插件补丁的形式更新App，提供App的调试速度，要开启这个功能，只需要在设置中设置Enable Instant Run即可

![](http://img.blog.csdn.net/20161114121530180)

接下来，在第一次全编译项目运行之后，启动和调试按钮旁边将多出一个闪电标识，如果开发者再对项目有更改，那么点击带闪电标识的启动或调试按钮，就可以非常快的应用修改，显示修改后的程序


----------
##Productivity Guide
Productivity Guide是一个非常有意思的功能，打开Help菜单，就可以打开这个功能

![](http://img.blog.csdn.net/20161114121744490)

打开之后，整个界面如图

![](http://img.blog.csdn.net/20161114121803571)

这里可以显示开发者本次使用Android Studio的总时间、活动时间、已经使用的快捷键次数、代码提示次数等统计信息


----------
##**3.4 Android Studio插件**
 Android Studio继承了Eclipse的插件化思想，因此它拥有非常多插件，开发者可以在网站上找到 Android Studio插件http://plugins.jetbrains.com/?androidstudio


----------
##Ignore
该插件的功能如它名字一样，就是为了给Git项目生成最合适的ignore文件

![](http://img.blog.csdn.net/20161114122057932)

在任意文件上单击鼠标右键选择New，选择.ignore file选项，选择gitignore file（Git）

![](http://img.blog.csdn.net/20161114122145120)

选择后会弹出新的界面，选择Android即可

![](http://img.blog.csdn.net/20161114122208682)


----------
##自动生成代码类插件

* ButterKnife：在代码中的布局文件上单击鼠标右键，选择Generate-Generate ButterKnift就可以自动生成ButterKnife所需要的注解文件
* Selector：可以讲一个drawable文件夹下的图像，自动生成对应的drawable Selector，只要文件名符合安装要求的规范即可
* Gson：将一段json生成所需要的Gson实体
* Parcelable：可以自动生成Parcelable接口所需要的代码
* ViewHolder：可以在getView方法中根据布局文件ID，快速生成对应的ViewHolder
* Prettify：可以根据Layout自动生成该Layout中的View在Java中的findViewById代码


----------
##主题插件
 Android Studio默认只提供了两种主题，即默认主题与Darcula主题，开发者通常都想定义自己的主题，那么下面这个插件就可以完成你的愿望，http://color-themes.com

下载好主题jar文件后，只需要在 Android Studio中选择File-Import Settings

![、](http://img.blog.csdn.net/20161114122837997)

在弹出的菜单中选择要导入jar文件，最后系统提示重启 Android Studio，主题安装完毕，在设置的Editor-Colors&Font选项中就可以找到安装的主题了


----------
##**3.5  Android Studio资源网站**

* Google官方网站：https://developer.android.com/studio/intro/index.html
*  Android Studio中文社区：http://tools.android-studio.org/index.php
*  Android Studio问答社区：http://ask.android-studio.org/?/explore/
*  Android Studio中文论坛：http://forum.android-studio.org/forum.php
*  Android Studio开发者工具下载：http://www.androiddevtools.cn/


----------
##附录

![](http://img.blog.csdn.net/20161114123456234)


----------
##总结

这一章节对于我使用了很久的Android Studio都有一个些新的认识，想想也是，如果没有新的认识自己就是大牛了，哈哈哈。这篇文章篇幅很长，主要是图片占的地方很多，可以说是标准的图文消息了，阅读起来很轻松，我都坚持下来，何况你们呢，加油