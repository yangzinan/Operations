# 1.Nginx介绍
  Nginx是一款轻量级的Web 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器，并在一个BSD-like 协议下发行。由俄罗斯的程序设计师Igor Sysoev所开发，供俄国大型的入口网站及搜索引擎Rambler（俄文：Рамблер）使用。其特点是占有内存少，并发能力强，事实上nginx的并发能力确实在同类型的网页服务器中表现较好，中国大陆使用nginx网站用户有：百度、京东、新浪、网易、腾讯、淘宝等。
# 2.Nginx功能
支持操作系统
FreeBSD 3— 10 / i386; FreeBSD 5— 10 / amd64;
Linux 2.2— 4 / i386; Linux 2.6— 4 / amd64; Linux 3— 4 / armv6l, armv7l, aarch64;
Solaris 9 / i386, sun4u; Solaris 10 / i386, amd64, sun4v;
AIX 7.1 / powerpc;
HP-UX 11.31 / ia64;
Mac OS X / ppc, i386;
Windows XP, Windows Server 2003.
结构与扩展
一个主进程和多个工作进程。工作进程是单线程的，且不需要特殊授权即可运行；
kqueue (FreeBSD 4.1+),epoll (Linux 2.6+),rt signals (Linux 2.2.19+),/dev/poll (Solaris 7 11/99+),select，以及 poll 支持；
kqueue支持的不同功能包括 EV_CLEAR,EV_DISABLE （临时禁止事件）， NOTE_LOWAT,EV_EOF，有效数据的数目，错误代码；
sendfile (FreeBSD 3.1+),sendfile (Linux 2.2+),sendfile64 (Linux 2.4.21+），和 sendfilev (Solaris 8 7/01+) 支持；
输入过滤 (FreeBSD 4.1+) 以及 TCP_DEFER_ACCEPT (Linux 2.4+) 支持；
10,000 非活动的 HTTP keep-alive 连接仅需要 2.5M内存。
最小化的数据拷贝操作；
其他HTTP功能；
基于IP 和名称的虚拟主机服务；
Memcached 的 GET 接口；
支持 keep-alive 和管道连接；
灵活简单的配置；
重新配置和在线升级而无须中断客户的工作进程；
可定制的访问日志，日志写入缓存，以及快捷的日志回卷；
4xx-5xx错误代码重定向；
基于 PCRE 的 rewrite 重写模块；
基于客户端IP 地址和 HTTP 基本认证的访问控制；
PUT,DELETE，和 MKCOL 方法；
支持 FLV （Flash 视频）；
带宽限制。
实验特性
内嵌的 perl；
通过 aio_read()/aio_write() 的套接字工作的实验模块，仅在 FreeBSD 下；
对线程的实验化支持，FreeBSD 4.x 的实现基于 rfork()；
Nginx 主要的英语站点是 http://sysoev. ru/en/；
英语文档草稿由 Aleksandar Lazic 完成 点击。
HTTP基础功能
处理静态文件，索引文件以及自动索引；
反向代理加速（无缓存），简单的负载均衡和容错；
FastCGI，简单的负载均衡和容错；
模块化的结构。过滤器包括gzipping,byte ranges,chunked responses，以及 SSI-filter。在SSI过滤器中，到同一个 proxy 或者 FastCGI 的多个子请求并发处理；
SSL 和 TLS SNI 支持；
IMAP/POP3代理服务功能：
使用外部 HTTP 认证服务器重定向用户到 IMAP/POP3 后端；
使用外部 HTTP 认证服务器认证用户后连接重定向到内部的 SMTP 后端；
# 3.Nginx的安装
## 3.1安装依赖
```shell
yum install -y pcre pcre-devel openssl openssl-devel
useradd -s /sbin/nologin -M nginx
```
## 3.2编译安装
```shell
tar -zxf nginx-1.11.10.tar.gz
cd nginx-1.11.10
useradd -s /sbin/nologin -M nginx
./configure --user=nginx \
--group=nginx \
--prefix=/usr/local/nginx-1.11.10 \
--with-http_stub_status_module \
--with-http_ssl_module \
--with-http_gzip_static_module \
--with-http_addition_module \
--with-http_realip_module \
--with-stream \
--with-pcre \
make
make install
```
* 其他编译参数
/configure --help
 
--help 显示本提示信息
 
--prefix=PATH 设定安装目录
 
--sbin-path=PATH 设定程序文件目录
 
--conf-path=PATH 设定配置文件(nginx.conf)目录
 
--error-log-path=PATH 设定错误日志目录
 
--pid-path=PATH 设定pid文件(nginx.pid)目录
 
--lock-path=PATH 设定lock文件(nginx.lock)目录
 
--user=USER 设定程序运行的用户环境(www)
 
--group=GROUP 设定程序运行的组环境(www)
 
--builddir=DIR 设定程序编译目录
 
--with-rtsig_module 允许rtsig模块rtsig模块是一种实时信号，在Linux 2.2.19 默认情况下，实时信号连接数不超过1024，但是对于高负载是肯定不够的。因此通过调整内核参数/proc/sys/kernel/rtsig-max达到效果。但是Linux 2.6.6-mm2开始，这个参数不再可用，并为每个进程有一个独立的信号队列，数字是由RLIMIT_SIGPENDING确定。当队列变得满载时，nginx开始抛弃连接并使用poll方法，直到负载恢复正常。
 
--with-select_module 允许select模块(一种轮询模式,不推荐用在高载环境)
 
--without-select_module 不使用select模块标准连接模式。默认情况下自动编译方式。您可以启用或禁用通过使用-select_module和不带- select_module配置参数这个模块。
 
--with-poll_module 允许poll模块(一种轮询模式,不推荐用在高载环境)标准连接模式。默认情况下自动编译方式。
 
--without-poll_module 不使用poll模块
 
--with-http_ssl_module 允许ngx_http_ssl_module模块(Apache对应:mod_ssl)
 
--with-http_realip_module 允许ngx_http_realip_module模块(mod_rpaf)此模块支持显示真实来源IP地址，主要用于NGINX做前端负载均衡服务器使用。
 
--with-http_addition_module 允许ngx_http_addition_module模块(mod_layout)游戏服务器不必安装，门户网站可以安装，有利于被搜索引擎收录页面信息。
 
--with-http_xslt_module 允许ngx_http_xslt_module模块这个模块是一个过滤器，它可以通过XSLT模板转换XML应答。0.7.8后面版本才可以使用。
 
--with-http_sub_module 允许ngx_http_sub_module模块这个模块可以能够在nginx的应答中搜索并替换文本。
 
--with-http_dav_module 允许ngx_http_dav_module模块(mod_dav)为文件和目录指定权限，限制不同类型的用户对于页面有不同的操作权限
 
--with-http_flv_module 允许ngx_http_flv_module模块(mod_flvx)这个模块支持对FLV（flash）文件的拖动播放。
 
--with-http_gzip_static_module 允许ngx_http_gzip_static_module模块(mod_dflate)。这个模块在一个预压缩文件传送到开启Gzip压缩的客户端之前检查是否已经存在以“.gz”结尾的压缩文件，这样可以防止文件被重复压缩。
 
--with-http_random_index_module 允许ngx_http_random_index_module模块(mod_autoindex)，从目录中选择一个随机主页
 
--with-http_stub_status_module 允许ngx_http_stub_status_module模块(mod_status)这个模块可以取得一些nginx的运行状态，如果是工业状况，可以直接取消。
 
--without-http_charset_module 不使用ngx_http_charset_module模块这个模块将在应答头中为"Content-Type"字段添加字符编码
 
--without-http_gzip_module 不使用ngx_http_gzip_module模块，文件压缩模式。
 
--without-http_ssi_module 不使用ngx_http_ssi_module模块，此模块处理服务器端包含文件(ssi)的处理.
 
--without-http_userid_module 不使用ngx_http_userid_module模块

--without-http_access_module 不使用ngx_http_access_module模块
 
--without-http_auth_basic_module 不使用ngx_http_auth_basic_module模块
 
--without-http_autoindex_module 不使用ngx_http_autoindex_module模块
 
--without-http_geo_module 不使用ngx_http_geo_module模块这个模块基于客户端的IP地址创建一些ngx_http_geoip_module变量，并与MaxMindGeoIP文件进行匹配，该模块仅用于0.7.63和0.8.6版本之后。但效果不太理想，对于城市的IP记录并不是特别准确，不过对于网站的来源访问区域的分析大致有一定参考性。
 
--without-http_map_module 不使用ngx_http_map_module模块这个模块允许你分类或者同时映射多个值到多个不同值并储存到一个变量中，可用于页面跳转其他域名使用。
 
--without-http_referer_module 不使用ngx_http_referer_module模块当一个请求头的Referer字段中包含一些非正确的字段，这个模块可以禁止这个请求访问站点。可以禁止盗链的情况发生。
 
--without-http_rewrite_module 不使用ngx_http_rewrite_module模块，跳转模块
 
--without-http_proxy_module 不使用ngx_http_proxy_module模块，代理模块
 
--without-http_fastcgi_module 不使用ngx_http_fastcgi_module模块
 
--without-http_memcached_module 不使用ngx_http_memcached_module模块
 
--without-http_limit_zone_module 不使用ngx_http_limit_zone_module模块此模块可以限制并发连接，达到减少攻击的效果
 
 
--without-http_empty_gif_module 不使用ngx_http_empty_gif_module模块这个模块在内存中保存一个能够很快传递的1×1透明GIF
 
--without-http_browser_module 不使用ngx_http_browser_module模块识别客户端浏览器版本来制定计划。可以实现更加来源浏览器的版本访问不同页面的效果。
 
--without-http_upstream_ip_hash_module，不使用ngx_http_upstream_ip_hash_module模块
 
--with-http_perl_module 允许ngx_http_perl_module模块，这个模块允许nginx使用SSI调用perl或直接执行perl 
 
--with-perl_modules_path=PATH 设置perl模块路径
 
--with-perl=PATH 设置perl库文件路径
 
--http-log-path=PATH 设置access log文件路径
 
--http-client-body-temp-path=PATH 设置客户端请求临时文件路径
 
--http-proxy-temp-path=PATH 设置http proxy临时文件路径
 
--http-fastcgi-temp-path=PATH 设置http fastcgi临时文件路径
 
--without-http 不使用HTTP server功能，如果只是做代理服务器，可以不提供http服务
 
--with-mail 允许POP3/IMAP4/SMTP代理模块
 
--with-mail_ssl_module 允许ngx_mail_ssl_module模块这个模块使得POP3／IMAP／SMTP可以使用SSL／TLS.配置已经定义了HTTP SSL模块，但是不支持客户端证书检测
 
--without-mail_pop3_module 不允许ngx_mail_pop3_module模块
 
--without-mail_imap_module 不允许ngx_mail_imap_module模块
 
--without-mail_smtp_module 不允许ngx_mail_smtp_module模块
 
--with-google_perftools_module 允许ngx_google_perftools_module模块(调试用)这个模块可以启用google性能分析工具套件，模块在版本0.6.29增加。
 
--with-cpp_test_module 允许ngx_cpp_test_module模块
 
--add-module=PATH 允许使用外部模块,以及路径
 
--with-cc=PATH 设置C编译器路径
 
--with-cpp=PATH 设置C预处理路径
 
--with-cc-opt=OPTIONS 设置C编译器参数
 
--with-ld-opt=OPTIONS 设置连接文件参数
 
--with-cpu-opt=CPU 为指定CPU优化,可选参数有:pentium, pentiumpro, pentium3, pentium4,athlon, opteron, sparc32, sparc64, ppc64
 
--without-pcre 不使用pcre库文件，pcre是包括 perl 兼容的正规表达式库
 
--with-pcre=DIR 设定PCRE库路径
 
--with-pcre-opt=OPTIONS 设置PCRE运行参数
 
--with-md5=DIR 设定md5库文件路径
 
--with-md5-opt=OPTIONS 设置md5运行参数
 
--with-md5-asm 使用md5源文件编译
 
--with-sha1=DIR 设定sha1库文件路径
sha1：安全哈希算法（Secure Hash Algorithm）主要适用于数字签名标准（Digital Signature Standard DSS）里面定义的数字签名算法（Digital Signature Algorithm DSA）。对于长度小于2^64位的消息，SHA1会产生一个160位的消息摘要。当接收到消息的时候，这个消息摘要可以用来验证数据的完整性。在传输的过程中，数据很可能会发生变化，那么这时候就会产生不同的消息摘要。
 
--with-sha1-opt=OPTIONS 设置sha1运行参数
 
--with-sha1-asm 使用sha1源文件编译
 
--with-zlib=DIR 设定zlib库文件路径很多程序中的压缩或者解压缩函数都会用到这个库
 
--with-zlib-opt=OPTIONS 设置zlib运行参数
 
--with-zlib-asm=CPU 使zlib对特定的CPU进行优化,可选参数:pentium, pentiumpro
 
--with-openssl=DIR 设定OpenSSL库文件路径Openssl作为一个基于密码学的安全开发包，OpenSSL提供的功能相当强大和全面，囊括了主要的密码算法、常用的密钥和证书封装管理功能以及SSL协议，并提供了丰富的应用程序供测试或其它目的使用。
 
--with-openssl-opt=OPTIONS 设置OpenSSL运行参数
 
--with-debug 允许调试日志
## 3.3 启动Nginx
```shell
/usr/local/nginx/sbin/nginx
```
* 检查80端口
```shell
root@template /usr/local/nginx/conf 09:07:26 # netstat -ntlup              
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1038/master         
tcp        0      0 0.0.0.0:3358                0.0.0.0:*                   LISTEN      17706/sshd          
tcp        0      0 ::1:25                      :::*                        LISTEN      1038/master         
tcp        0      0 :::3358                     :::*                        LISTEN      17706/sshd 
```
* 检查nginx进程
```shell
root@template /usr/local/nginx/conf 09:13:46 # ps -ef | grep nginx | grep -v grep
root      26396      1  0 09:00 ?        00:00:00 nginx: master process ../sbin/nginx
nginx     26432  26396  0 09:07 ?        00:00:00 nginx: worker process
root@template /usr/local/nginx/conf 09:13:54 #
```
* 通过端口反查
```shell
root@template /usr/local/nginx/conf 09:14:50 # lsof -i:80
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   26396  root    6u  IPv4  47532      0t0  TCP *:http (LISTEN)
nginx   26432 nginx    6u  IPv4  47532      0t0  TCP *:http (LISTEN)
```

# 4.NGINX WEB配置
## 4.1nginx目录结构
```shell
root@template /usr/local/nginx-1.6.2 08:15:07 # tree
├── client_body_temp
├── conf  #这是nginx的配置文件路径
│   ├── fastcgi.conf #fastcgi的配置文件
│   ├── fastcgi.conf.default
│   ├── fastcgi_params  #fastcgi的参数文件
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf  #nginx的主配置文件
│   ├── nginx.conf.back
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── fastcgi_temp  #fastcgi的临时目录
├── html  #nginx的默认站点目录
│   ├── 50x.html  #错误的现实页面
│   └── index.html #默认首页文件
├── iptables
├── logs  #这是nginx的日志路径
│   ├── access.log  #nginx默认的访问日志文件
│   ├── error.log  #nginx默认错误日志文件
│   └── nginx.pid  #nginx的pid文件,nginx进程启动后会把所有的ID号写到此文件
├── proxy_temp
├── sbin
│   └── nginx
├── scgi_temp  #临时目录
└── uwsgi_temp  #临时目录
```
## 4.2nginx主配置文件注解
```conf
worker_processes  1;  #进程个数(和CPU的核数相等或是两倍)
events {
    worker_connections  1024;  #单个进程最多同时多少个用户连接
	use epoll;     #使用epool模型
}
http {  #http标签
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
	#上面四行是http公用参数
    server {  #server即一个虚拟主机
        listen       80;  #监听端口
        server_name  localhost;  #虚拟主机名(域名)
        location / {  #定义根
            root   html;  #根目录（这里就是nginx安装目录的html文件夹）
            index  index.html index.htm;  #首页文件
        }
		fastcgi_intercept_errors on;  #若要开启404错误页面跳转则必须开启fastcgi_intercept_errors
        error_page   500 502 503 504  /50x.html;  #错误的显示页面，500，502，503，504都显示50x.html
        location = /50x.html {
            root   html;  #定义了上面的错误显示文件50x.html在html文件夹中去找
        }
    }
}
注:nginx的配置文件每一行以”;”结尾
```
# 5.NGINX的虚拟主机配置
## 5.1基于域名的虚拟主机配置
### 5.1.1备份原配置文件，生成没有注释的配置文件
```shell
root@template /usr/local/nginx/conf 08:37:27 # mv nginx.conf nginx.conf.back
grep -Ev "#|^$" nginx.conf.back > nginx.conf
root@template /usr/local/nginx/conf 08:40:57 # grep -Ev "#|^$" nginx.conf.back > nginx.conf
```
### 5.1.2添加配置(在http标签下)
```conf
server {
    listen       80;
    server_name  www.dgr.com;
    location / {
        root   /root/dgr;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /root/dgr;
    }
}
```
### 5.1.3优雅重启nginx
```shell
root@template /usr/local/nginx/conf 08:46:04 # ../sbin/nginx -s reload
```
### 5.1.4通过修改本地hosts用浏览器验证

![1](https://user-images.githubusercontent.com/12201941/29481829-a316b718-84b8-11e7-96e0-35ac53f65fc0.png)

![2](https://user-images.githubusercontent.com/12201941/29481836-c2156b50-84b8-11e7-89a8-72052f6065ee.png)

## 5.2基于端口的虚拟主机
### 5.2.1添加配置
```conf
server {
    listen       8080;
    server_name  localhost;
    location / {
        root   8080;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   8080;
    }
}
```
### 5.2.2建立虚拟主机目录
```shell
mkdir /usr/local/nginx/8080
echo 8080 > /usr/local/nginx/8080/index.html
```
### 5.2.3重启nginx并检查
```shell
root@template /usr/local/nginx/conf 09:29:45 # ../sbin/nginx -s reload                   
root@template /usr/local/nginx/conf 09:29:48 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:8080                0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1038/master         
tcp        0      0 0.0.0.0:3358                0.0.0.0:*                   LISTEN      17706/sshd          
tcp        0      0 ::1:25                      :::*                        LISTEN      1038/master         
tcp        0      0 :::3358                     :::*                        LISTEN      17706/sshd          
root@template /usr/local/nginx/conf 09:30:27 # lsof -i:8080
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   26396  root   10u  IPv4  49046      0t0  TCP *:webcache (LISTEN)
nginx   26457 nginx   10u  IPv4  49046      0t0  TCP *:webcache (LISTEN)
root@template /usr/local/nginx/conf 09:30:38 #
```
### 5.2.4本地浏览器访问

![3](https://user-images.githubusercontent.com/12201941/29481878-67baf28c-84b9-11e7-8e8c-63090a68e249.png)

# 6.NGINX的配置分离
## 6.1创建子配置文件目录
```shell
mkdir /usr/local/nginx/conf/conf.d
```
## 6.2修改配置文件删除所有server标签并引入之配置文件
```conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include       conf.d/*.conf;  #引入conf.d下面所有的.conf文件
}
```
## 6.3将所有的serve标签放到conf.d下面的vhost.conf文件中
```conf
server {
    listen       80;
    server_name  www.daguanren.com;
    location / {
        root   html;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }

}

server {
    listen       80;
    server_name  www.dgr.com;
    location / {
        root   dgr;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   dgr;
    }

}


server {
    listen       8080;
    server_name  localhost;
    location / {
        root   8080;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   8080;
    }
}
```
## 6.5重启nginx
```shell
/usr/local/nginx/sbin/nginx -s reload
```
# 7.NGINX的优化模块
## 7.1缓存模块
### 7.1.1配置缓存模块
```conf
location ~ .*\.(jpg|jpeg|gif|ico|bmp|png)$ {  #对括号内后缀名的文件进行缓存
			expires	3650d;	#缓存3650天
			root   html;  #指定站点目录
		}
	注：在server标签内添加
```
### 7.1.2上传图片查看没有缓存状态

![4](https://user-images.githubusercontent.com/12201941/29481918-0f0067b6-84ba-11e7-81d4-4e0820307d92.png)

### 7.1.3重启nginx查看缓存生效后状态
```shell
/usr/local/nginx/sbin/nginx -s reload
```

![5](https://user-images.githubusercontent.com/12201941/29481942-653af3b2-84ba-11e7-8c4c-8778c2893d33.png)

 * 可以看到缓存了10年
## 7.2压缩模块
### 7.2.1配置压缩模块
```conf
gzip on;  #开启压缩
gzip_min_length  1k;  #最小压缩的文件大小
gzip_buffers     4 16k;  #压缩缓冲区
gzip_http_version 1.0;  #压缩版本
gzip_comp_level 2;  #压缩比率
gzip_types       text/css application/javascript text/xml image/jpeg;  #压缩文件类型
gzip_vary on;
注1：在http标签下配置（server，location标签下都可以）
注2：在conf/mime.types文件里标注了各种文件添加压缩的语法格式
```
### 7.2.2测试没有启用压缩状态


![6](https://user-images.githubusercontent.com/12201941/29481981-f4f3710a-84ba-11e7-8d67-4ded1b986f10.png)

### 7.2.3重启nginx查看生效后的状态

![7](https://user-images.githubusercontent.com/12201941/29481982-ffdf2b54-84ba-11e7-9c93-4dc00ce077dc.png)

* 已经出现gzip压缩
# 8.NGINX的防盗链
* 在server标签添加以下内容
```conf
location ~ .*\.(wma|wmv|asf|mp3|mmf|zip|rar|jpg|gif|png|swf|flv)$ {  #要防盗链的文件类型
     valid_referers none blocked *.yiibase.com yiibase.com;  #允许连接的域名
     if ($invalid_referer) {
     		return 403;  #盗链直接返回403
      }
}
```
# 9.NGINX的PHP环境
## 9.1fastcgi介绍
FastCGI像是一个常驻(long-live)型的CGI，它可以一直执行着，只要激活后，不会每次都要花费时间去fork一次(这是CGI最为人诟病的fork-and-execute 模式)。它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行并且接受来自其它网站服务器来的请求。
FastCGI是语言无关的、可伸缩架构的CGI开放扩展，其主要行为是将CGI解释器进程保持在内存中并因此获得较高的性能。众所周知，CGI解释器的反复加载是CGI性能低下的主要原因，如果CGI解释器保持在内存中并接受FastCGI进程管理器调度，则可以提供良好的性能、伸缩性、Fail- Over特性等等。
## 9.2fastcgi原理
1、Web Server启动时载入FastCGI进程管理器（IIS ISAPI或Apache Module)

2、FastCGI进程管理器自身初始化，启动多个CGI解释器进程(可见多个php-cgi)并等待来自Web Server的连接。

3、当客户端请求到达Web Server时，FastCGI进程管理器选择并连接到一个CGI解释器。Web server将CGI环境变量和标准输入发送到FastCGI子进程php-cgi。

4、FastCGI子进程完成处理后将标准输出和错误信息从同一连接返回Web Server。当FastCGI子进程关闭连接时，请求便告处理完成。FastCGI子进程接着等待并处理来自FastCGI进程管理器(运行在Web Server中)的下一个连接。 在CGI模式中，php-cgi在此便退出了。

在上述情况中，你可以想象CGI通常有多慢。每一个Web请求PHP都必须重新解析php.ini、重新载入全部扩展并重初始化全部数据结构。使用FastCGI，所有这些都只在进程启动时发生一次。一个额外的好处是，持续数据库连接(Persistent database connection)可以工作。
## 9.3fastcgi特点
1、如CGI，FastCGI也具有语言无关性。

2、如CGI, FastCGI在进程中的应用程序，独立于核心web服务器运行,提供了一个比API更安全的环境。(API是把应用程序的代码与核心的web服务器链接在一起，这意味着在一个错误的API的应用程序可能会损坏其他应用程序或核心服务器; 恶意的API的应用程序代码甚至可以窃取另一个应用程序或核心服务器的密钥。)

3、FastCGI技术目前支持语言有 PHP、C/C++、Java、Perl、Tcl、Python、SmallTalk、Ruby、Aardio等。相关模块在Apache,IIS, Lighttpd,Nginx等流行的服务器上也是可用的。

4、如CGI，FastCGI的不依赖于任何Web服务器的内部架构，因此即使服务器技术的变化, FastCGI依然稳定不变。
## 9.4fastcgi的不足
因为是多进程，所以比CGI多线程消耗更多的服务器内存，PHP-CGI解释器每进程消耗7至25兆内存，将这个数字乘以50或100就是很大的内存数。Nginx 0.8.46+PHP 5.2.14(FastCGI)服务器在3万并发连接下，开启的10个Nginx进程消耗150M内存（15M*10=150M），开启的64个php-cgi进程消耗1280M内存（20M*64=1280M），加上系统自身消耗的内存，总共消耗不到2GB内存。如果服务器内存较小，完全可以只开启25个php-cgi进程，这样php-cgi消耗的总内存数才500M。
## 9.5为nginx安装配置fastcgi解析php
### 9.5.1安装php
#### 9.5.1.1安装依赖
```shell
yum install -y libxml2-devel libjpeg-devel freetype-devel libpng-devel  libxslt-devel curl-devel gcc gcc-c++ openssl* libxslt* libcurl*
tar -zxf libiconv-1.14.tar.gz
cd libiconv-1.14
./configure --prefix=/usr/local/libiconv
make
make install
```
#### 9.5.1.2编译安装PHP
```shell
tar -zxf php-5.5.20.tar.gz
cd php-5.5.20

./configure \
--prefix=/usr/local/php-5.5.20 \
--enable-mysqlnd \
--with-mysql=mysqlnd \
--with-mysqli=mysqlnd \
--with-pdo-mysql=mysqlnd \
--enable-bcmath \
--with-gettext \
--with-xmlrpc \
--with-openssl \
--with-zlib \
--with-freetype-dir \
--with-gd \
--with-jpeg-dir \
--with-png-dir \
--with-iconv=/usr/local/libiconv \
--enable-short-tags \
--enable-sockets \
--enable-soap \
--enable-mbstring \
--enable-static \
--enable-gd-native-ttf \
--enable-zip \
--with-curl \
--with-xsl \
--enable-ftp \
--enable-fpm \
--with-gettext \
--with-fpm-user=nginx\
--with-fpm-group=nginx \
--with-libxml-dir
make && make install
mkdir /var/log/php
```
## 9.6创建fastcgi配置文件
```conf
[global]
pid = /var/run/php-fpm.pid
error_log = /var/log/php/php-fpm.log
log_level = error
rlimit_files = 32768
events.mechanism = epoll
[www]
user = nginx
group = nginx
listen = 127.0.0.1:9000
;listen = 0.0.0.0:9000
listen.owner = nginx
listen.group = nginx
pm = dynamic
pm.max_children = 1024
pm.start_servers = 16
pm.min_spare_servers = 5
pm.max_spare_servers = 20
pm.max_requests = 2048
slowlog = /var/log/php/$pool.log.slow
request_slowlog_timeout = 10
```
## 9.7启动fastcgi并验证
```shell
root@template /usr/local/src/php-5.5.20 12:36:01 # /usr/local/php-5.5.20/sbin/php-fpm 
root@template /usr/local/src/php-5.5.20 12:36:05 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:8080                0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1038/master         
tcp        0      0 0.0.0.0:3358                0.0.0.0:*                   LISTEN      17706/sshd          
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      57788/php-fpm       
tcp        0      0 ::1:25                      :::*                        LISTEN      1038/master         
tcp        0      0 :::3358                     :::*                        LISTEN      17706/sshd          
root@template /usr/local/src/php-5.5.20 12:36:10 # ps -ef | grep php 
root      57788      1  0 12:36 ?        00:00:00 php-fpm: master process (/usr/local/php-5.5.20/etc/php-fpm.conf)
nginx     57789  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57790  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57791  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57792  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57793  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57794  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57795  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57796  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57797  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57798  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57799  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57800  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57801  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57802  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57803  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
nginx     57804  57788  0 12:36 ?        00:00:00 php-fpm: pool www                 
root      57807  16444  0 12:37 pts/1    00:00:00 grep php
root@template /usr/local/src/php-5.5.20 12:37:37 # lsof -i:9000
COMMAND   PID  USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
php-fpm 57788  root    7u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57789 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57790 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57791 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57792 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57793 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57794 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57795 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57796 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57797 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57798 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57799 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57800 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57801 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57802 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57803 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
php-fpm 57804 nginx    0u  IPv4 183139      0t0  TCP localhost:cslistener (LISTEN)
root@template /usr/local/src/php-5.5.20 12:37:50 #
```
##9.8配置nginx支持php
* 在一个server标签下添加如下
```conf
location ~ .*\.(php|php5)?$ {  
    root    dgr;   
    fastcgi_pass  127.0.0.1:9000;
    fastcgi_index index.php;
    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    include fastcgi.conf;
}
```
* 重启nginx并验证
```shell
root@template /usr/local/nginx/dgr 12:54:10 # /usr/local/nginx/sbin/nginx -s reload
```
![8](https://user-images.githubusercontent.com/12201941/29482035-33f41dea-84bc-11e7-8acc-8b55e7cfdb83.png)

## 9.9php其他模块
参见apache
# 10.NGINX的SSL配置
## 10.1生成测试用证书
参见apache
## 10.2配置nginx
* 添加一个server标签
```conf
server {
    listen       443;
    ssl on;
    ssl_certificate /usr/local/nginx/ssl/server.crt;
    ssl_certificate_key /usr/local/nginx/ssl/server.key;
    server_name  localhost;
    location / {
        root   ssl;
        index  index.html index.htm;
    }
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   ssl;
    }
}
```
## 10.3重启nginx检测并测试
```shell
root@template /usr/local/nginx/ssl 13:20:50 # /usr/local/nginx/sbin/nginx -s reload                                          
root@template /usr/local/nginx/ssl 13:22:48 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:8080                0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1038/master         
tcp        0      0 0.0.0.0:443                 0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:3358                0.0.0.0:*                   LISTEN      17706/sshd          
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      57788/php-fpm       
tcp        0      0 ::1:25                      :::*                        LISTEN      1038/master         
tcp        0      0 :::3358                     :::*                        LISTEN      17706/sshd 
```
## 10.4客户端浏览器测试

![9](https://user-images.githubusercontent.com/12201941/29482132-ef2c3c4a-84bd-11e7-9e6b-059b5c0a492e.png)

# 11.NGINX的反向代理
如果您的内容服务器具有必须保持安全的敏感信息，如信用卡号数据库，可在防火墙外部设置一个代理服务器作为内容服务器的替身。当外部客户机尝试访问内容服务器时，会将其送到代理服务器。实际内容位于内容服务器上，在防火墙内部受到安全保护。代理服务器位于防火墙外部，在外部客户机看来就像是内容服务器。

当客户机向站点提出请求时，请求将转到代理服务器。然后，代理服务器通过防火墙中的特定通路，将客户机的请求发送到内容服务器。内容服务器再通过该通道将结果回传给代理服务器。代理服务器将检索到的信息发送给客户机，好像代理服务器就是实际的内容服务器如果内容服务器返回错误消息，代理服务器会先行截取该消息并更改标头中列出的任何 URL，然后再将消息发送给客户机。如此可防止外部客户机获取内部内容服务器的重定向 URL。
这样，代理服务器就在安全数据库和可能的恶意攻击之间提供了又一道屏障。与有权访问整个数据库的情况相对比，就算是侥幸攻击成功，作恶者充其量也仅限于访问单个事务中所涉及的信息。未经授权的用户无法访问到真正的内容服务器，因为防火墙通路只允许代理服务器有权进行访问。

![11](https://user-images.githubusercontent.com/12201941/29482309-a2a229a8-84c1-11e7-956a-5bfcaf4d688a.png)

## 11.2在配置文件http部分添加反向代理配置
```conf
proxy_connect_timeout 1；
#后端服务器连接的超时时间_发起握手等候响应超时时间
proxy_read_timeout 30；
#连接成功后_等候后端服务器响应时间_其实已经进入后端的排队之中等候处理（也可以说是后端服务器处理请求的时间）
proxy_send_timeout 20；
#后端服务器数据回传时间_就是在规定时间之内后端服务器必须传完所有的数据
proxy_buffer_size          4k;
proxy_buffers              4 32k;
proxy_busy_buffers_size    64k;
proxy_temp_file_write_size 64k;
proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;  #如果出现上述错误则直接转发到下一个RS 
注：以上配置也可以添加在server标签下，在http下全局生效在server下仅对server标签生效。重复部分以server标签下的配置为准
```
## 11.3创建反向代理配置文件（conf.d/proxy.conf）
```conf
upstream dgr_webserver {
        #ip_hash;     #会话保持(一般不使用)。注：当使用ip_hash时不可添加weigth和backup
        server 192.168.44.12 weight=5 max_fails=1 fail_timeout=10s; #weight是权重 ，max_fails是检查次数 ，fail_timeout是检查间隔时间
        server 192.168.44.12:8080 weight=5;
	   server 192.168.44.14 weight=1 backup  #backup是热备只有上面两个全部宕机才会被启用
}   #一个RES组

server {
                listen 90;
                server_name localhost;
                location / {
                proxy_pass http://dgr_webserver;   #80端口转发到RES组(非80端口后面加上:port)
                proxy_set_header Host $host;  #通过后端Rs的域名虚拟主机获取不加这个则默认走第一个虚拟主机
		  proxy_set_header X-Forwarded-For $remote_addr;   #是RS获取客户真实IP
注：apache需要修改日志格式记录真实IP: LogFormat "\"%{ X-Forwarded-For}i\" %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
                }
}
```
## 11.4使用curl验证
```shell
root@template /usr/local/nginx/conf/conf.d 14:49:56 # curl 192.168.44.12:90                
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@template /usr/local/nginx/conf/conf.d 14:49:59 # curl 192.168.44.12:90
8080
注：可以看到两次curl的返回结果不一致，说明分别被反向代理到了后端不同的RSV
```
## 11.5nginx七层负载均衡
### 11.5.1实验准备
* 本机80端口为动态PHP页面


![12](https://user-images.githubusercontent.com/12201941/29482363-83ae17f4-84c2-11e7-838b-323fdb3ca96e.png)

* 本机8080端口为静态图片

![13](https://user-images.githubusercontent.com/12201941/29482371-9aa6896e-84c2-11e7-8fdb-23401e05366c.png)

### 11.5.2配置基于扩展名动静分离
```conf
upstream php {
        server 192.168.44.12 weight=5;
}   #动态php页面

upstream image {
        server 192.168.44.12:8080 weight=5;
}   #静态图片页面

server {
    listen 90;
    server_name localhost;
    location / {
    location ~ .*\.(jpg|jpeg|gif|ico|bmp|png|html|css|js)$ {    #静态文件走静态RS组
                proxy_pass http://image;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }
        location ~ .*\.(php|php5)$ {                  #动态文件走动态RS组
                proxy_pass http://php;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }

    }
}
```
### 11.5.3验证
* 静态

![14](https://user-images.githubusercontent.com/12201941/29482389-de748d9e-84c2-11e7-95df-615a41b468be.png)

* 动态

![15](https://user-images.githubusercontent.com/12201941/29482477-435f2c86-84c4-11e7-8bb5-355deabf5280.png)

### 11.5.4配置基于URL的动静分离
```conf
upstream php {	
        server 192.168.44.12 weight=5;
}   #动态php页面

upstream image {
        server 192.168.44.12:8080 weight=5;
}   #静态图片页面

server {
    listen 90;
    server_name localhost;
     location /image/ {                 #如果访问image目录则转到静态RS组
                proxy_pass http://image;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
      }
      location /php/ {                #如果访问php目录则转到动态RS组
                proxy_pass http://php;
                proxy_set_header Host $host;
                proxy_set_header X-Forwarded-For $remote_addr;
        }

}
```
### 11.5.5验证
* 静态

![14](https://user-images.githubusercontent.com/12201941/29482389-de748d9e-84c2-11e7-95df-615a41b468be.png)

* 动态

![15](https://user-images.githubusercontent.com/12201941/29482477-435f2c86-84c4-11e7-8bb5-355deabf5280.png)

# 12.NGINX缓存功能
## 12.1缓存的概念
Web内容可以缓存在客户端、代理服务器以及服务器端。研究表明，缓存技术可以显著地提高WWW性能，它可以带来以下好处：

（1）减少网络流量，从而减轻拥塞。

（2）降低客户访问延迟，其主要原因有：①缓存在代理服务器中的内容，客户可以直接从代理获取而不是从远程服务器获取，从而减小了传输延迟②没有被缓存的内容由于网络拥塞及服务器负载的减轻而可以较快地被客户获取。

（3）由于客户的部分请求内容可以从代理处获取，从而减轻了远程服务器负载。

（4）如果由于远程服务器故障或者网络故障造成远程服务器无法响应客户的请求，客户可以从代理中获取缓存的内容副本，使得WWW服务的鲁棒性得到了加强。

Web缓存系统也会带来以下问题：

（1）客户通过代理获取的可能是过时的内容。

（2）如果发生缓存失效，客户的访问延迟由于额外的代理处理开销而增加。因此在设计Web缓存系统时，应力求做到Cache命中率最大化和失效代价最小化。

（3）代理可能成为瓶颈。因此应为一个代理设定一个服务客户数量上限及一个服务效率下限，使得一个代理系统的效率至少同客户直接和远程服务器相连的效率一样。
## 12.2配置缓存
### 12.1.1在http部分添加如下
```conf
proxy_connect_timeout 30;   #链接超时时间
proxy_send_timeout 6000;   #发送超市时间
proxy_read_timeout 6000;   #读取超时时间
proxy_buffers 8 1m;
proxy_buffer_size 4m;
proxy_busy_buffers_size 6m;
proxy_max_temp_file_size 0;
proxy_temp_file_write_size 12m;
proxy_temp_path /tmp/proxy_temp  1 2;     #临时文件路径
proxy_cache_path /tmp/proxy_cache1 levels=1:2 keys_zone=cache0:10m inactive=1d max_size=1g;  #缓存路径，keys_zone=cache0:10m是缓存的名称和内存缓存的大小本例为10m（缓存的名称是cache0这里需要记住后面会用到）
注：以上配置也可以添加在server标签下，在http下全局生效在server下仅对server标签生效。重复部分以server标签下的配置为准
```
### 12.1.2配置server标签
```conf
server {
                listen 90;
                server_name localhost;
                 location /image/ {                 #如果访问static目录则转到静态RS组
                proxy_pass http://image;   
                                                                proxy_set_header Host $host;  
                                                                proxy_set_header X-Forwarded-For $remote_addr;
                                                                proxy_redirect off;
                                                                proxy_set_header Host $host;
                                                                proxy_cache cache0; #定义使用哪个缓存
                                                                proxy_cache_valid 200 302 1h;
                                                                proxy_cache_valid 301 1d;
                                                                proxy_cache_valid any 1m;
      }
}
```
## 12.3验证
* 访问静态页面生成缓存

![16](https://user-images.githubusercontent.com/12201941/29482525-2c8904ae-84c5-11e7-981d-6a7f5523666e.png)

* 停止后端的8080端口
```shell
root@template /usr/local/nginx/8080/image 09:47:22 # netstat -ntlup
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1038/master         
tcp        0      0 0.0.0.0:90                  0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:443                 0.0.0.0:*                   LISTEN      26396/nginx         
tcp        0      0 0.0.0.0:3358                0.0.0.0:*                   LISTEN      17706/sshd          
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      57788/php-fpm       
tcp        0      0 ::1:25                      :::*                        LISTEN      1038/master         
tcp        0      0 :::3358                     :::*                        LISTEN      17706/sshd          
#可以看到已经没有80端口
```

![17](https://user-images.githubusercontent.com/12201941/29482537-6b9fb4ee-84c5-11e7-8ff7-5550f5ed62ff.png)
已经无法访问8080
* 访问90端口

![18](https://user-images.githubusercontent.com/12201941/29482564-a4469f88-84c5-11e7-9c20-cf902d42d468.png)
90端口依然可以访问，证名nginx已经缓存了改文件


