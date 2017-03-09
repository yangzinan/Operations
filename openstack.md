# 一、环境约定和基础包安装
## 1.1环境约定
* 提示：请先初步理解openstack的各组件功能，阅读此部署文档，文档中不做详细介绍。

|     主机名    | 管理IP   |  私有ip  |  公有ip  |  操作系统  |
| --------   | :-----:  | :----:  | ----- | ---- |
|controller|eth0:10.0.10.10|---|---| CenOS7.2-x64  |
|compute|eth0:10.0.10.20|eth1:10.0.20.20|---| CenOS7.2-x64  |
|neutron|eth0:10.0.10.30|eth1:10.0.20.30|eth2:10.0.30.30| CenOS7.2-x64  |
|cinder|eth0:10.0.10.40|eth1:10.0.20.40|---| CenOS7.2-x64  |
## 1.2环境配置（在所有节点执行）
```shell
systemctl stop firewalld.service
systemctl disable firewalld.service
yum install -y wget
mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.back
wget http://mirrors.aliyun.com/repo/Centos-7.repo  -P /etc/yum.repos.d/
mv /etc/yum.repos.d/Centos-7.repo /etc/yum.repos.d/CentOS-Base.repo
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
yum install -y epel-release 
yum install -y centos-release-openstack-mitaka
sed -i "s@#baseurl@baseurl@g" /etc/yum.repos.d/epel.repo
sed -i "s@mirrorlist@#mirrorlist@g"  /etc/yum.repos.d/epel.repo
sed -i "s#http://download.fedoraproject.org/pub#https://mirrors.tuna.tsinghua.edu.cn#g" /etc/yum.repos.d/epel.repo
sed -i "s#http://elrepo.org/linux#https://mirror.tuna.tsinghua.edu.cn/elrepo#g" /etc/yum.repos.d/elrepo.repo 
yum clean all
yum makecache
yum install -y tree lrzsz nmap python-pip vim net-tools ntp iperf iftop screen zip unzip gcc gcc-c++ cmake
yum install -y python-openstackclient openstack-selinux ntp openstack-utils
systemctl start ntpd
systemctl enable ntpd
setenforce 0
sed -i 's#SELINUX=enforcing#SELINUX=disable#g' /etc/selinux/config
```
## 1.3选择执行（可以不执行）
```shell
cp /etc/vimrc /etc/vimrc.back
cat>>/etc/vimrc<<EOF
set nu
set tabstop=2
set fileencodings=utf-8,ucs-bom,gb18030,gbk,gb2312,cp936
set termencoding=utf-8
set encoding=utf-8
EOF
. /etc/vimrc
echo "export LC_ALL=C" >> /etc/profile
echo "unset MAILCHECK" >> /etc/profile
echo "export PS1='\[\e[31;1m\]\u@\[\e[34;1m\]\h \[\e[36;1m\]\w \[\e[33;1m\]\t # \[\e[37;1m\]'" >> /etc/profile
echo "export GREP_OPTIONS='--color=auto' GREP_COLOR='1;33'" >> /etc/profile
echo " *  -  nofile  65536" >> /etc/security/limits.conf
source /etc/profile
```
# 二、controller(控制节点)部署
## 2.1安装配置mysql
### 2.1.1安装
```shell
yum install -y mariadb mariadb-server MySQL-python
```
```conf
[mysqld]
bind-address = 0.0.0.0
default-storage-engine = innodb
innodb_file_per_table
collation-server = utf8_general_ci
init-connect = 'SET NAMES utf8'
character-set-server = utf8
```
### 2.1.2启动mysql
```shell
systemctl enable mariadb.service
systemctl start mariadb.service
```
### 2.1.3配置mysql密码
```shell
mysqladmin -uroot password "openstack"
```
## 2.2安装配置mongodb
### 2.2.1安装
```shell
yum install -y mongodb-server mongodb
```
### 2.2.2配置mongod（/etc/mongod.conf ）
```conf
bind_ip = 0.0.0.0
smallfiles = true
```
### 2.2.3启动mongodb
```shell
systemctl enable mongod.service
systemctl start mongod.service
```
## 2.3安装配置rabbitmq
### 2.3.1安装rabbitmq
```shell
yum install -y rabbitmq-server
```
### 启动rabbitmq
```shell
systemctl enable rabbitmq-server.service
systemctl start rabbitmq-server.service
```
### 2.3.3配置rabbitmq
```shell
rabbitmqctl add_user openstack openstack
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```
## 2.4安装memcached
### 2.4.1安装memcached
```shell
yum install -y memcached python-memcached
```
### 启动memcached
```shell
systemctl enable memcached.service
systemctl start memcached.service
```
# 三、安装keystone（认证服务，在controller）
## 3.1为keystone创建mysql数据库和表
```shell
mysql -uroot -popenstack -e "CREATE DATABASE keystone;"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'controller' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' IDENTIFIED BY 'openstack';"
```
## 3.2生成随机令牌
```shell
ADMIN_TOKEN=$(openssl rand -hex 10)
```
## 3.3安装keystone和相关组件
```shell
yum install -y openstack-keystone httpd mod_wsgi
```
## 3.4配置tuken
```shell
openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token $ADMIN_TOKEN
openstack-config --set /etc/keystone/keystone.conf token provider fernet
```
## 3.5配置mysql链接
```shell
openstack-config --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:openstackd@controller/keystone
```
## 3.6初始化数据库
```shell
su -s /bin/sh -c "keystone-manage db_sync" keystone
```
## 3.7初始化fernet keys
```shell
keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
```
## 3.8配置apache
### 1.配置servername
```conf
ServerName controller
```
### 2.配置keystone配置文件
```shell
cat>>/etc/httpd/conf.d/wsgi-keystone.conf<<EOF
Listen 5000
Listen 35357

<VirtualHost *:5000>
    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-public
    WSGIScriptAlias / /usr/bin/keystone-wsgi-public
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>

<VirtualHost *:35357>
    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}
    WSGIProcessGroup keystone-admin
    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin
    WSGIApplicationGroup %{GLOBAL}
    WSGIPassAuthorization On
    ErrorLogFormat "%{cu}t %M"
    ErrorLog /var/log/httpd/keystone-error.log
    CustomLog /var/log/httpd/keystone-access.log combined

    <Directory /usr/bin>
        Require all granted
    </Directory>
</VirtualHost>
EOF
```
### 3.启动aapche
```shell
systemctl enable httpd.service
systemctl start httpd.service
```
## 3.9创建keyston服务和节点
### 1.配置token环境变量
```shell
export OS_TOKEN=`echo $ADMIN_TOKEN`
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
### 2.创建keystone服务
```shell
openstack service create --name keystone --description "OpenStack Identity" identity
```
### 3.创建api
```shell
openstack endpoint create --region RegionOne identity public http://controller:5000/v3
openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
openstack endpoint create --region RegionOne identity admin http://controller:35357/v3
```
## 3.10创建域、用户和角色
### 1.创建默认域
```shell
openstack domain create --description "Default Domain" default
```
### 2.创建admin项目
```shell
openstack project create --domain default --description "Admin Project" admin
```
### 3.创建admin用户
```shell
openstack user create --domain default --password=admin admin
```
### 4.创建admin角色
```shell
openstack role create admin
```
### 5.添加admin角色到admin项目和用户
```shell
openstack role add --project admin --user admin admin
```
###  6.创建service项目
```shell
openstack project create --domain default --description "Service Project" service
```
### 7.创建demo项目
```shell
openstack project create --domain default --description "Demo Project" demo
```
### 8.创建demo用户
```shell
openstack user create --domain default --password=demo demo
```
### 9.创建user角色
```shell
openstack role create user
```
### 10.添加user角色到demo 项目和用户
```shell
openstack role add --project demo --user demo user
```
##  3.11验证
### 1.清空环境变量
```shell
unset OS_TOKEN OS_URL
```
### 2.验证admin用户请求
```shell
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin token issue
```
### 3.验证demo用户请求
```shell
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue
```
### 4.生成admin用户脚本
```shell
cat>>~/admin.rc<<EOF
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=admin
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_AUTH_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```
### 5.生成demo用户脚本
```shell
cat>>~/demo.rc<<EOF
export OS_PROJECT_DOMAIN_NAME=default
export OS_USER_DOMAIN_NAME=default
export OS_PROJECT_NAME=demo
export OS_USERNAME=demo
export OS_PASSWORD=DEMO_PASS
export OS_AUTH_URL=http://controller:5000/v3
export OS_IDENTITY_API_VERSION=3
export OS_IMAGE_API_VERSION=2
EOF
```
# 四、镜像服务（glance，在controller）
## 4.1创建数据库
```shell
mysql -uroot -popenstack -e "CREATE DATABASE glance;"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'controller' \
  IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON glance.* TO 'glance'@'%' \
  IDENTIFIED BY 'openstack';"
