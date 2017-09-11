[toc]

#Linux命令

##清空命令行

```
reset
```

##文件操作

```
ls -l  显示文件列表 
ls -la  显示所有文件列表
ls -l ja*  显示经过通配符查找的结果

touch today.c  创建文件

cp src.txt dest.txt  复制文件
cp -i src.txt dest.txt  复制文件并询问
cp src.txt .  复制文件到当前目录
cp -R 目录 dest  复制整个目录到指定路径
cp c_?1 ../  经过通配符查找的结果复制到上级目录

rm file  删除文件
rm -rf 目录  删除目录
file 文件  查看文件类型
cat file  查看文件内容
cat -n file 查看文件内容并显示行号
tail -n 10 file  查看文件最后10行
head -n 10 file  查看文件最开始的10行
```

##用户管理

```
useradd -m jack 创建用户的同时，创建了home目录
userdel -r jack 删除用户
passwd jack 修改密码

groupadd androidgroup 创建组
usermod -G androidgroup jack 分配用户到组

chown user.group file  改变文件的所属用户和组
chown user file  改变文件的所属用户
chown .group file  改变文件的所属组
```

##权限管理

```
chmod 644 file  修改文件权限
chmod u+x file  给用户加上执行权限
chmod u-x file  给用户删去执行权限

umask 026  改变创建目录的默认权限（777-026 = 751）
```

常见的文件权限

```
d目录 r读 w写 x执行
rwx必须是固定顺序
比如：drwxr-xr-x
```

d|rwx|r-x|r-x
--|--|
目录|文件所属用户具备的权限|文件所属用户的所属组具备的权限|统的其他用户具备的权限


权限对应的数字说明

权限|二进制|八进制
--|--|
--- |000|0
--x	|001|1
-w-	|010|2
-wx	|011|3
r--	|100|4
r-x	|101|5
rw-	|110|6
rwx	|111|7

##线程管理

Linux下安装

```
apt-get install manpages-posix-dev
```

Linux下查看文档

```
man -k pthread
#查看具体某个方法
man pthread_create
man pthread_join
```

Linux下创建线程

```
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

void* thr_fun(void* arg){
	char* no = (char*)arg;
	int i = 0;
	for(; i < 10; i++){
		printf("%s thread, i:%d\n",no,i);
		if(i==5){
			//线程退出（自己退出自己）
			//别人退出线程，使用pthread_cancel	
			pthread_exit(2);
					
		}
	}	
	return 1;
}

void main(){
	//创建线程
	//参数一：线程id，参数二：线程的属性，NULL默认属性
	//参数三：thr_fun，线程创建之后执行的函数，参数四：传进函数的值
	pthread_t tid;
	pthread_create(&tid,NULL,thr_fun,"1");
	
	//线程礼让，等待tid线程结束
	//参数一：线程id，参数二：thr_fun与退出时传入的参数的内容
	void* rval;
	pthread_join(tid,&rval);
	printf("rval:%d\n",(int)rval);
}
```

Linux下编译文件成可执行文件，记得使用pthread库要加上依赖

```
gcc 01.c -o 01 -lpthread
```

Linux下线程互斥锁

```
#include <stdlib.h>                                                         
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

int i = 0;
//互斥锁
pthread_mutex_t mutex;

void* thr_fun(void* arg){
	//加锁
	pthread_mutex_lock(&mutex);
	char* no = (char*)arg;
	for(;i < 5; i++){
		printf("%s thread, i:%d\n",no,i);
		sleep(1);
	}
	i=0;
	//解锁
	pthread_mutex_unlock(&mutex);
}

void main(){
	pthread_t tid1, tid2;
	//初始化互斥锁
	pthread_mutex_init(&mutex,NULL);

	pthread_create(&tid1,NULL,thr_fun,"No1");
	pthread_create(&tid2,NULL,thr_fun,"No2");

	pthread_join(tid1,NULL);
	pthread_join(tid2,NULL);

	//销毁互斥锁
	pthread_mutex_destroy(&mutex);
}
```

