# Codis安装文档
## 一、codis简介
### 1.1codis特点
	1.最新 release 版本为 codis-3.2，codis-server 基于 redis-3.2.8
	
	2.支持 slot 同步迁移、异步迁移和并发迁移，对 key 大小无任何限制，迁移性能大幅度提升
	
	3.相比 2.0：重构了整个集群组件通信方式，codis-proxy 与 zookeeper 实现了解耦，废弃了codis-config 等
	
	4.元数据存储支持 etcd/zookeeper/filesystem 等，可自行扩展支持新的存储，集群正常运行期间，即便元存储故障也不再影响 codis 集群，大大提升 codis-proxy 稳定性
	
	5.对 codis-proxy 进行了大量性能优化,通过控制GC频率、减少对象创建、内存预分配、引入 cgo、jemalloc 等，使其吞吐还是延迟，都已达到 codis 项目中最佳
	
	6.proxy 实现 select 命令，支持多 DB
	
	7.proxy 支持读写分离、优先读同 IP/同 DC 下副本功能
	
	8.基于 redis-sentinel 实现主备自动切换
	
	9.实现动态 pipeline 缓存区（减少内存分配以及所引起的 GC 问题）
	
	10.proxy 支持通过 HTTP 请求实时获取 runtime metrics，便于监控、运维
	
	11.支持通过 influxdb 和 statsd 采集 proxy metrics
	
	12.slot auto rebalance 算法从 2.0 的基于 max memory policy 变更成基于 group 下 slot 数量
	
	13.提供了更加友好的 dashboard 和 fe 界面，新增了很多按钮、跳转链接、错误状态等，有利于快速发现、处理集群故障
	
	14.新增 SLOTSSCAN 指令，便于获取集群各个 slot 下的所有 key
	
	15.codis-proxy 与 codis-dashbaord 支持 docker 部署
