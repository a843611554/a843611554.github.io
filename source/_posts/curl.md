---
title: curl
date: 2019-07-23 09:16:40
tags: 
- php
- curl
categories: php
---

### 使用CURL发送请求的基本流程

使用CURL的PHP扩展完成一个HTTP请求的发送一般有以下几个步骤：

1.初始化连接句柄；
2.设置CURL选项；
3.执行并获取结果；
4.释放VURL连接句柄。

下面的程序片段是使用CURL发送HTTP的典型过程
{% codeblock lang:php %}
// 1. 初始化
 $ch = curl_init();
 // 2. 设置选项，包括URL
 curl_setopt($ch,CURLOPT_URL,"http://www.devdo.net");
 curl_setopt($ch,CURLOPT_RETURNTRANSFER,1);
 curl_setopt($ch,CURLOPT_HEADER,0);
 // 3. 执行并获取HTML文档内容
 $output = curl_exec($ch);
 if($output === FALSE ){
 echo "CURL Error:".curl_error($ch);
 }
 // 4. 释放curl句柄
 curl_close($ch);
{% endcodeblock %}
上述代码中使用到了四个函数

curl_init() 和 curl_close() 分别是初始化CURL连接和关闭CURL连接，都比较简单。

		1.curl_exec() 执行CURL请求，如果没有错误发生，该函数的返回是对应URL返回的数据，以字符串表示满意；如果发生错误，该函数返回 

		2.FALSE。需要注意的是，判断输出是否为FALSE用的是全等号，这是为了区分返回空串和出错的情况。

		3.CURL函数库里最重要的函数是curl_setopt(),它可以通过设定CURL函数库定义的选项来定制HTTP请求。上述代码片段中使用了三个重要的选项：
			·CURLOPT_URL 指定请求的URL；
			·CURLOPT_RETURNTRANSFER 设置为1表示稍后执行的curl_exec函数的返回是URL的返回字符串，而不是把返回字符串定向到标准输出并返回TRUE；
			·CURLLOPT_HEADER设置为0表示不返回HTTP头部信息。



