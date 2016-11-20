# 高性能WEB服务器apache
## 一.APACHE简介
Apache是世界使用排名第一的Web服务器软件。它可以运行在几乎所有广泛使用的计算机平台上，由于其跨平台和安全性被广泛使用，是最流行的Web服务器端软件之一。它快速、可靠并且可通过简单的API扩充，将`Perl/Python`等解释器编译到服务器中。
## 二.APACHE的特点
1. 支持最新的HTTP/1.1通信协议
2. `拥有简单而强有力的基于文件的配置过程`
3. 支持通用网关接口
4. `支持基于IP和基于域名的虚拟主机`
5. 支持多种方式的HTTP认证
6. `集成Perl处理模块`
7. 集成代理服务器模块
8. 支持实时监视服务器状态和定制服务器日志
9. 支持服务器端包含指令(SSI)
10. 支持安全Socket层(SSL)
11. 提供用户会话过程的跟踪
12. 支持FastCGI
13. 通过第三方模块可以支持JavaServlet

`注：黄色背景部分是运维人员需要着重掌握部分，也是本文以下讲解的侧重点。其他作为了解`
## 三.APACHE版本介绍
 目前apache有两个并行的版本2.2.x和2.4.x。当前生产环境（国内）使用最多的是2.2.x版本，因此本文以下的内容都已2.2.x版本作为讲解的主要内容
## 四.APACHE的安装
### 4.1在CentOS下可以使用yum命令直接安装apache，命令如下
```shell
    yum install -y httpd
```
`注：一般生产环境不建议使用yum安装apache软件，主要因为yum安装不可以选择apache的版本,和定制化安装。`

### 4.2编译安装apache
#### 4.2.1首先安装apache的依赖apr
```shell
tar -zxf apr-1.4.5.tar.gz
cd apr-1.4.5
./configure --prefix=/usr/local/apr
make && make install
cd ..
```
#### 4.2.2安装apr-util
```shell
tar -zxf apr-util-1.3.12.tar.gz
cd apr-util-1.3.12
./configure --prefix=/usr/local/apr-util -with-apr=/usr/local/apr/bin/apr-1-config
make && make install
cd ..
```
#### 4.2.3安装pcre
```shell
unzip -o pcre-8.10.zip
cd pcre-8.10
./configure --prefix=/usr/local/pcre 
make && make install
cd ..
```
#### 4.2.4安装其他依赖
```shell
yum install –y zlib zlib-devel openssl openssl-devel
```
#### 4.2.5编译安装apache
```shell
tar zxf httpd-2.2.31.tar.gz
cd httpd-2.2.31
./configure \
--prefix=/usr/local/apach-2.2.31 \
--with-apxs2=/usr/local/apache-2.2.31/bin/apxs \
--enable-deflate \
--enable-expires \
--enable-headers \
--enable-modules=most \
--enable-so \
--with-ssl \
--enable-ssl \
--enable-cgid \
--enable-cgi \
--with-apr-util=/usr/local/apr-util \
--with-mpm=worker \
--enable-rewrite \
--with-pcre=/usr/local/pcre \
--with-included-apr
make && make install
```
#### 4.2.6启动apache并通过命令验证
```shell
/usr/local/apache/bin/apachectl –t  #检查apache配置文件的语法
```

##### 如果出现以下报警（并不影响使用）

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/apache/01.png?raw=true)
##### 解决办法：
> * 打开配置文件（httpd.conf）在安装目录下的conf目录下（本例在/usr/local/apache/conf下）
> * 修改ServerName打开注释并将ServerName改为localhost

##### 启动apache
```shell
/usr/local/apache/bin/apachectl start
```
##### 验证80端口是否启动
```shell
root@template /usr/local/apache/conf 16:53:10 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1140/sshd           
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1216/master         
tcp        0      0 :::80                       :::*                        LISTEN      110052/httpd   
```
##### 验证apache进程是否启动
```shell
root@template /usr/local/apache/conf 16:54:47 # ps -ef | grep httpd | grep -v grep
root     110052      1  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110053 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110054 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110055 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110056 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
```
##### 通过端口反查进程
```shell
root@template /usr/local/apache/conf 16:56:28 # lsof -i:80
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   110052   root    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110054 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110055 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110056 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
```
##### 客户端使用浏览器检查