### 1.2codis拓扑
![image] (https://github.com/CodisLabs/codis/raw/release3.2/doc/pictures/architecture.png)

### 1.3codis3.x组件介绍
	1.Codis Server：基于 redis-3.2.8 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。具体的修改可以参考文档 redis 的修改。
	
	2.Codis Proxy：客户端连接的 Redis 代理服务, 实现了 Redis 协议。 除部分命令不支持以外(不支持的命令列表)，表现的和原生的 Redis 没有区别（就像 Twemproxy）。 
	- 对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例； 
	- 不同 codis-proxy 之间由 codis-dashboard 保证状态同步。
	
	3.Redis sentinel：Redis官方推荐的高可用性(HA)解决方案。它可以实现对Redis的监控、通知、自动故障转移。如果Master不能工作，则会自动启动故障转移进程，将其中的一个Slave提升为Master，其他的Slave重新设置新的Master服务。
	
	4.Codis Dashboard：集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。在集群状态发生改变时，codis-dashboard 维护集群下所有 codis-proxy 的状态的一致性。 
	- 对于同一个业务集群而言，同一个时刻 codis-dashboard 只能有 0个或者1个； 
	- 所有对集群的修改都必须通过 codis-dashboard 完成。
	
	5.Codis Admin：集群管理的命令行工具。 
	- 可用于控制 codis-proxy、codis-dashboard 状态以及访问外部存储。
	
	6.Codis FE：集群管理界面。 
	- 多个集群实例共享可以共享同一个前端展示页面； 
	- 通过配置文件管理后端codis-dashboard列表，配置文件可自动更新。
	
	7.Storage：为集群状态提供外部存储。 
	- 提供namespace概念，不同集群的会按照不同product name进行组织； 
	- 目前仅提供了zookeeper、etcd、filesystem三种实现，但是提供了抽象的 interface 可自行扩展。
## 二、codis部署环境规划
|     主机名    | IP   |  部署程序  |
| --------   | :-----:  | :----:  |
|codis01|10.0.10.20|codis-server:(6379&6380)、zookeeper:2181|
|codis02|10.0.10.21|codis-server:(6379&6380)、zookeeper:2181|
|codis03|10.0.10.22|codis-server:(6379&6380)、zookeeper:2181|
|codis04|10.0.10.23|redis-sentinel:26379、codis-proxy:19000、|
|codis05|10.0.10.24|redis-sentinel:26379、codis-proxy:19000、|
|codis06|10.0.10.25|redis-sentinel:26379、codis-dashborad:18080、codis-fe:18090|
## 三、安装部署
### 3.1安装部署zookeeper
#### 3.1.1安装jdk
* 在codis01、02、03上执行

```shell
yum install ./jdk-8u144-linux-x64.rpm -y
```

#### 3.1.2安装zookeeper
##### (1)安装zk（在codis01上执行）
```shell
tar zxf zookeeper-3.4.5.tar.gz -C /usr/local/
ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper
echo export PATH=$PATH:/usr/local/zookeeper/bin >> /etc/profile
source /etc/profile
```
##### (2)修改zk配置文件
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
server.1=codis01:2888:3888
server.2=codis01:2888:3888
server.3=codis01:2888:3888
```
##### (3)创建zk数据路径（在codis01、02、03上执行）
```shell
mkdir -p /data/zookeeper
echo $num > /data/zookeeper/myid #num是server.后面的数字每个主机不同
```
##### (4)将zk目录拷贝到codis02、03上
```shell
scp -r /usr/local/zookeeper-3.4.5 root@codis02:/usr/local/
scp -r /usr/local/zookeeper-3.4.5 root@codis03:/usr/local/ 
```
##### (5)配置其他节点环境变量（在codis02、03上执行）
```shell
ln -s /usr/local/zookeeper-3.4.5 /usr/local/zookeeper
echo export PATH=$PATH:/usr/local/zookeeper/bin >> /etc/profile
source /etc/profile
```
##### (6)启动zk（在codis01、02、03执行）
```shell
[root@codis01 /usr/local/zookeeper/conf 17:45:49]# zkServer.sh start
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@codis01 /usr/local/zookeeper/conf 17:45:55]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
##############################
[root@codis02 ~ 17:45:43]# zkServer.sh start
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@codis02 ~ 17:45:55]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: follower
##############################
[root@codis03 ~ 17:45:43]# zkServer.sh start
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
[root@codis03 ~ 17:45:55]# zkServer.sh status
JMX enabled by default
Using config: /usr/local/zookeeper/bin/../conf/zoo.cfg
Mode: leader
```
	现在codis03是leader节点
### 3.2安装配置codis-server
#### 3.2.1安装codis-server
##### (1)安装在codis01执行）
```shell
tar zxf codis3.2.0-go1.7.5-linux.tar.gz -C /usr/local/
ln -s /usr/local/codis3.2.0-go1.7.5-linux /usr/local/codis
```
##### (2)配置主（在codis01执行）
```shell
mkdir /usr/local/codis/conf
cat >> /usr/local/codis/conf/redis-6379.conf << EOF
daemonize yes
pidfile /var/run/redis-6379.pid # 进程ID文件路径 
port 6379 # 绑定端口 
timeout 86400 
tcp-keepalive 60 
loglevel notice 
logfile /var/log/redis/redis-6379.log # 日志文件路径 
databases 16 
save “” 
save 900 1 
save 300 10 
save 60 10000 
stop-writes-on-bgsave-error no 
rdbcompression yes 
dbfilename dump-6379.rdb # dump文件 
dir /data/redis/redis_data_6379 # dump路径 
masterauth "123456" # Master密码（从主同步密码） 
slave-serve-stale-data yes 
repl-disable-tcp-nodelay no 
slave-priority 100 
requirepass "123456" # 鉴权密码（客户端连接密码） 
maxmemory 10gb 
maxmemory-policy allkeys-lru 
appendonly no 
appendfsync everysec 
no-appendfsync-on-rewrite yes 
auto-aof-rewrite-percentage 100 
slowlog-log-slower-than 10000 
slowlog-max-len 128 
hash-max-ziplist-entries 512 
hash-max-ziplist-value 64 
list-max-ziplist-entries 512 
list-max-ziplist-value 64 
set-max-intset-entries 512 
zset-max-ziplist-entries 128 
zset-max-ziplist-value 64 
client-output-buffer-limit normal 0 0 0 
client-output-buffer-limit slave 0 0 0 
client-output-buffer-limit pubsub 0 0 0 
hz 10 
aof-rewrite-incremental-fsync yes 
repl-backlog-size 33554432
EOF
```
##### (4)配置从codis-server（在codis01执行）
```shell
cat >> /usr/local/codis/conf/redis-6380.conf << EOF
daemonize yes 
pidfile /var/run/redis-6380.pid # 进程ID文件路径 
port 6380 # 绑定端口 
timeout 86400 
tcp-keepalive 60 
loglevel notice 
logfile /var/log/redis/redis-6380.log # 日志文件路径 
databases 16 
save “” # 如果不希望存储到磁盘，则可以使用井号注销save配置行 
save 900 1 # 如果不希望存储到磁盘，则可以使用井号注销save配置行 
save 300 10 # 如果不希望存储到磁盘，则可以使用井号注销save配置行 
save 60 10000 # 如果不希望存储到磁盘，则可以使用井号注销save配置行 
stop-writes-on-bgsave-error no 
rdbcompression yes 
dbfilename dump-6380.rdb # dump文件 
dir /data/redis/redis_data_6380 # dump路径 
masterauth "123456" # Master密码（适合主从集群） 
slave-serve-stale-data yes 
repl-disable-tcp-nodelay no 
slave-priority 100 
requirepass "123456" # 鉴权密码（客户端连接密码） 
maxmemory 10gb 
maxmemory-policy allkeys-lru 
appendonly no 
appendfsync everysec 
no-appendfsync-on-rewrite yes 
auto-aof-rewrite-percentage 100 
slowlog-log-slower-than 10000 
slowlog-max-len 128 
hash-max-ziplist-entries 512 
hash-max-ziplist-value 64 
list-max-ziplist-entries 512 
list-max-ziplist-value 64 
set-max-intset-entries 512 
zset-max-ziplist-entries 128 
zset-max-ziplist-value 64 
client-output-buffer-limit normal 0 0 0 
client-output-buffer-limit slave 0 0 0 
client-output-buffer-limit pubsub 0 0 0 
hz 10 
aof-rewrite-incremental-fsync yes 
repl-backlog-size 33554432
EOF
```
#### 3.2.2将codis拷贝到codis02、03
```shell
scp -r /usr/local/codis3.2.0-go1.7.5-linux root@codis02:/usr/local/
scp -r /usr/local/codis3.2.0-go1.7.5-linux root@codis03:/usr/local/
```
#### 3.2.3创建软连接（在codis02、03执行）
```shell
ln -s /usr/local/codis3.2.0-go1.7.5-linux/ /usr/local/codis
```
#### 3.2.4创建codis-server所需目录（在codis01、02、03上执行）
```shell
mkdir -p /var/log/redis /data/redis/redis_data_6379 /data/redis/redis_data_6380
```
#### 3.2.5启动codis-server（在codis01、02、03执行）
```shell
[root@codis01 /usr/local/codis/conf 18:25:53]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6379.conf 
[root@codis01 /usr/local/codis/conf 18:30:06]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6380.conf
[root@codis01 /usr/local/codis/conf 18:31:01]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      24668/codis-server  
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN      24719/codis-server  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      869/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1157/master         
tcp6       0      0 :::6379                 :::*                    LISTEN      24668/codis-server  
tcp6       0      0 :::6380                 :::*                    LISTEN      24719/codis-server  
tcp6       0      0 :::3888                 :::*                    LISTEN      24289/java          
tcp6       0      0 :::22                   :::*                    LISTEN      869/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1157/master         
tcp6       0      0 :::44801                :::*                    LISTEN      24289/java          
tcp6       0      0 :::2181                 :::*                    LISTEN      24289/java
#######################################################################################################
[root@codis02 ~ 18:26:15]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6379.conf
[root@codis02 ~ 18:30:36]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6380.conf
[root@codis02 ~ 18:31:04]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      22534/codis-server  
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN      22618/codis-server  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      864/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1547/master         
tcp6       0      0 :::45226                :::*                    LISTEN      22341/java          
tcp6       0      0 :::6379                 :::*                    LISTEN      22534/codis-server  
tcp6       0      0 :::6380                 :::*                    LISTEN      22618/codis-server  
tcp6       0      0 :::3888                 :::*                    LISTEN      22341/java          
tcp6       0      0 :::22                   :::*                    LISTEN      864/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1547/master         
tcp6       0      0 :::2181                 :::*                    LISTEN      22341/java
#######################################################################################################
[root@codis03 ~ 18:23:01]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6379.conf
[root@codis03 ~ 18:28:09]# /usr/local/codis/codis-server /usr/local/codis/conf/redis-6380.conf
[root@codis03 ~ 18:31:08]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      22461/codis-server  
tcp        0      0 0.0.0.0:6380            0.0.0.0:*               LISTEN      22479/codis-server  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      869/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1212/master         
tcp6       0      0 :::2888                 :::*                    LISTEN      22150/java          
tcp6       0      0 :::6379                 :::*                    LISTEN      22461/codis-server  
tcp6       0      0 :::6380                 :::*                    LISTEN      22479/codis-server  
tcp6       0      0 :::3888                 :::*                    LISTEN      22150/java          
tcp6       0      0 :::37265                :::*                    LISTEN      22150/java          
tcp6       0      0 :::22                   :::*                    LISTEN      869/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1212/master         
tcp6       0      0 :::2181                 :::*                    LISTEN      22150/java 
```
### 3.3安装部署codis-dashboard
#### 3.3.1安装
```shell
tar zxf codis3.2.0-go1.7.5-linux.tar.gz -C /usr/local/
ln -s /usr/local/codis3.2.0-go1.7.5-linux/ /usr/local/codis 
```
#### 3.3.2生成配置文件
```shell
mkdir /usr/local/codis/conf
/usr/local/codis/codis-dashboard --default-config | tee /usr/local/codis/conf/dashboard.conf
```
#### 3.3.3配置
```conf

