# Nagios监控系统

#### By：大官人

#### Email：DaGuanR@gmail.com

#### QQ:375739049
## 一.NAGIOS简介
Nagios是一个监视系统运行状态和网络信息的监视系统。Nagios能监视所指定的本地或远程主机以及服务，同时提供异常通知功能等。Nagios可运行在Linux/Unix平台之上，同时提供一个可选的基于浏览器的WEB界面以方便系统管理人员查看网络状态，各种系统问题，以及日志等等。
## 二.NAGIOS功能特点
* 监控网络服务（SMTP、POP3、HTTP、NNTP、PING等）；
* 监控主机资源（处理器负荷、磁盘利用率等）；
* 简单地插件设计使得用户可以方便地扩展自己服务的检测方法；
* 并行服务检查机制；
* 具备定义网络分层结构的能力，用"parent"主机定义来表达网络主机间的关系，这种关系可被用来发现和明晰主机宕机或不可达状态；
* 当服务或主机问题产生与解决时将告警发送给联系人（通过EMail、短信、用户定义方式）；
* 可以定义一些处理程序，使之能够在服务或者主机发生故障时起到预防作用；
* 自动的日志滚动功能；
* 可以支持并实现对主机的冗余监控；
* 可选的WEB界面用于查看当前的网络状态、通知和故障历史、日志文件等；
* 可以通过手机查看系统监控信息；
* 可指定自定义的事件处理控制器；

## 三.NAGIOS的工作原理
Nagios的功能是监控服务和主机，但是他自身并不包括这部分功能，所有的监控、检测功能都是通过各种插件来完成的。
启动Nagios后，它会周期性的自动调用插件去检测服务器状态，同时Nagios会维持一个队列，所有插件返回来的状态信息都进入队列，Nagios每次都从队首开始读取信息，并进行处理后，把状态结果通过web显示出来。
Nagios提供了许多插件，利用这些插件可以方便的监控很多服务状态。安装完成后，在nagios主目录下的/libexec里放有nagios自带的可以使用的所有插件，如，check_disk是检查磁盘空间的插件，check_load是检查CPU负载的，等等。每一个插件可以通过运行./check_xxx –h 来查看其使用方法和功能。
Nagios可以识别4种状态返回信息，即 0(OK)表示状态正常/绿色、1(WARNING)表示出现警告/黄色、2(CRITICAL)表示出现非常严重的错误/红色、3(UNKNOWN)表示未知错误/深黄色。Nagios根据插件返回来的值，来判断监控对象的状态，并通过web显示出来，以供管理员及时发现故障。

`四种监控状态`

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/nagios/01.jpg?raw=true)

再说报警功能，如果监控系统发现问题不能报警那就没有意义了，所以报警也是nagios很重要的功能之一。但是，同样的，Nagios 自身也没有报警部分的代码，甚至没有插件，而是交给用户或者其他相关开源项目组去完成的。
Nagios 安装，是指基本平台，也就是Nagios软件包的安装。它是监控体系的框架，也是所有监控的基础。
打开Nagios官方的文档，会发现Nagios基本上没有什么依赖包，只要求系统是Linux或者其他Nagios支持的系统。不过如果你没有安装apache（http服务），那么你就没有那么直观的界面来查看监控信息了，所以apache姑且算是一个前提条件。关于apache的安装，网上有很多，照着安装就是了。安装之后要检查一下是否可以正常工作。
知道Nagios 是如何通过插件来管理服务器对象后，现在开始研究它是如何管理远端服务器对象的。Nagios 系统提供了一个插件NRPE。Nagios 通过周期性的运行它来获得远端服务器的各种状态信息。它们之间的关系如下图所示：

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/nagios/02.jpg?raw=true)

Nagios 通过NRPE 来远端管理服务
* 1. Nagios 执行安装在它里面的check_nrpe 插件，并告诉check_nrpe 去检测哪些服务。
* 2. 通过SSL，check_nrpe 连接远端机子上的NRPE daemon
* 3. NRPE 运行本地的各种插件去检测本地的服务和状态(check_disk,..etc)
* 4. 最后，NRPE 把检测的结果传给主机端的check_nrpe，check_nrpe 再把结果送到Nagios状态队列中。
* 5. Nagios 依次读取队列中的信息，再把结果显示出来。