```
## 4.2获取admin权限
```shell
source ~/admin.rc
```
## 4.3创建服务和api
### 1.创建glance用户
```shell
openstack user create --domain default --password=glance glance
```
###  2.添加admin角色到glance用户和service项目上
```shell
openstack role add --project service --user glance admin
```
### 3.创建glance服务
```shell
openstack service create --name glance --description "OpenStack Image" image
```
### 4.创建galnce的api
```shell
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```
## 4.4安装配置glance
### 1.安装glance
```shell
yum install -y openstack-glance
```
### 2.配置glance-api的数据库连接
```shell
openstack-config --set /etc/glance/glance-api.conf database \
connection mysql+pymysql://glance:openstack@controller/glance
```
### 3.配置glance-api的认证
```shell
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-api.conf keystone_authtoken password glance
openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone 
```
### 4.配置镜像存储位置
```shell
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```
### 5.配置glance-registry的数据库连接
```shell
openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:glance@controller/glance
```
### 6.配置glance-registry的认证
```shell
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken auth_type password
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken project_name service
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken username glance
openstack-config --set /etc/glance/glance-registry.conf keystone_authtoken password glance
openstack-config --set /etc/glance/glance-registry.conf paste_deploy flavor keystone 
```
## 4.5同步数据库
```shell
su -s /bin/sh -c "glance-manage db_sync" glance
```
## 4.6启动glance
```shell
systemctl enable openstack-glance-api.service \
  openstack-glance-registry.service