##################################################
#                                                #
#                  Codis-Dashboard               #
#                                                #
##################################################

# Set Coordinator, only accept "zookeeper" & "etcd" & "filesystem".
# Quick Start
coordinator_name = "zookeeper"
coordinator_addr = "codis01:2181,codis02:2181,codis03:2181"
#coordinator_name = "zookeeper"
#coordinator_addr = "127.0.0.1:2181"

# Set Codis Product Name/Auth.
product_name = "codis-demo"
#主从密码要和Redis主从配置一致
product_auth = "123456"

# Set bind address for admin(rpc), tcp only.
admin_addr = "0.0.0.0:18080"

# Set arguments for data migration (only accept 'sync' & 'semi-async').
migration_method = "semi-async"
migration_parallel_slots = 100
migration_async_maxbulks = 200
migration_async_maxbytes = "32mb"
migration_async_numkeys = 500
migration_timeout = "30s"

# Set configs for redis sentinel.
sentinel_quorum = 2
sentinel_parallel_syncs = 1
sentinel_down_after = "30s"
sentinel_failover_timeout = "5m"
sentinel_notification_script = ""
sentinel_client_reconfig_script = ""
```
#### 3.3.4启动验证
```shell
[root@codis06 ~ 18:58:29]# nohup /usr/local/codis/codis-dashboard --ncpu=1 --config=/usr/local/codis/conf/dashboard.conf --log=/var/log/codis/dashboard.log --log-level=WARN &
[1] 10046
[root@codis06 ~ 18:58:23]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      860/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1162/master         
tcp6       0      0 :::22                   :::*                    LISTEN      860/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1162/master         
tcp6       0      0 :::18080                :::*                    LISTEN      10046/codis-dashboa 
```
### 3.4安装部署codis-proxy
#### 3.4.1安装
```shell
tar zxf codis3.2.0-go1.7.5-linux.tar.gz -C /usr/local/
ln -s /usr/local/codis3.2.0-go1.7.5-linux/ /usr/local/codis 
```
#### 3.4.2生成配置文件
```shell
mkdir /usr/local/codis/conf
/usr/local/codis/codis-proxy --default-config | tee /usr/local/codis/conf/proxy.conf
```
#### 3.4.3配置
```conf
##################################################
#                                                #
#                  Codis-Proxy                   #
#                                                #
##################################################

