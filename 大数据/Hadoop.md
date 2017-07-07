# Hadoop安装文档
#### By：大官人

#### Email：DaGuanR@gmail.com

#### QQ:375739049
## 一·伪分布式
### 1.1环境准备
#### 1.1.1关闭iptables和selinux
```shell
service iptables stop
chkconfig iptables off
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disable#g' /etc/selinux/config
```
#### 1.1.2安装JDK
```shell
tar zxf jdk-8u71-linux-x64.tar.gz
mv jdk1.8.0_71 /usr/local/jdk1.8.0_71
ln -s /usr/local/jdk1.8.0_71 /usr/local/jdk
cat >>/etc/profile<<EOF
export JAVA_HOME=/usr/local/jdk
export PATH=/usr/local/jdk/bin:/bin:$PATH
export CLASSPATH=.:/usr/local/jdk/lib/dt.jar:/usr/local/jdk/lib/tools.jar
EOF
source /etc/profile
```
### 1.2安装hadoop
#### 1.2.1解压
```shell
tar zxf hadoop-2.7.3.tar.gz -C /usr/local
ln -s /usr/local/hadoop-2.7.3 /usr/local/hadoop
```
#### 1.2.2配置JAVA_HOME
```shell
sed -i "s#export JAVA_HOME=\${JAVA_HOME}#export JAVA_HOME=$JAVA_HOME#g" /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```
#### 1.2.3 修改vim core-site.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!--添加hdfs的namenode-->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://localhost:9000</value>
	</property>

	<!--配置hadoop的运行时产生文件的存放目录-->
	<property>
  		<name>hadoop.tmp.dir</name>
 		<value>/usr/local/hadoop/tmp</value>
	</property>
</configuration>
```
#### 1.2.4修改hdfs-site.xml
```xml
 <?xml version="1.0" encoding="UTF-8"?>
 <?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
 <!--
   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at
 
     http://www.apache.org/licenses/LICENSE-2.0
 
   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License. See accompanying LICENSE file.
 -->
 
 <!-- Put site-specific property overrides in this file. -->
 
 <configuration>
	<!--配置hdfs的副本数量-->
	<property>
		<name>dfs.replication</name>
 		<value>1</value>
	</property>
 </configuration>
```
#### 1.2.5配置mapred-site.xml
* 使用模板创建一个mapred-site.xml

```shell
cp mapred-site.xml.template mapred-site.xml
```

* 编辑配置mapred-site.xml

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!--告诉hadoop以后MR运行在yarn上-->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```
#### 1.2.6配置yarn-site.xml 
```xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>

<!-- Site specific YARN configuration properties -->
	<!--表示MR applicatons所使用的shuffle工具类-->
	<property>
		<name>yarn.nodemanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>

	<!--用来指定ResourceManager主机地址-->
	<property>
		<name>yarn.resourcemanager.hostname</value>
		<value>0.0.0.0</value>
	</property>
</configuration>
```
#### 1.2.7配置环境变量
```shell
cat >>/etc/profile<<EOF
export HADOOP_HOME=/usr/local/hadoop
PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:$PATH
EOF
source /etc/profile
```
#### 1.2.8格式化文件系统（hdfs）
```shell
root@localhost /usr/local/hadoop 21:49:38 # hdfs namenode -format
16/11/28 21:49:44 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = localhost/127.0.0.1
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.3
STARTUP_MSG:   classpath = /usr/local/hadoop-2.7.3/etc/hadoop:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpcore-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsch-0.1.42.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsp-api-2.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-client-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-httpclient-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/mockito-all-1.8.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-configuration-1.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jets3t-0.9.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpclient-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-api-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-digester-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/gson-2.2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-auth-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-framework-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-math3-3.1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-net-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-all-4.0.23.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-client-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-sharedcachemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-api-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-registry-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-tests-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-client-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.7.3.jar:/usr/local/hadoop/contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff; compiled by 'root' on 2016-08-18T01:41Z
STARTUP_MSG:   java = 1.8.0_71
************************************************************/
16/11/28 21:49:44 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
16/11/28 21:49:44 INFO namenode.NameNode: createNameNode [-format]
Formatting using clusterid: CID-e13c760e-f908-4f67-aae2-84ea36afe690
16/11/28 21:49:45 INFO namenode.FSNamesystem: No KeyProvider found.
16/11/28 21:49:45 INFO namenode.FSNamesystem: fsLock is fair:true
16/11/28 21:49:45 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
16/11/28 21:49:45 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
16/11/28 21:49:45 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
16/11/28 21:49:46 INFO blockmanagement.BlockManager: The block deletion will start around 2016 Nov 28 21:49:46
16/11/28 21:49:46 INFO util.GSet: Computing capacity for map BlocksMap
16/11/28 21:49:46 INFO util.GSet: VM type       = 64-bit
16/11/28 21:49:46 INFO util.GSet: 2.0% max memory 966.7 MB = 19.3 MB
16/11/28 21:49:46 INFO util.GSet: capacity      = 2^21 = 2097152 entries
16/11/28 21:49:46 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
16/11/28 21:49:46 INFO blockmanagement.BlockManager: defaultReplication         = 1
16/11/28 21:49:46 INFO blockmanagement.BlockManager: maxReplication             = 512
16/11/28 21:49:46 INFO blockmanagement.BlockManager: minReplication             = 1
16/11/28 21:49:46 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
16/11/28 21:49:46 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
16/11/28 21:49:46 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
16/11/28 21:49:46 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
16/11/28 21:49:46 INFO namenode.FSNamesystem: fsOwner             = root (auth:SIMPLE)
16/11/28 21:49:46 INFO namenode.FSNamesystem: supergroup          = supergroup
16/11/28 21:49:46 INFO namenode.FSNamesystem: isPermissionEnabled = true
16/11/28 21:49:46 INFO namenode.FSNamesystem: HA Enabled: false
16/11/28 21:49:46 INFO namenode.FSNamesystem: Append Enabled: true
16/11/28 21:49:46 INFO util.GSet: Computing capacity for map INodeMap
16/11/28 21:49:46 INFO util.GSet: VM type       = 64-bit
16/11/28 21:49:46 INFO util.GSet: 1.0% max memory 966.7 MB = 9.7 MB
16/11/28 21:49:46 INFO util.GSet: capacity      = 2^20 = 1048576 entries
16/11/28 21:49:46 INFO namenode.FSDirectory: ACLs enabled? false
16/11/28 21:49:46 INFO namenode.FSDirectory: XAttrs enabled? true
16/11/28 21:49:46 INFO namenode.FSDirectory: Maximum size of an xattr: 16384
16/11/28 21:49:46 INFO namenode.NameNode: Caching file names occuring more than 10 times
16/11/28 21:49:46 INFO util.GSet: Computing capacity for map cachedBlocks
16/11/28 21:49:46 INFO util.GSet: VM type       = 64-bit
16/11/28 21:49:46 INFO util.GSet: 0.25% max memory 966.7 MB = 2.4 MB
16/11/28 21:49:46 INFO util.GSet: capacity      = 2^18 = 262144 entries
16/11/28 21:49:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
16/11/28 21:49:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
16/11/28 21:49:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
16/11/28 21:49:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
16/11/28 21:49:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
16/11/28 21:49:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
16/11/28 21:49:46 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
16/11/28 21:49:46 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
16/11/28 21:49:46 INFO util.GSet: Computing capacity for map NameNodeRetryCache
16/11/28 21:49:46 INFO util.GSet: VM type       = 64-bit
16/11/28 21:49:46 INFO util.GSet: 0.029999999329447746% max memory 966.7 MB = 297.0 KB
16/11/28 21:49:46 INFO util.GSet: capacity      = 2^15 = 32768 entries
16/11/28 21:49:46 INFO namenode.FSImage: Allocated new BlockPoolId: BP-449643598-127.0.0.1-1480340986462
16/11/28 21:49:46 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name has been successfully formatted.
16/11/28 21:49:46 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
16/11/28 21:49:46 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 351 bytes saved in 0 seconds.
16/11/28 21:49:46 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
16/11/28 21:49:46 INFO util.ExitUtil: Exiting with status 0
16/11/28 21:49:46 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at localhost/127.0.0.1
************************************************************/
```
#### 1.2.9启动hadoop
* 配置免密钥登录

