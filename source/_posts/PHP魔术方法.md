---
title: PHP魔术方法
date: 2019-07-30 10:51:31
tags: 
- php
- 魔术方法
categories: php
comments: true
---

# PHP的十六个魔术方法

PHP中把以两个下划线__开头的方法称为魔术方法(Magic methods)，这些方法在PHP中充当了举足轻重的作用。

## __construct()，类的构造函数

php中构造方法是对象创建完成后第一个被对象自动调用的方法。在每个类中都有一个构造方法，如果没有显示地声明它，那么类中都会默认存在一个没有参数且内容为空的构造方法。

### 构造方法的作用

通常构造方法被用来执行一些有用的初始化任务，如对成员属性在创建对象时赋予初始值。

### 1.构造方法的在类中的声明格式

{% codeblock lang:php %}
function __constrct([参数列表]){

    方法体 //通常用来对成员属性进行初始化赋值

}
{% endcodeblock %}
### 2.在类中声明构造方法需要注意的事项

	1、在同一个类中只能声明一个构造方法，原因是，PHP不支持构造函数重载。

	2、构造方法名称是以两个下画线开始的__construct()。

下面是它的例子：	
{% codeblock lang:php %}
<?php
    class Person
    {  
        public $name;        
        public $age;        
        public $sex;    

        /**
         * 显示声明一个构造方法且带参数
         */                                                                                      
        public function __construct($name="", $sex="男", $age=22)
        {      
            $this->name = $name;
            $this->sex = $sex;
            $this->age = $age;
        }

        /**
         * say 方法
         */
        public function say()
        { 
            echo "我叫：" . $this->name . "，性别：" . $this->sex . "，年龄：" . $this->age;
        }   
                                                                                          

    }
{% endcodeblock %}

创建对象$Person1且不带任参数
{% codeblock lang:php %}
$Person1 = new Person();
echo $Person1->say(); //输出:我叫：，性别：男，年龄：27
{% endcodeblock %}
创建对象$Person2且带参数“小明”
{% codeblock lang:php %}
$Person2 = new Person("小明");
echo $Person2->say(); //输出：我叫：张三，性别：男，年龄：27
{% endcodeblock %}
创建对象$Person3且带三个参数
{% codeblock lang:php %}
$Person3 = new Person("李四","男",25);
echo $Person3->say(); //输出：我叫：李四，性别：男，年龄：25
{% endcodeblock %}

## __destruct()，类的析构函数
析构方法允许在销毁一个类之前执行的一些操作或完成一些功能，比如说关闭文件、释放结果集等。

析构方法是PHP5才引进的新内容。

析造方法的声明格式与构造方法 __construct() 比较类似，也是以两个下划线开始的方法 __destruct() ，这种析构方法名称也是固定的。

### 1.析构方法的声明格式
{% codeblock lang:php %}
function __destruct()
{
 //方法体
}
{% endcodeblock %}
注意：析构函数不能带有任何参数。

### 2.析构方法的作用
一般来说，析构方法在PHP中并不是很常用，它属类中可选择的一部分，通常用来完成一些在对象销毁前的清理任务。
举例演示，如下：
{% codeblock lang:php %}
<?php

class Person{

    public $name;         
    public $age;         
    public $sex;         

    public function __construct($name="", $sex="男", $age=22)
    {   
        $this->name = $name;
        $this->sex  = $sex;
        $this->age  = $age;
    }

         /**
     * say 说话方法
     */
    public function say()
    {  
        echo "我叫：".$this->name."，性别：".$this->sex."，年龄：".$this->age;
    }    
    /**
     * 声明一个析构方法
     */
    public function __destruct()
    {
        echo "我觉得我还可以再抢救一下，我的名字叫".$this->name;
    }
}

$Person = new Person("小明");
unset($Person); //销毁上面创建的对象$Person
{% endcodeblock %}

上面的程序运行时输出：
{% codeblock lang:php %}
我觉得我还可以再抢救一下，我的名字叫小明
{% endcodeblock %}


## __call()，在对象中调用一个不可访问方法时调用

该方法有两个参数，第一个参数 $function_name 会自动接收不存在的方法名，第二个 $arguments 则以数组的方式接收不存在方法的多个参数。