# Set Codis Product Name/Auth.
product_name = "codis-demo"
product_auth = "123456"

# Set auth for client session
#   1. product_auth is used for auth validation among codis-dashboard,
#      codis-proxy and codis-server.
#   2. session_auth is different from product_auth, it requires clients
#      to issue AUTH <PASSWORD> before processing any other commands.
session_auth = "56789"

# Set bind address for admin(rpc), tcp only.
admin_addr = "0.0.0.0:11080"

# Set bind address for proxy, proto_type can be "tcp", "tcp4", "tcp6", "unix" or "unixpacket".
proto_type = "tcp4"
proxy_addr = "0.0.0.0:19000"

# Set jodis address & session timeout
#   1. jodis_name is short for jodis_coordinator_name, only accept "zookeeper" & "etcd".
#   2. jodis_addr is short for jodis_coordinator_addr
#   3. proxy will be registered as node:
#        if jodis_compatible = true (not suggested):
#          /zk/codis/db_{PRODUCT_NAME}/proxy-{HASHID} (compatible with Codis2.0)
#        or else
#          /jodis/{PRODUCT_NAME}/proxy-{HASHID}
jodis_name = "zookeeper"
jodis_addr = "codis01:2181,codis02:2181,codis03:2181"
jodis_timeout = "20s"
jodis_compatible = false