```shell
root@localhost ~/.ssh 21:55:07 # ssh-keygen -t rsa 
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
f6:6d:f2:19:de:0d:2e:36:2e:2b:3d:65:2e:11:71:80 root@localhost
The key's randomart image is:
+--[ RSA 2048]----+
|         ...     |
|        E . .    |
|           o     |
|          .      |
|        S  .     |
|       . ...o    |
|         .o=+ .  |
|        . ==== o |
|         ..B=oo .|
+-----------------+
root@localhost ~/.ssh 22:00:36 # ssh-copy-id -i ~/.ssh/id_rsa.pub root@localhost
The authenticity of host 'localhost (::1)' can't be established.
RSA key fingerprint is 57:22:33:d6:43:40:31:84:88:19:5d:f5:fc:7d:4e:ce.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'localhost' (RSA) to the list of known hosts.
Now try logging into the machine, with "ssh 'root@localhost'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.
```

* 启动hadoop

```shell
root@localhost /usr/local/hadoop/etc/hadoop 22:11:15 # start-all.sh 
This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
Starting namenodes on [localhost]
localhost: starting namenode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-namenode-localhost.out
localhost: starting datanode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-datanode-localhost.out
Starting secondary namenodes [0.0.0.0]
0.0.0.0: starting secondarynamenode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-secondarynamenode-localhost.out
starting yarn daemons
starting resourcemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-root-resourcemanager-localhost.out
localhost: starting nodemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-root-nodemanager-localhost.out
```
### 1.3.验证
#### 1.3.1通过JAVA进程验证
```shell
root@localhost /usr/local/hadoop/etc/hadoop 22:13:39 # jps
6689 NodeManager
6898 Jps
6439 SecondaryNameNode
6279 DataNode
6155 NameNode
6591 ResourceManager
root@localhost /usr/local/hadoop/etc/hadoop 22:13:44 # 
```
### 1.3.2端口验证
```shell
root@localhost /usr/local/hadoop/etc/hadoop 22:16:40 # netstat -ntlup | grep java
tcp        0      0 0.0.0.0:50020               0.0.0.0:*                   LISTEN      6279/java           
tcp        0      0 127.0.0.1:9000              0.0.0.0:*                   LISTEN      6155/java           
tcp        0      0 0.0.0.0:50090               0.0.0.0:*                   LISTEN      6439/java           
tcp        0      0 127.0.0.1:40368             0.0.0.0:*                   LISTEN      6279/java           
tcp        0      0 0.0.0.0:50070               0.0.0.0:*                   LISTEN      6155/java           
tcp        0      0 0.0.0.0:50010               0.0.0.0:*                   LISTEN      6279/java           
tcp        0      0 0.0.0.0:50075               0.0.0.0:*                   LISTEN      6279/java           
tcp        0      0 ::ffff:127.0.0.1:8030       :::*                        LISTEN      6591/java           
tcp        0      0 ::ffff:127.0.0.1:8031       :::*                        LISTEN      6591/java           
tcp        0      0 ::ffff:127.0.0.1:8032       :::*                        LISTEN      6591/java           
tcp        0      0 ::ffff:127.0.0.1:8033       :::*                        LISTEN      6591/java           
tcp        0      0 :::37732                    :::*                        LISTEN      6689/java           
tcp        0      0 :::8040                     :::*                        LISTEN      6689/java           
tcp        0      0 :::8042                     :::*                        LISTEN      6689/java           
tcp        0      0 ::ffff:127.0.0.1:8088       :::*                        LISTEN      6591/java           
```
### 1.3.3浏览器验证

* yarn管理界面

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/01.png?raw=true)

* hdfs管理界面

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/02.png?raw=true)

