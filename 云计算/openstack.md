# 一、环境约定和基础包安装
## 1.1环境约定
* 提示：请先初步理解openstack的各组件功能，阅读此部署文档，文档中不做详细介绍。

|     主机名    | 管理IP   |  私有ip  |  公有ip  |  操作系统  |
| --------   | :-----:  | :----:  | ----- | ---- |
|controller|eth0:10.0.10.10|eth1:10.0.20.10|eth2:10.0.30.10| CenOS7.2-x64  |
|compute|eth0:10.0.10.20|eth1:10.0.20.20|eth2:10.0.30.20| CenOS7.2-x64  |
|cinder|eth0:10.0.10.30|eth1:10.0.20.30|---| CenOS7.2-x64  |

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
sed -i "s#mirror.centos.org#mirrors.163.com#g" /etc/yum.repos.d/CentOS-OpenStack-mitaka.repo 
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
## 3.4配置token
```shell
openstack-config --set /etc/keystone/keystone.conf DEFAULT admin_token $ADMIN_TOKEN
openstack-config --set /etc/keystone/keystone.conf token provider fernet
```
## 3.5配置mysql链接
```shell
openstack-config --set /etc/keystone/keystone.conf database connection mysql+pymysql://keystone:openstack@controller/keystone
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
### 3.8.1配置servername
```conf
ServerName controller
```
### 3.8.2配置keystone配置文件
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
### 3.8.3启动aapche
```shell
systemctl enable httpd.service
systemctl start httpd.service
```
## 3.9创建keyston服务和节点
### 3.9.1配置token环境变量
```shell
export OS_TOKEN=`echo $ADMIN_TOKEN`
export OS_URL=http://controller:35357/v3
export OS_IDENTITY_API_VERSION=3
```
### 3.9.2创建keystone服务
```shell
openstack service create --name keystone --description "OpenStack Identity" identity
```
### 3.9.3创建api
```shell
openstack endpoint create --region RegionOne identity public http://controller:5000/v3
openstack endpoint create --region RegionOne identity internal http://controller:5000/v3
openstack endpoint create --region RegionOne identity admin http://controller:35357/v3
```
## 3.10创建域、用户和角色
### 3.10.1创建默认域
```shell
openstack domain create --description "Default Domain" default
```
### 3.10.2创建admin项目
```shell
openstack project create --domain default --description "Admin Project" admin
```
### 3.10.3创建admin用户
```shell
openstack user create --domain default --password=admin admin
```
### 3.10.4创建admin角色
```shell
openstack role create admin
```
### 3.10.5添加admin角色到admin项目和用户
```shell
openstack role add --project admin --user admin admin
```
### 3.10.6创建service项目
```shell
openstack project create --domain default --description "Service Project" service
```
### 3.10.7创建demo项目
```shell
openstack project create --domain default --description "Demo Project" demo
```
### 3.10.8创建demo用户
```shell
openstack user create --domain default --password=demo demo
```
### 3.10.9创建user角色
```shell
openstack role create user
```
### 3.10.10添加user角色到demo 项目和用户
```shell
openstack role add --project demo --user demo user
```
##  3.11验证
### 3.11.1清空环境变量
```shell
unset OS_TOKEN OS_URL
```
### 3.11.2验证admin用户请求
```shell
openstack --os-auth-url http://controller:35357/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name admin --os-username admin token issue
```
### 3.11.3验证demo用户请求
```shell
openstack --os-auth-url http://controller:5000/v3 \
  --os-project-domain-name default --os-user-domain-name default \
  --os-project-name demo --os-username demo token issue
