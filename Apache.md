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
1. 首先安装apache的依赖apr
```shell
tar -zxf apr-1.4.5.tar.gz
cd apr-1.4.5
./configure --prefix=/usr/local/apr
make && make install
cd ..
```
2. 安装apr-util
```shell
tar -zxf apr-util-1.3.12.tar.gz
cd apr-util-1.3.12
./configure --prefix=/usr/local/apr-util -with-apr=/usr/local/apr/bin/apr-1-config
make && make install
cd ..
```
3. 安装pcre
```shell
unzip -o pcre-8.10.zip
cd pcre-8.10
./configure --prefix=/usr/local/pcre 
make && make install
cd ..
```
4. 安装其他依赖
```shell
yum install –y zlib zlib-devel openssl openssl-devel
```
5. 编译安装apache
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
6. 启动apache并通过命令验证
```shell
/usr/local/apache/bin/apachectl –t  #检查apache配置文件的语法
```

如果出现以下报警（并不影响使用）

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/apache/01.png?raw=true)
解决办法：
> * 打开配置文件（httpd.conf）在安装目录下的conf目录下（本例在/usr/local/apache/conf下）
> * 修改ServerName打开注释并将ServerName改为localhost

启动apache
```shell
/usr/local/apache/bin/apachectl start
```
验证80端口是否启动
```shell
root@template /usr/local/apache/conf 16:53:10 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1140/sshd           
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1216/master         
tcp        0      0 :::80                       :::*                        LISTEN      110052/httpd   
```
验证apache进程是否启动
```shell
root@template /usr/local/apache/conf 16:54:47 # ps -ef | grep httpd | grep -v grep
root     110052      1  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110053 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110054 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110055 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
daemon   110056 110052  0 16:49 ?        00:00:00 /usr/local/apach-2.2.31/bin/httpd -k start
```
通过端口反查进程
```shell
root@template /usr/local/apache/conf 16:56:28 # lsof -i:80
COMMAND    PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
httpd   110052   root    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110054 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110055 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
httpd   110056 daemon    4u  IPv6 100734      0t0  TCP *:http (LISTEN)
```
客户端使用浏览器检查

![image] (https://github.com/yangzinan/Operations/blob/master/iamge/apache/02.png?raw=true)