## 二.完全分布式

### 2.1实验部署环境

|     主机名    | IP   |  安装的软件  |  运行的进程  |
| --------   | :-----:  | :----:  | ----- |
|node1|192.168.241.10|jdk、hadoop|NameNode、DFSZKFailoverController(zkfc)|
|node2|192.168.241.20|jdk、hadoop|NameNode、DFSZKFailoverController(zkfc)|
|node3|192.168.241.30|jdk、hadoop|ResourceManager|
|node4|192.168.241.40|jdk、hadoop|ResourceManager|
|node5|192.168.241.50|jdk、hadoop、zookeeper|DataNode、NodeManager、JournalNode、QuorumPeerMain|
|node6|192.168.241.60|jdk、hadoop、zookeeper	|DataNode、NodeManager、JournalNode、QuorumPeerMain
|node7|192.168.241.70|jdk、hadoop、zookeeper|DataNode、NodeManager、JournalNode、QuorumPeerMain|

### 2.2配置基本环境
#### 2.2.1配置host
```conf
 192.168.241.10	node1
 192.168.241.20	node2
 192.168.241.30	node3
 192.168.241.40	node4
 192.168.241.50	node5
 192.168.241.60	node6
 192.168.241.70 node7
```
#### 2.2.2在node1上配置免密钥
```shell
[root@node1 ~]# ssh-key
ssh-keygen   ssh-keyscan  
[root@node1 ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
e9:90:fc:48:86:22:e1:6e:75:92:3a:aa:b2:a1:9b:58 root@node1
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|.                |
|..  .o . .       |
|...+..* S        |
|..o.oo =         |
|.=E   . o        |
|*+.              |
|@o               |
+-----------------+
```
#### 2.2.3 将公钥发送到其他节点
```shell
[root@node1 ~]# ssh-copy-id node1         
The authenticity of host 'node1 (192.168.241.10)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node1,192.168.241.10' (RSA) to the list of known hosts.
root@node1's password: 
Now try logging into the machine, with "ssh 'node1'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node2
The authenticity of host 'node2 (192.168.241.20)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node2,192.168.241.20' (RSA) to the list of known hosts.
root@node2's password: 
Now try logging into the machine, with "ssh 'node2'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node3
The authenticity of host 'node3 (192.168.241.30)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node3,192.168.241.30' (RSA) to the list of known hosts.
root@node3's password: 
Now try logging into the machine, with "ssh 'node3'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node4
The authenticity of host 'node4 (192.168.241.40)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node4,192.168.241.40' (RSA) to the list of known hosts.
root@node4's password: 
Now try logging into the machine, with "ssh 'node4'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node5
The authenticity of host 'node5 (192.168.241.50)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node5,192.168.241.50' (RSA) to the list of known hosts.
root@node5's password: 
Now try logging into the machine, with "ssh 'node5'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node6
The authenticity of host 'node6 (192.168.241.60)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node6,192.168.241.60' (RSA) to the list of known hosts.
root@node6's password: 
Now try logging into the machine, with "ssh 'node6'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# ssh-copy-id node7
The authenticity of host 'node7 (192.168.241.70)' can't be established.
RSA key fingerprint is 43:51:ed:ef:d7:b2:12:67:ab:2e:e8:0a:15:1b:36:d4.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'node7,192.168.241.70' (RSA) to the list of known hosts.
root@node7's password: 
Now try logging into the machine, with "ssh 'node7'", and check in:

  .ssh/authorized_keys

to make sure we haven't added extra keys that you weren't expecting.

[root@node1 ~]# 
```
#### 2.2.4将hosts文件分发到其他节点
```shell
[root@node1 ~]# scp /etc/hosts root@node2:/etc/
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# scp /etc/hosts root@node3:/etc/ 
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# scp /etc/hosts root@node4:/etc/ 
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# scp /etc/hosts root@node5:/etc/ 
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# scp /etc/hosts root@node6:/etc/ 
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# scp /etc/hosts root@node7:/etc/ 
hosts                                                                                                          100%  314     0.3KB/s   00:00    
[root@node1 ~]# 
```

#### 2.2.5安装JDK
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
### 2.3安装zookeeper集群
#### 2.3.1安装zookeeper（在node5上执行）
```shell
tar zxf zookeeper-3.4.5.tar.gz -C /usr/local/
ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper
echo export PATH=$PATH:/usr/local/zookeeper/bin >> /etc/profile
source /etc/profile
```
#### 2.3.2配置zookeeper（在node5上执行）
```conf
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.1=node5:2888:3888
server.2=node6:2888:3888
server.3=node7:2888:3888
```
#### 2.3.3创建zookeeper数据目录（在node5.6.7上分别执行）
```shell
mkdir -p /data/zookeeper
echo $num > /data/zookeeper/myid #num是server.后面的数字每个主机不同
```
#### 2.3.4将配置好的zookeeper发送到node6.7上（在node5上执行）
```shell
scp -r /usr/local/zookeeper-3.4.5 root@node6:/usr/local/
scp -r /usr/local/zookeeper-3.4.5 root@node7:/usr/local/  
```
#### 2.3.5配置其他节点环境变量（在node6.7上执行）
```shell
ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper
echo export PATH=$PATH:/usr/local/zookeeper/bin >> /etc/profile
source /etc/profile
```

### 2.4安装hadoop(在node1上执行)

```shell
tar zxf hadoop-2.7.3.tar.gz
mv hadoop-2.7.3 /usr/local/ 
ln -s /usr/local/hadoop-2.7.3 /usr/local/hadoop 
cat >>/etc/profile<<EOF
export HADOOP_HOME=/usr/local/hadoop/
export PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:$PATH
EOF
source /etc/profile
```
#### 2.4.1配置hadoop-env.sh
```shell
sed -i "s#export JAVA_HOME=\${JAVA_HOME}#export JAVA_HOME=$JAVA_HOME#g" /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```

