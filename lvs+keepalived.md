# Lvs+Keepalived高可用负载均衡
#### By：大官人
#### Email：DaGuanR@gmail.com
#### QQ：375739049
## 一.ARP协议
### 1.1什么是arp协议
地址解析协议，即ARP（Address Resolution Protocol），是根据IP地址获取物理地址的一个TCP/IP协议。主机发送信息时将包含目标IP地址的ARP请求广播到网络上的所有主机，并接收返回消息，以此确定目标的物理地址；收到返回消息后将该IP地址和物理地址存入本机ARP缓存中并保留一定时间，下次请求时直接查询ARP缓存以节约资源。地址解析协议是建立在网络中各个主机互相信任的基础上的，网络上的主机可以自主发送ARP应答消息，其他主机收到应答报文时不会检测该报文的真实性就会将其记入本机ARP缓存；由此攻击者就可以向某一主机发送伪ARP应答报文，使其发送的信息无法到达预期的主机或到达错误的主机，这就构成了一个ARP欺骗。ARP命令可用于查询本机ARP缓存中IP地址和MAC地址的对应关系、添加或删除静态对应关系等。相关协议有RARP、代理ARP。NDP用于在IPv6中代替地址解析协议。
### 1.2arp工作原理
* 主机A的IP地址为192.168.1.1，MAC地址为0A-11-22-33-44-01；
* 主机B的IP地址为192.168.1.2，MAC地址为0A-11-22-33-44-02；
* 当主机A要与主机B通信时，地址解析协议可以将主机B的IP地址（192.168.1.2）解析成主机B的MAC地址，以下为工作流程：
* 第1步：根据主机A上的路由表内容，IP确定用于访问主机B的转发IP地址是192.168.1.2。然后A主机在自己的本地ARP缓存中检查主机B的匹配MAC地址。
* 第2步：如果主机A在ARP缓存中没有找到映射，它将询问192.168.1.2的硬件地址，从而将ARP请求帧广播到本地网络上的所有主机。源主机A的IP地址和MAC地址都包括在ARP请求中。本地网络上的每台主机都接收到ARP请求并且检查是否与自己的IP地址匹配。如果主机发现请求的IP地址与自己的IP地址不匹配，它将丢弃ARP请求。
* 第3步：主机B确定ARP请求中的IP地址与自己的IP地址匹配，则将主机A的IP地址和MAC地址映射添加到本地ARP缓存中。
* 第4步：主机B将包含其MAC地址的ARP回复消息直接发送回主机A。
* 第5步：当主机A收到从主机B发来的ARP回复消息时，会用主机B的IP和MAC地址映射更新ARP缓存。本机缓存是有生存期的，生存期结束后，将再次重复上面的过程。主机B的MAC地址一旦确定，主机A就能向主机B发送IP通信了
### 1.3arp代理
C1和PC2虽然属于不同的广播域，但它们处于同一网段中，因此PC1会向PC2发出ARP请求广播包，请求获得PC2的MAC地址。由于路由器不会转发广播包，因此ARP请求只能到达路由器，不能到达PC2。当在路由器上启用ARP代理后，路由器会查看ARP请求，发现IP地址172.16.20.100属于它连接的另一个网络，因此路由器用自己的接口MAC地址代替PC2的MAC地址，向PC1发送了一个ARP应答。