### 1. __call() 方法的格式：
{% codeblock lang:php %}
function __call(string $function_name, array $arguments)
{
    // 方法体
}
{% endcodeblock %}
### 2.__call() 方法的作用：
为了避免当调用的方法不存在时产生错误，而意外的导致程序中止，可以使用 __call() 方法来避免。

该方法在调用的方法不存在时会自动调用，程序仍会继续执行下去。

请参考如下代码：

{% codeblock lang:php %}
<?php
class Person{                             
    
    function say()
    {                           
         echo "Hello, world!<br>"; 
    }      

    /**
     * 声明此方法用来处理调用对象中不存在的方法
     */
    function __call($funName, $arguments)
    { 
          echo "你所调用的函数：" . $funName . "(参数：" ;  // 输出调用不存在的方法名
          print_r($arguments); // 输出调用不存在的方法时的参数列表
          echo ")不存在！<br>\n"; // 结束换行                      
    }                                          

}
$Person = new Person();            
$Person->run("teacher"); // 调用对象中不存在的方法，则自动调用了对象中的__call()方法
$Person->eat("小明", "苹果");             
$Person->say();
{% endcodeblock %}
运行结果：
{% codeblock lang:php %}
你所调用的函数：run(参数：Array ( [0] => teacher ) )不存在！
你所调用的函数：eat(参数：Array ( [0] => 小明 [1] => 苹果 ) )不存在！
Hello, world!
{% endcodeblock %}

## __callStatic()，用静态方式中调用一个不可访问方法时调用
此方法与上面所说的 __call() 功能除了 __callStatic() 是未静态方法准备的之外，其它都是一样的。
## __get()，获得一个类的成员变量时调用
在 php 面向对象编程中，类的成员属性被设定为 private 后，如果我们试图在外面调用它则会出现“不能访问某个私有属性”的错误。那么为了解决这个问题，我们可以使用魔术方法 __get()。

### 1.魔术方法__get()的作用

在程序运行过程中，通过它可以在对象的外部获取私有成员属性的值。
我们通过下面的 __get() 的实例来更进一步的连接它吧：

{% codeblock lang:php %}
<?php
class Person
{
    private $name;
    private $age;
    function __construct($name="", $age=1)
    {
        $this->name = $name;
        $this->age = $age;
    }

    /**
     * 在类中添加__get()方法，在直接获取属性值时自动调用一次，以属性名作为参数传入并处理
     * @param $propertyName
     *
     * @return int
     */
    public function __get($propertyName)
    {   
        if ($propertyName == "age") {
            if ($this->age > 30) {
                return $this->age - 10;
            } else {
                return $this->$propertyName;
            }
        } else {
            return $this->$propertyName;
        }
    }
}
$Person = new Person("小明", 60);   // 通过Person类实例化的对象，并通过构造方法为属性赋初值
echo "姓名：" . $Person->name . "<br>";   // 直接访问私有属性name，自动调用了__get()方法可以间接获取
echo "年龄：" . $Person->age . "<br>";    // 自动调用了__get()方法，根据对象本身的情况会返回不同的值
{% endcodeblock %}
运行结果：
{% codeblock lang:php %}
姓名：小明
年龄：50
{% endcodeblock %}

## __set()，设置一个类的成员变量时调用