systemctl start openstack-glance-api.service \
  openstack-glance-registry.service
```
## 4.7验证操作
```shell
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
 #查看上传镜像
 root@controller ~ 08:20:16 # openstack image list
+--------------------------------------+--------+--------+
| ID                                   | Name   | Status |
+--------------------------------------+--------+--------+
| 293a2b7f-7338-4900-90b7-2c1d6df56021 | cirros | active |
+--------------------------------------+--------+--------+
```
# 五、安装compute（计算服务）
## 5.1安装配置控制节点（controller）
### 5.1.1创建数据库
```shell
mysql -uroot -popenstack -e "CREATE DATABASE nova_api;"
mysql -uroot -popenstack -e "CREATE DATABASE nova;"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'controller' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON nova_api.* TO 'nova'@'%' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'controller' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e "GRANT ALL PRIVILEGES ON nova.* TO 'nova'@'%' IDENTIFIED BY 'openstack';"
```
### 5.1.2创建nova用户
```shell
openstack user create --domain default --password=nova nova
```
### 5.1.3给nova用户添加admn角色
```shell
openstack role add --project service --user nova admin
```
### 5.1.4创建nova服务
```shell
openstack service create --name nova --description "OpenStack Compute" compute
```
### 5.1.5创建nova的api
```shell
openstack endpoint create --region RegionOne compute public http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute internal http://controller:8774/v2.1/%\(tenant_id\)s
openstack endpoint create --region RegionOne compute admin http://controller:8774/v2.1/%\(tenant_id\)s
```
### 5.1.6a安装nova
```shell
yum install openstack-nova-api openstack-nova-conductor \
  openstack-nova-console openstack-nova-novncproxy \
  openstack-nova-scheduler
