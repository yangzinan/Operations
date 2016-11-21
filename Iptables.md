# Iptables 
#### By：大官人
#### Email：DaGuanR@gmail.com
#### QQ：375739049
## 一.IPTABLES简介
iptables 是与最新的 3.5 版本 Linux 内核集成的 IP 信息包过滤系统。如果 Linux 系统连接到因特网或 LAN、服务器或连接 LAN 和因特网的代理服务器， 则该系统有利于在 Linux 系统上更好地控制 IP 信息包过滤和防火墙配置。
防火墙在做信息包过滤决定时，有一套遵循和组成的规则，这些规则存储在专用的信 息包过滤表中，而这些表集成在 Linux 内核中。在信息包过滤表中，规则被分组放在我们所谓的链（chain）中。而netfilter/iptables IP 信息包过滤系统是一款功能强大的工具，可用于添加、编辑和移除规则。
虽然 netfilter/iptables IP 信息包过滤系统被称为单个实体，但它实际上由两个组件netfilter 和 iptables 组成。
netfilter 组件也称为内核空间（kernelspace），是内核的一部分，由一些信息包过滤表组成，这些表包含内核用来控制信息包过滤处理的规则集。
iptables 组件是一种工具，也称为用户空间（userspace），它使插入、修改和除去信息包过滤表中的规则变得容易。除非您正在使用 Red Hat Linux 7.1 或更高版本，否则需要下载该工具并安装使用它。
## 二.IPTABLES的组成
### 2.1表
#### 2.1.1filter表
Filter表是控制所有流入和流出主机的数据包
#### 2.1.2nat表
Nat表是地址转换表，负责来源与目的的ip和port的转换，一般用于多人共享上网和地址转换使用
#### 2.1.3 mangle和raw表
仅做了解
### 2.2链
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/01.png?raw=true)
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/02.png?raw=true)
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/03.png?raw=true)
## 三.IPTABLES的工作流程
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/04.png?raw=true)

`简化版本`

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/05.png?raw=true)
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/06.png?raw=true)

`注：filter链的默认规则为拒绝所有`
## 四.IPTABLES FILTER表的操作
### 4.1实例允许192.168.44/24网段访问80
`iptables -I  INPUT-s 192.168.44.0/24 -p tcp --dport 80 -j ACCEPT`
```shell
root@template /var/www/html 16:54:48 # iptables -I INPUT -s 192.168.44.0/24 -p tcp --dport 80 -j ACCEPT  
root@template /var/www/html 16:55:01 # iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.44.0/24      0.0.0.0/0           tcp dpt:80 
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
root@template /var/www/html 16:55:04 #
```
`注：此时我们重启iptables刚才的设置将会丢失。必须要执行service iptables save`
```shell
root@template /var/www/html 16:55:04 # /etc/init.d/iptables restart
iptables: Setting chains to policy ACCEPT: nat filter      [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
root@template /var/www/html 16:57:59 # iptables -L -n              
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
root@template /var/www/html 16:58:01 #
```
`执行save命令后`
```shell
root@template /var/www/html 16:58:01 # iptables -I INPUT -s 192.168.44.0/24 -p tcp --dport 80 -j ACCEPT
root@template /var/www/html 16:58:35 # /etc/init.d/iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
root@template /var/www/html 16:58:47 # iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.44.0/24      0.0.0.0/0           tcp dpt:80 
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
root@template /var/www/html 16:58:49 # /etc/init.d/iptables restart
iptables: Setting chains to policy ACCEPT: nat filter      [  OK  ]
iptables: Flushing firewall rules:                         [  OK  ]
iptables: Unloading modules:                               [  OK  ]
iptables: Applying firewall rules:                         [  OK  ]
root@template /var/www/html 16:58:53 # iptables -L -n              
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  192.168.44.0/24      0.0.0.0/0           tcp dpt:80 
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           state RELATED,ESTABLISHED 
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0           state NEW tcp dpt:22 
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0           reject-with icmp-host-prohibited 

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
```
### 4.2filter表操作参数详解
> * -s 源地址
> * -d 目的地址
> * -i 流入网卡
> * -o 流出网卡
> * -p 协议（tcp，udp，icmp，all默认不加-p参数即为all）
> * --dport 目的端口（port:port可以用分号制定端口范围）
> * --sport 源端口（port:port可以用分号制定端口范围）
> * -m multiport 多端口 （-m mutilport --dport 21,22,65,89）
> * -j动作（DROP,ACCEPT）
### 4.3一般通用安全配置
```shell
iptables -F
iptabels -X
iptables -Z
iptables -I INPUT -s [安全网段] -p tcp --dport [ssh端口] -j ACCEPT
iptables -I INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD DROP
```


## 五.IPTABLES NAT表的操作
### 5.1DNAT功能
#### 5.1.1DNAT介绍
DNAT 的全称为Destination Network Address Translation目的地址转换，常用于防火墙中。
DNAT：目的地址转换的作用是将一组本地内部的地址映射到一组全球地址。通常来说，合法地址的数量比起本地内部的地址数量来要少得多。RFC1918中的地址保留可以用地址重叠的方式来达到。当一个内部主机第一次放出的数据包通过防火墙时，动态NAT的实现方式与静态NAT相同，然后这次NAT就以表的形式保留在防火墙中。除非由于某种个原因会引起这次NAT的结束，否则这次NAT就一直保留在防火墙中。引起NAT结束最常见的原因就是发出连接的主机在预定的时间内一直没有响应，这时空闲计时器就会从表中删除该主机的NAT。


#### 5.1.2生产实例
`例如：www.baidu.com的ip为220.181.112.244我们要把所有访问这个ip地址且为80端口的请求转到192.168.1.10这个内网地址`

`iptables -t nat -I PREROUTING -d 220.181.112.244 -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80`
### 5.2SNAT功能
#### 5.2.1SNAT介绍
`SNAT，是源地址转换，其作用是将ip数据包的源地址转换成另外一个地址。`
#### 5.2.2生产实例
`例如：办公网上网路由功能将192.168.1.0/24这个网段转换为外网地址220.18.11.23这个外网地址进行上网`

`iptables -t nat -I POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 220.18.11.23`

