---
title: GitHub常用命令
date: 2019-07-22 17:11:25
tags: git
categories: git
comments: true
---

## SSH
``` bash
ssh-keygen -t -rsa -C "TaylorApril947939@gmail" //（在github上new SSH，内容为id_rsa.pub）

```
## 分支

``` bash
创建分支：git branch dev
切换分支：git checkout dev
创建+切换分支：git checkout -b dev
查看当前分支：git branch
切换回master分支：git checkout master
合并指定分支到当前分支：git merge dev
(fast-forward 快进模式)
删除分支：git branch -d dev
```

## 上传更新代码
``` bash
git init  //初始化本地仓库
git add -A //添加本地所有文件到仓库
git commit -m "blog源文件" //添加commit
git branch hexo //添加本地仓库分支hexo
git remote add origin <server>  //添加远程仓库 <server> 是指在线仓库的地址 origin是本地分支,remote add操作会将本地仓库映射到云端
git push origin hexo //将本地仓库的源文件推送到远程仓库hexo分支
```

## 拷贝仓库
``` bash
git clone <server> hexo //<server> 是指在线仓库的地址
```