## 四.NAGIOS服务端的安装部署
### 4.1安装依赖添加用户
```shell
yum install  httpd php php-gd gcc glibc glibc-common gd gd-devel libjpeg-devel libpng-devel pango* libart_lgpl-devel pango-devel* cairo-devel* libxml2-devel libjpeg-devel libpng-devel php-gd gd-devel perl-GD libtoul-ltdl-devel rrdtool-perl perl-devel perl-ExtUtils-Embed perl-Time-HiRes mysql openssl* rrdtool sysstat mailx
useradd nagios
groupadd nagcmd    
usermod -a -G nagcmd nagios
usermod -a -G nagcmd apache
```
### 4.2编译安装nagios
```shell
tar zxf nagios-3.5.1.tar
cd nagios-3.5.1
./configure --with-command-group=nagcmd
make all
make install
make install-init
make install-commandmode
make install-config
make install-webconf
cd ..
```
### 4.3安装nagios-plugins
```shell
tar zxf nagios-plugins-2.1.3.tar.gz
cd nagios-plugins-2.1.3
./configure --with-nagios-user=nagios --with-nagios-group=nagios --enable-perl-modules
make && make install
cd ..
```
`安装完成后会在/usr/local/nagios/libexec生成一些监控脚本`
```shell
root@template ~ 08:37:32 # ls /usr/local/nagios/libexec/ 
check_apt       check_dummy         check_ifstatus  check_mrtgtraf  check_ntp_time  check_rpc      check_tcp     process_perfdata.pl
check_breeze    check_file_age      check_imap      check_nagios    check_nwstat    check_sensors  check_time    urlize
check_by_ssh    check_flexlm        check_ircd      check_nntp      check_oracle    check_simap    check_udp     utils.pm
check_clamd     check_ftp           check_jabber    check_nntps     check_overcr    check_smtp     check_ups     utils.sh
check_cluster   check_http          check_load      check_nrpe      check_ping      check_spop     check_uptime
check_dhcp      check_icmp          check_log       check_nt        check_pop       check_ssh      check_users
check_disk      check_ide_smart     check_mailq     check_ntp       check_procs     check_ssmtp    check_wave
check_disk_smb  check_ifoperstatus  check_mrtg      check_ntp_peer  check_real      check_swap     negate
```
### 4.4安装nrpe
```shell
tar zxf cd nrpe-2.15.tar.gz
cd nrpe-2.15
./configure
make all
make install-plugin
make install-daemon
make install-daemon-config
cd ..
```
### 4.5安装pnp4
```shell
tar zxf pnp4nagios-0.6.6.tar.gz
cd pnp4nagios-0.6.6
./configure --prefix=/usr/local/pnp4nagios --with-nagios-user=nagios --with-nagios-group=nagcmd
make all && make install
make instal-webconf
make instal-config
make instal-init
cp contrib/ssi/status-header.ssi /usr/local/nagios/share/ssi/
cd /usr/local/pnp4nagios/etc
mv misccommands.cfg-sample  misccommands.cfg
mv nagios.cfg-sample  nagios.cfg
mv npcd.cfg-sample npcd.cfg
mv process_perfdata.cfg-sample  process_perfdata.cfg
mv rra.cfg-sample rra.cfg
cd /usr/local/pnp4nagios/etc/pages
mv web_traffic.cfg-sample web_traffic.cfg
cd ../check_commands
mv check_all_local_disks.cfg-sample  check_all_local_disks.cfg
mv check_nrpe.cfg-sample  check_nrpe.cfg
mv check_nwstat.cfg-sample  check_nwstat.cfg
cp /usr/local/pnp4nagios/libexec/process_perfdata.pl /usr/local/nagios/libexec/
chmod 755 /usr/local/nagios/libexec/process_perfdata.pl
chown -R nagios.nagios /usr/local/nagios/libexec/*
mv /usr/local/pnp4nagios/share/install.php /usr/local/pnp4nagios/share/install.php.bak
/etc/init.d/npcd restart
cat>>/etc/httpd/conf/httpd.conf<<EOF
<Directory "/usr/local/pnp4nagios/share">
    AllowOverride None
    Order allow,deny
    Allow from all
    AuthName "Nagios Access"
    AuthType Basic
    AuthUserFile /usr/local/nagios/etc/htpasswd.users
    Require valid-user
</Directory>
EOF
service httpd restart
cd ..
```
### 4.6替换配置文件
```shell
\cp nagios.cfg /usr/local/nagios/etc/
\cp commands.cfg /usr/local/nagios/etc/objects
\cp templates.cfg /usr/local/nagios/etc/objects
\cp hosts.cfg /usr/local/nagios/etc/objects
\cp services.cfg /usr/local/nagios/etc/objects
mkdir -p /usr/local/nagios/etc/objects/services
sed -i 's#nagiosadmin#nagios#g' /usr/local/nagios/etc/cgi.cfg
chown -R nagios.nagios /usr/local/nagios
```
### 4.7配置密码启动nagios
```shell
root@template /usr/local/pnp4nagios/etc 19:50:52 # htpasswd -cb /usr/local/nagios/etc/htpasswd.users nagios 7758521
Adding password for user nagios
root@template /usr/local/src 20:36:22 # /etc/init.d/nagios start 
Starting nagios: done.
```

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/nagios/03.jpg?raw=true)