#### 2.4.2配置修改core-site.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!-- 指定hdfs的nameservice为ns1 -->
	<property>
		<name>fs.defaultFS</name>
		<value>hdfs://ns1</value>
	</property>
	<!-- 指定hadoop临时目录 -->
	<property>
		<name>hadoop.tmp.dir</name>
		<value>/usr/local/hadoop/tmp</value>
	</property>
	<!-- 指定zookeeper地址 -->
	<property>
		<name>ha.zookeeper.quorum</name>
		<value>node5:2181,node6:2181,node7:2181</value>
	</property>
</configuration>
```

#### 2.4.3配置hdfs-site.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!--指定hdfs的nameservice为ns1，需要和core-site.xml中的保持一致 -->
	<property>
		<name>dfs.nameservices</name>
		<value>ns1</value>
	</property>
	<!-- ns1下面有两个NameNode，分别是nn1，nn2 -->
	<property>
		<name>dfs.ha.namenodes.ns1</name>
		<value>nn1,nn2</value>
	</property>
	<!-- nn1的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.ns1.nn1</name>
		<value>node1:9000</value>
	</property>
	<!-- nn1的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.ns1.nn1</name>
		<value>node1:50070</value>
	</property>
	<!-- nn2的RPC通信地址 -->
	<property>
		<name>dfs.namenode.rpc-address.ns1.nn2</name>
		<value>node2:9000</value>
	</property>
	<!-- nn2的http通信地址 -->
	<property>
		<name>dfs.namenode.http-address.ns1.nn2</name>
		<value>node2:50070</value>
	</property>
	<!-- 指定NameNode的元数据在JournalNode上的存放位置 -->
	<property>
		<name>dfs.namenode.shared.edits.dir</name>
		<value>qjournal://node5:8485;node6:8485;node7:8485/ns1</value>
	</property>
	<!-- 制定datanode数据的存储位置 -->
	<property>
		<name>dfs.datanode.data.dir</name>
		<value>/data/hadoop</value>
	</property>
	<!-- 指定JournalNode在本地磁盘存放数据的位置 -->
	<property>
		<name>dfs.journalnode.edits.dir</name>
		<value>/usr/local/hadoop/journal</value>
	</property>
	<!-- 开启NameNode失败自动切换 -->
	<property>
		<name>dfs.ha.automatic-failover.enabled</name>
		<value>true</value>
	</property>
	<!-- 配置失败自动切换实现方式 -->
	<property>
		<name>dfs.client.failover.proxy.provider.ns1</name>
		<value>org.apache.hadoop.hdfs.server.namenode.ha.ConfiguredFailoverProxyProvider</value>
	</property>
	<!-- 配置隔离机制方法，多个机制用换行分割，即每个机制暂用一行-->
	<property>
		<name>dfs.ha.fencing.methods</name>
		<value>
			sshfence
			shell(/bin/true)
		</value>
	</property>
	<!-- 使用sshfence隔离机制时需要ssh免登陆 -->
	<property>
		<name>dfs.ha.fencing.ssh.private-key-files</name>
		<value>/home/hadoop/.ssh/id_rsa</value>
	</property>
	<!-- 配置sshfence隔离机制超时时间 -->
	<property>
		<name>dfs.ha.fencing.ssh.connect-timeout</name>
		<value>30000</value>
	</property>
</configuration>
```
#### 2.4.4配置mapred-site.xml
* 首先拷贝一份
```shell
cp mapred-site.xml.template mapred-site.xml
```

```xml
<?xml version="1.0"?>
<?xml-stylesheet type="text/xsl" href="configuration.xsl"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->

<!-- Put site-specific property overrides in this file. -->

<configuration>
	<!-- 指定mr框架为yarn方式 -->
	<property>
		<name>mapreduce.framework.name</name>
		<value>yarn</value>
	</property>
</configuration>
```

#### 2.4.5配置yarn-site.xml
```xml
<?xml version="1.0"?>
<!--
  Licensed under the Apache License, Version 2.0 (the "License");
  you may not use this file except in compliance with the License.
  You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

  Unless required by applicable law or agreed to in writing, software
  distributed under the License is distributed on an "AS IS" BASIS,
  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
  See the License for the specific language governing permissions and
  limitations under the License. See accompanying LICENSE file.
-->
<configuration>
	<!-- 开启RM高可靠 -->
	<property>
	   <name>yarn.resourcemanager.ha.enabled</name>
	   <value>true</value>
	</property>
	<!-- 指定RM的cluster id -->
	<property>
	   <name>yarn.resourcemanager.cluster-id</name>
	   <value>yrc</value>
	</property>
	<!-- 指定RM的名字 -->
	<property>
	   <name>yarn.resourcemanager.ha.rm-ids</name>
	   <value>rm1,rm2</value>
	</property>
	<!-- 分别指定RM的地址 -->
	<property>
	   <name>yarn.resourcemanager.hostname.rm1</name>
	   <value>node3</value>
	</property>
	<property>
	   <name>yarn.resourcemanager.hostname.rm2</name>
	   <value>node4</value>
	</property>
	<!-- 配置自动切换 -->
	<property>
		<name>yarn.resourcemanager.recovery.enabled</name>
		<value>true</value>
	</property>
	<!-- 配置使用zk -->
	<property>
		<name>yarn.resourcemanager.store.class</name>
		<value>org.apache.hadoop.yarn.server.resourcemanager.recovery.ZKRMStateStore</value>
	</property>
	<!-- 指定zk集群地址 -->
	<property>
	   <name>yarn.resourcemanager.zk-address</name>
	   <value>node5:2181,node6:2181,node7:2181</value>
	</property>
	<property>
	   <name>yarn.nodemanager.aux-services</name>
	   <value>mapreduce_shuffle</value>
	</property>
</configuration>
```

