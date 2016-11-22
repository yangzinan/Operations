# MySQL
#### By：大官人
#### Email：DaGuanR@gmail.com
#### QQ：375739049
## 一.	MYSQL介绍
MySQL是一个关系型数据库管理系统，由瑞典MySQL AB 公司开发，目前属于 Oracle 旗下产品。MySQL 最流行的关系型数据库管理系统，在 WEB 应用方面MySQL是最好的 RDBMS (Relational Database Management System，关系数据库管理系统) 应用软件之一。MySQL是一种关联数据库管理系统，关联数据库将数据保存在不同的表中，而不是将所有数据放在一个大仓库内，这样就增加了速度并提高了灵活性。MySQL所使用的 SQL 语言是用于访问数据库的最常用标准化语言。MySQL 软件采用了双授权政策，它分为社区版和商业版，由于其体积小、速度快、总体拥有成本低，尤其是开放源码这一特点，一般中小型网站的开发都选择 MySQL 作为网站数据库。由于其社区版的性能卓越，搭配 PHP 和 Apache 可组成良好的开发环境。本文主要以运维角度讲解mysql的安装和维护知识，不会过多涉及DBA相关内容。
## 二.MYSQL版本选择
截至当前mysql的最新版本是5.7.x。但是在mysql5.5以后oracle公司已经收购了mysql。所以大部分用户担心日后mysql会走上商业化的道路，在版本的选择上都是小于等于5.6.x。本文以mysql5.6.10为例讲解。
## 三.MYSQL的安装
### 3.1安装依赖建立mysql用户
```shell
yum -y install cmake gcc gcc-c++ ncurses-devel
useradd -M -s /sbin/nologin mysql
```
### 3.2安装mysql
#### 3.2.1源码编译安装
```shell
tar zxf mysql-5.6.33.tar.gz
cd mysql-5.6.33
cmake \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \  #mysql的安装目录
-DMYSQL_DATADIR=/data/mysql \      #mysql的数据目录
-DSYSCONFDIR=/etc \                  #默认的配置文件目录
-DEXTRA_CHARSETS=all \
-DDEFAULT_CHARSET=utf8 \           #字符集为utf8
-DDEFAULT_COLLATION=utf8_general_ci \         
-DWITH_INNOBASE_STORAGE_ENGINE=1 \     #编译innodb引擎
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_FEDERATED_STORAGE_ENGINE=1 \
-DWITH_PARTITION_STORAGE_ENGINE=1 \
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \
-DMYSQL_UNIX_ADDR=/var/lib/mysql \   #socket路径
-DMYSQL_TCP_PORT=3306 \      #默认端口
-DWITH_DEBUG=0 \
-DWITH_INNODB_MEMCACHED=1 \
-DWITH_SPHINX_STORAGE_ENGINE=1 \    #安装sphinx引擎
-DENABLED_LOCAL_INFILE=1
make
make install
```
#### 3.2.2二进制安装
```shell
tar zxf mysql-5.6.33-linux-glibc2.5-x86_64.tar.gz
mv mysql-5.6.33-linux-glibc2.5-x86_64 /usr/local/
ln –s /usr/local/mysql-5.6.33-linux-glibc2.5-x86_64 /usr/local/mysql  #创建软连接方便版本的升级
```
### 3.3初始化数据库
```shell
root@template /usr/local 18:41:20 # /usr/local/mysql/scripts/mysql_install_db --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql --datadir=/data/mysql --user=mysql
WARNING: The host 'template' could not be looked up with resolveip.
This probably means that your libc libraries are not 100 % compatible
with this binary MySQL version. The MySQL daemon, mysqld, should work
normally with the exception that host name resolving will not work.
This means that you should use IP addresses instead of hostnames
when specifying MySQL privileges !
Installing MySQL system tables...
160909 18:41:48 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.5.52) starting as process 1347 ...
OK
Filling help tables...
160909 18:41:48 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.5.52) starting as process 1354 ...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/local/mysql/bin/mysqladmin -u root password 'new-password'
/usr/local/mysql/bin/mysqladmin -u root -h template password 'new-password'

Alternatively you can run:
/usr/local/mysql/bin/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr/local/mysql ; /usr/local/mysql/bin/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/local/mysql/mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/
root@template /usr/local 18:41:48 # ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
root@template /usr/local 18:44:21 # ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
root@template /usr/local 18:44:34 # ln -s /usr/local/mysql/bin/mysqld_safe /usr/bin/mysqld_safe
root@template /usr/local 18:44:34 # ln -s /usr/local/mysql/bin/mysqladmin /usr/bin/mysqladmin
root@template /usr/local 18:44:34 # ln -s /usr/local/mysql/bin/mysqldump /usr/bin/mysqldump
#以上我们是使用软连接的方法，我们也可以使用环境变量的方法
echo "PATH=/usr/local/mysq/bin:$PATH" >> /etc/profile
source /etc/profile
```
#### 3.4生成配置文件并启动mysql
#### 3.4.1mysql配置文件
```shell
root@template /usr/local/mysql/support-files 22:14:44 # ll /usr/local/mysql/support-files/     #mysql的配置文件全部在此目录下
total 96
-rwxr-xr-x. 1 7161 31415  1153 Aug 26 23:12 binary-configure
-rw-r--r--. 1 7161 31415  4528 Aug 26 23:12 config.huge.ini
-rw-r--r--. 1 7161 31415  2382 Aug 26 23:12 config.medium.ini
-rw-r--r--. 1 7161 31415  1626 Aug 26 23:12 config.small.ini       
-rw-r--r--. 1 7161 31415   773 Aug 26 19:24 magic
-rw-r--r--. 1 7161 31415  4691 Aug 26 23:12 my-huge.cnf
-rw-r--r--. 1 7161 31415 19759 Aug 26 23:12 my-innodb-heavy-4G.cnf    #生产环境可以使用这个配置文件
-rw-r--r--. 1 7161 31415  4665 Aug 26 23:12 my-large.cnf
-rw-r--r--. 1 7161 31415  4676 Aug 26 23:12 my-medium.cnf
-rw-r--r--. 1 7161 31415  2840 Aug 26 23:12 my-small.cnf       #我们是测试环境直接使用这个配置文件
-rwxr-xr-x. 1 7161 31415   839 Aug 26 23:12 mysql-log-rotate
-rwxr-xr-x. 1 7161 31415 10875 Aug 26 23:12 mysql.server
-rwxr-xr-x. 1 7161 31415  1061 Aug 26 23:12 mysqld_multi.server
-rw-r--r--. 1 7161 31415  1326 Aug 26 23:12 ndb-config-2-node.ini
root@template /usr/local/mysql/support-files 22:18:19 # cp my-small.cnf /etc/my.cnf
cp overwrite `/etc/my.cnf'? y
```
#### 3.4.2添加服务启动mysql
```shell
root@template /usr/local/mysql/support-files 22:19:42 # chkconfig --add mysqld
root@template /usr/local/mysql/support-files 22:22:12 # chkconfig mysqld on
root@template /usr/local/mysql/support-files 22:22:34 # service mysqld start
Starting MySQL... SUCCESS! 
root@template /usr/local/mysql/support-files 22:22:40 # mysql    #直接输入mysql进入mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
## 四.MYSQL的基本操作
### 4.1MySQL的登录
#### 4.1.1设置MySQL的登录密码
* 设置密码
```shell
root@template /usr/local/mysql 16:38:55 # mysqladmin -uroot password '123456'
root@template /usr/local/mysql 16:39:03 # mysql
ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: NO)
#这时再登录MySQL就需要提供密码
root@template /usr/local/mysql 16:39:05 # mysql -uroot -p123456
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 6
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
`当我们设置登录密码和使用密码登录的时候，为确保安全一般不希望显示出密码，所以我们一般采用如下方式`
```shell
root@template /usr/local/mysql 16:42:56 # mysqladmin -uroot password   #使用互交方式
New password: 
Confirm new password:
root@template /usr/local/mysql 16:43:29 # mysql -uroot -p      #使用互交方式
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 11
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
* 修改密码
```shell
root@template /usr/local/mysql 16:46:36 # mysqladmin -uroot -p password
Enter password: 
New password: 
Confirm new password:
```
### 4.2MySQL的用户权限管理
#### 4.2.1用户的优化
* 查看默认的用户和用户的介绍
```shell
mysql> select user,host from mysql.user;   
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | ::1       |
|      | localhost |
| root | localhost |
|      | template  |
| root | template  |
+------+-----------+
6 rows in set (0.00 sec)
#以上为mysql安装后默认创建的用户，mysql的用户是由两部分组成，用户名和主机名。所以显示的root有很多但是后面对应的主机名是不同的。
```
* 删除多余的用户
```shell
mysql> drop user 'root'@'::1';
Query OK, 0 rows affected (0.04 sec)
mysql> drop user ''@'localhost';                 
Query OK, 0 rows affected (0.00 sec)
mysql> drop user ''@'template';         
Query OK, 0 rows affected (0.00 sec)
mysql> drop user 'root'@'template';         
Query OK, 0 rows affected (0.00 sec)

mysql> select user,host from mysql.user;
+------+-----------+
| user | host      |
+------+-----------+
| root | 127.0.0.1 |
| root | localhost |
+------+-----------+
注：这里只保留着两个用户即可
```
#### 4.2.2创建用户
* 创建用的语句和权限
	`grant [grants] on '[database_name]'.'[table_name]' to [user_name]@[hostname] identified by '[password]';`

	`grants是要赋予的权限（权限见下表） database_name 是数据库的名称 user_name是用户名 host_name是主机名 最后是密码`
	
	`权限列表`
