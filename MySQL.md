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

`create database [database_name];
create table 表名(id int(4)   #创建表 id 整数型
                  not null   #id不可谓空
                  primary key auto_increment  #id为主键且自动增长 
                  COMMENT '内容' , #注释
                  name char(20)   #name字符串型
                  not null   #不可为空
				  default 值  #定义默认值
                  );
`
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
* 5.1.2方法2
`mysqldump –u[user_name] –p[passwoed] -B [databasename] > [file_name];`
```shell
root@template /backup/mysql 21:09:02 # mysqldump -uroot -p7758521 -B daguanren > back2.sql 
root@template /backup/mysql 21:09:07 # ll
total 8
-rw-r--r--. 1 root root 1906 Sep 12 21:09 back1.sql
-rw-r--r--. 1 root root 2058 Sep 12 21:09 back2.sql
```