Linux下线程生产消费机制

```
#include <stdlib.h>                                                      
#include <stdio.h>
#include <unistd.h>
#include <pthread.h>

//消费者数量
#define CONSUMER_NUM 2
//生产者数量
#define PRODUCER_NUM 2
pthread_t pids[CONSUMER_NUM+PRODUCER_NUM];
//产品队列
int ready = 0;
//互斥锁
pthread_mutex_t mutex;
//条件变量（类似于等待唤醒机制）
pthread_cond_t has_product;

//生产者
void* producer(void* arg){
	int no = (int)arg;
	for(;;){
		pthread_mutex_lock(&mutex);

		//往队列中添加产品
		ready++;
		printf("生产者%d号, 生产出了产品\n",no);
		
		//通知消费者，有新的产品可以消费了
		pthread_cond_signal(&has_product);

		pthread_mutex_unlock(&mutex);
		sleep(1);
	}
}

//消费者
void* consumer(void* arg){
	int num = (int)arg;
	for(;;){
		pthread_mutex_lock(&mutex);

		while(ready==0){
			//没有产品，继续等待
			pthread_cond_wait(&has_product,&mutex);
		}

		//有产品，消费产品
		ready--;
		printf("消费者%d号，消费了产品\n",num);

		pthread_mutex_unlock(&mutex);
		sleep(1);
	}
}

void main(){
	//初始化互斥锁和条件变量                                                
 	pthread_mutex_init(&mutex,NULL);
  	pthread_cond_init(&has_product,NULL);

	int i;
	for(i=0; i<PRODUCER_NUM;i++){
		//生产者线程
		pthread_create(&pids[i],NULL,producer,(void*)i);
	}
	
	for(i=0; i<CONSUMER_NUM;i++){
		//消费者线程
		pthread_create(&pids[PRODUCER_NUM+i],NULL,consumer,(void*)i);
	}
	
	//等待
	sleep(2);

	for(i=0; i<PRODUCER_NUM+CONSUMER_NUM;i++){
		pthread_join(pids[i],NULL);
	}
	
	//销毁互斥锁和条件变量
	pthread_mutex_destroy(&mutex);
	pthread_cond_destroy(&has_product);
}
```

#shell命令

##定义脚本的版本

```
#!/bin/bash   
```

##引入sh文件

```
#引入当前目录的my脚本
source ./my.sh
#写法等同于
. ./my.sh
```

##定义变量       

```                                           
a=10
echo $a
``` 

##定义字符串

``` 
text="i love shell"
echo $text
``` 

##命令变量

命令的执行结果的输出作为变量的值，date和who都是可以输出结果的命令

``` 
textdate=`date`
textwho=$(who)
echo $textdate
echo $textwho
```

##输出重定向

* >：将ls -al 这个命令输出结果全部复制到file文件中
* >>：将end字符串追加到file文件中

标准输入输出

* 0：STDIN 标准输入
* 1：STDOUT 标准输出
* 2：STDERR 标准错误

```
ls -al >file
echo "end" >>file
```

将sh文件的所有输出信息复制到file文件中
```
exec 1>file
```

将sh文件的错误信息复制到file文件中
```
exec 2>file
```

自定义输出信息

```
exec 7>file
echo "Hi" >&7
```

在Linux模式下，可以执行某个sh文件，将输出的结果重定向到file文件中，不在屏幕输出

```
./aa.sh 1>file
```

##输入重定向

将file文件中的行数、单词数、字节数输出

```
wc < file
```

##整数计算

expr命令一般用于整数值和字符串的计算中

```
a=10
b=20
c=$(expr $b / $a)
#写法等同于
d=$[$b / $a]
```

##浮点数计算

* bc命令：用于计算浮点数的base calculate
* scale=4：是bc命令里说明显示的小数位数
* 管道：一个命令的输出作为另一个命令的输入

```
e=$(echo "scale=4; 10 / 3" | bc)
```

##内联输入重定向

这里将输入的式子作为bc指令的表达式，结果是5*4/3

