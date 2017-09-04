[toc]

##变量

变量不分类型，用$符号可以代替所有类型，这点跟JS的var很像
```
$a = 10;
$b = 5;
echo $a+$b
```

##常量

常量分为const和define，它们的区别是：

1. define是在编译时，会自动将声明的变量替换到程序中
2. const是在运行是，会自动将声明的变量替换到程序中

```
const THE_VALUE = 10;
define('THE_VALUE' = 10);
```

##函数

1、无参函数
```
function helloWord(){
	echo 'Hello World';
}
```
2、有参函数

```
function sum($a,$b){
	echo $a+$b;
}
```
##数组

1、创建数组
```
$arr = array("java","c","php");
echo "i like".$arr[0]."and".$arr[1]."and".$arr[2];
```
2、遍历数组
```
$arr = array('0'->"java",'1'->"c",'2'->"php");
for($i = 0;$i < count($arr);$i++){
	echo "i like".$arr[$i];
}
```

##字符串常用操作

1、确定字符串长度
```
$str = 'Hello PHP Java C# C++';
strlen($str);//21
```
2、定位字符串
```
$str = 'Hello PHP Java C# C++';
echo strpos($str, 'PH');//6
```

3、截取字符串

```
$str = 'Hello PHP Java C# C++';
echo substr($str, 2);//llo PHP Java C# C++
```

4、根据字符长度切割字符串

```
$result = str_split($str, 2);
print_r($result);//Array ( [0] => He [1] => ll [2] => o [3] => PH [4] => P [5] => Ja [6] => va [7] => C [8] => # [9] => C+ [10] => + )
```

5、根据定界符切割字符串

```
$result = explode(' ', $str);
print_r($result);//Array ( [0] => Hello [1] => PHP [2] => Java [3] => C# [4] => C++ )
```
6、连接字符串

```
$num = 123;
$newStr = "$str$num";
//或者
$newStr = $str.$num;
echo $newStr;//Hello PHP Java C# C++123
```

##面向对象
1、对象的引用

```
$h = new Hello();
$h->sayHello();
```
2、通过命名空间创建同名的不同类

```
require_once 'one/Hello.php';
require_once 'two/Hello.php';
$h = new \one\Hello();
$h->sayHello();
$h = new \two\Hello();
$h->sayHello();
```
3、构造方法、静态方法

```
class Man{
	private $_age,$_name;
	
	public function __construct($age,$name){
		$this->_age = $age;
		$this->_name = $name;
	}

	public function getAge(){
		return $this->_age;
	}
	
	public function getName(){
		return $this->_name;
	}

	public static function sayHello(){
		echo 'Hello world';
	}
}
```

4、类的继承

```
public function __construct($age,$name){
	parent::__construct($age,$name,'男');
}
```


##时间和日期

```
date_default_timezone_set("Asia/Shanghai");
echo "当前时间是 " . date("Y-m-d H:i:s");//当前时间是 2017-04-30 21:47:13
```

##处理请求

1、Get请求

```
if(isset($_GET['name'])&&$_GET['name']){
	echo "hello".$_GET['name'];
}else{
	echo "请输入名字";
}	
```

2、Post请求

```
if(isset($_POST['name'])&&$_POST['name']){
	echo "hello".$_POST['name'];
}else{
	echo "请输入名字";
}	
```


##处理Cookie和Session
1、设置cookie

```
setcookie('name','hensen');
```

2、JS中读取cookie

```
<scipt>
	alert(document.cookie);
</scipt>
```
3、设置session

```
session_start();
$_SESSION['name'] = 'hensen';
session_destory();
```

4、读取session

```
session_start();
echo $_SESSION['name'];
```