# Set datacenter of proxy.
proxy_datacenter = ""

# Set max number of alive sessions.
proxy_max_clients = 1000

# Set max offheap memory size. (0 to disable)
proxy_max_offheap_size = "1024mb"

# Set heap placeholder to reduce GC frequency.
proxy_heap_placeholder = "256mb"

# Proxy will ping backend redis (and clear 'MASTERDOWN' state) in a predefined interval. (0 to disable)
backend_ping_period = "5s"

# Set backend recv buffer size & timeout.
backend_recv_bufsize = "128kb"
backend_recv_timeout = "30s"

# Set backend send buffer & timeout.
backend_send_bufsize = "128kb"
backend_send_timeout = "30s"

# Set backend pipeline buffer size.
backend_max_pipeline = 20480

# Set backend never read replica groups, default is false
backend_primary_only = false

# Set backend parallel connections per server
backend_primary_parallel = 1
backend_replica_parallel = 1

# Set backend tcp keepalive period. (0 to disable)
backend_keepalive_period = "75s"

# Set number of databases of backend.
backend_number_databases = 16

# If there is no request from client for a long time, the connection will be closed. (0 to disable)
# Set session recv buffer size & timeout.
session_recv_bufsize = "128kb"
session_recv_timeout = "30m"

# Set session send buffer size & timeout.
session_send_bufsize = "64kb"
session_send_timeout = "30s"

# Make sure this is higher than the max number of requests for each pipeline request, or your client may be blocked.
# Set session pipeline buffer size.
session_max_pipeline = 10000

# Set session tcp keepalive period. (0 to disable)
session_keepalive_period = "75s"