```
f=$(bc << EOF
scale=4
a1 = (5 * 4)
al / 3
EOF
)
```

##命令执行退出的状态

```
echo $?
```

返回值

* 0 成功
* 1 未知错误
* 126 命令不可执行
* 127 没有找到命令

自定义退出状态

```
#!/bin/bash
echo "my shell exit"
exit_5
```

##条件控制语句

if语法的如下，elif、else、&& 命令，可以根据情况增加或者不增加，注意只有if命令的退出状态码为0，才会执行then部分

```
if 命令 && 命令
then 
   命令
elif 命令
then 
   命令
else
   命令
fi
```

例子：查找本地是否包含user这个用户

```
#!/bin/bash
user=Hensen
if grep $user /etc/passwd
then 
   echo "ok"
   ls -a /home/$user/
fi
```

任意的数学赋值或者比较表达式，用双括号表示

```
if(( a++ > 90 ))
```

##test命令

用方括号来表示text命令，注意需要空格

```
#!/bin/bash
a=10
b=5
if [ $a -gt $b ]
then
	echo "$a greater than $b"
else
	echo "$a smaller than $b"
fi
```

test数值比较

* -gt：大于
* -eq：等于
* -le：小于
* -ne：不等于

test字符串比较

* str1 == str2
* str1 != str2
* str1 < str2
* -n str1：长度是否非0
* -z str1：长度是否为0

文件比较

* -d：检查是否存在，并且是一个目录
* -e：检查file是否存在
* -f：检查是否存在，并且是一个文件
* -r：检查是否存在，并且可读
* -w：检查是否存在，并且可写
* -x：检查是否存在，并且可执行
* file1 -nt file2：file1比file2新
* file1 -ot file2：file1比file2旧

##case命令

语法

```
case 变量 in
pattern1) 命令;;
pattern2) 命令;;
*) 默认命令;;
esac
```

例子

```
#!/bin/bash
user=rose
case $user in
rose)
	echo "hi,i am rose";;
ricky)
	echo "hi,i am ricky";;
*)
	echo "defaults"
esac
```

##for循环

语法

```
for var in list
do
	命令
done
```

例子

```
#!/bin/bash
#第一种方式
for name in Mother Father Brother
do
	echo $name
done
#第二种方式
list = "Mother,Father,Brother"
IFS=$,
for name in $list
do
	echo $name
done
```

##while命令

语法

```
while [ 命令 ]
do
	命令
done	
```

例子，与break一起用

```
#!/bin/bash
a=10
while [ $a -gt 0 ]
do
	echo "num:$a"
	a=$[ $a - 1 ]
	if[ $a -eq 5 ]
	then 
		echo "break"
		break
	fi
done
```

##函数

函数定义

```
#第一种写法
function func
{
	echo "func";
}
#第二种写法
func(){
	echo "func2"
}
```

函数传参和返回值

```
function func
{
	echo $[ $1 + $2 ];
}
value=$(func 10 90)
echo "value:$value"
```

##脚本传参

```
#!/bin/bash
echo "参数的总数:$#"
#遍历所有传进来的参数
for param in "$@"
do
	echo "param:$param"
done
```

#MakeFile命令

##介绍

MakeFile主要是将多个.c文件统一编译打包成可执行文件，它可以自动识别某个目标文件是否被修改过，从而决定需不需要进行重新编译，主要是通过目标文件的修改时间是否大于.c文件，如果大于则说明.c文件未被修改过，不用重新编译该文件

##场景

* 在很多C/C++开源项目中，configure文件用来检查系统配置生成配置文件
* MakeFile文件会引入configure生成的配置文件，根据内容生成我们需要的动态库文件
* Shell脚本就是先运行configure文件之后再运行MakeFile文件继续编译生成动态库文件

##使用

正规写法

