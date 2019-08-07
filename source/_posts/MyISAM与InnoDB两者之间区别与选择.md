---
title: MyISAM与InnoDB两者之间区别与选择
date: 2019-08-07 09:24:49
tags: 
- mysql
- MyISAM
- InnoDB
categories: mysql
comments: true
---

MyISAM与InnoDB两者之间区别与选择，详细总结，性能对比

1、MyISAM：默认表类型，它是基于传统的ISAM类型，ISAM是Indexed Sequential Access Method (有索引的顺序访问方法) 的缩写，它是存储记录和文件的标准方法。不是事务安全的，而且不支持外键，如果执行大量的select，insert MyISAM比较适合。

2、InnoDB：支持事务安全的引擎，支持外键、行锁、事务是他的最大特点。如果有大量的update和insert，建议使用InnoDB，特别是针对多个并发和QPS较高的情况。

## 表锁差异

MyISAM:

myisam只支持表级锁，用户在操作myisam表时，select，update，delete，insert语句都会给表自动加锁，如果加锁以后的表满足insert并发的情况下，可以在表的尾部插入新的数据。也可以通过lock table命令来锁表，这样操作主要是可以模仿事务，但是消耗非常大，一般只在实验演示中使用。

InnoDB ：

Innodb支持事务和行级锁，是innodb的最大特色。

事务的ACID属性：atomicity,consistent,isolation,durable。

并发事务带来的几个问题：更新丢失，脏读，不可重复读，幻读。

事务隔离级别：未提交读(Read uncommitted)，已提交读(Read committed)，可重复读(Repeatable read)，可序列化(Serializable)

查看mysql的默认事务隔离级别“show global variables like ‘tx_isolation’; ”

Innodb的行锁模式有以下几种：共享锁，排他锁，意向共享锁(表锁)，意向排他锁(表锁)，间隙锁。

注意：当语句没有使用索引，innodb不能确定操作的行，这个时候就使用的意向锁，也就是表锁

关于死锁：

什么是死锁？当两个事务都需要获得对方持有的排他锁才能完成事务，这样就导致了循环锁等待，也就是常见的死锁类型。

解决死锁的方法：

1、  数据库参数

2、  应用中尽量约定程序读取表的顺序一样

3、  应用中处理一个表时，尽量对处理的顺序排序

4、  调整事务隔离级别（避免两个事务同时操作一行不存在的数据，容易发生死锁）

## 数据库文件差异

MyISAM ：

myisam属于堆表

myisam在磁盘存储上有三个文件，每个文件名以表名开头，扩展名指出文件类型。

.frm 用于存储表的定义

.MYD 用于存放数据

.MYI 用于存放表索引

myisam表还支持三种不同的存储格式：

静态表(默认，但是注意数据末尾不能有空格，会被去掉)

动态表

压缩表

InnoDB ：

innodb属于索引组织表

innodb有两种存储方式，共享表空间存储和多表空间存储

两种存储方式的表结构和myisam一样，以表名开头，扩展名是.frm。

如果使用共享表空间，那么所有表的数据文件和索引文件都保存在一个表空间里，一个表空间可以有多个文件，通过innodb_data_file_path和innodb_data_home_dir参数设置共享表空间的位置和名字，一般共享表空间的名字叫ibdata1-n。

如果使用多表空间，那么每个表都有一个表空间文件用于存储每个表的数据和索引，文件名以表名开头，以.ibd为扩展名。

## 索引差异

1、关于自动增长

myisam引擎的自动增长列必须是索引，如果是组合索引，自动增长可以不是第一列，他可以根据前面几列进行排序后递增。

innodb引擎的自动增长咧必须是索引，如果是组合索引也必须是组合索引的第一列。

2、关于主键

myisam允许没有任何索引和主键的表存在，

myisam的索引都是保存行的地址。

innodb引擎如果没有设定主键或者非空唯一索引，就会自动生成一个6字节的主键(用户不可见)

innodb的数据是主索引的一部分，附加索引保存的是主索引的值。

3、关于count()函数

myisam保存有表的总行数，如果select count(*) from table;会直接取出出该值

innodb没有保存表的总行数，如果使用select count(*) from table；就会遍历整个表，消耗相当大，但是在加了wehre       条件后，myisam和innodb处理的方式都一样。

4、全文索引

myisam支持 FULLTEXT类型的全文索引

innodb不支持FULLTEXT类型的全文索引，但是innodb可以使用sphinx插件支持全文索引，并且效果更好。（sphinx   是一个开源软件，提供多种语言的API接口，可以优化mysql的各种查询）

5、delete from table

