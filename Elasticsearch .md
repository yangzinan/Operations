# Elasticsearch 
#### By：大官人
#### Email：DaGuanR@gmail.com
#### QQ：375739049

# 一.安装JDK

```shell
tar zxf jdk-8u71-linux-x64.tar.gz -C /usr/local
ln -s /usr/local/jdk1.8.0_71 /usr/local/jdk
cat >>/etc/profile<<EOF
export JAVA_HOME=/usr/local/jdk
export PATH=/usr/local/jdk/bin:/bin:$PATH
export CLASSPATH=.:/usr/local/jdk/lib/dt.jar:/usr/local/jdk/lib/tools.jar
EOF
source /etc/profile
```

# 二.安装el

```shell
tar zxf elasticsearch-5.1.1.tar.gz -C /usr/local
ln -s /usr/local/elasticsearch-5.1.1 /usr/local/elasticsearch
```

# 三.配置系统

```shell
echo "vm.max_map_count= 262144" /etc/sysctl.conf
sysctl -p
cat>>/etc/security/limits.conf<<EOF
*  hard nofile 65536*  soft nofile 65536
EOF
sed -i "s#1024#2048#g" /etc/security/limits.d/90-nproc.conf
```

# 四.配置el配置文件

```conf
cluster.name: [cluster_name] #配置集群名称所有节点的配置必须一致
node.name: [node_name] #在一个集群内nodename必须是唯一的
network.host: [ipaddr|host] #配置IP地址或者主机名（本机的）
discovery.zen.ping.unicast.hosts: ["[ip|host]", "[ip|host]", "ip|host"] #配置集群所有的节点的IP或主机名
discovery.zen.minimum_master_nodes: [num] #配置集群主节点的个数
```

# 五.启动el集群

```shell
useradd el
chown -R el.el /usr/local/elasticsearch-5.1.1
/usr/local/elasticsearch/bin/elasticsearch -d
echo "/bin/su - el -c /usr/local/elasticsearch/bin/elasticsearch -d" >> /etc/rc.d/rc.local
```