```conf
    SELECT                     --查询
	UPDATE                     --更新，修改表内容
	INSERT                     --插入
	DELETE                     --删除表内容
	CREATE                     --创建数据库和表
	DROP                       --删除数据库和表
	REFERENCES                 --外键
	INDEX                      --索引
	ALTER                      --修改表结构
	CREATE TEMPORARY TABLES    --创建临时表
	LOCK TABLES                --锁表
	EXECUTE                    --执行  
	CREATE VIEW                --创建视图
	SHOW VIEW                  --显示视图
	CREATE ROUTINE             --创建存储过程
	ALTER ROUTINE              --修改存储过程
	EVENT                      --事件
	TRIGGER                    --触发器
```
* 创建一个用户
```shell
mysql> grant all on *.* to daguanren@'%' identified by '123456';  #注意这个%代表是任意主机的意思，生产环境不建议使用可以组合使用例如192.168.1.%
Query OK, 0 rows affected (0.02 sec)
mysql> select user,host from mysql.user;
+-----------+-----------+
| user      | host      |
+-----------+-----------+
| daguanren | %         |
| root      | 127.0.0.1 |
| root      | localhost |
+-----------+-----------+
3 rows in set (0.00 sec)
```
* 权限的刷新

	`目前我们创建的用户已经可以直接生效，但是mysql官方建议创建完成用户后要刷新权限表，所以生产环境为了避免意外的出现我们还是执行刷新权限`