![image] (https://github.com/yangzinan/Operations/blob/master/iamge/apache/02.png?raw=true)
## 五.APACHE配置详解
### 5.1apache主配置文件重要参数
```conf
ServerRoot "/usr/local/ apach-2.2.31"  #Apache的安装目录
    Listen 80  #监听80端口（可以写成多个端口--多行。ip:端口的形式）
    User daemon  #Apache的默认用户名（建议修改）
    Group daemon  #Apache的默认用户组（建议修改）
    ServerAdmin you@example.com  #系统管理员邮箱 
    DocumentRoot "/usr/local/ apach-2.2.31/htdocs"  #Apache默认web站点的路径末尾不要添加/
    <Directory />  #禁止访问文件系统所在的目录，并添加你希望允许访问的模块目录
       Options FollowSymLinks  #表示允许使用符号连接
       AllowOverride None  #表示用户禁止对目录配置文件（.htaccess）重载
       Order deny,allow  #一deny方式优先处理，没有明确说明拒绝的话都通过
       Deny from all  #明确指出拒绝所有访问
   </Directory>
   <Directory "/usr/local/apache2.4.10/htdocs">  
#设置/usr/local/ apach-2.2.31/htdocs目录块权限
         Options Indexes FollowSymLinks 
 #表示禁止符号连接，Indexes表示允许目录浏览应删除（修改后Options
          #FollowSymLinks）
         AllowOverride None   #表示用户禁止对目录配置文件（.htaccess）重载
         Order allow,deny  #一deny方式优先处理，没有明确说明拒绝的话都通过
         Allow from all  #允许所有访问 此处是web服务必须开启
    </Directory>
    <IfModule dir_module>
         DirectoryIndex index.html  #Apache默认的主页配置多个以空格隔开
    </IfModule>
    <FilesMatch "^\.ht">  #防止.htaccess和htpasswd等隐藏文件被web用户看到
         Order allow,deny
         Deny from all
         Satisfy All
    </FilesMatch>
    ErrorLog "logs/error_log"  #错误日志
    <IfModule log_config_module>
         LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined 
 #访问日志格式
         LogFormat "%h %l %u %t \"%r\" %>s %b" common  
#普通访问日志格式
        <IfModule logio_module>
LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\" %I %O" combinedio
        </IfModule>
             CustomLog "logs/access_log" common  #默认站点访问日志配置
    </IfModule>
    <IfModule alias_module>
         ScriptAlias /cgi-bin/ "/usr/local/ apach-2.2.31/cgi-bin/"  #配置cgi别名
    </IfModule>
    <IfModule cgid_module>
    </IfModule>
    <Directory "/usr/local/ apach-2.2.31/cgi-bin">  #配置cgi访问路径
         AllowOverride None
         Options None
         Order allow,deny
         Allow from all
    </Directory>
          DefaultType text/plain 
 #定义当前不能确定MIME时服务器提供的默认MIME类型，如果你的服务只要包括test或html文档
          #text/plain是一个好的选择
    <IfModule mime_module>
         TypesConfig conf/mime.types
         AddType application/x-compress .Z
         AddType application/x-gzip .gz .tgz
    </IfModule>
    <IfModule ssl_module>
```
### 5.2配置apache的虚拟主机
#### 5.2.1基于主机名的虚拟主机
##### 1.首先要使用apache的虚拟主机功能必须引入虚拟主机的配置文件。在主配置文件解除注释如下：
```shell
sed	-i 's@#Includeconf/extra/httpd-vhosts.conf@Includeconf/extra/httpd-vhosts.conf@g' /usr/local/apache/conf/httpd.conf
注：可以使用sed命令在非交互模式下快速修改，但是后面的文件路径需要读者根据自己的路径修改
```
##### 2.3)	修改虚拟主机的配置文件（httpd-vhosts.conf位于conf目录下的extra目录里面）
```conf
#
# Virtual Hosts
#
# If you want to maintain multiple domains/hostnames on your
# machine you can setup VirtualHost containers for them. Most configurations
# use only name-based virtual hosts so the server doesn't need to worry about
# IP addresses. This is indicated by the asterisks in the directives below.
#
# Please see the documentation at 
# <URL:http://httpd.apache.org/docs/2.2/vhosts/>
# for further details before you try to setup virtual hosts.
#
# You may use the command line option '-S' to verify your virtual host
# configuration.

#
# Use name-based virtual hosting.
#
NameVirtualHost *:80

#
# VirtualHost example:
# Almost any Apache directive may go into a VirtualHost container.
# The first VirtualHost section is used for all requests that do not
# match a ServerName or ServerAlias in any <VirtualHost> block.
#
<VirtualHost *:80>
    ServerAdmin DaguanR@gmail.com   #管理员的邮箱
    DocumentRoot "/usr/local/daguanren"   #当前虚拟主机的目录
    ServerName www.daguanren.com      #当前虚拟主机的域名
    ServerAlias www.dummy-host.example.com   #当前虚拟主机的别名
    ErrorLog "logs/daguanren.com-error_log"  #当前虚拟主机的错误日志
    CustomLog "logs/daguanren-access_log" common   #当前虚拟主机的访问日志
</VirtualHost>
```
##### 4.检测语法重启apache
```shell
root@template /usr/local/apache/conf/extra 17:34:23 # /usr/local/apache/bin/apachectl -t      
Syntax OK
root@template /usr/local/apache/conf/extra 17:34:27 # /usr/local/apache/bin/apachectl graceful
注：这里使用graceful而不是用stop再start，因为graceful是优雅的重启只是重新加载配置文件而不会影响正在访问的用户
```
##### 5.创建一个index.html文件方便测试
```shell
cd /usr/local/daguanren
echo "Hello World" > index.html
```
##### 6.修改windows主机的hosts文件进行测试
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/apache/03.png?raw=true)

