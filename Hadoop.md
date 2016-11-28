# Hadoop安装文档
#### By：大官人

#### Email：DaGuanR@gmail.com

#### QQ:375739049
## 一·伪分布式
### 1.环境准备
#### 1.1关闭iptables和selinux
```shell
service iptables stop
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disable#g' /etc/selinux/config
```
#### 1.2安装JDK
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
### 2.安装hadoop
#### 2.1解压
```shell
tar zxf hadoop-2.7.3.tar.gz -C /usr/local
ln -s /usr/local/hadoop-2.7.3 /usr/local/hadoop
```
#### 2.2配置JAVA_HOME
```shell
sed -i "s#export JAVA_HOME=\${JAVA_HOME}#export JAVA_HOME=$JAVA_HOME#g" /usr/local/hadoop/etc/hadoop/hadoop-env.sh
```
#### 2.3 修改vim core-site.xml
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
#### 2.3修改hdfs-site.xml
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
#### 2.4 配置mapred-site.xml
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
#### 2.5配置yarn-site.xml 
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
		<name>yarn.nademanager.aux-services</name>
		<value>mapreduce_shuffle</value>
	</property>

	<!--用来指定ResourceManager主机地址-->
	<property>
		<name>yarn.resourcemanager.hostname</value>
		<value>0.0.0.0</value>
	</property>
</configuration>
```
#### 2.6配置环境变量
```shell
cat >>/etc/profile<<EOF
export HADOOP_HOME=/usr/local/hadoop
PATH=\$HADOOP_HOME/bin:\$HADOOP_HOME/sbin:$PATH
EOF
source /etc/profile
```
#### 2.7格式化文件系统（hdfs）
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
#### 2.8启动hadoop
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
## 三.验证
### 1.通过JAVA进程验证
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
### 2.端口验证
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
### 3.浏览器验证

* yarn管理界面

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/01.png?raw=true)

* hdfs管理界面

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/hadoop/02.png?raw=true)

