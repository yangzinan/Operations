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
### 2.2.链
![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/iptables/01.png?raw=true)

