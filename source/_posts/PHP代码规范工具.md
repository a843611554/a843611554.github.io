---
title: PHP代码规范工具
date: 2022-05-05 10:31:36
tags:
- php
- phpstorm
categories: php
comments: true
---

## 为什么需要规范

1. 高效编码 - 避免了过多的选择造成的『决策时间』浪费
2. 风格统一 - 最大程度统一了开发团队成员代码书写风格和思路，代码阅读起来如出一辙
3. 减少错误 - 减小初级工程师的犯错几率
   
   ==一个人开发可以随喜好编码，但团队协作的情况下，如果没有规范，整个团队产出的代码可读性极低，代码结构混乱，也为后面的项目代码的维护带来了难度。==
   
   **开发规范一旦统一，就需要团队成员严格遵守**，编码愉悦感提高了，整个项目代码阅读起来更加流畅，工作效率自然也会因此提高，同时代码的健壮性也得到了保障。

## 基础规范

- PSR-2 作为代码风格规范
- phpcs 作为代码检查工具
- phpcbf 作为代码修正工具
- phpstorm 作为开发 IDE
  
  <br/>

## 工具搭建指南

### 设置 phpstorm code style

使用 php 7.3 作为 IDE 版本+

{% asset_img af13ab357ee6b9c567069ba479768e16.png %}

<br/>

设置风格为 psr-2

{% asset_img bd4b4578579bb506a486cb3b566937c2.png %}
<br/>

### 设置 phpcs 和 phpcbf

配置工具

{% asset_img 8a52658ecddb0ea5b65d05ffccd9f274.png %}

在文件选择框里分别选择 phpcs 和 phpcbf ，上面一项选 cs 下面一项选 cbf
文件在 bin 下，因为是软连接的缘故，所以最终指向 php_codesniffer 包，但这个不用管，只要选好 vendor/bin 下的文件就行，选完之后点击 validate 验证一下是否配置成功

配置语法检查

{% asset_img c8a52097e2ef36d8bc7faecb7c2ff102.png %}

注意这里也要设置为 PSR-2 ，如下图

{% asset_img 647df85050a805bbe885eaf7975c8b62.png %}

### 配置 Extends Tools

{% asset_img 2806f7118ad0d8d3164888913d5f7623.png %}

如上，填入相关信息，其中

Arguments 为 

```
--standard=PSR2 $FileDir$/$FileName$
```

Working Directory 为 

```
$ProjectFileDir$
```

<br/>

### 验证配置结果

<br/>

上述内容配置完成后，可以打开一个肯定不规范的文件看看，如：

{% asset_img 1690c63dea12e1d9a570eb23384d4052.png %}

规范要求单行代码不可超过 120 列，而有些文件超过了 120 列，就会报 warning 提示，此时我们就要想办法通过换行、拼接等方式将单行过长的代码给分成多行写，过长的代码是非常影响阅读的。

当我们在编辑器内右键时，可以看到以下内容：

{% asset_img afe7bde13de58f744fefbce5644a37db.png %}

phpcbf 工具会自动为我们修正部分代码，我们可以使用 phpstorm 自带的 format 功能做第一步格式化，再使用 phpcbf 进行规范化修正。

<br/>

### 为 phpcbf 添加快捷键

{% asset_img e62dd635ad65ba3debdde6673c9c7251.png %}

### 配置 git hook -- 看项目情况是否需要强制校验规范

**配置 pre-commit hook 是为了在项目发生 commit 的时候，先进行 phpcs 检测，如果检测出有不符合规范的文件，则需要优化后再提交，否则无法通过校验，无法提交。只有这样才能确保变更内容中都符合规范。**

找到自己项目的 .git 目录，这是个隐藏文件夹（对于 windows 而言）

{% asset_img df88de315cc7fb930b1e378e03ac3565.png %}

如果没有 pre-commit 文件，可以新建，新建后进行编辑，复制以下代码：

```
#!/bin/bash

PROJECT=$(git rev-parse --show-toplevel) 
cd $PROJECT 
SFILES=$(git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\.php) 
TMP_DIR=$PROJECT."/tmp" 

if [ "$#" -ne 0 ] 
then 
    exit 0 
fi 

echo "语法检查："
for FILE in $SFILES 
do
    # 语法检查
    php -l -d display_errors=0 $FILE 
    if [ $? != 0 ] 
    then 
        echo "提交前请先处理上述语法错误" 
        exit 1 
    fi 
    FILES="$FILES $PROJECT/$FILE" 
done 

if [ "$FILES" != "" ] 
then 
    echo "规范检查：" 

    TMP_DIR=/tmp/$(uuidgen) 
    mkdir -p $TMP_DIR 
    for FILE in $SFILES 
    do 
        mkdir -p $TMP_DIR/$(dirname $FILE) 
        git show :$FILE > $TMP_DIR/$FILE 
    done 
    $PROJECT/vendor/bin/phpcs --standard=PSR2 --encoding=utf-8 -n $TMP_DIR 
    PHPCS_ERROR=$? 
    rm -rf $TMP_DIR 
    if [ $PHPCS_ERROR != 0 ] 
    then 
        echo "提交前请先处理上述规范问题" 
        exit 1 
    fi 
fi 

exit $?
```