```
### 3.11.4生成admin用户脚本
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
### 3.11.5生成demo用户脚本
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
### 4.3.1创建glance用户
```shell
openstack user create --domain default --password=glance glance
```
### 4.3.2添加admin角色到glance用户和service项目上
```shell
openstack role add --project service --user glance admin
```
### 4.3.3创建glance服务
```shell
openstack service create --name glance --description "OpenStack Image" image
```
### 4.3.4创建galnce的api
```shell
openstack endpoint create --region RegionOne image public http://controller:9292
openstack endpoint create --region RegionOne image internal http://controller:9292
openstack endpoint create --region RegionOne image admin http://controller:9292
```
## 4.4安装配置glance
### 4.4.1安装glance
```shell
yum install -y openstack-glance
```
### 4.4.2配置glance-api的数据库连接
```shell
openstack-config --set /etc/glance/glance-api.conf database \
connection mysql+pymysql://glance:openstack@controller/glance
```
### 4.4.3配置glance-api的认证
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
### 4.4.4配置镜像存储位置
```shell
openstack-config --set /etc/glance/glance-api.conf glance_store stores file,http
openstack-config --set /etc/glance/glance-api.conf glance_store default_store file
openstack-config --set /etc/glance/glance-api.conf glance_store filesystem_store_datadir /var/lib/glance/images/
```
### 4.4.5配置glance-registry的数据库连接
```shell
openstack-config --set /etc/glance/glance-registry.conf database connection mysql+pymysql://glance:openstack@controller/glance
```
### 4.4.6配置glance-registry的认证
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
### 5.1.6安装nova
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
openstack-config --set /etc/nova/nova.conf vnc vncserver_listen 10.0.10.10
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address 10.0.10.10
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
openstack-config --set /etc/nova/nova.conf vnc vncserver_proxyclient_address 10.0.10.20
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
# 六、安装网络服务（neutron）
## 6.1安装控制节点（controller）
### 6.1.1创建数据库
```shell
mysql -uroot -popenstack -e"CREATE DATABASE neutron;"
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'openstack';"
```
### 6.1.2创建neutron用户
```shell
openstack user create --domain default --password=neutron neutron
```
### 6.1.3添加admin角色到neutron角色
```shell
openstack role add --project service --user neutron admin
```
### 6.1.4创建neutron服务
```shell
openstack service create --name neutron --description "OpenStack Networking" network
```
### 6.1.5创建neutron的api
```shell
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
```
### 6.1.6安装相关组件
```shell
yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```
### 6.1.7配置数据库连接
```shell
openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:openstack@controller/neutron
```
### 6.1.8配置使用ml2插件
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
```
### 6.1.9配置rabbitmq
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password openstack
```
### 6.1.10配置keystone
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password neutron
```
### 6.1.11配置nova
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf nova auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf nova auth_type password
openstack-config --set /etc/neutron/neutron.conf nova project_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova user_domain_name default
openstack-config --set /etc/neutron/neutron.conf nova region_name RegionOne
openstack-config --set /etc/neutron/neutron.conf nova project_name service
openstack-config --set /etc/neutron/neutron.conf nova username nova
openstack-config --set /etc/neutron/neutron.conf nova password nova
```
###  6.1.12配置锁路径
```shell
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```
### 6.1.13配置ml2
```shell
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks provider
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```
###  6.1.14配置Linuxbridge代理
```shell
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth2
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.20.10
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
### 6.1.15配置DHCP代理
```shell
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
```
###  6.1.16配置layer－3代理
```shell
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT external_network_bridge 
```
### 6.1.17配置nova
```shell
openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696
openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf neutron auth_type password
openstack-config --set /etc/nova/nova.conf neutron project_domain_name default
openstack-config --set /etc/nova/nova.conf neutron user_domain_name default
openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
openstack-config --set /etc/nova/nova.conf neutron project_name service
openstack-config --set /etc/nova/nova.conf neutron username neutron
openstack-config --set /etc/nova/nova.conf neutron password neutron
openstack-config --set /etc/nova/nova.conf neutron service_metadata_proxy True
openstack-config --set /etc/nova/nova.conf neutron metadata_proxy_shared_secret METADATA_SECRET
```
### 6.1.18配置元数据代理
```shell
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET
```
### 6.1.19创建软连接
```shell
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
### 6.1.20同步数据库
```shell
su -s /bin/sh -c "neutron-db-manage --config-file /etc/neutron/neutron.conf \
  --config-file /etc/neutron/plugins/ml2/ml2_conf.ini upgrade head" neutron
```
### 6.1.21重启nova-api
```shell
systemctl restart openstack-nova-api.service
```
### 6.1.22启动neutron
```shell
systemctl enable neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl start neutron-server.service \
  neutron-linuxbridge-agent.service neutron-dhcp-agent.service \
  neutron-metadata-agent.service