```shell
mysql> flush privileges;
Query OK, 0 rows affected (0.01 sec)
```
* 移出用户权限
```shell
mysql> grant insert,update on *.* to znyang@'%' identified by '123456';  #先创建一个具有insert和update权限的用户
Query OK, 0 rows affected (0.00 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql> show grants for znyang@'%';   #查看用户权限
+----------------------------------------------------------------------------------------------------------------+
| Grants for znyang@%                                                                                            |
+----------------------------------------------------------------------------------------------------------------+
| GRANT INSERT, UPDATE ON *.* TO 'znyang'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
+----------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql>
mysql> revoke update on *.* from znyang@'%';  #移除update
Query OK, 0 rows affected (0.00 sec)

mysql>
mysql> show grants for znyang@'%';    #再次查看用户权限
+--------------------------------------------------------------------------------------------------------+
| Grants for znyang@%                                                                                    |
+--------------------------------------------------------------------------------------------------------+
| GRANT INSERT ON *.* TO 'znyang'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' |
+--------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql>
```
#### 4.2.3常用SQL语句
* 查看数据
```shell
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| log                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
5 rows in set (0.04 sec)

mysql>
```
* 切换数据库
```shell
mysql> use test
Database changed
mysql>
```
* 查看数据库里面的表
```shell
mysql> use mysql
Database changed
mysql> show tables;
+---------------------------+
| Tables_in_mysql           |
+---------------------------+
| columns_priv              |
| db                        |
| event                     |
| func                      |
| general_log               |
| help_category             |
| help_keyword              |
| help_relation             |
| help_topic                |
| host                      |
| ndb_binlog_index          |
| plugin                    |
| proc                      |
| procs_priv                |
| proxies_priv              |
| servers                   |
| slow_log                  |
| tables_priv               |
| time_zone                 |
| time_zone_leap_second     |
| time_zone_name            |
| time_zone_transition      |
| time_zone_transition_type |
| user                      |
+---------------------------+
24 rows in set (0.00 sec)

mysql>
```
* 创建数据库和表
```sql
create database [database_name];
create table 表名(id int(4)   #创建表 id 整数型
                  not null   #id不可谓空
                  primary key auto_increment  #id为主键且自动增长 
                  COMMENT '内容' , #注释
                  name char(20)   #name字符串型
                  not null   #不可为空
				  default 值  #定义默认值
                  );
```
```shell
mysql> create database daguanren;
Query OK, 1 row affected (0.01 sec)

mysql> use daguanren
Database changed
mysql> create table users(id int(4) not null primary key auto_increment COMMENT 'id',name char(20) not null default '');  
Query OK, 0 rows affected (0.01 sec)

mysql>
```
* 查看数据库的表结构
```shell
mysql> desc users;
+-------+----------+------+-----+---------+----------------+
| Field | Type     | Null | Key | Default | Extra          |
+-------+----------+------+-----+---------+----------------+
| id    | int(4)   | NO   | PRI | NULL    | auto_increment |
| name  | char(20) | NO   |     |         |                |
+-------+----------+------+-----+---------+----------------+
2 rows in set (0.01 sec)
```
* 插入数据
`insert into [table_name]( field,…) values(value….);`
```shell
mysql> insert into users(name) values('daguanren');
Query OK, 1 row affected (0.01 sec)

mysql> select * from users;    
+----+-----------+
| id | name      |
+----+-----------+
|  1 | daguanren |
+----+-----------+
1 row in set (0.03 sec)
```
* 修改数据
`updata [table_name] set [field]=[value] ……`
```shell
mysql> insert into users(name) values('xiaoguanren');           
Query OK, 1 row affected (0.00 sec)

mysql> select * from users;                          
+----+-------------+
| id | name        |
+----+-------------+
|  1 | daguanren   |
|  2 | xiaoguanren |
+----+-------------+
2 rows in set (0.00 sec)
mysql> update users set name='bigguanren' where id=2;     
Query OK, 1 row affected (0.03 sec)
Rows matched: 1  Changed: 1  Warnings: 0
mysql> select * from users;                               
+----+------------+
| id | name       |
+----+------------+
|  1 | daguanren  |
|  2 | bigguanren |
+----+------------+
2 rows in set (0.00 sec)
```
* 删除数据