# Set session to be sensitive to failures. Default is false, instead of closing socket, proxy will send an error response to client.
session_break_on_failure = false

# Set metrics server (such as http://localhost:28000), proxy will report json formatted metrics to specified server in a predefined period.
metrics_report_server = ""
metrics_report_period = "1s"

# Set influxdb server (such as http://localhost:8086), proxy will report metrics to influxdb.
metrics_report_influxdb_server = ""
metrics_report_influxdb_period = "1s"
metrics_report_influxdb_username = ""
metrics_report_influxdb_password = ""
metrics_report_influxdb_database = ""

# Set statsd server (such as localhost:8125), proxy will report metrics to statsd.
metrics_report_statsd_server = ""
metrics_report_statsd_period = "1s"
metrics_report_statsd_prefix = ""
```
#### 3.4.4启动验证
```shell
[root@codis04 ~ 19:45:31]# nohup /usr/local/codis/codis-proxy --ncpu=1 --config=/usr/local/codis/conf/proxy.conf --log=/var/log/codis/proxy.log --log-level=WARN &     
[1] 10165
root@codis04 ~ 19:45:52]# netstat -ntlp                      
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      865/sshd            
tcp        0      0 0.0.0.0:19000           0.0.0.0:*               LISTEN      10165/codis-proxy   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1427/master         
tcp6       0      0 :::11080                :::*                    LISTEN      10165/codis-proxy   
tcp6       0      0 :::22                   :::*                    LISTEN      865/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1427/master
################################################################################################################
[root@codis05 ~ 19:46:47]# nohup /usr/local/codis/codis-proxy --ncpu=1 --config=/usr/local/codis/conf/proxy.conf --log=/var/log/codis/proxy.log --log-level=WARN &
[1] 9983
[root@codis05 ~ 19:46:52]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      857/sshd            
tcp        0      0 0.0.0.0:19000           0.0.0.0:*               LISTEN      9983/codis-proxy    
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1135/master         
tcp6       0      0 :::11080                :::*                    LISTEN      9983/codis-proxy    
tcp6       0      0 :::22                   :::*                    LISTEN      857/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1135/master
```
### 3.5安装redis-sentinel（codis04、05、06）
#### 3.5.1安装
```shell
tar zxf codis3.2.0-go1.7.5-linux.tar.gz -C /usr/local/
ln -s /usr/local/codis3.2.0-go1.7.5-linux/ /usr/local/codis 
```
#### 3.5.2配置(/usr/local/codis/conf/sentinel.conf)
```conf
# *** IMPORTANT ***
#
# By default Sentinel will not be reachable from interfaces different than
# localhost, either use the 'bind' directive to bind to a list of network
# interfaces, or disable protected mode with "protected-mode no" by
# adding it to this configuration file.
#
# Before doing that MAKE SURE the instance is protected from the outside
# world via firewalling or other means.
#
# For example you may use one of the following:
#
daemonize yes
bind 0.0.0.0
#
protected-mode no

# port <sentinel-port>
# The port that this sentinel instance will run on
port 26379

# sentinel announce-ip <ip>
# sentinel announce-port <port>
#
# The above two configuration directives are useful in environments where,
# because of NAT, Sentinel is reachable from outside via a non-local address.
#
# When announce-ip is provided, the Sentinel will claim the specified IP address
# in HELLO messages used to gossip its presence, instead of auto-detecting the
# local address as it usually does.
#
# Similarly when announce-port is provided and is valid and non-zero, Sentinel
# will announce the specified TCP port.
#
# The two options don't need to be used together, if only announce-ip is
# provided, the Sentinel will announce the specified IP and the server port
# as specified by the "port" option. If only announce-port is provided, the
# Sentinel will announce the auto-detected local IP and the specified port.
#
# Example:
#
# sentinel announce-ip 1.2.3.4

dir /data/redis-sentinel/
# Every long running process should have a well-defined working directory.
# For Redis Sentinel to chdir to /tmp at startup is the simplest thing
# for the process to don't interfere with administrative tasks such as
# unmounting filesystems.
dir /tmp