使用这条命令时，innodb不会从新建立表，而是一条一条的删除数据，在innodb上如果要清空保存有大量数据的表，最       好不要使用这个命令。(推荐使用truncate table，不过需要用户有drop此表的权限)

6、索引保存位置

myisam的索引以表名+.MYI文件分别保存。

innodb的索引和数据一起保存在表空间里。

## 开发的注意事项

1、可以用 show create table tablename 命令看表的引擎类型。

2、对不支持事务的表做start/commit操作没有任何效果，在执行commit前已经提交。

3、可以执行以下命令来切换非事务表到事务（数据不会丢失），innodb表比myisam表更安全：alter table tablename type=innodb;或者使用 alter table tablename engine = innodb;

4、默认innodb是开启自动提交的，如果你按照myisam的使用方法来编写代码页不会存在错误，只是性能会很低。如何在编写代码时候提高数据库性能呢？

5、尽量将多个语句绑到一个事务中，进行提交，避免多次提交导致的数据库开销。

6、在一个事务获得排他锁或者意向排他锁以后，如果后面还有需要处理的sql语句，在这两条或者多条sql语句之间程序应尽量少的进行逻辑运算和处理，减少锁的时间。

7、尽量避免死锁

8、sql语句如果有where子句一定要使用索引，尽量避免获取意向排他锁。

9、针对我们自己的数据库环境，日志系统是直插入，不修改的，所以我们使用混合引擎方式，ZION_LOG_DB照旧使用myisam存储引擎，只有ZION_GAME_DB，ZION_LOGIN_DB，DAUM_BILLING使用Innodb引擎。


## 究竟该怎么选择

下面先让我们回答一些问题：   

◆你的数据库有外键吗？   

◆你需要事务支持吗？   

◆你需要全文索引吗？   

◆你经常使用什么样的查询模式？   

◆你的数据有多大？   
  
myisam只有索引缓存   
innodb不分索引文件数据文件 innodb buffer   
myisam只能管理索引，在索引数据大于分配的资源时，会由操作系统来cache；数据文件依赖于操作系统的cache。innodb不管是索引还是数据，都是自己来管理  
  
思考上面这些问题可以让你找到合适的方向，但那并不是绝对的。如果你需要事务处理或是外键，那么InnoDB 可能是比较好的方式。如果你需要全文索引，那么通常来说 MyISAM是好的选择，因为这是系统内建的，然而，我们其实并不会经常地去测试两百万行记录。所以，就算是慢一点，我们可以通过使用Sphinx从InnoDB中获得全文索引。  
  
数据的大小，是一个影响你选择什么样存储引擎的重要因素，大尺寸的数据集趋向于选择InnoDB方式，因为其支持事务处理和故障恢复。数据库的在小决定了故障恢复的时间长短，InnoDB可以利用事务日志进行数据恢复，这会比较快。而MyISAM可能会需要几个小时甚至几天来干这些事，InnoDB只需要几分钟。  
  
操作数据库表的习惯可能也会是一个对性能影响很大的因素。比如： COUNT() 在 MyISAM 表中会非常快，而在InnoDB 表下可能会很痛苦。而主键查询则在InnoDB下会相当相当的快，但需要小心的是如果我们的主键太长了也会导致性能问题。大批的inserts 语句在 MyISAM下会快一些，但是updates 在InnoDB下会更快一些——尤其在并发量大的时候。  
  
所以，到底你检使用哪一个呢？根据经验来看，如果是一些小型的应用或项目，那么MyISAM 也许会更适合。当然，在大型的环境下使用 MyISAM 也会有很大成功的时候，但却不总是这样的。如果你正在计划使用一个超大数据量的项目，而且需要事务处理或外键支持，那么你真的应该直接使用 InnoDB方式。但需要记住InnoDB 的表需要更多的内存和存储，转换100GB 的MyISAM 表到InnoDB 表可能会让你有非常坏的体验。  
  
对于支持事务的InnoDB类型的表，影响速度的主要原因是AUTOCOMMIT默认设置是打开的，而且程序没有显式调用BEGIN 开始事务，导致每插入一条都自动Commit，严重影响了速度。可以在执行sql前调用begin，多条sql形成一个事务（即使autocommit打开也可以），将大大提高性能。  

 

InnoDB

InnoDB 给 MySQL 提供了具有事务(commit)、回滚(rollback)和崩溃修复能力 (crash recovery capabilities)的事务安全(transaction-safe (ACID compliant))型表。 InnoDB 提供了行锁(locking on row level)，提供与 Oracle 类型一致的不加锁读取(non- locking read in SELECTs)。这些特性均提高了多用户并发操作的性能表现。在InnoDB表中不需要扩大锁定 (lock escalation)，因为 InnoDB 的列锁定(row level locks)适宜非常小的空间。 InnoDB 是 MySQL 上第一个提供外键约束(FOREIGN KEY constraints)的表引擎。  

