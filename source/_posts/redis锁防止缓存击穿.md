---
title: redis锁防止缓存击穿
date: 2019-08-27 17:24:27
tags: 
- php
- redis
categories: redis
comments: true
---

## 概念和场景
最近工作上遇到一个缓存上的问题，去统计一个海量数据的时候，我给当前月的加上了生存时间的缓存，但是没有考虑到在缓存数据失效的时候，如果这时候有多个管理员进入系统去查看数据，那么当前月的数据缓存为空，这时候2个请求会同时去查询数据库。本身因为每次查询都要去统计十几个数据表，数十万百万的日志数据，压力已经很大了，这时候同时去查询，很容易造成请求超时，数据库停止请求等问题。这个在某度上的说法叫 缓存击穿。万能的某度告诉我，可以用redis锁去解决。

那么，直接看部分处理源码

## RedisModel.php

```php
<?php
namespace Authority\Model;
class RedisModel{

  protected $handler;

  protected $host;
  protected $port;
  protected $password;
  protected $select;
  protected $timeout;
  protected $prefix;

  public function __construct(){
    
    $this->host     = C("redis.host");
    $this->port     = C("redis.port");
    $this->password = C("redis.password");
    $this->select   = C("redis.select");
    $this->timeout  = C("redis.timeout");
    $this->prefix   = C("redis.prefix");

    try{

      $this->handler = new \Redis;

      $this->handler->connect($this->host, $this->port, $this->timeout);

      $this->password && $this->handler->auth($this->password);
      
      $this->handler->select($this->select);

    }catch(\RedisException $e){

      service_error("系统内部错误",99999);
    }
  }

  public function get($key){

    if(!$key){return false;}

    $realkey = $this->prefix.$key;

    $result  = $this->handler->get($realkey);

    return $result ? json_decode($result,true) : false;
  }

  public function set($key,$val = "",$expire = 0){

    $realkey = $this->prefix.$key;

    $expire  = intval($expire);

    if($expire > 0){

      $result  = $this->handler->set($realkey,json_encode($val),$expire);
    }else{

      $result  = $this->handler->set($realkey,json_encode($val));
    }

    return $result;
  }

  public function clear($key){

    $realkey = $this->prefix.$key;

    $result  = $this->handler->delete($realkey);

    return $result;
  }

  public function setnx($key){
      $realkey = $this->prefix.$key;

      $result  = $this->handler->setnx($realkey,1);

      return $result;
  }

  public function ttl($key){
      $realkey = $this->prefix.$key;

      $result  = $this->handler->ttl($realkey);

      return $result;
  }

  public function expire($key,$expire = 10){
      $realkey = $this->prefix.$key;

      $result  = $this->handler->expire($realkey,$expire);

      return $result;
  }
}
```

以上是封装的类

## 主要逻辑代码

这是我在主要执行逻辑查询的文件中封装的缓存获取和设置的两个方法

```php
//获取缓存数据
    public function getcache($key)
    {
        $cache_key= session("sterminal_cityid") . '_' . MODULE_NAME . "_" . CONTROLLER_NAME  . '_' . $key . '_cache';
        $lock_key = 'lock'.$cache_key;
        $redis = new RedisModel();
//        $redis->clear(md5($cache_key));
//        $redis->clear($lock_key);
        $cache = $redis->get(md5($cache_key));
        if(!$cache){
            //上锁 $locak_key
            $is_lock = $redis->setnx($lock_key);
            if($is_lock){ //如果上锁成功
                //返回false  程序开始查询数据，再存入缓存，解锁
                return false;
            }else{
                if($redis->ttl($lock_key) == -1){
                    $redis->expire($lock_key, 60);//防止死锁,定义锁生存时间为60s判断超时，自动解锁
                    exit('等待解锁');
                }
            }
        }
        return  $cache;
    }
//设置缓存
    public function setcache($key, $data, $time)
    {
        $cache_key = session("sterminal_cityid") . '_' . MODULE_NAME . "_" . CONTROLLER_NAME  . '_' . $key . '_cache';

        $lock_key = 'lock'.$cache_key;

        $redis = new RedisModel();

        $res = $redis->set(md5($cache_key), $data, $time);

        //成功插入缓存后，解锁
        if($res){
            $redis->clear($lock_key);
        }
    }
```

下面是使用的地方

```php
$key = $y_m.'_business';
$data = $this->getcache($key);
if (empty($data)) {
    $data = $this->business_statistics($time_interval, $last_time_intervale, $y_m);
}
if ($i > 0) {//非本月数据查询
    $this->setcache($key, $data);
} else {//本月数据查询
    $this->setcache($key, $data, 3600 * 2); //缓存2小时
}
yield $data;
```

## 执行逻辑流程

查询缓存->存在->直接读取缓存数据返回

查询缓存->不存在->上锁->查数据库->存入缓存->解锁->返回数据

## ps

可能逻辑上还存在纰漏，希望能够有大佬指点完善。