```
#myapp是目标文件，默认的make指令会执行第一行的指令
#:后的文件都是依赖，注意gcc前面必须是TAB键开头
myapp:main.o plus.o minus.o multi.o divi.o
	gcc main.o plus.o minus.o multi.o divi.o -o myapp
main.o:main.c
	gcc -c main.c
plus.o:plus.c
	gcc -c plus.c
minus.o:minus.c
	gcc -c minus.c
multi.o:multi.c
	gcc -c multi.c
divi.o:divi.c
	gcc -c divi.c

#声明伪目标，表示make clean会执行下面的指令
.PHONY:clean
clean:
	rm -f *.o
	rm -f myapp
```

简化写法，此段代码与上面代码效果一样

```
OBJECTS=main.o plus.o minus.o multi.o divi.o
myapp:$(OBJECTS)
	gcc $^ -o $@

%.o:%.c
	gcc -c $^ -o $@

.PHONY:clean
clean:
	rm -f *.o
	rm -f myapp
```

获取当前目录所有.c文件编译成.o文件

```
#所有.c源文件，wildcard是个函数
SOURCES=$(wildcard *.c)
#把.c后缀，替换成.o后缀，patsubst是个函数
OBJECTS=$(patsubst %.c,%.o,$(SOURCES))

myapp:$(OBJECTS)
	gcc $^ -o $@

%.o:%.c
	gcc -c $^ -o $@

.PHONY:clean
clean:
	rm -f *.o
	rm -f myapp
```

我们可以通过echo打印出SOURCES和OBJECTS，输入make test命令即可

```
#增加"@"符号echo不会打印出命令
test:
	@echo $(SOURCES)
	@echo $(OBJECTS)
```

##变量

```
#递归展开式
#可以引用还没有定义的变量，展开是引用时展开
str2=$(str1)
str1=hello

#直接展开式
#必须引用定义好了的变量，定义之后就会展开
str3 := android
str4 := $(str3)
str5 := $(str1) world

#变量的值追加
str5 += hello
```

##函数

```
#自定义函数
myfun=$2 $1
#变量等于函数的执行结构
myfun_ret=$(call myfun,20,10)
```

##Android.mk

* LOCAL_PATH := $(call my-dir)：放在第一行，地址当前所在目录
* include $(CLEAR_VARS)：编译模块时，清空LOCAL_MODULE等参数
* LOCAL_MODULE：模块名称
* LOCAL_SRC_FILES：编译需要的源文件
* LOCAL_C_INCLUDES：需要的头文件
* LOCAL_SHARED_LIBRARIES：编译需要的动态库

#gba命令

gba是一款在Linux下的调试工具，安装gba

```
apt-get install gba
```

如果需要gba调试程序的话，编译加上-g参数

```
gcc file.c -g -o file
```

进入调试

```
gdb file
```

开始调试

```
start
```

显示代码

```
list ：查看当前代码，简写l
list 函数名称：查看函数内容
list 行数：查看某行代码
```

执行下一步

```
next：简写n
```

查看变量

```
print 变量名：简写p
```

进入到某个函数

```
step：简写s
```

设置断点

```
break 行号：简写b
```

全速运行

```
continue：遇到断点会停止，简写b
```

查看断点信息

```
info breakpoints
```

删除断点

```
delete breakpoints 断点编号
```

修改变量的值

```
set var 变量=值
```

程序调用堆栈，当前函数之前的所有已调用函数列表，每一个都分配一个“帧”，最近调用的函数在0号帧里

```
backtrace：简写bt
```

切换栈帧

```
frame 帧数：切换指定栈帧，方便查看指定栈帧的变量
```

自动显示，在调试每一步的过程中都会输出

```
display 变量名
```

取消自动显示

```
undisplay 行号
```

查看内存布局

```
x /20 地址
x /20 buff：查看buff数组的前20个元素
```

程序非正常退出，如何查看错误

1. ulimit -a：查看core文件是否分配大小
2. ulimit -c 1024：创建的core文件大小为1024字节
3. gcc file.c -g -o file：编译链接得到带有-g选项的可执行程序
4. ./file：执行程序，会生成core日志文件
5. gdb file core：打开日志文件，定位错误信息到具体的代码行数