InnoDB 的设计目标是处理大容量数据库系统，它的 CPU 利用率是其它基于磁盘的关系数据库引擎所不能比的。在技术上，InnoDB 是一套放在 MySQL 后台的完整数据库系统，InnoDB 在主内存中建立其专用的缓冲池用于高速缓冲数据和索引。 InnoDB 把数据和索引存放在表空间里，可能包含多个文件，这与其它的不一样，举例来说，在 MyISAM 中，表被存放在单独的文件中。InnoDB 表的大小只受限于操作系统的文件大小，一般为 2 GB。  
InnoDB所有的表都保存在同一个数据文件 ibdata1 中（也可能是多个文件，或者是独立的表空间文件）,相对来说比较不好备份，免费的方案可以是拷贝数据文件、备份 binlog，或者用 mysqldump。  


MyISAM   
MyISAM 是MySQL缺省存贮引擎 .   
每张MyISAM 表被存放在三个文件 。frm 文件存放表格定义。 数据文件是MYD (MYData) 。 索引文件是 MYI (MYIndex) 引伸。   
因为MyISAM相对简单所以在效率上要优于InnoDB..小型应用使用MyISAM是不错的选择.   
MyISAM表是保存成文件的形式,在跨平台的数据转移中使用MyISAM存储会省去不少的麻烦   
  
以下是一些细节和具体实现的差别：   
  
1.InnoDB不支持FULLTEXT类型的索引。   
2.InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行，但是MyISAM只要简单的读出保存好的行数即可。注意的是，当count(*)语句包含 where条件时，两种表的操作是一样的。  
3.对于AUTO_INCREMENT类型的字段，InnoDB中必须包含只有该字段的索引，但是在MyISAM表中，可以和其他字段一起建立联合索引。   
4.DELETE FROM table时，InnoDB不会重新建立表，而是一行一行的删除。   
5.LOAD TABLE FROM MASTER操作对InnoDB是不起作用的，解决方法是首先把InnoDB表改成MyISAM表，导入数据后再改成InnoDB表，但是对于使用的额外的InnoDB特性（例如外键）的表不适用。  

另外，InnoDB表的行锁也不是绝对的，如果在执行一个SQL语句时MySQL不能确定要扫描的范围，InnoDB表同样会锁全表，例如 update table set num=1 where name like “%aaa%”  

任何一种表都不是万能的，只用恰当的针对业务类型来选择合适的表类型，才能最大的发挥MySQL的性能优势。   


## 总结

1、MyISAM不支持事务，InnoDB是事务类型的存储引擎，当我们的表需要用到事务支持的时候，那肯定是不能选择MyISAM了。

2、MyISAM只支持表级锁，BDB支持页级锁和表级锁默认为页级锁，而InnoDB支持行级锁和表级锁默认为行级锁
 
表级锁：直接锁定整张表，在锁定期间，其他进程无法对该表进行写操作，如果设置的是写锁，那么其他进程读也不允许
 
MyISAM是表级锁定的存储引擎，它不会出现死锁问题
 
对于write，表锁定原理如下：
 
如果表上没有锁，在其上面放置一个写锁，否则，把锁定请求放在写锁队列中。
 
对于read，表锁定原理如下 ：
 
如果表上没有写锁定，那么把一个读锁放在其上面，否则把锁请求放在读锁定队列中
 
当一个锁定被释放时，表可被写锁定队列中的线程得到，然后才是读锁定队列中的线程。这意味着，如果你在一个表上有许多更新，那么你的SELECT语句将等到所有的写锁定线程执行完。

行级锁：只对指定的行进行锁定，其他进程还是可以对表中的其他行进行操作的。
 
行级锁是Mysql粒度最小的一种锁，它能大大的减少数据库操作的冲突，但是粒度越小实现成本也越大。
 
行级锁可能会导致“死锁”，那到底是怎么导致的呢，分析原因：Mysql行级锁并不是直接锁记录，而是锁索引。索引分为主键索引和非主键索引两种，如果一条sql语句操作了主键索引，那么Mysql就会锁定这个主键索引，如果sql语句操作的是非主键索引，那么Mysql会先锁定这个非主键索引，再去锁定主键索引。
 
在UPDATE 和 DELETE操作时Mysql不仅会锁定所有WHERE 条件扫描过得索引，还会锁定相邻的键值。
 
“死锁”举例分析：
 
表Test：（ID,STATE,TIME）  主键索引：ID  非主键索引：STATE
 
