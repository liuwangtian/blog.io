---
layout: post
title: Oracle11g体系结构
date: 2018-07-25
categories: blog
tags: [数据库,Oracle]
description: 文章金句。
---

oracle数据库=实例(instance)+数据文件(datafiles)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例=内存结构(memory)+进程结构(process)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内存=SGA+PGA<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;进程=后台进程+前台进程(服务器进程)<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例:是连接数据库的一种方式<br/>

连接(connection):通过客户端与数据库服务器连接产生的进程
会话(session):就是通过连接在数据库产生的对话
事务(transcation):能commit或rollback,在会话中所执行的操作

内存=SGA+PGA
SGA:
*共享池,
*高速数据缓冲区,
*重做日志缓冲区,
大池,JAVA池,流池

update t1 set a=10 where b=100; 
校验:语法,对象是否存在,是否有权限操作,锁信息,等等 
校验方式:通过数据字典(由数据库自动运维),或者动态性能视图
硬解析:会生成多种执行方式,然后选取其中消耗性能最小的方式执行,并生成执行计划,存放到sql共享区
软解析:如果数据库之前执行过类似的语句,则直接使用之前语句的执行计划

后台进程结构:
DBWn  数据写进程
LGWR  日志写进程
CKPT  检查点进程
SMON  系统监视进程
PMON  进程监视进程
ARCn  归档进程


参数文件:默认放置在:$ORACLE_HOME/dbs/spfile实例名.ora
	参数文件分为两种形式:
	二进制形式 ,数据库默认使用的形式 ,不可以vi修改,在数据库运行过程中,对部分参数的修改,可以同时记录在此文件中

	文本形式,以文本形式存在,可以vi修改,参数只会在开启数据库时加载,数据库运行过程中修改的参数,无法记录

二进制文件和文本文件可以相互生成
create spfile from pfile;
create pfile from spfile;


密码文件:默认放置在:$ORACLE_HOME/dbs orapw库名.ora
	密码文件只记录具有sysdba权限的用户的密码


预警日志:/u01/app/oracle/diag/rdbms/prod4/PROD4/trace/alert实例名.log
	文件负责记录数据库引擎的运行状态,比如数据库的启停,数据库运行中,改变结构的信息或报错
	从后往前看 找error 找到后看附近的ORA-


数据库启动:
nomount:
开启后台进程并且分配内存 nomount阶段又叫做实例启动阶段
参数文件是本阶段启动所依赖的文件
数据库在启动时 默认寻找放在$ORACLE_HOME/dbs/spfile实例名.ora文件  如果不存在 则寻找 $ORACLE_HOME/dbs/init实例名.ora文件  
要求服务器中$ORACLE_SID与参数文件中的实例名完全一致才可以使用                      

mount:
从nomount到mount阶段转换的关键 在于参数文件中记录的控制文件
通过控制文件记录的日志文件与数据文件的位置 将这些文件挂载起来

open:
利用控制文件记录的日志文件与数据文件进行校验 如果上次关机是非一致性关闭 则在本阶段根据redo文件的记录恢复数据文件 达到一致性后开启 开启后所有数据文件可正常读写




判断一个服务器 安装了多少个数据库/运行了多少个数据库
cat /etc/oratab --安装了多少库
ps -ef |grep smon --正在运行的库


如果数据库中有数据文件使用了ASM形式存放 那么在服务器中 一定会出现一个叫做+ASM的实例
+ASM这个实例的作用是负责管理磁盘和磁盘组的 只能开启到nomount
而数据库实例,比如:PROD4,asmdb,fsdb 是用来存放数据的 需要开启到open


数据库实例依赖ASM实例
在启动时 先开启ASM实例 后开启数据库实例
在关闭时 先关闭数据库实例 后关闭ASM实例


重要视图
V$ASM_CLIENT  这个视图显示了所连接ASM实例的实例信息。
V$ASM_DISKGROUP  这个视图列出了在ASM中创建的磁盘组，还有元数据信息，如磁盘组的空闲空间、分配单元大小和状态。
V$ASM_DISK
V$ASM_DISK_IOSTAT  这个视图列出了V$ASM_DISKGROUP视图中所列每个磁盘的磁盘I/O性能统计信息。
V$ASM_OPERATION  这个视图显示了当前操作，例如在V$ASM_DISKGROUP视图中所列磁盘组上发生的任何再均衡操作。


对于asm磁盘组来说 
AU是磁盘组的属性 而细粒度条带 是针对某些使用磁盘组的数据文件设置的

OLTP:事务型数据库,事务量比较大,事物之间联系性强,单次操作的数据量不大
小条带的优点是数据更分散，有助于分散热点。

OLAP:分析型数据库,事务量很小,不同事务之间联系性差,,单次操作的数据量巨大
大AU、大条带的优点是数据连续存储，显著提高OLAP类操作性能

总结:
对于ASM管理:了解ASM的优点,了解ASM的参数,内存,后台进程
熟练掌握,ASM磁盘组的增删改查,GRID软件的安装,ASM服务的开启
深究,ASM磁盘组的冗余策略,AU与细粒度的关系,OLAP/OLTP型服务器需要的AU大小问题,重平衡的原理


表空间     <-------------> 数据文件
tablespace <-------------> datafile

表空间是存放段的 
表空间与数据文件相对应 一个表空间包含那些个数据文件 一个数据文件属于哪个表空间
segment
段是若干个区的组合
数据段：表
索引段：每个索引都有一个索引段，存储其所有数据。
还原段：系统会为每个数据库实例创建一个UNDO 表空间。
临时段：临时段是SQL 语句需要临时工作区来完成执行时由Oracle DB 创建的。
注：另外还有一些上面未列出的其它类型的段
extent
区是若干个*连续*的数据库块的组合 并且在使用时 一次性分配 最小的区要包含8个数据库块
data block  
数据库块大小通常8k一个 操作系统块(windows/linux/AIX/HP UNIX/SOLARIS)系统块大小通常不一致
oracle数据为了解决在不同系统中系统块差异问题 设定一个数据库块大小 组成数据库块的系统块不必相连
一个数据库块=若干个系统块
若干个系统块印射成一个数据块


创建表空间
create  类型1 tablespace 名字2 datafile 数据文件3 size extent4 segment5 6 7 8
1.smallfile--默认 小文件类型的表空间 可以有多个数据文件 但是每个数据文件最大32G
  bigfile 大文件类型的表空间 只能有1个数据文件 这个数据文件最大32T blob和clob类型文件的 word文档 pdf文档 视频 音频 图片
  undo
  temporary
2.表空间的名字
  名字不区分大小写 
   但是唯一
3.对于数据文件来说:
  存在形式 文件系统,裸设备,ASM
  大小 如果是小文件类型的表空间 单数据文件大小不可超过32G
       可以开启数据文件的自动扩展 达到当前文件最大值时 会根据步长设置进行增长但是不能超过最大值32G
4.对于区有两个概念
  (1)区的管理方式 
  本地管理--默认值
  数据字典管理
  (2)区的分配方式
  自动分配--默认值
  指定分配
5.段的管理方式
  手动管理
  自动管理--默认值 ASSM automatic segment space management
6.压缩选项 
  选择不压缩--默认值
7.日志记录
  记录redo--默认值
  不记录redo
8.本表空间所用数据块大小
  可以设置非8k大小的数据块

区分配 使用指定大小 空间使用率不高 分配次数少 适用于单次操纵数据量大
	   自动分配 空间使用率高 分配次数频繁 比较消耗计算资源 适用于单次操纵数据量小