`delete from [table_name] …..;`
	
`#...代表条件生产一定要加上不然整个表都会被删除`

```shell	
mysql> select * from users;                               
+----+------------+
| id | name       |
+----+------------+
|  1 | daguanren  |
|  2 | bigguanren |
+----+------------+
2 rows in set (0.00 sec)

mysql> delete from users where id=2;
Query OK, 1 row affected (0.01 sec)

mysql> select * from users;
+----+-----------+
| id | name      |
+----+-----------+
|  1 | daguanren |
+----+-----------+
1 row in set (0.00 sec)
```
* 其他
```sql
#清除表中所有内容
truncate table 表名;
#向表中插入新字段
alter table 表名 add 新字段 字段类型;
#删除字段
alter table 表名 drop 字段名;
#修改表名
alter table 表名 rename to 新表名;
rename table 表名 to 新表名;
#删除表
drop table 表名;
```
## 五.MYSQL的备份
### 5.1.1方法1
`mysqldump –u[user_name] –p[passwoed] [databasename] > [file_name];`
```shell
root@template /usr/local/mysql 21:04:49 # mkdir /backup/mysql -p                         
root@template /usr/local/mysql 21:05:41 # cd /backup/mysql/
root@template /backup/mysql 21:05:46 # mysqldump -uroot -p7758521 daguanren > back1.sql
root@template /backup/mysql 21:06:10 # ll
total 4
-rw-r--r--. 1 root root 1906 Sep 12 21:06 back1.sql
```
### 5.1.2方法2
`mysqldump –u[user_name] –p[passwoed] -B [databasename] > [file_name];`
```shell
root@template /backup/mysql 21:09:02 # mysqldump -uroot -p7758521 -B daguanren > back2.sql 
root@template /backup/mysql 21:09:07 # ll
total 8
-rw-r--r--. 1 root root 1906 Sep 12 21:09 back1.sql
-rw-r--r--. 1 root root 2058 Sep 12 21:09 back2.sql
```
###5.1.3还原数据库和方法1 方法2的比较

