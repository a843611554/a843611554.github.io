---
title: php获取一个类里面的所有方法
date: 2019-07-24 10:12:54
tags: 
- php
- Reflection
categories: php
comments: true
---

## 场景
因为后台开发服务配置需要，我需要获取目前已有的服务列表。
如下图
{% asset_img 1.png %}
因为当时快速开发需要，对于这个服务列表我临时全部写在了一个固定的数组里返回。显然这是不利于后期开发维护的。
这时候就需要想办法获取一个类里的方法，属性等信息。

## get_class_methods

{% codeblock lang:php %}
var_dump(get_class_methods('\Authority\Service\Alipay\Authorize'));exit;
{% endcodeblock %}

输出如下

{% asset_img 2.png %}

但是这个答案显然不是我想要的，它同时输出了\Authority\Service\Alipay\Authorize 继承 BaseController 类的方法。
但是php给我们提供一个ReflectionClass反射类，可以解决这些问题。

## ReflectionClass

{% codeblock lang:php %}
$service1= new \ReflectionClass('\Authority\Service\Alipay\Authorize');
var_dump($service1->getMethods());exit;
{% endcodeblock %}

输出如下

{% asset_img 3.png %}

这个时候我们可以得到方法名称的同时，还可以获取到这个方法来自于哪个类，接下来只需要一个简单的foreach循环就可以拿到需要的方法名称！

## 关于ReflectionClass反射类的其他用法

基本可以获取一个类任何你所需要的信息！
其他功能可见手册：{% link 点击前往 https://php.golaravel.com/class.reflectionclass.html  %}



