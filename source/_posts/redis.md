---
title: redis
date: 2019-07-29 15:24:40
tags: 
- php
- redis
categories: php
comments: true
---

# Redis

Redis 是完全开源免费的，遵守BSD协议，是一个高性能的key-value数据库。

Redis 与其他 key - value 缓存产品有以下三个特点：

	1.Redis支持数据的持久化，可以将内存中的数据保存在磁盘中，重启的时候可以再次加载进行使用。
	2.Redis不仅仅支持简单的key-value类型的数据，同时还提供list，set，zset，hash等数据结构的存储。
	3.Redis支持数据的备份，即master-slave模式的数据备份。

## 安装

### Windows

	下载地址：https://github.com/MSOpenTech/redis/releases。

	Redis 支持 32 位和 64 位。这个需要根据你系统平台的实际情况选择，这里我们下载 Redis-x64-xxx.zip压缩包到 C 盘，解压后，将文件夹重新命名为 redis。	

	打开一个 cmd 窗口 使用 cd 命令切换目录到 C:\redis 运行：

	``` bash
	redis-server.exe redis.windows.conf
	```
	服务开启成功
	{% asset_img 1.png %}

	这时候另启一个 cmd 窗口，原来的不要关闭，不然就无法访问服务端了。

	切换到 redis 目录下运行:
	``` bash
	redis-cli.exe -h 127.0.0.1 -p 6379
	```
### Linux

	下载地址：http://redis.io/download，下载最新稳定版本。

	本教程使用的最新文档版本为 2.8.17，下载并安装：
	``` bash
	$ wget http://download.redis.io/releases/redis-2.8.17.tar.gz
	$ tar xzf redis-2.8.17.tar.gz
	$ cd redis-2.8.17
	$ make
	```
	make完后 redis-2.8.17目录下会出现编译后的redis服务程序redis-server,还有用于测试的客户端程序redis-cli,两个程序位于安装目录 src 目录下：
	下面启动redis服务.
	``` bash
	$ cd src
	$ ./redis-server
	```
	注意这种方式启动redis 使用的是默认配置。也可以通过启动参数告诉redis使用指定配置文件使用下面命令启动。
	``` bash
	$ cd src
	$ ./redis-server ../redis.conf
	```

	redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。
	启动redis服务进程后，就可以使用测试客户端程序redis-cli和redis服务交互了。 比如：
	``` bash
	$ cd src
	$ ./redis-cli
	redis> set foo bar
	OK
	redis> get foo
	"bar"
	```
## 配置

### 查看配置
	Redis CONFIG 命令格式如下：	
	``` bash
	redis 127.0.0.1:6379> CONFIG GET CONFIG_SETTING_NAME    （配置字段用 * 可查询全部配置）
	```
### 编辑配置	
	CONFIG SET 命令基本语法：
	``` bash
	redis 127.0.0.1:6379> CONFIG SET loglevel "notice"
	OK
	redis 127.0.0.1:6379> CONFIG GET loglevel

	1) "loglevel"
	2) "notice"
	```
## 使用消息队列处理高并发demo

分别创建两个php文件  index.php  demo.php

index.php(模拟高并发请求)
{% codeblock lang:php %}

$redis = new Redis();
$redis->connect('127.0.0.1',6379);
$redis->auth('59989301');

for($i=0;$i<50;$i++){//设置50条模拟
    try{
        $request_id =rand(1000,5000);
        $data=[
            'request_id' => $request_id,
            'event' => '这是一个事件'.$i
        ];
        $redis->lPush('list',json_encode($data,true));
        echo '请求'.$request_id."已进入队列处理<br>";
    }catch(Exception $e){
        echo $e->getMessage();
    }
}

{% endcodeblock %}	
输出如下：
{% asset_img 2.png %}

demo.php(模拟处理请求)
{% codeblock lang:php %}

$redis = new Redis();
$redis->pconnect('127.0.0.1',6379);
$redis->auth('59989301');
while(true){
    try{
        $data = $redis->lPop('list');
        if(!$data){
            break;
        }
        $data=json_decode($data,true);
        /*
         *  利用$data进行逻辑和数据处理
         */
        echo $data['request_id'].'请求已被处理'.'<br>';
    }catch(Exception $e){
        echo $e->getMessage();
    }
}
{% endcodeblock %}	
输出如下：
{% asset_img 3.png %}