```sql
mysql –u[user_name] –p[password] < [file_name]
```
```shell
root@template /backup/mysql 21:10:48 #mysql -uroot -p7758521 < back1.sql; 
ERROR 1046 (3D000) at line 22: No database selected
root@template /backup/mysql 21:11:45 # mysql -uroot -p7758521 < back2.sql;
```
**我们可以看到用方法1备份的文件还原失败了提示没有选在数据库，而用方法2备份的可以成功，我们对比一下俩个备份文件**
```shell
root@template /backup/mysql 21:15:58 # diff back1.sql back2.sql 
18a19,26
> -- Current Database: `daguanren`
> --
> 
> CREATE DATABASE /*!32312 IF NOT EXISTS*/ `daguanren` /*!40100 DEFAULT CHARACTER SET utf8 */;
> 
> USE `daguanren`;                             
> 
> --
51c59
< -- Dump completed on 2016-09-12 21:09:02
---
> -- Dump completed on 2016-09-12 21:09:07
#从这里可以看出用方法2备份的文件里面有创建数据库并切换数据库，而方法1 的没有，所以使用方法1备份的文件必须先要创建数据库才能恢复所以一般生产我们使用方法2备份
```
### 5.2备份还原多个数据库
```sql
mysqldump –u[user_name] –p[password] -B [database_name] [database_name] ... > [file_name]
```
```shell
root@template /backup/mysql 16:29:15 # mysqldump -uroot -p7758521 -B test daguanren > back3.sql
mysql> drop database test;
Query OK, 0 rows affected (0.00 sec)

mysql> drop database daguanren;
Query OK, 1 row affected (0.00 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| log                |
| mysql              |
| performance_schema |
+--------------------+
4 rows in set (0.00 sec)
root@template /backup/mysql 16:31:10 # mysql -uroot -p7758521 < back3.sql 
root@template /backup/mysql 16:31:19 # mysql -uroot -p       
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 36
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| daguanren          |
| log                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)
```
### 5.3数据表的备份还原

```sql
mysqldump -uroot -p7758521 [database_name] [table_name] [table_name] … > [file_name]
```
```shell
mysql> use daguanren
Database changed
mysql> show tables;
+---------------------+
| Tables_in_daguanren |
+---------------------+
| users               |
+---------------------+
1 row in set (0.00 sec)

mysql> create table users2(id int(4) not null primary key auto_increment COMMENT 'id',name char(20) not null default '');
Query OK, 0 rows affected (0.04 sec)	

mysql> show tables;
+---------------------+
| Tables_in_daguanren |
+---------------------+
| users               |
| users2              |
+---------------------+
2 rows in set (0.00 sec)
mysql> \q
Bye
root@template /backup/mysql 16:39:37 # mysqldump -uroot -p7758521 daguanren users users2 > back4.sql
root@template /backup/mysql 16:40:32 # mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 39
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use daguanren
Database changed
mysql> show tables;
+---------------------+
| Tables_in_daguanren |
+---------------------+
| users               |
| users2              |
+---------------------+
2 rows in set (0.00 sec)

mysql> drop table users; 
Query OK, 0 rows affected (0.00 sec)

mysql> drop table users2;
Query OK, 0 rows affected (0.01 sec)

mysql> show tables;      
Empty set (0.00 sec)

mysql>\q
Bye
root@template /backup/mysql 16:43:38 # mysql -uroot -p7758521 daguanren < back4.sql 
root@template /backup/mysql 16:43:44 # mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 42
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use daguanren
Database changed
mysql> show tables;
+---------------------+
| Tables_in_daguanren |
+---------------------+
| users               |
| users2              |
+---------------------+
2 rows in set (0.00 sec)

mysql>
```
### 5.4mysql的全库备份