{% link CURL的选项还有很多，可以到PHP的官方网站上查看CURL支持的所有选项列表。（点击前往） http://www.php.net/manual/en/function.curl-setopt.php  %}


## 获取CURL请求的输出信息

在curl_exec()函数执行之后，可以使用curl_getinfo()函数获取CURL请求输出的相关信息，示例代码如下：

{% codeblock lang:php %}
curl_exec($ch);
$info = curl_getinfo($sh);
echo ' 获取 '.$info['url'].'耗时'.$info['total_time'].'秒';
{% endcodeblock %}

上述代码中curl_getinfo返回的是一个关联数组，包含以下数据：

	url:网络地址。
	content_type:内容编码。
	http_code:HTTP状态码。
	header_size:header的大小。
	request_size:请求的大小。
	filetime:文件创建的时间。
	ssl_verify_result:SSL验证结果。
	redirect_count:跳转计数。
	total_time:总耗时。
	namelookup_time:DNS查询耗时。
	connect_time:等待连接耗时。
	pretransfer_time:传输前准备耗时。
	size_uplpad:上传数据的大小。
	size_download:下载数据的大小。
	speed_download:下载速度。
	speed_upload:上传速度。
	download_content_length:下载内容的长度。
	upload_content_length:上传内容的长度。
	starttransfer_time:开始传输的时间表。
	redirect_time:重定向耗时。

curl_getinfo()函数还有一个可选择参数$opt,通过这个参数可以设置一些常量，对应到上术这个字段，如果设置了第二个参数，那么返回的只有指定的信息。例如设置$opt为CURLINFO_TOTAL_TIME，则curl_getinfo()函数只返回total_time,即总传输消耗的时间，在只需要关注某些传输信息时，设置$opt参数很有意义。

## curl封装方法

{% codeblock lang:php %}
 /*
     * Curl模拟HTTP请求
     * @param string $url 请求目标地址url
     * @param string $cookie 请求时附带的cookie
     * @param string $cookie_name 保存返回信息中cookie的变量名
     * @param string|array $data 请求时POST的数据
     * @param string|array $header 自定义请求Header
     * @return 响应结果主体内容
     */
    public static function curl($url, $cookie = '', $cookie_name = '', $data='', $header = '',$type='',$refer = ''){
        if($type!=1){
            if(is_array($data)){
                $data = http_build_query($data);
            }
        }
        $curl = curl_init(); // 启动一个CURL会话
        curl_setopt($curl, CURLOPT_SAFE_UPLOAD, false);
        curl_setopt($curl, CURLOPT_URL, $url); // 要访问的地址
        curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, false); // 对认证证书来源的检查
        curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, false); // 从证书中检查SSL加密算法是否存在
        curl_setopt($curl, CURLOPT_USERAGENT, $_SERVER['HTTP_USER_AGENT']); // 模拟用户使用的浏览器
        curl_setopt($curl,CURLOPT_COOKIESESSION,true);
        if($cookie){
            curl_setopt($curl, CURLOPT_COOKIE, $cookie);
        }
        if($data){
            curl_setopt($curl, CURLOPT_POST, 1); // 发送一个常规的Post请求
            curl_setopt($curl, CURLOPT_POSTFIELDS, $data); // Post提交的数据包
        }else{
            curl_setopt($curl, CURLOPT_HTTPGET, 1 ); // 发送一个常规的GET请求
        }
        curl_setopt($curl, CURLOPT_FOLLOWLOCATION, 1); // 使用自动跳转
        curl_setopt($curl, CURLOPT_AUTOREFERER, 1); // 自动设置Referer
        curl_setopt($curl, CURLOPT_TIMEOUT, 30); // 设置超时限制防止死循环
        curl_setopt($curl, CURLOPT_HEADER, 1); // 显示返回的Header区域内容
        curl_setopt($curl, CURLOPT_RETURNTRANSFER, 1); // 获取的信息以文件流的形式返回
        if($header){
            // curl_setopt($curl, CURLINFO_CONTENT_TYPE, $content_type); // 获取的信息以文件流的形式返回
            if(is_array($header)){
                foreach ($header as $key => $val){
                    $headers[] = $key.": " . $val; // browsers keep this blank.
                }
            }else {
                $headers[] = "Content-Type: " . $header; // browsers keep this blank.
            }
            curl_setopt($curl, CURLOPT_HTTPHEADER, $headers);
        }

        $tmp_info = curl_exec($curl); // 执行操作

        $headerSize = curl_getinfo($curl, CURLINFO_HEADER_SIZE);
        // 根据头大小去获取头信息内容
        $header = substr($tmp_info, 0, $headerSize);

        $content = substr($tmp_info, $headerSize);

        if($cookie_name){
            preg_match_all('/Set-Cookie:(.*);/imU',$header,$cookie_arr); //正则匹配
            $cookie_new = $cookie_arr[1]; //获得COOKIE（SESSIONID）
            if($cookie_new){
                $cookie_arr1 = ToolsModel::cookie_match($cookie_new);
                $cookie_arr2 = [];
                if($_SESSION[$cookie_name]){
                    $cookie_old_arr = explode('; ', $_SESSION[$cookie_name]);
                    $cookie_arr2 = ToolsModel::cookie_match($cookie_old_arr);
                }
                $cookie_arr_new = array_filter(array_merge($cookie_arr2, $cookie_arr1));
                if(is_array($cookie_arr_new)){
                    $cookie_new_str = '';
                    foreach ($cookie_arr_new as $key => $val){
                        if($cookie_new_str) $cookie_new_str .= '; ';
                        $cookie_new_str .= $key.'='.$val;
                    }
                    $_SESSION[$cookie_name] = $cookie_new_str;
                }
            }
        }
        if (curl_errno($curl)) {
            echo 'Errno'.curl_error($curl);die;//捕抓异常
            IS_AJAX ? exit(json_encode(["code" => 400,"msg" => "请求错误！"])) : header("location:/index.php");
        }
        curl_close($curl); // 关闭CURL会话
        return $content; // 返回数据
    }

{% endcodeblock %}