#### 2.4.6修改slaves
```conf
node5
node6
node7
```
#### 2.4.7将配置好的hadoop拷贝到其他节点(在node1上执行)
```shell
for ((i=2;i<=7;i++)); do
  scp -r /usr/local/hadoop-2.7.3 root@node$i:/usr/local/
done
```
#### 2.4.8配置其他节点的环境变量
```shell
ln -s /usr/local/hadoop-2.7.3 /usr/local/hadoop 
cat >>/etc/profile<<EOF
export HADOOP_HOME=/usr/local/hadoop/
export PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:$PATH
EOF
source /etc/profile
```
### 2.5启动Hadoop
#### 2.5.1启动zookeeper集群(在node5.6.7)
```shell
[root@node5 src]# zkServer.sh start  
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node5 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
#############################
[root@node6 src]# zkServer.sh start  
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node6 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader
#############################
[root@node7 src]# zkServer.sh start  
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@node7 zookeeper]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
```
	现在node6是leader节点
#### 2.5.2启动journalnode（分别在在node05、node6、node7上执行）
```shell
[root@node5 hadoop]# hadoop-daemon.sh start journalnode
starting journalnode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-journalnode-node5.out
[root@node5 hadoop]# jps
2580 QuorumPeerMain
3134 JournalNode
3183 Jps
#########################################
[root@naode6 src]# hadoop-daemon.sh start journalnode
starting journalnode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-journalnode-naode6.out
[root@naode6 src]# jps
1745 QuorumPeerMain
2002 JournalNode
2037 Jps
########################################
[root@node7 src]# hadoop-daemon.sh start journalnode
starting journalnode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-journalnode-node7.out
[root@node7 src]# jps
1878 JournalNode
1927 Jps
1720 QuorumPeerMain
```
#### 2.5.3 格式化HDFS(/在node1上执行)
```shell
[root@node1 hadoop]# hdfs namenode -format                                
16/12/01 21:31:44 INFO namenode.NameNode: STARTUP_MSG: 
/************************************************************
STARTUP_MSG: Starting NameNode
STARTUP_MSG:   host = node1/192.168.241.10
STARTUP_MSG:   args = [-format]
STARTUP_MSG:   version = 2.7.3
STARTUP_MSG:   classpath = /usr/local/hadoop-2.7.3/etc/hadoop:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpclient-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/mockito-all-1.8.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-math3-3.1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-digester-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-httpclient-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsp-api-2.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jets3t-0.9.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-api-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-framework-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpcore-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsch-0.1.42.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-net-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-configuration-1.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/gson-2.2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-auth-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-client-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-all-4.0.23.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-client-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-api-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-sharedcachemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-registry-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-client-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-tests-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.7.3.jar:/usr/local/hadoop//contrib/capacity-scheduler/*.jar
STARTUP_MSG:   build = https://git-wip-us.apache.org/repos/asf/hadoop.git -r baa91f7c6bc9cb92be5982de4719c1c8af91ccff; compiled by 'root' on 2016-08-18T01:41Z
STARTUP_MSG:   java = 1.8.0_71
************************************************************/
16/12/01 21:31:44 INFO namenode.NameNode: registered UNIX signal handlers for [TERM, HUP, INT]
16/12/01 21:31:44 INFO namenode.NameNode: createNameNode [-format]
Formatting using clusterid: CID-325c8295-f748-4e65-89eb-178b7d127a07
16/12/01 21:31:45 INFO namenode.FSNamesystem: No KeyProvider found.
16/12/01 21:31:45 INFO namenode.FSNamesystem: fsLock is fair:true
16/12/01 21:31:45 INFO blockmanagement.DatanodeManager: dfs.block.invalidate.limit=1000
16/12/01 21:31:45 INFO blockmanagement.DatanodeManager: dfs.namenode.datanode.registration.ip-hostname-check=true
16/12/01 21:31:45 INFO blockmanagement.BlockManager: dfs.namenode.startup.delay.block.deletion.sec is set to 000:00:00:00.000
16/12/01 21:31:45 INFO blockmanagement.BlockManager: The block deletion will start around 2016 Dec 01 21:31:45
16/12/01 21:31:45 INFO util.GSet: Computing capacity for map BlocksMap
16/12/01 21:31:45 INFO util.GSet: VM type       = 64-bit
16/12/01 21:31:45 INFO util.GSet: 2.0% max memory 966.7 MB = 19.3 MB
16/12/01 21:31:45 INFO util.GSet: capacity      = 2^21 = 2097152 entries
16/12/01 21:31:45 INFO blockmanagement.BlockManager: dfs.block.access.token.enable=false
16/12/01 21:31:45 INFO blockmanagement.BlockManager: defaultReplication         = 3
16/12/01 21:31:45 INFO blockmanagement.BlockManager: maxReplication             = 512
16/12/01 21:31:45 INFO blockmanagement.BlockManager: minReplication             = 1
16/12/01 21:31:45 INFO blockmanagement.BlockManager: maxReplicationStreams      = 2
16/12/01 21:31:45 INFO blockmanagement.BlockManager: replicationRecheckInterval = 3000
16/12/01 21:31:45 INFO blockmanagement.BlockManager: encryptDataTransfer        = false
16/12/01 21:31:45 INFO blockmanagement.BlockManager: maxNumBlocksToLog          = 1000
16/12/01 21:31:45 INFO namenode.FSNamesystem: fsOwner             = root (auth:SIMPLE)
16/12/01 21:31:45 INFO namenode.FSNamesystem: supergroup          = supergroup
16/12/01 21:31:45 INFO namenode.FSNamesystem: isPermissionEnabled = true
16/12/01 21:31:45 INFO namenode.FSNamesystem: Determined nameservice ID: ns1
16/12/01 21:31:45 INFO namenode.FSNamesystem: HA Enabled: true
16/12/01 21:31:45 INFO namenode.FSNamesystem: Append Enabled: true
16/12/01 21:31:46 INFO util.GSet: Computing capacity for map INodeMap
16/12/01 21:31:46 INFO util.GSet: VM type       = 64-bit
16/12/01 21:31:46 INFO util.GSet: 1.0% max memory 966.7 MB = 9.7 MB
16/12/01 21:31:46 INFO util.GSet: capacity      = 2^20 = 1048576 entries
16/12/01 21:31:46 INFO namenode.FSDirectory: ACLs enabled? false
16/12/01 21:31:46 INFO namenode.FSDirectory: XAttrs enabled? true
16/12/01 21:31:46 INFO namenode.FSDirectory: Maximum size of an xattr: 16384
16/12/01 21:31:46 INFO namenode.NameNode: Caching file names occuring more than 10 times
16/12/01 21:31:46 INFO util.GSet: Computing capacity for map cachedBlocks
16/12/01 21:31:46 INFO util.GSet: VM type       = 64-bit
16/12/01 21:31:46 INFO util.GSet: 0.25% max memory 966.7 MB = 2.4 MB
16/12/01 21:31:46 INFO util.GSet: capacity      = 2^18 = 262144 entries
16/12/01 21:31:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.threshold-pct = 0.9990000128746033
16/12/01 21:31:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.min.datanodes = 0
16/12/01 21:31:46 INFO namenode.FSNamesystem: dfs.namenode.safemode.extension     = 30000
16/12/01 21:31:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.window.num.buckets = 10
16/12/01 21:31:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.num.users = 10
16/12/01 21:31:46 INFO metrics.TopMetrics: NNTop conf: dfs.namenode.top.windows.minutes = 1,5,25
16/12/01 21:31:46 INFO namenode.FSNamesystem: Retry cache on namenode is enabled
16/12/01 21:31:46 INFO namenode.FSNamesystem: Retry cache will use 0.03 of total heap and retry cache entry expiry time is 600000 millis
16/12/01 21:31:46 INFO util.GSet: Computing capacity for map NameNodeRetryCache
16/12/01 21:31:46 INFO util.GSet: VM type       = 64-bit
16/12/01 21:31:46 INFO util.GSet: 0.029999999329447746% max memory 966.7 MB = 297.0 KB
16/12/01 21:31:46 INFO util.GSet: capacity      = 2^15 = 32768 entries
16/12/01 21:31:48 INFO namenode.FSImage: Allocated new BlockPoolId: BP-1570520983-192.168.241.10-1480599108162
16/12/01 21:31:48 INFO common.Storage: Storage directory /usr/local/hadoop/tmp/dfs/name has been successfully formatted.
16/12/01 21:31:48 INFO namenode.FSImageFormatProtobuf: Saving image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 using no compression
16/12/01 21:31:48 INFO namenode.FSImageFormatProtobuf: Image file /usr/local/hadoop/tmp/dfs/name/current/fsimage.ckpt_0000000000000000000 of size 351 bytes saved in 0 seconds.
16/12/01 21:31:48 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
16/12/01 21:31:49 INFO util.ExitUtil: Exiting with status 0
16/12/01 21:31:49 INFO namenode.NameNode: SHUTDOWN_MSG: 
/************************************************************
SHUTDOWN_MSG: Shutting down NameNode at node1/192.168.241.10
************************************************************/
```
#### 2.5.4将/usr/local/hadoop/tmp拷贝到node2的/usr/local/hadoop下
```shell
scp -r /usr/local/hadoop/tmp node2:/usr/local/hadoop/
```
#### 2.5.5格式化zk
```shell
[root@node1 hadoop]# hdfs zkfc -formatZK
16/12/01 21:36:56 INFO tools.DFSZKFailoverController: Failover controller configured for NameNode NameNode at node1/192.168.241.10:9000
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:zookeeper.version=3.4.6-1569965, built on 02/20/2014 09:09 GMT
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:host.name=node1
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.version=1.8.0_71
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.vendor=Oracle Corporation
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.home=/usr/local/jdk1.8.0_71/jre
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.class.path=/usr/local/hadoop-2.7.3/etc/hadoop:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-core-1.8.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpclient-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-beanutils-1.7.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/mockito-all-1.8.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-math3-3.1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/java-xmlbuilder-0.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-digester-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-httpclient-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsp-api-2.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jets3t-0.9.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-api-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-framework-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/httpcore-4.2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsch-0.1.42.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-net-3.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-configuration-1.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-recipes-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-kerberos-codec-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/gson-2.2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hadoop-auth-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/apacheds-i18n-2.0.0-M15.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-util-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/curator-client-2.7.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/lib/api-asn1-api-1.0.0-M20.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/common/hadoop-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xercesImpl-2.9.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xml-apis-1.3.04.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/htrace-core-3.1.0-incubating.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/netty-all-4.0.23.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/xmlenc-0.52.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/commons-daemon-1.0.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-nfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/hdfs/hadoop-hdfs-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-impl-2.2.3-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guava-11.0.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jaxb-api-2.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-lang-2.6.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-xc-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/activation-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/stax-api-1.0-2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-logging-1.1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-collections-3.2.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/servlet-api-2.5.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jettison-1.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-json-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-codec-1.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-jaxrs-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/commons-cli-1.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jersey-client-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jsr305-3.0.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jetty-util-6.1.26.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/lib/zookeeper-3.4.6-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-web-proxy-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-applicationhistoryservice-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-nodemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-api-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-sharedcachemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-registry-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-unmanaged-am-launcher-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-resourcemanager-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-client-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-applications-distributedshell-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/yarn/hadoop-yarn-server-tests-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/log4j-1.2.17.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/avro-1.7.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/paranamer-2.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/asm-3.2.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/protobuf-java-2.5.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-io-2.4.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/guice-servlet-3.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-core-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-guice-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hadoop-annotations-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/javax.inject-1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/netty-3.6.2.Final.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/junit-4.11.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-mapper-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/commons-compress-1.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/leveldbjni-all-1.8.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/xz-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/snappy-java-1.0.4.1.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jersey-server-1.9.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/hamcrest-core-1.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/jackson-core-asl-1.9.13.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/lib/aopalliance-1.0.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-common-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3-tests.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-core-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-shuffle-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-jobclient-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-plugins-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-app-2.7.3.jar:/usr/local/hadoop-2.7.3/share/hadoop/mapreduce/hadoop-mapreduce-client-hs-2.7.3.jar:/usr/local/hadoop//contrib/capacity-scheduler/*.jar
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.library.path=/usr/local/hadoop-2.7.3/lib/native
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.io.tmpdir=/tmp
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:java.compiler=<NA>
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:os.name=Linux
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:os.arch=amd64
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:os.version=2.6.32-431.el6.x86_64
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:user.name=root
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:user.home=/root
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Client environment:user.dir=/usr/local/hadoop-2.7.3
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Initiating client connection, connectString=node5:2181,node6:2181,node7:2181 sessionTimeout=5000 watcher=org.apache.hadoop.ha.ActiveStandbyElector$WatcherWithClientRef@8f4ea7c
16/12/01 21:36:56 INFO zookeeper.ClientCnxn: Opening socket connection to server node7/192.168.241.70:2181. Will not attempt to authenticate using SASL (unknown error)
16/12/01 21:36:56 INFO zookeeper.ClientCnxn: Socket connection established to node7/192.168.241.70:2181, initiating session
16/12/01 21:36:56 INFO zookeeper.ClientCnxn: Session establishment complete on server node7/192.168.241.70:2181, sessionid = 0x358ba945a200000, negotiated timeout = 5000
16/12/01 21:36:56 INFO ha.ActiveStandbyElector: Session connected.
16/12/01 21:36:56 INFO ha.ActiveStandbyElector: Successfully created /hadoop-ha/ns1 in ZK.
16/12/01 21:36:56 INFO zookeeper.ClientCnxn: EventThread shut down
16/12/01 21:36:56 INFO zookeeper.ZooKeeper: Session: 0x358ba945a200000 closed
```
#### 2.5.6启动hdfs(在node1上执行)
```shell
[root@node1 hadoop]# start-dfs.sh
Starting namenodes on [node1 node2]
node2: starting namenode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-namenode-node2.out
node1: starting namenode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-namenode-node1.out
node6: starting datanode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-datanode-node6.out
node5: starting datanode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-datanode-node5.out
node7: starting datanode, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-datanode-node7.out
Starting journal nodes [node5 node6 node7]
node6: journalnode running as process 1008. Stop it first.
node7: journalnode running as process 1878. Stop it first.
node5: journalnode running as process 3263. Stop it first.
Starting ZK Failover Controllers on NN hosts [node1 node2]
node1: starting zkfc, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-zkfc-node1.out
node2: starting zkfc, logging to /usr/local/hadoop-2.7.3/logs/hadoop-root-zkfc-node2.out
```
#### 2.5.7检查hdfs启动情况
* 在node1，2上检查NameNode和DFSZKFailoverController
```shell
[root@node1 hadoop]# jps
2769 Jps
2413 NameNode
2703 DFSZKFailoverController
############################
[root@node2 ~]# jps 
1924 DFSZKFailoverController
1972 Jps
1832 NameNode
```
* 在node5，6，7上检查DataNode
```shell
[root@node5 hadoop]# jps
2580 QuorumPeerMain
3494 Jps
3386 DataNode
3263 JournalNode
################
[root@node6 ~]# jps
1088 DataNode
1008 JournalNode
944 QuorumPeerMain
1176 Jps
################
[root@node7 src]# jps
1878 JournalNode
1720 QuorumPeerMain
2026 DataNode
2122 Jps
```
* 浏览器验证主从

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/03.png?raw=true)