```
### 5.1.7配置nova的default
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT enabled_apis osapi_compute,metadata
```
### 5.1.8配合nova数据库连接
```shell
openstack-config --set /etc/nova/nova.conf \
api_database connection mysql+pymysql://nova:openstack@controller/nova_api
openstack-config --set /etc/nova/nova.conf \
database connection mysql+pymysql://nova:openstack@controller/nova
```
### 5.1.9配置rabbitmq
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password openstack
```
### 5.1.10配置keystone认证
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password nova
```
### 5.1.11配置ip和网络
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.0.10.10
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
```
### 5.1.12配置vnc和glance
```shell
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen $my_ip
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address $my_ip
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
```
### 5.1.13配置锁路径
```shell
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
```
### 5.1.14同步数据库
```shell
su -s /bin/sh -c "nova-manage api_db sync" nova
su -s /bin/sh -c "nova-manage db sync" nova
```
### 5.1.15启动controller上的nova
```shell
systemctl enable openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service
systemctl start openstack-nova-api.service \
  openstack-nova-consoleauth.service openstack-nova-scheduler.service \
  openstack-nova-conductor.service openstack-nova-novncproxy.service

```
### 5.1.16验证
```shell
root@controller ~ 10:20:52 # openstack compute service list
+----+------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-conductor   | controller | internal | enabled | up    | 2017-03-09T02:21:25.000000 |
|  2 | nova-consoleauth | controller | internal | enabled | up    | 2017-03-09T02:21:25.000000 |
|  3 | nova-scheduler   | controller | internal | enabled | up    | 2017-03-09T02:21:25.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
root@controller ~ 10:21:31 # 
```
## 5.2计算节点安装（compute）
### 5.2.1 安装nova-compute
```shell
yum install -y openstack-nova-compute
```
### 5.2.2配置rabbitmq链接
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/nova/nova.conf oslo_messaging_rabbit rabbit_password openstack
```
### 5.2.3配置keystone认证
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/nova/nova.conf keystone_authtoken auth_type password
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/nova/nova.conf keystone_authtoken project_name service
openstack-config --set /etc/nova/nova.conf keystone_authtoken username nova
openstack-config --set /etc/nova/nova.conf keystone_authtoken password nova
```
### 5.2.4配置IP和网络
```shell
openstack-config --set /etc/nova/nova.conf DEFAULT my_ip 10.0.10.20
openstack-config --set /etc/nova/nova.conf DEFAULT use_neutron True
openstack-config --set /etc/nova/nova.conf DEFAULT firewall_driver nova.virt.firewall.NoopFirewallDriver
```
### 5.2.5配置vnc
```shell
openstack-config --set /etc/nova/nova.conf vnc enabled True
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 0.0.0.0
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address $my_ip
openstack-config --set /etc/nova/nova.conf vnc novncproxy_base_url http://controller:6080/vnc_auto.html
```
### 5.2.6配置glance
```shell
openstack-config --set /etc/nova/nova.conf glance api_servers http://controller:9292
```
### 5.2.7配置所路径
```shell
openstack-config --set /etc/nova/nova.conf oslo_concurrency lock_path /var/lib/nova/tmp
```
### 5.2.8配置硬件加速
```shell
egrep -c '(vmx|svm)' /proc/cpuinfo
#如果返回0则执行下面命令非0不做任何操作
openstack-config --set /etc/nova/nova.conf libvirt virt_type qemu
```
### 5.2.9启动nova-compute
```shell
systemctl enable libvirtd.service openstack-nova-compute.service
systemctl start libvirtd.service openstack-nova-compute.service
```
### 5.2.10验证安装
```shell
#在controller节点执行
root@controller ~ 11:10:24 # openstack compute service list      
+----+------------------+------------+----------+---------+-------+----------------------------+
| Id | Binary           | Host       | Zone     | Status  | State | Updated At                 |
+----+------------------+------------+----------+---------+-------+----------------------------+
|  1 | nova-conductor   | controller | internal | enabled | up    | 2017-03-09T03:10:46.000000 |
|  2 | nova-consoleauth | controller | internal | enabled | up    | 2017-03-09T03:10:46.000000 |
|  3 | nova-scheduler   | controller | internal | enabled | up    | 2017-03-09T03:10:55.000000 |
|  6 | nova-compute     | compute    | nova     | enabled | up    | 2017-03-09T03:10:55.000000 |
+----+------------------+------------+----------+---------+-------+----------------------------+
root@controller ~ 11:10:55 # 
```