### 1.__set() 的作用：
__set( $property, $value )` 方法用来设置私有属性， 给一个未定义的属性赋值时，此方法会被触发，传递的参数是被设置的属性名和值。

请看下面的演示代码：

{% codeblock lang:php %}
<?php

class Person
{

    private $name;

    private $age;

    public function __construct($name="",  $age=25)
    {
        $this->name = $name;
        $this->age  = $age;
    }

    /**
     * 声明魔术方法需要两个参数，真接为私有属性赋值时自动调用，并可以屏蔽一些非法赋值
     * @param $property
     * @param $value
     */
    public function __set($property, $value) {
        if ($property=="age")
        {
            if ($value > 150 || $value < 0) {
                return;
            }
        }
        $this->$property = $value;
    }

    /**
     * 在类中声明说话的方法，将所有的私有属性说出
     */
    public function say(){
        echo "我叫".$this->name."，今年".$this->age."岁了";
    }

}
$Person=new Person("小明", 25); //注意，初始值将被下面所改变
//自动调用了__set()函数，将属性名name传给第一个参数，将属性值”李四”传给第二个参数
$Person->name = "小红";     //赋值成功。如果没有__set()，则出错。
//自动调用了__set()函数，将属性名age传给第一个参数，将属性值26传给第二个参数
$Person->age = 16; //赋值成功
$Person->age = 160; //160是一个非法值，赋值失效
$Person->say();  //输出：我叫小红，今年16岁了
{% endcodeblock %}
运行结果：
{% codeblock lang:php %}
我叫小红，今年16岁了
{% endcodeblock %}


## __isset()，当对不可访问属性调用isset()或empty()时调用
在看这个方法之前我们看一下isset()函数的应用，isset()是测定变量是否设定用的函数，传入一个变量作为参数，如果传入的变量存在则传回true，否则传回false。

那么如果在一个对象外面使用isset()这个函数去测定对象里面的成员是否被设定可不可以用它呢？

分两种情况，如果对象里面成员是公有的，我们就可以使用这个函数来测定成员属性，如果是私有的成员属性，这个函数就不起作用了，原因就是因为私有的被封装了，在外部不可见。那么我们就不可以在对象的外部使用isset()函数来测定私有成员属性是否被设定了呢？当然是可以的，但不是一成不变。

你只要在类里面加上一个__isset()方法就可以了，当在类外部使用isset()函数来测定对象里面的私有成员是否被设定时，就会自动调用类里面的__isset()方法了帮我们完成这样的操作。

### __isset()的作用：

当对不可访问属性调用 isset() 或 empty() 时，__isset() 会被调用。
请看下面代码演示：
{% codeblock lang:php %}
<?php
class Person
{
    public $sex;
    private $name;
    private $age;

    public function __construct($name="",  $age=25, $sex='男')
    {
        $this->name = $name;

        $this->age  = $age;

        $this->sex  = $sex;
    }

    /**
     * @param $content
     *
     * @return bool
     */
    public function __isset($content) {
        echo "当在类外部使用isset()函数测定私有成员{$content}时，自动调用<br>";
        echo  isset($this->$content);
    }

$person = new Person("小明", 25); // 初始赋值
echo isset($person->sex),"<br>";
echo isset($person->name),"<br>";
echo isset($person->age),"<br>";
{% endcodeblock %}

运行结果如下：
{% codeblock lang:php %}
1 // public 可以 isset()
当在类外部使用isset()函数测定私有成员name时，自动调用 // __isset() 内 第一个echo
1 // __isset() 内第二个echo
当在类外部使用isset()函数测定私有成员age时，自动调用 // __isset() 内 第一个echo
1 // __isset() 内第二个echo
{% endcodeblock %}

## __unset()，当对不可访问属性调用unset()时被调用。

用法同__isset()。

## __sleep()，执行serialize()时，先会调用这个函数

serialize() 函数会检查类中是否存在一个魔术方法 __sleep()。如果存在，则该方法会优先被调用，然后才执行序列化操作。

此功能可以用于清理对象，并返回一个包含对象中所有应被序列化的变量名称的数组。

如果该方法未返回任何内容，则 NULL 被序列化，并产生一个 E_NOTICE 级别的错误。

注意：

__sleep() 不能返回父类的私有成员的名字。这样做会产生一个 E_NOTICE 级别的错误。可以用 Serializable 接口来替代。

作用：

__sleep() 方法常用于提交未提交的数据，或类似的清理操作。同时，如果有一些很大的对象，但不需要全部保存，这个功能就很好用。

具体请参考如下代码：
{% codeblock lang:php %}
<?php
class Person
{
    public $sex;
    public $name;
    public $age;
    public function __construct($name="",  $age=25, $sex='男')
    {
        $this->name = $name;
        $this->age  = $age;
        $this->sex  = $sex;
    }
    /**
     * @return array
     */
    public function __sleep() {
        echo "当在类外部使用serialize()时会调用这里的__sleep()方法<br>";
        $this->name = base64_encode($this->name);
        return array('name', 'age'); // 这里必须返回一个数值，里边的元素表示返回的属性名称
    }

}
$person = new Person('小明'); // 初始赋值
echo serialize($person);
echo '<br/>';
{% endcodeblock %}
代码运行结果：
{% codeblock lang:php %}
当在类外部使用serialize()时会调用这里的__sleep()方法
O:6:"Person":2:{s:4:"name";s:8:"5bCP5piO";s:3:"age";i:25;}
{% endcodeblock %}
## __wakeup()，执行unserialize()时，先会调用这个函数
如果说 __sleep() 是白的，那么 __wakeup() 就是黑的了。
因为：

与之相反，`unserialize()` 会检查是否存在一个 `__wakeup()` 方法。如果存在，则会先调用 `__wakeup` 方法，预先准备对象需要的资源。

作用：

__wakeup() 经常用在反序列化操作中，例如重新建立数据库连接，或执行其它初始化操作。

还是看代码：
{% codeblock lang:php %}
<?php
class Person
{
    public $sex;
    public $name;
    public $age;
    public function __construct($name="",  $age=25, $sex='男')
    {
        $this->name = $name;
        $this->age  = $age;
        $this->sex  = $sex;
    }

    /**
     * @return array
     */
    public function __sleep() {
        echo "当在类外部使用serialize()时会调用这里的__sleep()方法<br>";
        $this->name = base64_encode($this->name);
        return array('name', 'age'); // 这里必须返回一个数值，里边的元素表示返回的属性名称
    }

    /**
     * __wakeup
     */
    public function __wakeup() {
        echo "当在类外部使用unserialize()时会调用这里的__wakeup()方法<br>";
        $this->name = 2;
        $this->sex = '男';
        // 这里不需要返回数组
    }
}
$person = new Person('小明'); // 初始赋值
var_dump(serialize($person));
var_dump(unserialize(serialize($person)));
{% endcodeblock %}
运行结果：
{% codeblock lang:php %}
当在类外部使用serialize()时会调用这里的__sleep()方法
string(58) "O:6:"Person":2:{s:4:"name";s:8:"5bCP5piO";s:3:"age";i:25;}" 当在类外部使用serialize()时会调用这里的__sleep()方法
当在类外部使用unserialize()时会调用这里的__wakeup()方法
object(Person)#2 (3) { ["sex"]=> string(3) "男" ["name"]=> int(2) ["age"]=> int(25) }
{% endcodeblock %}
## __toString()，类被当成字符串时的回应方法
作用：

__toString() 方法用于一个类被当成字符串时应怎样回应。例如 `echo $obj;` 应该显示些什么。

注意：

此方法必须返回一个字符串，否则将发出一条 `E_RECOVERABLE_ERROR` 级别的致命错误。

警告：

不能在 __toString() 方法中抛出异常。这么做会导致致命错误。

代码：
{% codeblock lang:php %}
<?php
class Person
{
    public $sex;
    public $name;
    public $age;
    public function __construct($name="",  $age=25, $sex='男')
    {
        $this->name = $name;
        $this->age  = $age;
        $this->sex  = $sex;
    }
    public function __toString()
    {
        return  'go go go';
    }
}
$person = new Person('小明'); // 初始赋值
echo $person;

{% endcodeblock %}

结果：
{% codeblock lang:php %}
go go go
{% endcodeblock %}

那么如果类中没有 __toString() 这个魔术方法运行
{% codeblock lang:php %}
Catchable fatal error: Object of class Person could not be converted to string in D:\phpStudy\WWW\test\index.php on line 18
很明显，页面报了一个致命错误，这是语法所不允许的。
{% endcodeblock %}

## __invoke()，调用函数的方式调用一个对象时的回应方法
作用：

当尝试以调用函数的方式调用一个对象时，__invoke() 方法会被自动调用。

注意：

本特性只在 PHP 5.3.0 及以上版本有效。

直接上代码：
{% codeblock lang:php %}
<?php
class Person
{
    public $sex;
    public $name;
    public $age;
    public function __construct($name="",  $age=25, $sex='男')
    {
        $this->name = $name;
        $this->age  = $age;
        $this->sex  = $sex;
    }

    public function __invoke() {
        echo '这可是一个对象哦';
    }
}
$person = new Person('小明'); // 初始赋值
$person();
{% endcodeblock %}
查看运行结果：
{% codeblock lang:php %}
这可是一个对象哦
{% endcodeblock %}
当然，如果你执意要将对象当函数方法使用，那么会得到下面结果：
{% codeblock lang:php %}
Fatal error: Function name must be a string in D:\phpStudy\WWW\test\index.php on line 18
{% endcodeblock %}
## __set_state()，调用var_export()导出类时，此静态方法会被调用。

## __clone()，当对象复制完成时调用

## __autoload()，尝试加载未定义的类

## __debugInfo()，打印所需调试信息
