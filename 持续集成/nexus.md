# NEXUS
## 1.安装JDk
```shell
tar zxf jdk-8u71-linux-x64.tar.gz
mv jdk1.8.0_71 /usr/local
ln -s /usr/local/jdk1.8.0_71 /usr/local/jdk
cat >>/etc/profile<<EOF
export JAVA_HOME=/usr/local/jdk/
export PATH=\$JAVA_HOME/bin:$PATH 
export CLASSPATH=.:\$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar 
EOF
source /etc/profile
[root@node1 src]# java -version          
java version "1.8.0_71"
Java(TM) SE Runtime Environment (build 1.8.0_71-b15)
Java HotSpot(TM) 64-Bit Server VM (build 25.71-b15, mixed mode)
```
## 2.安装
```shell
mkdir /usr/local/nexus
tar zxf nexus-2.11.2-03-bundle.tar.gz -C /usr/local/nexus/
```
![](https://user-images.githubusercontent.com/12201941/28052536-495b0906-663e-11e7-9d39-291c1d4c66d0.png)
## 3.配置
### 3.1配置(/usr/local/nexus/nexus-2.11.2-03/conf/nexus.properties)
```conf
application-port=8081 
application-host=0.0.0.0 
nexus-webapp=${bundleBasedir}/nexus 
nexus-webapp-context-path=/nexus# Nexus section 
nexus-work=${bundleBasedir}/../sonatype-work/nexus 
runtime=${bundleBasedir}/nexus/WEB-INF
```
**此文件基本不用修改**
### 3.2配置(/usr/local/nexus/nexus-2.11.2-03/bin/nexus)
```conf
RUN_AS_USER=nexus
```
## 4.启动
```shell
useradd -M -s /sbin/nologin nexus
chown -R nexus:nexus /usr/local/nexus
/usr/local/nexus/nexus-2.11.2-03/bin/nexus start
```
## 5.仓库介绍
![](https://user-images.githubusercontent.com/12201941/28052872-06db73fc-6640-11e7-9586-c72f71ef8fec.png)
### 5.1仓库种类
* hosted：主要用于内部项目的发布仓库或第三方的项目构件
* proxy：公共远程仓库的代理
* virtual：虚拟仓库
* 
**一般用到的仓库种类是 hosted、proxy**

### 5.2Hosted说明

* releases 内部的模块中 release 模块的发布仓库
* snapshots 发布内部的 SNAPSHOT 模块的仓库
* 3rd party 第三方依赖的仓库，这个数据通常是由内部人员自行下载之后发布上去(如oracle的JDBC)