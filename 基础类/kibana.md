# Kibana
#### By：大官人
#### Email：DaGuanR@gmail.com
#### QQ：375739049

# 一.安装jdk

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

# 二.安装kibana

```shell
tar zxf kibana-5.1.1.tar.gz -C /usr/local
ln -s /usr/local/kibana-5.1.1 /usr/local/kibana
```

# 三.安装-xpack

```shell
/usr/local/kibana/bin/kibana-plugin install file:///usr/local/src/x-pack-5.1.1.zip
```

# 四.修改密码

```shell
修改 elastic 的密码：
[root@controller config]# Curl XPUT -u elastic:changeme '192.168.63.246:9200/_x
pack/security/user/elastic/_password' -d '{
 "password" : "123456"
}'
###现在 elastic 的密码已经变成了 123,在改一下 kibana 的登录密码：

[root@controller config]# curl -XPUT -u elastic:123456 '192.168.63.246:9200/_xp
ack/security/user/kibana/_password' -d '{
 "password" : "123456"
}'
```

# 五.修改配置文件

```conf
server.host: "192.168.63.246"
elasticsearch.url: http://192.168.63.246:9200
elasticsearch.username: "elastic"
elasticsearch.password: "123456 ##x-pack 默认超级用户的登录密码(es 和
kibana 两个的超级管理员账号密码都一样)
```