![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/04.png?raw=true)

#### 2.5.8启动yarn
* 在node3.4上运行
```shell
[root@node3 hadoop]# yarn-daemon.sh start resourcemanager
starting resourcemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-root-resourcemanager-node3.out
```
* 在node5.6.7上运行
```shell
[root@node5 hadoop]# yarn-daemon.sh start nodemanager        
starting nodemanager, logging to /usr/local/hadoop-2.7.3/logs/yarn-root-nodemanager-node5.out
```
* 验证主从
```shell
[root@node1 hadoop]# yarn rmadmin -getServiceState rm1
active  #活动
[root@node1 hadoop]# yarn rmadmin -getServiceState rm2
standby #备用
[root@node1 hadoop]# 
```
### 2.6.验证使用
#### 2.6.1验证hdfs上传文件
```shell
[root@node1 hadoop]# hadoop fs -put /etc/profile /profile

[root@node1 hadoop]# 
[root@node1 hadoop]# hadoop fs -ls /
Found 2 items
-rw-r--r--   3 root supergroup       2311 2016-12-01 22:43 /profile #上传成功
drwxr-xr-x   - root supergroup          0 2016-12-01 22:09 /usr
```
#### 2.6.2验证yran
```shell
[root@node1 hadoop]# hadoop jar /usr/local/hadoop/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.7.3.jar wordcount /profile /out    
16/12/01 22:50:15 INFO input.FileInputFormat: Total input paths to process : 1
16/12/01 22:50:15 INFO mapreduce.JobSubmitter: number of splits:1
16/12/01 22:50:15 INFO mapreduce.JobSubmitter: Submitting tokens for job: job_1480601841303_0001
16/12/01 22:50:16 INFO impl.YarnClientImpl: Submitted application application_1480601841303_0001
16/12/01 22:50:16 INFO mapreduce.Job: The url to track the job: http://node3:8088/proxy/application_1480601841303_0001/
16/12/01 22:50:16 INFO mapreduce.Job: Running job: job_1480601841303_0001
16/12/01 22:51:04 INFO mapreduce.Job: Job job_1480601841303_0001 running in uber mode : false
16/12/01 22:51:04 INFO mapreduce.Job:  map 0% reduce 0%
16/12/01 22:51:40 INFO mapreduce.Job:  map 100% reduce 0%
16/12/01 22:52:00 INFO mapreduce.Job:  map 100% reduce 100%
16/12/01 22:52:01 INFO mapreduce.Job: Job job_1480601841303_0001 completed successfully
16/12/01 22:52:01 INFO mapreduce.Job: Counters: 49
        File System Counters
                FILE: Number of bytes read=2572
                FILE: Number of bytes written=247013
                FILE: Number of read operations=0
                FILE: Number of large read operations=0
                FILE: Number of write operations=0
                HDFS: Number of bytes read=2394
                HDFS: Number of bytes written=1906
                HDFS: Number of read operations=6
                HDFS: Number of large read operations=0
                HDFS: Number of write operations=2
        Job Counters 
                Launched map tasks=1
                Launched reduce tasks=1
                Data-local map tasks=1
                Total time spent by all maps in occupied slots (ms)=32441
                Total time spent by all reduces in occupied slots (ms)=17945
                Total time spent by all map tasks (ms)=32441
                Total time spent by all reduce tasks (ms)=17945
                Total vcore-milliseconds taken by all map tasks=32441
                Total vcore-milliseconds taken by all reduce tasks=17945
                Total megabyte-milliseconds taken by all map tasks=33219584
                Total megabyte-milliseconds taken by all reduce tasks=18375680
        Map-Reduce Framework
                Map input records=86
                Map output records=275
                Map output bytes=3155
                Map output materialized bytes=2572
                Input split bytes=83
                Combine input records=275
                Combine output records=165
                Reduce input groups=165
                Reduce shuffle bytes=2572
                Reduce input records=165
                Reduce output records=165
                Spilled Records=330
                Shuffled Maps =1
                Failed Shuffles=0
                Merged Map outputs=1
                GC time elapsed (ms)=219
                CPU time spent (ms)=1960
                Physical memory (bytes) snapshot=295219200
                Virtual memory (bytes) snapshot=3747471360
                Total committed heap usage (bytes)=138731520
        Shuffle Errors
                BAD_ID=0
                CONNECTION=0
                IO_ERROR=0
                WRONG_LENGTH=0
                WRONG_MAP=0
                WRONG_REDUCE=0
        File Input Format Counters 
                Bytes Read=2311
        File Output Format Counters 
                Bytes Written=1906
[root@node1 hadoop]# hadoop fs -ls /out
Found 2 items
-rw-r--r--   3 root supergroup          0 2016-12-01 22:51 /out/_SUCCESS
-rw-r--r--   3 root supergroup       1906 2016-12-01 22:51 /out/part-r-00000
[root@node1 hadoop]# hadoop fs -cat /out/part-r-00000  #已得到计算结果
!=      1
"$-"    1
"$2"    1
"$EUID" 2
"$HISTCONTROL"  1
"$i"    3
"${-#*i}"       1
"0"     1
":${PATH}:"     1
"`id    2
"after" 1
"ignorespace"   1
#       13
$UID    1
&&      1
()      1
*)      1
*:"$1":*)       1
-f      1
-gn`"   1
-gt     1
-r      1
-ru`    1
-u`     1
-un`"   2
-x      1
-z      1
.       2
/etc/bashrc     1
/etc/profile    1
/etc/profile.d/ 1
/etc/profile.d/*.sh     1
/sbin   2
/usr/bin/id     1
/usr/local/sbin 2
/usr/sbin       2
/usr/share/doc/setup-*/uidgid   1
002     1
022     1
199     1
200     1
2>&1    1
2>/dev/null`    1
;       3
;;      1
=       4
>/dev/null      1
By      1
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:/usr/local/jdk//lib/tools.jar 1
Current 1
EUID=`id        1
Functions       1
GREP_COLOR='1;33'       1
GREP_OPTIONS='--color=auto'     1
HADOOP_HOME=/usr/local/hadoop/  1
HISTCONTROL     1
HISTCONTROL=ignoreboth  1
HISTCONTROL=ignoredups  1
HISTSIZE        1
HISTSIZE=1000   1
HOSTNAME        1
HOSTNAME=`/bin/hostname 1
It's    2
JAVA_HOME=/usr/local/jdk/       1
LC_ALL=C        1
LOGNAME 1
LOGNAME=$USER   1
MAIL    1
MAIL="/var/spool/mail/$USER"    1
MAILCHECK       1
NOT     1
PATH    1
PATH=$1:$PATH   1
PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin:/usr/local/jdk//bin:/usr/local/jdk//bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin:/root/bin 1
PATH=$JAVA_HOME/bin:/usr/local/jdk//bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin  1
PATH=$PATH:$1   1
Path    1
System  1
This    1
UID=`id 1
USER    1
USER="`id       1
You     1
[       9
]       3
];      6
a       2
after   3
aliases 1
and     2
are     1
as      1
better  1
case    1
change  1
changes 1
check   1
could   1
create  1
custom  1
custom.sh       1
default,        1
do      1
doing.  1
done    1
else    5
environment     1
environment,    1
esac    1
export  10
fi      8
file    2
for     5
future  1
get     1
go      1
good    1
i       2
idea    1
if      8
in      6
is      1
it      1
know    1
ksh     1
login   2
make    1
manipulation    1
merging 1
much    1
need    1
pathmunge       8
prevent 1
programs,       1
reservation     1
reserved        1
script  1
set.    1
sets    1
setup   1
shell   2
startup 1
system  1
the     1
then    8
this    2
threshold       1
to      5
uid/gids        1
uidgid  1
umask   3
unless  1
unset   3
updates.        1
validity        1
want    1
we      1
what    1
wide    1
will    1
workaround      1
you     2
your    1
{       1
}       1
```
#### 2.6.3hdfs的HA验证
* 杀死node1上NameNode进程
```shell
[root@node1 hadoop]# jps
3597 Jps
2413 NameNode
2703 DFSZKFailoverController
[root@node1 hadoop]# kill -9 2413
[root@node1 hadoop]# jps
3607 Jps
2703 DFSZKFailoverController
```

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/05.png?raw=true)

	如图node2已经变为活跃状态
#### 2.6.4验证resoucemanager的HA

* 杀死node3上的resoucemanager
```shell
[root@node3 hadoop]# jps          
3873 Jps
3631 ResourceManager
[root@node3 hadoop]# kill -9 3631
[root@node3 hadoop]# yarn rmadmin -getServiceState rm2 #rm2即node4已经变为活跃
active
[root@node3 hadoop]# 
``` 