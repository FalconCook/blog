## mysql简介

概述

![](images/01.png)

## 高级Mysql

- 完整的mysql优化需要很深的功底，大公司甚至有专门的DBA写上述
    
    - mysql内核
    
    - sql优化工程师
    
    - mysql服务器的优化

    - 各种参数常量设定
    
    - 查询语句优化
    
    - 主从复制
    
    - 软硬件升级
    
    - 容灾备份
    
    - sql编程

## mysqlLinux版的安装

- mysql5.5

    - 下载地址：https://dev.mysql.com/downloads/mysql/5.5.html#downloads

    - 检查当前系统是否安装过mysql：

        - 查询命令：rpm -qa|grep -i mysql

        - 删除命令：rpm -e RPM软件包名称

            - 删除自带的mysql：yum -y remove mysql-libs-5.1.73-7.el6.x86_64

- 安装mysql服务端（注意提示）：

    rpm -ivh MySQL-server-5.5.48-1.linux2.6.i386.rpm

        - 如果报错libc.so.6：https://blog.csdn.net/xiyuliuyang/article/details/90750049

        - 如果警告key ID 5072e1f5: NOKEY：https://blog.csdn.net/Aaron960214/article/details/78451321

- 安装mysql客户端

    - rpm -ivh MySQL-client-5.5.48-1.linux2.6.i386.rpm

- 查看MySQL安装时创建的mysql用户和mysql组

    - cat /etc/passwd|grep mysql

        - cat /etc/group|grep mysql

        - mysqladmin --version

- mysql服务的启+停

    - service mysql start

    - service mysql start

        - 如果报错ERROR! The server quit without updating PID file (/var/lib/mysql/localhost.localdomain.pid).

        - 解决办法：https://www.cnblogs.com/bingco/p/8068243.html

            ```
            1 mysql_install_db --datadir=/var/lib/mysql
            2 chown mysql:mysql /var/lib/mysql -R
            ```

    - 查看mysql的进程：ps -ef|grep mysql

- mysql服务启动后，开始连接

    - 首次连接成功：mysql（不需要输入密码）

        - 给root用户设置密码：/usr/bin/mysqladmin -u root password 123456

- 自启动mysql服务

    - 设置开机自启动mysql：chkconfig mysql on

        - 查看mysql的等级：chkconfig --list | grep mysql

        - 查看不同等级代表的含义：cat /etc/inittab

        - 查看开机自动服务有哪些：ntsysv

- 修改配置文件位置

    - 版本5.5：cp /usr/share/mysql/my-huge.cnf /etc/my.cnf

    - 版本5.6：cp /usr/share/mysql/my-default.cnf /etc/my.cnf

- 修改字符集和数据存储路径

    - 查看字符集

        - show variables like ‘character%’;

        - show variables like ‘%char%’;

        - 由于默认的是客户端和服务器都使用的latin1，所以都是乱码

        - 修改
        ![](images/02.png)

        - 重启mysql

        - 重新连接后，原来的库由于建立于修改字符集之前，所以中文依然是乱码，而新建表中文不是乱码

- MySQL的安装位置

    - /var/lib/mysql：mysql数据库文件的存放路径

    - /usr/share/mysql：配置文件目录

    - /usr/bin：相关命令目录

    - /etc/init.d/mysql：启停相关脚本

## mysql配置文件

- 主要配置文件

    - 二进制日志log-bin

        - 主从复制

    - 错误日志log-error
        
        - 默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等。
    
    - 查询日志log

        - 默认关闭，记录查询的sql语句，如果开启会降低mysql的整体性能，因为记录日志也是需要消耗系统资源的。
    
    - 数据文件

        - 两系统
            
            - windows：D:\devSoft\MySQLServer5.5\data目录下可以挑选很多库
            
            - linux
                看看当前系统中的全部库后再进去
                默认路径：/var/lib/mysql

        - frm文件：存放表结构
        - myd文件：存放表数据
        - myi文件：存放表索引

    - 如何配置
        - windows：my.ini文件
        - Linux：/etc/my.cnf文件

## mysql逻辑架构介绍

- 和其它数据库相比，MySQL有点与众不同，它的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的架构上，**插件式的存储引擎架构将查询处理和其它的系统任务以及数据的存储提取相分离。**这种架构可以根据业务的需求和时机需要选择合适的存储引擎。

![](images/03.png)

- 从上到下，连接层，服务层，引擎层，存储层

![](images/04.png)

## mysql存储引擎

- 查看命令

    - 如何用命令查看
        - 看你的mysql现在已提供什么存储引擎：show engines;
        - 看你的mysql当前默认的存储引擎：show variables like ‘%storage_engine%’;

- MyISAM和InnoDB

![](images/05.png)

- 阿里巴巴、淘宝用哪个

![](images/06.png)

## 参考资料

MySQL_基础+高级篇
https://www.bilibili.com/video/BV12b411K7Zu