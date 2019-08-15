---
title: LNMP源码安装
date: 2019-08-08 09:08:04
tags: 
- LNMP
- Linux
categories: Linux
comments: true
---


## 安装nginx1.8.1

准备工作，安装依赖库

```bash
yum -y install gcc automake autoconf libtool make gcc-c++ glibc libxslt-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel libxml2 libxml2-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel bzip2 bzip2-devel ncurses ncurses-devel curl curl-devel e2fsprogs e2fsprogs-devel krb5-devel libidn libidn-devel openssl openssl-devel pcre pcre-devel libmcrypt libmcrypt-devel wget vim lrzsz
```
1、下载nginx（stable版本）
```bash
wget -P /usr/local/src http://nginx.org/download/nginx-1.8.1.tar.gz
```
-P指定下载文件目录
或者
```bash
cd /usr/local/src
wget http://nginx.org/download/nginx-1.8.1.tar.gz
```
2、解压nginx
```bash
tar xf nginx-1.8.1.tar.gz
cd nginx-1.8.1
```

3、编译nginx
如果指定用户和用户组，需要先创建
```bash
//创建用户www和用户组www

groupadd www

useradd -g www www

./configure --prefix=/usr/local/nginx  --user=www --group=www --with-http_ssl_module --with-http_gzip_static_module

make && make install
```
4、启动nginx
**第一种方式 指定--sbin-path=/usr/sbin/nginx**
```bash	
    nginx //启动

    nginx -s stop// 停止

    nginx -s reload // 重新加载

 

    **第二种方式 不指定--sbin-path**

    cd /usr/local/nginx

    ./sbin/nginx

    重启nginx  /usr/local/nginx/sbin/nginx -s reload

 

    **第三种方式**

     配置开机启动

    首先写一个shell脚本，脚本名称：nginx

    vi /etc/rc.d/init.d/nginx

 

    #! /bin/bash

    # chkconfig: 35 85 15 

    # description: Nginx is an HTTP(S) server, HTTP(S) reverse

    set -e

    PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

    DESC="nginx daemon"

    NAME=nginx

    DAEMON=/usr/local/nginx/sbin/$NAME (这里是nginx安装是 --sbin-path指定的路径)

    SCRIPTNAME=/etc/init.d/$NAME

    test -x $DAEMON || exit 0

    d_start(){

        $DAEMON || echo -n " already running"

    }

    d_stop() {

        $DAEMON -s quit || echo -n " not running"

    }

    d_reload() {

        $DAEMON -s reload || echo -n " counld not reload"

    }

    case "$1" in

    start)

        echo -n "Starting $DESC:$NAME"

        d_start

        echo "."

    ;;

    stop)

        echo -n "Stopping $DESC:$NAME"

        d_stop

        echo "."

    ;;

    reload)

        echo -n "Reloading $DESC configuration..."

        d_reload

        echo "reloaded."

    ;;

    restart)

        echo -n "Restarting $DESC: $NAME"

        d_stop

        sleep 2

        d_start

        echo "."

    ;;

    *)

        echo "Usage: $SCRIPTNAME {start|stop|restart|reload}" >&2

        exit 3

    ;;

    esac

    exit 0

 

    //将shell脚本放入到 /etc/rc.d/init.d/中，并执行下列命令

 

    chmod +x /etc/rc.d/init.d/nginx （设置可执行权限）

    chkconfig --add nginx （添加系统服务）

 

    service nginx start

    service nginx stop

    service nginx restart

    service nginx reload

 

    浏览器访问：http://localhost如能出现nginx页面则表示成功

 

    // 查看nginx进程

    ps -ef | grep nginx

    // 查看进程个数 去掉首位的

    ps -ef | grep nginx | wc -l

 

    // 查看80端口

    netstat -anpt

useradd -M -s /sbin/nologin www

#修改nginx配置文件

vim /usr/local/nginx/conf/nginx.conf

把#user nobody 改成 user www;

#测试一下nginx配置文件

/usr/local/nginx/sbin/nginx -t

#启动nginx

/usr/local/nginx/sbin/nginx

#一般来说在nginx的配置文件修改后进行如下操作，

/usr/local/nginx/sbin/nginx -t检测一下配置文件是否正确，如果正确的话

再使用/usr/local/nginx/sbin/nginx -s reload 使nginx平滑启动

```


## 安装mysql5.6.30

1.下载 MySQL
```bash
cd /usr/local/src

wget http://dev.mysql.com/get/Downloads/MySQL-5.6/mysql-5.6.30.tar.gz
```

2.安装编译源码所需的工具和库
如果缺少cmake库
```bash
yum install cmake

(可以从http://www.cmake.org下载源码并编译安装) （安装包安装cmake）
wget http://www.cmake.org/files/v2.8/cmake-2.8.10.2.tar.gz
tar -xzvf cmake-2.8.10.2.tar.gz
./bootstrap make make
install
```

3.设置MySQL用户和组

新增mysql用户组 新增mysql用户
```bash
groupadd mysql
useradd -r -g mysql mysql
```

4. 新建MySQL所需要的目录
```bash
新建mysql安装目录 新建mysql数据库数据文件目录
mkdir -p /usr/local/mysql

mkdir -p /data/mysqldb
```

5.MySQL源码包解压
```bash
cd /usr/local/src
tar zxvf mysql-5.6.30.tar.gz
cd mysql-5.6.30

cmake -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_MYISAM_STORAGE_ENGINE=1 -DWITH_MEMORY_STORAGE_ENGINE=1 -DWITH_PARTITION_STORAGE_ENGINE=1 -DMYSQL_DATADIR=/data/mysqldb -DMYSQL_TCP_PORT=3306 -DENABLE_DOWNLOADS=1 -DEXTRA_CHARSETS=all -DENABLED_LOCAL_INFILE=1 -DWITH_READLINE=1

(注：重新运行配置，需要删除CMakeCache.txt文件 rm CMakeCache.txt ) make && make install
```

6.修改mysql目录所有者和组
```bash
修改mysql安装目录 cd /usr/local/mysql chown -R mysql:mysql .
修改mysql数据库文件目录 cd /data/mysqldb chown -R mysql:mysql .（注意后面的.指的是当前目录）
```

7.初始化mysql数据库
```bash
cd /usr/local/mysql
./scripts/mysql_install_db --user=mysql --datadir=/data/mysqldb
```

8.复制mysql服务启动配置文件
```bash
cp /usr/local/mysql/support-files/my-default.cnf /etc/my.cnf
（注：如果/etc/my.cnf文件存在，则覆盖。）
```

9.复制mysql服务启动脚本及加入PATH路径
```bash
cp support-files/mysql.server /etc/init.d/mysqld
```

10.启动mysql服务并加入开机自启动
```bash
service mysqld start
chkconfig –level 35 mysqld on
```

11.检查mysql服务是否启动
```bash
netstat -tulnp | grep 3306
需要把mysql加入环境变量 vim /etc/profile
在最后加入如下命令
MYSQL_HOME=/usr/local/mysql
PATH=$MYSQL_HOME/bin:$PATH
export PATH MYSQL_HOME
source /etc/profile
mysql -u root -p （密码为空，如果能登陆上，则安装成功）
```

12.修改MySQL用户root的密码
```bash
方法一：mysql -u root -p
use mysql
update user set password=password(‘123456’) where user=’root’
flush privileges(刷新权限)
方法二：/usr/local/mysql/bin/mysql_secure_installation
（修改MySQL用户root的密码，同时可禁止root远程连接，移除test数据库和匿名用户。）

方法三：mysqladmin -uroot password '123456’
```