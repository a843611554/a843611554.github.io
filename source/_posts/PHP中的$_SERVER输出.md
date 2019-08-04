---
title: PHP中的$_SERVER输出
date: 2019-08-04 18:07:28
tags: 
- php
categories: php
comments: true
---

## $_SERVER全局变量输出
```php
$_SERVER['HTTP_ACCEPT_LANGUAGE']//浏览器语言 
$_SERVER['REMOTE_ADDR'] //当前用户 IP 。
$_SERVER['HTTP_X_FORWARDED_FOR'] //透过代理服务器取得客户端的真实 IP 地址
不过要注意的事，并不是每个代理服务器都能用 $_SERVER["HTTP_X_FORWARDED_FOR"] 来读取客户端的真实IP，有些用此
方法读取到的仍然是代理服务器的 IP。 
$_SERVER['REMOTE_HOST'] //当前用户主机名 
$_SERVER['REQUEST_URI'] //URL
$_SERVER['REMOTE_PORT'] //端口。 
$_SERVER['SERVER_NAME'] //服务器主机的名称。 
$_SERVER['PHP_SELF']//正在执行脚本的文件名 
$_SERVER['argv'] //传递给该脚本的参数。 
$_SERVER['argc'] //传递给程序的命令行参数的个数。 
$_SERVER['GATEWAY_INTERFACE']//CGI 规范的版本。 
$_SERVER['SERVER_SOFTWARE'] //服务器标识的字串 
$_SERVER['SERVER_PROTOCOL'] //请求页面时通信协议的名称和版本 
$_SERVER['REQUEST_METHOD']//访问页面时的请求方法 
$_SERVER['QUERY_STRING'] //查询(query)的字符串。 
$_SERVER['DOCUMENT_ROOT'] //当前运行脚本所在的文档根目录 
$_SERVER['HTTP_ACCEPT'] //当前请求的 Accept: 头部的内容。 
$_SERVER['HTTP_ACCEPT_CHARSET'] //当前请求的 Accept-Charset: 头部的内容。 
$_SERVER['HTTP_ACCEPT_ENCODING'] //当前请求的 Accept-Encoding: 头部的内容 
$_SERVER['HTTP_CONNECTION'] //当前请求的 Connection: 头部的内容。例如：“Keep-Alive”。 
$_SERVER['HTTP_HOST'] //当前请求的 Host: 头部的内容。 
$_SERVER['HTTP_REFERER'] //链接到当前页面的前一页面的 URL 地址。 
$_SERVER['HTTP_USER_AGENT'] //当前请求的 User_Agent: 头部的内容。 
$_SERVER['HTTPS']//如果通过https访问,则被设为一个非空的值(on)，否则返回off 
$_SERVER['SCRIPT_FILENAME'] #当前执行脚本的绝对路径名。 
$_SERVER['SERVER_ADMIN'] #管理员信息 
$_SERVER['SERVER_PORT'] #服务器所使用的端口 
$_SERVER['SERVER_SIGNATURE'] #包含服务器版本和虚拟主机名的字符串。 
$_SERVER['PATH_TRANSLATED'] #当前脚本所在文件系统（不是文档根目录）的基本路径。 
$_SERVER['SCRIPT_NAME'] #包含当前脚本的路径。这在页面需要指向自己时非常有用。 
$_SERVER['PHP_AUTH_USER'] #当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的用户名。 
$_SERVER['PHP_AUTH_PW'] #当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是用户输入的密码。 
$_SERVER['AUTH_TYPE'] #当 PHP 运行在 Apache 模块方式下，并且正在使用 HTTP 认证功能，这个变量便是认证的类型
```

## PHP获取服务器端IP地址的方法

PHP获取服务器IP地址方法一：

```php
if('/'==DIRECTORY_SEPARATOR){
    $server_ip=$_SERVER['SERVER_ADDR'];
}else{
    $server_ip=@gethostbyname($_SERVER['SERVER_NAME']);
}
echo $server_ip;
```

php获取服务器ip地址方法二：

```php
/**
* 获取服务器端IP地址
 * @return string
*/
 
function get_server_ip(){
    if(isset($_SERVER)){
        if($_SERVER['SERVER_ADDR']){
            $server_ip=$_SERVER['SERVER_ADDR'];
        }else{
            $server_ip=$_SERVER['LOCAL_ADDR'];
        }
    }else{
        $server_ip = getenv('SERVER_ADDR');
    }
    return $server_ip;
}
 
echo get_server_ip();
```

但是以上方法在局域网代理模式下，无法获取真实的ip
所以参考 thinkphp封装方法（根据场景和配置的不同 不一定生效）

```php
function get_client_ip($type = 0) {
    $type       =  $type ? 1 : 0;
    static $ip  =   NULL;
    if ($ip !== NULL) return $ip[$type];
    if($_SERVER['HTTP_X_REAL_IP']){//nginx 代理模式下，获取客户端真实IP
        $ip=$_SERVER['HTTP_X_REAL_IP'];     
    }elseif (isset($_SERVER['HTTP_CLIENT_IP'])) {//客户端的ip
        $ip     =   $_SERVER['HTTP_CLIENT_IP'];
    }elseif (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {//浏览当前页面的用户计算机的网关
        $arr    =   explode(',', $_SERVER['HTTP_X_FORWARDED_FOR']);
        $pos    =   array_search('unknown',$arr);
        if(false !== $pos) unset($arr[$pos]);
        $ip     =   trim($arr[0]);
    }elseif (isset($_SERVER['REMOTE_ADDR'])) {
        $ip     =   $_SERVER['REMOTE_ADDR'];//浏览当前页面的用户计算机的ip地址
    }else{
        $ip=$_SERVER['REMOTE_ADDR'];
    }
    // IP地址合法验证
    $long = sprintf("%u",ip2long($ip));
    $ip   = $long ? array($ip, $long) : array('0.0.0.0', 0);
    return $ip[$type];
}
```