当执行"UPDATE  STATE =1011 WHERE STATE=1000"  语句的时候会锁定STATE索引，由于STATE 是非主键索引,所以Mysql还会去请求锁定ID索引
 
当另一个SQL语句与语句1几乎同时执行时：“UPDATE STATE=1010 WHERE ID=1”  对于语句2 Mysql会先锁定ID索引，由于语句2操作了STATE字段，所以Mysql还会请求锁定STATE索引。这时。彼此锁定着对方需要的索引，又都在等待对方释放锁定。所以出现了"死锁"的情况。
 
行级锁的优点：
 
有许多线程访问不同的行时，只存在少量的冲突。
 
回滚时只有少量的更改
 
可以长时间锁定单一的行
 
行级锁缺点：
 
相对于页级锁和表级锁来说占用了更多的内存
 
当表的大部分行在使用时，比页级锁和表级锁慢，因为你必须获得更多的锁
 
当在大部分数据上经常使用GROUP BY操作，肯定会比表级锁和页级锁慢。
 
页级锁：表级锁速度快，但是冲突多；行级锁速度慢，但冲突少；页级锁就是他俩折中的，一次锁定相邻的一组记录。

3、MyISAM引擎不支持外键，InnoDB支持外键

4、MyISAM引擎的表在大量高并发的读写下会经常出现表损坏的情况
 
我们以前做的项目就遇到这个问题，表的INSERT 和 UPDATE操作很频繁，原来用的MyISAM引擎，导致表隔三差五就损坏，后来更换成了InnoDB引擎。
 
其他容易导致表损坏原因：
 
服务器突然断电导致数据文件损坏，强制关机（mysqld未关闭情况下）导致表损坏
 
mysqld进程在写入操作的时候被杀掉
 
磁盘故障
 
表损坏常见症状：
 
查询表不能返回数据或返回部分数据
 
打开表失败： Can’t open file: ‘×××.MYI’ (errno: 145) 。
 
Error: Table 'p' is marked as crashed and should be repaired 。

Incorrect key file for table: '...'. Try to repair it
 
Mysql表的恢复：

对于MyISAM表的恢复：

可以使用Mysql自带的myisamchk工具： myisamchk -r tablename  或者 myisamchk -o tablename（比前面的更保险） 对表进行修复

5、对于count()查询来说MyISAM更有优势

因为MyISAM存储了表中的行数记录，执行SELECT COUNT() 的时候可以直接获取到结果，而InnoDB需要扫描全部数据后得到结果。

但是注意一点：对于带有WHERE 条件的 SELECT COUNT()语句两种引擎的表执行过程是一样的，都需要扫描全部数据后得到结果

6、 InnoDB是为处理巨大数据量时的最大性能设计，它的CPU效率可能是任何其它基于磁盘的关系数据库引擎所不能匹敌的。

7、MyISAM支持全文索引（FULLTEXT），InnoDB不支持

8、MyISAM引擎的表的查询、更新、插入的效率要比InnoDB高

网上截取了前辈们测试结论： 

测试方法：连续提交10个query， 表记录总数：38万 ， 时间单位 s
 
        引擎类型                    MyISAM                InnoDB              性能相差
 
        count                      0.0008357            3.0163                3609
 
        查询主键                  0.005708              0.1574                27.57
 
        查询非主键                  24.01                  80.37                3.348
 
        更新主键                  0.008124            0.8183                100.7
 
        更新非主键                0.004141            0.02625              6.338
 
        插入                        0.004188            0.3694                88.21
 

    （1）加了索引以后，对于MyISAM查询可以加快：4 206.09733倍，对InnoDB查询加快510.72921倍，同时对MyISAM更新速度减慢为原来的1/2，InnoDB的更
  新速度减慢为原来的1/30。要看情况决定是否要加索引，比如不查询的log表，不要做任何的索引。
 
    （2）如果你的数据量是百万级别的，并且没有任何的事务处理，那么用MyISAM是性能最好的选择。
 
    （3）InnoDB表的大小更加的大，用MyISAM可省很多的硬盘空间。
 
        在我们测试的这个38w的表中，表占用空间的情况如下：
            引擎类型                    MyISAM              InnoDB
            数据                      53,924 KB          58,976 KB
            索引                      13,640 KB          21,072 KB
            占用总空间              67,564 KB          80,048 KB
  
        另外一个176W万记录的表， 表占用空间的情况如下：
 
            引擎类型                MyIsam              InnorDB
            数据                  56,166 KB          90,736 KB
            索引                  67,103 KB          88,848 KB
            占用总空间        123,269 KB        179,584 KB
 