```sql
mysqldump –u[user_name] –p[password] -A -B > [file_name]
```
```shell
root@template /backup/mysql 16:49:25 # mysqldump -uroot -p7758521 -A -B > back5.sql
-- Warning: Skipping the data of table mysql.event. Specify the --events option explicitly.
root@template /backup/mysql 16:49:50 # mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 45
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| daguanren          |
| log                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)
mysql> drop database daguanren;         
Query OK, 2 rows affected (0.03 sec)
mysql> drop database test; 
Query OK, 0 rows affected (0.00 sec)
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| log                |
| mysql              |
| performance_schema |
+--------------------+
root@template /backup/mysql 16:53:09 # mysql -uroot -p7758521 < back5.sql 
root@template /backup/mysql 16:53:34 # mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 47
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| daguanren          |
| log                |
| mysql              |
| performance_schema |
| test               |
+--------------------+
6 rows in set (0.00 sec)
```
## 六.MYSQL的多实例
### 6.1建立目录
```shell
mkdir -p /data/{3306,3307}/data/log
chown -R mysql.mysql /data
```
## 6.2建立多实例配置文件
```conf
#3306端口配置文件
cat>/etc/my_3306.cnf<<EOF
[client]								
port = 3306							
socket = /data/3306/mysql.socket		
[mysqld]								
server-id = 1							
port = 3306							
socket = /data/3306/mysql.socket		
datadir = /data/3306/data 			
pid-file = /data/3306/mysql.pid 		
EOF
#3307端口配置文件
cat>/etc/my_3307.cnf<<EOF
[client]								
port = 3307							
socket = /data/3307/mysql.socket		
[mysqld]								
server-id = 2							
port = 3307							
socket = /data/3307/mysql.socket		
datadir = /data/3307/data 			
pid-file = /data/3307/mysql.pid 		
EOF
```
### 6.3初始化数据库
```shell
/usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/3306/data --user=mysql
/usr/local/mysql/scripts/mysql_install_db --basedir=/usr/local/mysql --datadir=/data/3307/data --user=mysql
```
### 6.4启动多实例
```shell
/usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my_3306.cnf 2>&1 > /dev/null &
/usr/local/mysql/bin/mysqld_safe --defaults-file=/etc/my_3307.cnf 2>&1 > /dev/null &
```
### 6.5登录多实例
```shell
root@template /data/mysql 18:08:48 # mysql -S /data/3306/mysql.socket                         
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.52 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
#########################################
root@template /data/mysql 18:12:07 # mysql -S /data/3307/mysql.socket 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.52 MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
### 6.6关闭多实例
```shell
root@template /data/mysql 18:14:19 # mysqladmin -uroot -p -S /data/3306/mysql.socket shutdown  
Enter password: 
root@template /data/mysql 18:14:28 # mysqladmin -uroot -p -S /data/3307/mysql.socket shutdown 
Enter password: 
root@template /data/mysql 18:14:35 #
```
## 七.MYSQL的主从同步
### 7.1设置主库的ID和binlog
```conf
server-id = 1
log-bin = /data/mysql/binlog
```
```shell
root@template /data/mysql 19:07:50 # mkdir /data/mysql/binlog
root@template /data/mysql 19:08:19 # chown -R mysql.mysql /data/
```
### 7.2创建用于主从同步的mysql用户
```shell
root@template /data/mysql 19:08:27 # mysql -uroot -p            
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant replication slave on *.* to slave@'%' identified by 'slave';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for slave@'%';
+------------------------------------------------------------------------------------------------------------------+
| Grants for slave@%                                                                                               |
+------------------------------------------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY PASSWORD '*51125B3597BEE0FC43E0BCBFEE002EF8641B44CF' |
+------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
### 7.3锁库锁表备份数据库并记录binlog
```shell
root@template /data/mysql 19:08:27 # mysql -uroot -p            
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 2
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> grant replication slave on *.* to slave@'%' identified by 'slave';
Query OK, 0 rows affected (0.01 sec)

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)

mysql> show grants for slave@'%';
+------------------------------------------------------------------------------------------------------------------+
| Grants for slave@%                                                                                               |
+------------------------------------------------------------------------------------------------------------------+
| GRANT REPLICATION SLAVE ON *.* TO 'slave'@'%' IDENTIFIED BY PASSWORD '*51125B3597BEE0FC43E0BCBFEE002EF8641B44CF' |
+------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql> flush tables with read lock;
Query OK, 0 rows affected (0.00 sec)

mysql> \q
Bye
root@template /data/mysql 19:13:43 # mysqldump -uroot -p7758521 -A -B > all.sql
-- Warning: Skipping the data of table mysql.event. Specify the --events option explicitly.
root@template /data/mysql 19:14:10 # mysql -uroot -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
mysql> show master status;
+---------------+----------+--------------+------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+---------------+----------+--------------+------------------+
| binlog.000001 |      107 |              |                  |
+---------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```
### 7.4从库导入数据
```shell
root@mysql01 ~ 11:22:01 # mysql -uroot -p7758521 < /tmp/all.sql
mysql> select * from users; 
+----+-------------+
| id | name        |
+----+-------------+
|  1 | daguanren   |
|  2 | xiaoguanren |
+----+-------------+
2 rows in set (0.00 sec)

mysql> select * from users2;
+----+--------------+
| id | name         |
+----+--------------+
|  1 | bigguanren   |
|  2 | smallguanren |
+----+--------------+
2 rows in set (0.00 sec)
```
### 7.5从库执行同步
```sql
CHANGE MASTER TO MASTER_HOST='master',MASTER_PORT=3306,MASTER_USER='slave',MASTER_PASSWORD='slave',MASTER_LOG_FILE='misql-bin.000003',MASTER_LOG_POS=318;
mysql> CHANGE MASTER TO MASTER_HOST='192.168.44.11',MASTER_PORT=3306,MASTER_USER='slave',MASTER_PASSWORD='slave',MASTER_LOG_FILE='binlog.000001',MASTER_LOG_POS=107;
Query OK, 0 rows affected, 2 warnings (0.17 sec)

mysql> start slave;
Query OK, 0 rows affected (0.00 sec)

mysql> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.44.11
                  Master_User: slave
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000001
          Read_Master_Log_Pos: 107
               Relay_Log_File: mysql-relay-bin.000002
                Relay_Log_Pos: 263
        Relay_Master_Log_File: binlog.000001
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 107
              Relay_Log_Space: 432
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 
             Master_Info_File: /data/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
1 row in set (0.00 sec)
```
### 7.6验证
#### 7.6.1主库插入数据
```shell
root@template /data 19:26:09 # mysql -uroot -p7758521
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 5
Server version: 5.5.52-log MySQL Community Server (GPL)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> use daguanren
Database changed
mysql> select * from users;       
+----+-------------+
| id | name        |
+----+-------------+
|  1 | daguanren   |
|  2 | xiaoguanren |
+----+-------------+
2 rows in set (0.00 sec)

mysql> insert into users(name) values('yangzinan');              
Query OK, 1 row affected (0.02 sec)

mysql> select * from users;                        
+----+-------------+
| id | name        |
+----+-------------+
|  1 | daguanren   |
|  2 | xiaoguanren |
|  3 | yangzinan   |
+----+-------------+
3 rows in set (0.00 sec)
```
### 7.6.2从库检测
```sql
mysql> use daguanren
Database changed
mysql> select * from users;
+----+-------------+
| id | name        |
+----+-------------+
|  1 | daguanren   |
|  2 | xiaoguanren |
|  3 | yangzinan   |
+----+-------------+
3 rows in set (0.00 sec)
```
## 八.其他优化参数
## 8.1.开启关闭事务自动提交
```conf
SET AUTOCOMMIT=0 #关闭自动提交
SET AUTOCOMMIT=1 #开启自动提交
```
### 8.2.设置数据库只读
```conf
read-only
注：对all权限用户无效
```
### 8.3.其他参数优化
```conf
innodb_buffer_pool_size = 物理内存的1/3或1/2一般不超过50%
max_connections = 100 #最大连接数，此值最大16384
long_query_time = 2 #记录超过两秒的sql语句
long-slow-queries = /data/mysql/slow.log  #记录超过两秒的sql语句
log-error = /data/mysql/error.log  #错误日志
expire_logs_days = 7 #binlog保存时间
skip-name-resolve  #解决连接失败
innodb_data_file_path = ibdata1:10M: ibdata2:10M:autoextend  #InnoDB数据文件初始化大小，本例10M
```