# sentinel monitor <master-name> <ip> <redis-port> <quorum>
#
# Tells Sentinel to monitor this master, and to consider it in O_DOWN
# (Objectively Down) state only if at least <quorum> sentinels agree.
#
# Note that whatever is the ODOWN quorum, a Sentinel will require to
# be elected by the majority of the known Sentinels in order to
# start a failover, so no failover can be performed in minority.
#
# Slaves are auto-discovered, so you don't need to specify slaves in
# any way. Sentinel itself will rewrite this configuration file adding
# the slaves using additional configuration options.
# Also note that the configuration file is rewritten when a
# slave is promoted to master.
#
# Note: master name should not include special characters or spaces.
# The valid charset is A-z 0-9 and the three characters ".-_".
# sentinel monitor mymaster 127.0.0.1 6379 2

# sentinel auth-pass <master-name> <password>
#
# Set the password to use to authenticate with the master and slaves.
# Useful if there is a password set in the Redis instances to monitor.
#
# Note that the master password is also used for slaves, so it is not
# possible to set a different password in masters and slaves instances
# if you want to be able to monitor these instances with Sentinel.
#
# However you can have Redis instances without the authentication enabled
# mixed with Redis instances requiring the authentication (as long as the
# password set is the same for all the instances requiring the password) as
# the AUTH command will have no effect in Redis instances with authentication
# switched off.
#
# Example:
#
# sentinel auth-pass mymaster MySUPER--secret-0123passw0rd

# sentinel down-after-milliseconds <master-name> <milliseconds>
#
# Number of milliseconds the master (or any attached slave or sentinel) should
# be unreachable (as in, not acceptable reply to PING, continuously, for the
# specified period) in order to consider it in S_DOWN state (Subjectively
# Down).
#
# Default is 30 seconds.
# sentinel down-after-milliseconds mymaster 30000

# sentinel parallel-syncs <master-name> <numslaves>
#
# How many slaves we can reconfigure to point to the new slave simultaneously
# during the failover. Use a low number if you use the slaves to serve query
# to avoid that all the slaves will be unreachable at about the same
# time while performing the synchronization with the master.
# sentinel parallel-syncs mymaster 1

# sentinel failover-timeout <master-name> <milliseconds>
#
# Specifies the failover timeout in milliseconds. It is used in many ways:
#
# - The time needed to re-start a failover after a previous failover was
#   already tried against the same master by a given Sentinel, is two
#   times the failover timeout.
#
# - The time needed for a slave replicating to a wrong master according
#   to a Sentinel current configuration, to be forced to replicate
#   with the right master, is exactly the failover timeout (counting since
#   the moment a Sentinel detected the misconfiguration).
#
# - The time needed to cancel a failover that is already in progress but
#   did not produced any configuration change (SLAVEOF NO ONE yet not
#   acknowledged by the promoted slave).
#
# - The maximum time a failover in progress waits for all the slaves to be
#   reconfigured as slaves of the new master. However even after this time
#   the slaves will be reconfigured by the Sentinels anyway, but not with
#   the exact parallel-syncs progression as specified.
#
# Default is 3 minutes.
# sentinel failover-timeout mymaster 180000

# SCRIPTS EXECUTION
#
# sentinel notification-script and sentinel reconfig-script are used in order
# to configure scripts that are called to notify the system administrator
# or to reconfigure clients after a failover. The scripts are executed
# with the following rules for error handling:
#
# If script exits with "1" the execution is retried later (up to a maximum
# number of times currently set to 10).
#
# If script exits with "2" (or an higher value) the script execution is
# not retried.
#
# If script terminates because it receives a signal the behavior is the same
# as exit code 1.
#
# A script has a maximum running time of 60 seconds. After this limit is
# reached the script is terminated with a SIGKILL and the execution retried.

