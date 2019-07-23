---
title: 生成器yield
date: 2019-07-23 13:45:15
tags: 
- php
- yield
categories: php
---
## 理解

生成器的关键字yield，我个人将其理解为一个临时的内存空间用于存放数据。

比如我们如果要去读取一个文件或者一组数据的里的数据，如果数据只有10条，100条简单的foreach循环存入数组里没什么问题

## 示例传统方式

{% codeblock lang:php %}

function createRange($number){
    $data = [];
    for($i=0;$i<$number;$i++){
        $data[] = time();
    }
    return $data;
}

$result = createRange(10); // 这里调用上面我们创建的函数
foreach($result as $value){
    sleep(1);//这里停顿1秒，我们后续有用
    echo $value.'<br />';
}
{% endcodeblock %}
得到如下输出结果：
{% asset_img 1.png %}
这个时候，10个时间戳数据 先是全部执行完成存入$data数组中，之后在foreach循环中再被依次输出。

## 思考

在调用函数 createRange 的时候给 $number 的传值是10，一个很小的数字。假设，现在传递一个值10000000（1000万）。

那么，在函数 createRange 里面，for循环就需要执行1000万次。且有1000万个值被放到 $data 里面，而$data数组在是被放在内存内。所以，在调用函数时候会占用大量内存。

这里，生成器就可以大显身手了。

## 使用生成器

{% codeblock lang:php %}

## 使用生成器
function createRange($number){
    for($i=0;$i<$number;$i++){
        yield time();
    }
}

$result = createRange(10); // 这里调用上面我们创建的函数
foreach($result as $value){
    sleep(1);
    echo $value.'<br />';
}
{% endcodeblock %}
得到如下输出结果：
{% asset_img 2.png %}

可以看出:

·未使用生成器时： createRange 函数内的 for 循环结果被很快放到 $data 中，并且立即返回。所以， foreach 循环的是一个固定的数组。

·使用生成器时： createRange 的值不是一次性快速生成，而是依赖于 foreach 循环。 foreach 循环一次， for 执行一次。

## 执行过程

 1.首先调用 createRange 函数，传入参数10，但是 for 值执行了一次然后停止了，并且告诉 foreach 第一次循环可以用的值。

 2.foreach 开始对 $result 循环，进来首先 sleep(1) ，然后开始使用 for 给的一个值执行输出。

 3.foreach 准备第二次循环，开始第二次循环之前，它向 for 循环又请求了一次。

 4.for 循环于是又执行了一次，将生成的时间戳告诉 foreach .

 5.foreach 拿到第二个值，并且输出。由于 foreach 中 sleep(1) ，所以， for 循环延迟了1秒生成当前时间

所以，整个代码执行中，始终只有一个记录值参与循环，内存中也只有一条信息。

无论开始传入的 $number 有多大，由于并不会立即生成所有结果集，所以内存始终是一条循环的值。

注意：生成器yield关键字不是返回值，他的专业术语叫产出值，只是生成一个值
那么代码中 foreach 循环的是什么？其实是PHP在使用生成器的时候，会返回一个Generator类的对象。foreach可以对该对象进行迭代，每一次迭代，PHP会通过 Generator 实例计算出下一次需要迭代的值。这样foreach就知道下一次需要迭代的值了。

而且，在运行中for循环执行后，会立即停止。等待foreach下次循环时候再次和for索要下次的值的时候，循环才会再执行一次，然后立即再次停止。直到不满足条件不执行结束。

## 应用

由此可以看出，yield在优化内存占用上有着极大的作用。当我们要去读取 1G、2G.....等大文件时，传统的直接读取方法显然是不可取的。
这时如果使用生成器去实现读取，每次循环只取一行文本，再输出，系统的内存占用量一直只需要那存放一行文本的容量。