## 二.LVS简介
### 2.1lvs概念
LVS集群采用IP负载均衡技术和基于内容请求分发技术。调度器具有很好的吞吐率，将请求均衡地转移到不同的服务器上执行，且调度器自动屏蔽掉服务器的故障，从而将一组服务器构成一个高性能的、高可用的虚拟服务器。整个服务器集群的结构对客户是透明的，而且无需修改客户端和服务器端的程序。为此，在设计时需要考虑系统的透明性、可伸缩性、高可用性和易管理性。
### 2.2lvs三种基本模型
#### 2.2.1lvs-DR模型是lvs的默认模型，也是企业中用到的最多的模型
![image](https://github.com/yangzinan/Operations/blob/master/iamge/lvs+keepalived/01.png?raw=true)

`LVS_NAT模型，通常应用与rs较少，rs节点无要求，端口转换的场景`

`解读：地址转换模型，vs通过修改目的ip将报文发送到rs.rs通过dip网关将报文发给vs,vs再将报文的源ip进行修改发送给客户端`

### 2.2.2LVS-TUN 模式
![image](https://github.com/yangzinan/Operations/blob/master/iamge/lvs+keepalived/02.png?raw=true)

`lvs-TUN模型可以运用于异地机房的负载调度上。`

`解读：隧道模型，跟DR模型比较相似，都是由rs直接回复给cs .跟dr模型不同的是，vs和rs之间可以存在路由，原因是tun模型在报文源ip和目的ip后又加入了一层源ip和目的ip的信息。`

### 2.2.3LVS-NAT 模式
![image](https://github.com/yangzinan/Operations/blob/master/iamge/lvs+keepalived/03.png?raw=true)

`LVS_NAT模型，通常应用与rs较少，rs节点无要求，端口转换的场景`

`解读：地址转换模型，vs通过修改目的ip将报文发送到rs.rs通过dip网关将报文发给vs,vs再将报文的源ip进行修改发送给客户端。`
## 三.LVS-DR模式详解
* 场景假设：
    * 客户端：ip：192.168.1.1，mac：00：00：00：00：00：01
    * Lvs：vip：192.168.0.2，mac：00：00：00：00：00：02
    * Web服务器：ip：192.168.0.3，mac：00：00：00：00：00：03，lo网卡ip：192.168.0.2
* Lvs工作流程流程：
    * 客户向lvs发送一个请求后lvs收到的报文为源IP地址为：192.168.0.1，源mac为：00：00：00：00：00：01。目标IP地址为：192.168.0.2，目标mac地址为：00：00：00：00：00：02
    * 当lvs收到请求后会根据自身算法选择一个web服务器对报文进行修改并转发给web服务器，此时转发的报文为IP地址为：192.168.0.1，源mac为：00：00：00：00：00：01，目标IP地址为：192.168.0.2，目标mac地址为：00：00：00：00：00：03
    * Web服务器收到lvs抓发过来的请求进行回复，此时web参看报文发现自己的lo网卡有目标地址那么进行恢复。回复的报文为源IP地址：192.168.0.2，源mac地址为：00：00：00：00：00：03，目标IP地址为：192.168.0.1，目标Imac、地址为：00：00：00：00：00：01
* 总结：
    * 整个lvs-的dr模式客户端的计算机可以理解为arp欺骗，是客户的计算机始终认为和同一台计算机通信。
## 四.LVS的安装
### 4.1升级内核并安装内核开发包开启内核转发
```shell
yum update -y kernel
yum install -y kernel-devel
reboot
ln -s /usr/src/kernels/`uname -r` /usr/src/linux
sed -i 's#net.ipv4.ip_forward = 0#net.ipv4.ip_forward = 1#g' /etc/sysctl.conf  
sysctl -p
```
### 4.2安装lvs
```shell
yum install popt-devel ipvsadm -y
root@template ~ 23:58:02 # lsmod | grep ip_vs
root@template ~ 23:58:14 # modprobe ip_vs
root@template ~ 23:58:23 # lsmod | grep ip_vs
ip_vs                 126897  0 
libcrc32c               1246  1 ip_vs
ipv6                  336282  282 ip_vs,ip6t_REJECT,nf_conntrack_ipv6,nf_defrag_ipv6
```
## 五.LVS应用案例 
### 5.1配置lvs主机
`ifconfig eth网卡号:编号 VIP netmask 255.255.255.0 up`

```shell
[root@lvs01 ~]# ifconfig eth0:12 192.168.241.11 netmask 255.255.255.0 up
[root@lvs01 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:cc:ec:94 brd ff:ff:ff:ff:ff:ff
    inet 192.168.241.12/24 brd 192.168.241.255 scope global eth0
    inet 192.168.241.11/24 brd 192.168.241.255 scope global secondary eth0:12
    inet6 fe80::20c:29ff:fecc:ec94/64 scope link 
       valid_lft forever preferred_lft forever
```
### 5.2添加转发VIP
`ipvsadm -A -t VIP:port -s wrr -p 20`

#-s 参数使用wrr轮询算法 -p会话保持20秒

```shell
[root@lvs01 ~]# ipvsadm -A -t 192.168.241.11:80 -s wrr -p 20
[root@lvs01 ~]#
```
### 5.3添加后端RSV（web）
`ipvsadm -a -t VIP:port -r RIP:port -g -w 1`

```shell
[root@lvs01 ~]# ipvsadm -a -t 192.168.241.11:80 -r 192.168.241.15:80 -g -w 1
[root@lvs01 ~]# ipvsadm -a -t 192.168.241.11:80 -r 192.168.241.16:80 -g -w 1
[root@lvs01 ~]#
```
### 5.4查看设置
```shell
[root@lvs01 ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.241.11:80 wrr persistent 20
  -> 192.168.241.15:80            Route   1      0          0         
  -> 192.168.241.16:80            Route   1      0          0         
[root@lvs01 ~]#
```
### 5.5配置后端rsv的lo网卡
`ifconfig lo:num VIP netmask 255.255.255.255 up`

```shell
[root@apache ~]# ifconfig lo:0 192.168.241.11 netmask 255.255.255.255 up      
[root@apache ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 16436 qdisc noqueue state UNKNOWN 
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
    inet 192.168.241.11/32 brd 192.168.241.11 scope global lo:0
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 00:0c:29:d9:fc:b8 brd ff:ff:ff:ff:ff:ff
    inet 192.168.241.15/24 brd 192.168.241.255 scope global eth0
    inet6 fe80::20c:29ff:fed9:fcb8/64 scope link 
       valid_lft forever preferred_lft forever
```
### 5.6抑制rsv的arp
```shell
echo "1" > /proc/sys/net/ipv4/conf/all/arp_ignore
echo "1" > /proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" > /proc/sys/net/ipv4/conf/lo/arp_announce
echo "2" > /proc/sys/net/ipv4/conf/all/arp_announce
```
### 5.7用curl命令验证
![image](https://github.com/yangzinan/Operations/blob/master/iamge/lvs+keepalived/04.png?raw=true)
![image](https://github.com/yangzinan/Operations/blob/master/iamge/lvs+keepalived/05.png?raw=true)










