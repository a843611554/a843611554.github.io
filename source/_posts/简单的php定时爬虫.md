---
title: 简单的php定时爬虫
date: 2019-08-25 11:47:42
tags: 
- php
- curl
categories: php
comments: true
---

首先声明一点：PHP是世界上最好的语言

本文本臭弟弟尝试一次简单用php进行简单的爬虫，把罪恶的双手伸向百度，如有问题，请诸位前辈评论区指正

## 一个简单的开关demo

config.php

```php
<?php
//控制器，1：执行，0：停止

return 1;
```
当然其中还可以去添加很多配置选项，这里就简单的设置一下开关

## 一个简单的爬虫操作

先看一下受害者
{% asset_img 1.png %}
{% asset_img 2.png %}
这样就知道正则需要怎么去处理了

file.php

```php
<?php

header('content-type:text/html;charset=utf-8');

//控制器

$run = include 'config.php';

if(!$run) die('stop');


$url = 'https://tophub.today/n/Jb0vmloB1G';

$filename = 'data/hot'.date('Ymdhis').'.txt';
//使用 curl
//启用curl configure -with-curl

$ch = curl_init();

curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, 0); // 对认证证书来源的检查
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, 2); // 从证书中检查SSL加密算法是否存在
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);//返回数据不输出

$content = curl_exec($ch);
if(!$content){
	echo "error:".curl_error($ch);
}
curl_error($ch);
// echo $content;
preg_match_all('/<td class="al">(.)*<\/td>/', $content, $match);

foreach ($match[0] as $key => $value) {
	preg_match_all('/<a(\s)+href="(.*?)"(.)*target="_blank" rel="nofollow"(.)*>(.+?)<\/a>/iU', $value, $res);
	//把字符串写入文件
	
    $txt =  'href:'.$res[2][0]."  title:".$res[5][0]."\r\n";

    $fp = fopen($filename, 'a');

    fwrite($fp, $txt);
    
}
echo "爬取数据完成，时间为：".date('Y-m-d h:i:s');
fclose($fp);

```

这里只是简单的做一个读取写入文件的操作，实际工作中，应该去捕获异常，和更多的错误处理，这里就简单的写下基本的逻辑

## 运行index.php的结果
{% asset_img 3.png %}
{% asset_img 4.png %}
## ps
实现定时任务的方法有很多
大多数时候会去使用linux下的定时功能去执行
crontab传送门 ：  https://blog.csdn.net/rfyuan/article/details/21022335