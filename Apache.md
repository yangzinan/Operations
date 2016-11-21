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
## 六.APACHE模块
### 6.1mod_expires模块（用于客户端浏览器允许使用缓存网页的图片等）
#### 6.1.1检查模块是否安装
```shell
root@template /usr/local/apache/conf 19:07:55 # /usr/local/apache/bin/apachectl -M | grep expires
 expires_module (static)
Syntax OK
#以上是显示静态安装结果
root@template /usr/local/apache/conf 19:07:55 # /usr/local/apache/bin/apachectl -M | grep expires
 expires_module (shared)
Syntax OK
#以上是动态安装结果
```
#### 6.1.2手动安装
```shell
/usr/local/apache/bin/apxs -c -i -a mod_expires.c
注：mod_expires.c文件在apache安装包的modules/metadata/下
注：如果我们是编译安装时已经编译进去的需要在住配置文件（httpd.conf）中解除注释如下行
LoadModule expires_module modules/mod_expires.so
sed -i 's#\#LoadModule mod_expires modules/mod_expires.so#LoadModule mod_expires modules/mod_expires.so#g' /usr/local/apache/conf/httpd.conf
```
#### 6.1.3配置缓存（写在主配置文件结尾即可)
```conf
ExpiresActive on 
#开启缓存
ExpiresDefault "access plus 12 month"
ExpiresByType text/html "access plus 12 months"
#压缩的文件为text和html文件 缓存时间为12个月
ExpiresByType text/css "access plus 12 months"
ExpiresByType image/gif "access plus 12 months"
ExpiresByType image/jpeg "access plus12  12 months"
ExpiresByType image/jpg "access plus 12 months"
ExpiresByType image/png "access plus 12 months"
EXpiresByType application/x-shockwave-flash "access plus 12 months"
EXpiresByType application/x-javascript "access plus 12 months"
ExpiresByType video/x-flv "access plus 12 months"
```
#### 6.1.4检查语法重启apache
```shell
root@template /usr/local/apache/conf 19:43:03 # /usr/local/apache/bin/apachectl -t      
Syntax OK
root@template /usr/local/apache/conf 19:43:11 # /usr/local/apache/bin/apachectl graceful
```
#### 6.1.5通过浏览器查看配置效果
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/08.png?raw=true)
可以看到缓存到2017年
### 6.2mod_deflate模块（压缩发送）
#### 6.2.1检查模块是否安装
```shell
root@template /usr/local/apache/conf 20:20:45 # /usr/local/apache/bin/apachectl -M | grep deflate
 deflate_module (static)
Syntax OK
#以上是显示静态安装结果
root@template /usr/local/apache/conf 20:20:45 # /usr/local/apache/bin/apachectl -M | grep deflate
 deflate_module (shared)
Syntax OK
#以上是动态安装结果
```
#### 6.2.2手动安装
```shell
/usr/local/apache/bin/apxs -c -i -a mod_deflate.c
注：mod_expires.c文件在apache安装包的/modules/filters/下
注：如果我们是编译安装时已经编译进去的需要在住配置文件（httpd.conf）中解除注释如下行
LoadModule deflate_module modules/deflate_module.so
sed -i 's#\#LoadModule deflate_module modules/mod_deflate.so#LoadModule deflate_module modules/mod_deflate.so#g' /usr/local/apache/conf/httpd.conf                
注：若出现错误undefined symbol: inflateEnd请在LoadModule expires_module
modules/mod_expires.so前面加上
LoadFile /usr/lib64/libz.so
```
6.2.3配置压缩（写在主配置文件即可）
```conf
<ifmodule mod_deflate.c> 
DeflateCompressionLevel 9      #压缩等级，越大效率越高，消耗CPU也越高 
SetOutputFilter DEFLATE           #启用压缩 
DeflateFilterNote Input instream #声明输入流的byte数量 
DeflateFilterNote Output outstream  #声明输出流的byte数量 
DeflateFilterNote Ratio ratio   #声明压缩的百分比 
AddOutputFilterByType DEFLATE text/html text/plain text/xml     
AddOutputFilterByType DEFLATE application/javascript
AddOutputFilterByType DEFLATE text/css
#声明压缩的文件格式
AddOutputFilterByType DEFLATE image/gif image/png  image/jpe image/swf image/jpeg image/bmp
#DeflateFilterNote ratio     #在日志中放置压缩率标记，下面是记录日志的，这个功能一般不用 
#LogFormat '"%r" %{outstream}n/%{instream}n (%{ratio}n%%)' deflate 
#CustmLog logs/deflate_log.log deflate 
</ifmodule>
```
#### 6.2.4查看无压缩的返回
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/09.png?raw=true)
无压缩
#### 6.2.5检查语法重启apache
```shell
root@template /usr/local/apache/conf 20:30:19 # /usr/local/apache/bin/apachectl -t
Syntax OK
root@template /usr/local/apache/conf 20:31:12 # /usr/local/apache/bin/apachectl graceful
```
#### 6.2.6查看压缩结果
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/10.png?raw=true)
发现已有gzip
## 七.APACHE的两种工作模式
### 7.1prefork模式
#### 7.1.1prefork模式简介
如果不用“--with-mpm”显式指定某种MPM，prefork就是Unix平台上缺省的MPM。它所采用的预派生子进程方式也是 Apache 1.3中采用的模式。prefork本身并没有使用到线程，2.0版使用它是为了与1.3版保持兼容性；另一方面，prefork用单独的子进程来处理不同的请求，进程之间是彼此独立的，这也使其成为最稳定的MPM之一。
prefork的工作原理是，控制进程在最初建立“StartServers”个子进程后，为了满足MinSpareServers设置的需要创建一个进程，等待一秒钟，继续创建两个，再等待一秒钟，继续创建四个……如此按指数级增加创建的进程数，最多达到每秒32个，直到满足 MinSpareServers设置的值为止。这就是预派生（prefork）的由来。这种模式可以不必在请求到来时再产生新的进程，从而减小了系统开销以增加性能。servrelimit指系统限制最大的进程数，默认值很小如果MaxClients设的比unix默认servrelimit还大，就无效，返回默认值。因此servrelimit肯定要远大于apache的MaxClients
preforkMPM使用多个子进程，但每个子进程并不包含多线程。每个进程只处理一个链接。在许多系统上它的速度和workerMPM一样快，但是需要更多的内存。这种无线程的设计在某些情况下优于workerMPM：它可以应用于不具备线程安全的第三方模块(比如php)，且在不支持线程调试的平台上易于调试，而且还具有比workerMPM更高的稳定性。
#### 7.1.2prefork配置详解（conf/extra/http-mpm.conf）
```conf
< IfModule mpm_prefork_module>  #工作在prefork模式下
    StartServers 5  # StartServers是开始的进程数
    MinSpareServers 3  # MinSpareServers是最小空闲进数
    MaxSpareServers 10  # MaxSpareServers是最大空闲进程数
    ServerLimit 16  # ServerLimit是最大的进程数，最大值为2000
    MaxClients 16  # MaxClients是最大的请求并发
    MaxRequestsPerChild 2000
</IfModule>
```
### 7.2worker模式
#### 7.2.1 worker模式详解
相对于prefork，worker全新的支持多线程和多进程混合模型的MPM。由于使用线程来处理，所以可以处理相对海量的请求，而系统资源的开销要小 于基于进程的服务器。但是，worker也使用了多进程，每个进程又生成多个线程，以获得基于进程服务器的稳定性。在configure ?with-mpm=worker后，进行make编译、make install安装。
Worker 由主控制进程生成“StartServers”个子进程，每个子进程中包含固定的ThreadsPerChild线程数，各个线程独立地处理请求。同样， 为了不在请求到来时再生成线程，MinSpareThreads和MaxSpareThreads设置了最少和最多的空闲线程数；而MaxClients 设置了同时连入的clients最大总数。如果现有子进程中的线程总数不能满足负载，控制进程将派生新的子进程。MinSpareThreads和 MaxSpareThreads的最大缺省值分别是75和250。这两个参数对Apache的性能影响并不大，可以按照实际情况相应调节。 ThreadsPerChild是worker MPM中与性能相关最密切的指令。ThreadsPerChild的最大缺省值是64，如果负载较大，64也是不够的。这时要显式使用 ThreadLimit指令，它的最大缺省值是20000。Worker模式下所能同时处理的请求总数是由子进程总数乘以ThreadsPerChild 值决定的，应该大于等于MaxClients。如果负载很大，现有的子进程数不能满足时，控制进程会派生新的子进程。默认最大的子进程总数是16，加大时 也需要显式声明ServerLimit（最大值是20000）。需要注意的是，如果显式声明了ServerLimit，那么它乘以 ThreadsPerChild的值必须大于等于MaxClients，而且MaxClients必须是ThreadsPerChild的整数倍，否则 Apache将会自动调节到一个相应值。
#### 7.2.2 worker配置详解（conf/extra/http-mpm.conf）
```conf
< IfModule mpm_worker_module>   #工作在worker模式下
    StartServers         4  # StartServers是开始的进程数
    MaxClients         300
    MinSpareThreads     25
    MaxSpareThreads     75
    ThreadsPerChild     25
    MaxRequestsPerChild  0
</IfModule>
```
## 八.APACHE的SSL证书配置
### 8.1生成ssl证书（注意此步骤只用于测试正式生产环境需要到正规的证书机构购买）
#### 8.1.1生成一个2048为的私钥
```shell
root@template /usr/local/apache/conf 21:05:44 # openssl genrsa -out server.key 2048
Generating RSA private key, 2048 bit long modulus
...........+++
....+++
e is 65537 (0x10001)
```
#### 8.1.2生成证书签名请求（CSR），这里需要填写许多信息，如国家，省市，公司等
```shell
root@template /usr/local/apache/conf 21:10:51 # openssl req -new -key server.key -out server.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:dd
State or Province Name (full name) []:daguanren
Locality Name (eg, city) [Default City]:tianjin
Organization Name (eg, company) [Default Company Ltd]:daguanren
Organizational Unit Name (eg, section) []:daguanren
Common Name (eg, your name or your server's hostname) []:192.168.44.10
Email Address []:daguanren@gmail.com

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:daguanren
```
#### 8.1.3生成类型为X509的自签名证书。有效期设置3650天，即有效期为10年
```shell 
root@template /usr/local/apache/conf 21:11:50 # openssl x509 -req -days 3650 -in server.csr -signkey server.key -out server.crt
Signature ok
subject=/C=dd/ST=daguanren/L=tianjin/O=daguanren/OU=daguanren/CN=192.168.44.10/emailAddress=daguanren@gmail.com
Getting Private key
```
### 8.2配置apache ssl
#### 8.2.1配置主配置文件监听443端口
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/11.png?raw=true)
#### 8.2.2配置ssl虚拟主机
```conf
<VirtualHost *:443>
    SSLEngine on
    SSLCertificateFile /usr/local/apache/conf/server.crt  #证书签名
    SSLCertificateKeyFile /usr/local/apache/conf/server.key  #公钥
    <Directory /usr/local/apache/ssl>
        Options Indexes FollowSymLinks
        AllowOverride None
        Order allow,deny
        Allow from all
    </Directory>
    ServerAdmin email@example.com
DocumentRoot /usr/local/apache/ssl
ServerName proxy.mimvp.com
</VirtualHost>
```
#### 8.2.3检查语法重启apache
```shell
root@template /usr/local/apache/conf 21:49:08 # /usr/local/apache/bin/apachectl -t      
Syntax OK
root@template /usr/local/apache/conf 21:49:15 # /usr/local/apache/bin/apachectl graceful
```
8.2.4通过浏览器使用https访问
![image](https://github.com/yangzinan/Operations/blob/master/iamge/apache/12.png?raw=true)