`出现403权限拒绝解决办法：`
`打开主配置文件添加目录的权限`
```conf
<Directory "/usr/local/daguanren "> #设置目录块权限（虚拟主机的根目录及index.html的文件目录）
    Options Indexes FollowSymLinks 
#表示禁止符号连接，Indexes表示允许目录浏览应删除（修改后Options#FollowSymLinks）
    AllowOverride None
#表示用户禁止对目录配置文件（.htaccess）重载 
    Order allow,deny
#一deny方式优先处理，没有明确说明拒绝的话都通过 
    Allow from all   #apache2.4 版本为Require all granted
#允许所有访问 此处是web服务必须开启
</Directory>
```
`可以使用如下命令快速执行`
```shell
cat>>/usr/local/apache/conf/httpd.conf<<EOF
<Directory "/usr/local/daguanren"> 
    Options Indexes FollowSymLinks 
    AllowOverride None
    Order allow,deny
    Allow from all  
</Directory>
EOF
```
`再次检测语法并重启`
```shell
root@template /usr/local/daguanren 17:46:05 # /usr/local/apache/bin/apachectl -t
Syntax OK
root@template /usr/local/daguanren 17:46:09 # /usr/local/apache/bin/apachectl graceful
```
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/04.png?raw=true)
访问成功
#### 5.2.2基于端口的虚拟主机
##### 1.修改主配置文件添加一个监听端口（httpd.conf）
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/05.png?raw=true)
##### 2.修改主配置文件添加虚拟主机host
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/06.png?raw=true)
##### 3.修改httpd-vhosts.conf添加一个基于端口的虚拟主机
```shell
cat>>/usr/local/apache/conf/extra/httpd-vhosts.con<<EOF
<VirtualHost *:8080>
    	ServerAdmin admin@your-domain.com
    	DocumentRoot "/usr/local/port"
    	ServerName www.daguanren.com
    	ServerAlias www.dummy-host.example.com
    	ErrorLog "logs/port.com-error_log"
    	CustomLog "logs/port-access_log" common
</VirtualHost>
EOF
```
##### 4.在主配置文件添加目录权限并写一个测试文件
```shell
cat>>/usr/local/apache/conf/httpd.conf<<EOF
<Directory "/usr/local/port"> 
    	Options Indexes FollowSymLinks 
    	AllowOverride None
   	Order allow,deny
    	Allow from all  
</Directory>
EOF
echo "port:8080" > /usr/local/port/index.html
```
##### 5.检测语法重启apache
```shell
root@template /usr/local/apache/conf 18:59:06 # /usr/local/apache/bin/apachectl -t           
Syntax OK
root@template /usr/local/apache/conf 18:59:15 # /usr/local/apache/bin/apachectl graceful
```
##### 6.查看端口
```shell
root@template /usr/local/apache/conf 19:07:48 # netstat -ntlup 
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1140/sshd           
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1216/master         
tcp        0      0 :::8080                     :::*                        LISTEN      110052/httpd        8080端口已经开始监听
tcp        0      0 :::80                       :::*                        LISTEN      110052/httpd        
tcp        0      0 :::22                       :::*                        LISTEN      1140/sshd           
tcp        0      0 ::1:25                      :::*                        LISTEN      1216/master
```
##### 7.通过浏览器查看
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/07.png?raw=true)