systemctl enable neutron-l3-agent.service
systemctl start neutron-l3-agent.service
```
## 6.2安装计算节点（compute）
### 6.2.1安装组件
```shell
yum install -y openstack-neutron-linuxbridge ebtables ipset
```
### 6.2.2配置rabbitmq
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password openstack
```
### 6.2.3配置keystone
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password neutron
```
### 6.2.4配置锁路径
```shell
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```
### 6.2.5配置Linuxbridge代理
```shell
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings provider:eth2
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.20.20
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
### 6.2.6配置nova
```shell
openstack-config --set /etc/nova/nova.conf neutron url http://controller:9696
openstack-config --set /etc/nova/nova.conf neutron auth_url http://controller:35357
openstack-config --set /etc/nova/nova.conf neutron auth_type password
openstack-config --set /etc/nova/nova.conf neutron project_domain_name default
openstack-config --set /etc/nova/nova.conf neutron user_domain_name default
openstack-config --set /etc/nova/nova.conf neutron region_name RegionOne
openstack-config --set /etc/nova/nova.conf neutron project_name service
openstack-config --set /etc/nova/nova.conf neutron username neutron
openstack-config --set /etc/nova/nova.conf neutron password neutron
```
### 6.2.7重启nova-api
```shell
systemctl restart openstack-nova-compute.service
```
### 6.2.8启动Linuxbridge代理
```shell
systemctl enable neutron-linuxbridge-agent.service
systemctl start neutron-linuxbridge-agent.service
```
### 6.2.9 验证安装
```shell
root@controller ~ 20:04:49 # neutron agent-list
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host       | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
| 14259fcd-448f-4fb6-ba30-fb5885ebe878 | Metadata agent     | controller |                   | :-)   | True           | neutron-metadata-agent    |
| 1994d93b-f6cf-4dd3-8a2a-3623b473c675 | L3 agent           | controller | nova              | :-)   | True           | neutron-l3-agent          |
| 9c44da60-3e2a-466e-9185-9c30840ab699 | DHCP agent         | controller | nova              | :-)   | True           | neutron-dhcp-agent        |
| afd1d41d-d83d-46ec-b916-e5f9e6d3725e | Linux bridge agent | compute    |                   | :-)   | True           | neutron-linuxbridge-agent |
| e5006fdb-b45d-4d9c-b129-6fefbad8a3d5 | Linux bridge agent | controller |                   | :-)   | True           | neutron-linuxbridge-agent |
+--------------------------------------+--------------------+------------+-------------------+-------+----------------+---------------------------+
```
### 6.2.10创建网络
```shell
neutron net-create --shared --provider:physical_network provider \
  --provider:network_type flat provider
neutron subnet-create --name provider \
  --allocation-pool start=10.0.30.100,end=10.0.30.200 \
  --dns-nameserver 8.8.4.4 --gateway 10.0.30.1 \
  provider 10.0.30.0/24
neutron net-update provider --router:external
```
# 七、安装dashboard
## 7.1安装
```shell
yum install openstack-dashboard -y
```
## 7.2配置
```conf
sed -i "s#OPENSTACK_HOST = \"127.0.0.1\"#OPENSTACK_HOST = \"controller\"#g" /etc/openstack-dashboard/local_settings
sed -i "s#ALLOWED_HOSTS = \['horizon.example.com', 'localhost'\]#ALLOWED_HOSTS = \['*', \]#g" /etc/openstack-dashboard/local_settings
echo "SESSION_ENGINE = 'django.contrib.sessions.backends.cache'" >> /etc/openstack-dashboard/local_settings
sed -i "s@#CACHES = {@CACHES = {@g" /etc/openstack-dashboard/local_settings
sed -i "s@#    'default': {@    'default': {@g" /etc/openstack-dashboard/local_settings
sed -i "s@#        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',@        'BACKEND': 'django.core.cache.backends.memcached.MemcachedCache',@g" /etc/openstack-dashboard/local_settings
sed -i "s@#        'LOCATION': '127.0.0.1:11211',@        'LOCATION': 'controller:11211',@g" /etc/openstack-dashboard/local_settings
sed -i "s@#    },@    },@g" /etc/openstack-dashboard/local_settings
sed -i "s@#OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = False@OPENSTACK_KEYSTONE_MULTIDOMAIN_SUPPORT = True@g" /etc/openstack-dashboard/local_settings
sed -i "s@OPENSTACK_KEYSTONE_DEFAULT_ROLE = \"_member_\"@OPENSTACK_KEYSTONE_DEFAULT_ROLE = \"user\"@g" /etc/openstack-dashboard/local_settings
sed -i "s@#OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'default'@OPENSTACK_KEYSTONE_DEFAULT_DOMAIN = 'default'@g" /etc/openstack-dashboard/local_settings
sed -i "s@#OPENSTACK_API_VERSIONS@OPENSTACK_API_VERSIONS@g" /etc/openstack-dashboard/local_settings
sed -i "s@#    \"identity\": 3,@    \"identity\": 3,@g" /etc/openstack-dashboard/local_settings
sed -i "s@#    \"volume\": 2@    \"volume\": 2@g" /etc/openstack-dashboard/local_settings
sed -i "s@#    \"compute\": 2,@    \"compute\": 2,@g" /etc/openstack-dashboard/local_settings
```
### 7.3重启apache
```shell
systemctl restart httpd
```
# 八、安装存储服务（cinder）
## 8.1安装controller节点
### 8.1.1创建数据库
```
mysql -uroot -popenstack -e"CREATE DATABASE cinder;"
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'controller' \
  IDENTIFIED BY 'openstack';:
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON cinder.* TO 'cinder'@'%' \
  IDENTIFIED BY 'openstack';"
```
### 8.1.2创建cinder用户
```shell
openstack user create --domain default --password=cinder cinder
```
### 8.1.3添加admin角色到cinder用户
```shell
openstack role add --project service --user cinder admin
```
### 8.1.4创建cinder服务
```shell
openstack service create --name cinder --description "OpenStack Block Storage" volume
openstack service create --name cinderv2 --description "OpenStack Block Storage" volumev2
```
### 8.1.5创建cindr的api
```shell
openstack endpoint create --region RegionOne \
  volume public http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume internal http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volume admin http://controller:8776/v1/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 public http://controller:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 internal http://controller:8776/v2/%\(tenant_id\)s
openstack endpoint create --region RegionOne \
  volumev2 admin http://controller:8776/v2/%\(tenant_id\)s
```
### 8.1.6安装cinder
```shell
yum install -y openstack-cinder
```
### 8.1.7配置数据库
```shell
openstack-config --set /etc/cinder/cinder.conf database connection = mysql+pymysql://cinder:openstack@controller/cinder
```
### 8.1.8配置rabbitmq
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_password openstack
```
### 8.1.9配置keystone
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
openstack-config --set /etc/neutron/cinder/cinder.conf keystone_authtoken password cinder
```
### 8.1.10配置ip和锁路径
```shell
openstack-config --set /etc/neutron/cinder/cinder.conf DEFAULT my_ip 10.0.20.30
openstack-config --set /etc/neutron/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp
```
### 8.1.11初始化数据库
```shell
su -s /bin/sh -c "cinder-manage db sync" cinder
```
### 8.1.12配置nova使用cinder
```shell
penstack-config --set /etc/neutron/nova/nova.conf cinder os_region_name RegionOne
```
### 8.1.13重启nova-api
```shell
systemctl restart openstack-nova-api.service
```
### 8.1.14启动cinder
```shell
systemctl enable openstack-cinder-api.service openstack-cinder-scheduler.service
systemctl start openstack-cinder-api.service openstack-cinder-scheduler.service
```
## 8.2安装存储节点
### 8.2.1安装lvm
```shell
yum install lvm2 -y
```
### 8.2.2启动lvm
```shell
systemctl enable lvm2-lvmetad.service
systemctl start lvm2-lvmetad.service
```
### 8.2.3配置物理卷并创建卷组
```shell
pvcreate /dev/sdb
vgcreate cinder-volumes /dev/sdb
```
### 8.2.4配置lvm(/etc/lvm/lvm.conf)
```conf
devices {
...
filter = [ "a/sdb/", "r/.*/"]
```
### 8.2.4安装cinder
```shell
yum install openstack-cinder targetcli python-keystone -y
```
### 8.2.5配置rabbitmq
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/cinder/cinder.conf oslo_messaging_rabbit rabbit_password openstack
```
### 8.2.6配置keystone
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken auth_type password
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken project_name service
openstack-config --set /etc/cinder/cinder.conf keystone_authtoken username cinder
openstack-config --set /etc/neutron/cinder/cinder.conf keystone_authtoken password cinder
```
### 8.2.7配置ip和锁路径
```shell
openstack-config --set /etc/neutron/cinder/cinder.conf DEFAULT my_ip 10.0.20.30
openstack-config --set /etc/neutron/cinder/cinder.conf oslo_concurrency lock_path /var/lib/cinder/tmp
```
### 8.2.8配置cinder使用lvm
```shell
openstack-config --set /etc/neutron/cinder/cinder.conf lvm volume_driver cinder.volume.drivers.lvm.LVMVolumeDriver
openstack-config --set /etc/neutron/cinder/cinder.conf lvm volume_group cinder-volumes
openstack-config --set /etc/neutron/cinder/cinder.conf lvm iscsi_protocol iscsi
openstack-config --set /etc/neutron/cinder/cinder.conf lvm iscsi_helper lioadm
openstack-config --set /etc/neutron/cinder/cinder.conf DEFAULT enabled_backends lvm
```
### 8.2.9启动cinder
```shell
systemctl enable openstack-cinder-volume.service target.service
systemctl start openstack-cinder-volume.service target.service
```
# 九、dashboard操作
## 9.1登录

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/01.png?raw=true)

## 9.2创建网络

### 9.2.1创建网络

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/02.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/03.png?raw=true)

### 9.2.2创建子网

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/04.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/05.png?raw=true)

## 9.3创建路由

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/06.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/07.png?raw=true)

## 9.4路由连接网络

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/08.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/09.png?raw=true)

## 9.5清空默认网络规则并先暂时允许所有

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/10.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/11.png?raw=true)

## 9.6创建云主机

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/12.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/13.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/14.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/15.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/16.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/17.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/18.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/19.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/20.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/21.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/22.png?raw=true)

## 9.7创建云硬盘

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/23.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/24.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/25.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/26.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/27.png?raw=true)

## 9.8格式化并挂载

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/28.png?raw=true)

![iamge](https://github.com/yangzinan/Operations/blob/master/iamge/openstack/29.png?raw=true)