# NOTIFICATION SCRIPT
#
# sentinel notification-script <master-name> <script-path>
# 
# Call the specified notification script for any sentinel event that is
# generated in the WARNING level (for instance -sdown, -odown, and so forth).
# This script should notify the system administrator via email, SMS, or any
# other messaging system, that there is something wrong with the monitored
# Redis systems.
#
# The script is called with just two arguments: the first is the event type
# and the second the event description.
#
# The script must exist and be executable in order for sentinel to start if
# this option is provided.
#
# Example:
#
# sentinel notification-script mymaster /var/redis/notify.sh

# CLIENTS RECONFIGURATION SCRIPT
#
# sentinel client-reconfig-script <master-name> <script-path>
#
# When the master changed because of a failover a script can be called in
# order to perform application-specific tasks to notify the clients that the
# configuration has changed and the master is at a different address.
# 
# The following arguments are passed to the script:
#
# <master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
#
# <state> is currently always "failover"
# <role> is either "leader" or "observer"
# 
# The arguments from-ip, from-port, to-ip, to-port are used to communicate
# the old address of the master and the new address of the elected slave
# (now a master).
#
# This script should be resistant to multiple invocations.
#
# Example:
#
# sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```
#### 3.5.3创建数据目录
```shell
mkdir -p /data/redis-sentinel/
```
#### 3.5.4启动验证
```shell
[root@codis04 ~ 20:18:26]# /usr/local/codis/redis-sentinel /usr/local/codis/conf/sentinel.conf
[root@codis04 ~ 20:20:51]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:26379           0.0.0.0:*               LISTEN      10398/redis-sentinel  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      865/sshd            
tcp        0      0 0.0.0.0:19000           0.0.0.0:*               LISTEN      10165/codis-proxy   
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1427/master         
tcp6       0      0 :::11080                :::*                    LISTEN      10165/codis-proxy   
tcp6       0      0 :::22                   :::*                    LISTEN      865/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1427/master 
##################################################################################################
[root@codis05 ~ 20:14:35]# /usr/local/codis/redis-sentinel /usr/local/codis/conf/sentinel.conf
[root@codis05 ~ 20:18:05]# 
[root@codis05 ~ 20:23:42]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:26379           0.0.0.0:*               LISTEN      10074/redis-sentinel  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      857/sshd            
tcp        0      0 0.0.0.0:19000           0.0.0.0:*               LISTEN      9983/codis-proxy    
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1135/master         
tcp6       0      0 :::11080                :::*                    LISTEN      9983/codis-proxy    
tcp6       0      0 :::22                   :::*                    LISTEN      857/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1135/master
##################################################################################################
[root@codis06 ~ 20:18:26]# /usr/local/codis/redis-sentinel /usr/local/codis/conf/sentinel.conf
[root@codis06 ~ 20:18:28]# 
[root@codis06 ~ 20:24:04]# netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:26379           0.0.0.0:*               LISTEN      21524/redis-sentinel  
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      860/sshd            
tcp        0      0 127.0.0.1:25            0.0.0.0:*               LISTEN      1162/master         
tcp6       0      0 :::22                   :::*                    LISTEN      860/sshd            
tcp6       0      0 ::1:25                  :::*                    LISTEN      1162/master         
tcp6       0      0 :::18080                :::*                    LISTEN      10046/codis-dashboa
```
### 3.6安装codis-fe
#### 3.6.1安装
```shell
tar zxf codis3.2.0-go1.7.5-linux.tar.gz -C /usr/local/
ln -s /usr/local/codis3.2.0-go1.7.5-linux/ /usr/local/codis 
```
#### 3.6.2生成配置文件
```shell
/usr/local/codis/codis-dashboard --default-config | tee /usr/local/codis/conf/dashboard.conf
```
#### 3.6.3启动
```shell
nohup /usr/local/codis/codis-proxy --ncpu=1 --config=/usr/local/codis/conf/proxy.conf --log=/var/log/codis/proxy.log --log-level=WARN &
```
