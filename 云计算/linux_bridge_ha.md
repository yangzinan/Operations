# 一、控制节点的安装
## 1.1创建数据库
```shell
mysql -uroot -popenstack -e"CREATE DATABASE neutron;"
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'controller' IDENTIFIED BY 'openstack';"
mysql -uroot -popenstack -e"GRANT ALL PRIVILEGES ON neutron.* TO 'neutron'@'%' IDENTIFIED BY 'openstack';"
```
## 1.2创建用户相关
```shell
openstack user create --domain default --password=neutron neutron
openstack role add --project service --user neutron admin
```
## 1.3创建neutron服务API
```shell
openstack service create --name neutron --description "OpenStack Networking" network
openstack endpoint create --region RegionOne network public http://controller:9696
openstack endpoint create --region RegionOne network internal http://controller:9696
openstack endpoint create --region RegionOne network admin http://controller:9696
```
## 1.4安装controller相关组件
```shell
yum install -y openstack-neutron python-neutronclient
```
## 1.5配置neutron
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy = keystone
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_status_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT notify_nova_on_port_data_changes True
openstack-config --set /etc/neutron/neutron.conf DEFAULT router_distributed False
openstack-config --set /etc/neutron/neutron.conf DEFAULT l3_ha True
openstack-config --set /etc/neutron/neutron.conf DEFAULT l3_ha_net_cidr 169.254.192.0/18
openstack-config --set /etc/neutron/neutron.conf DEFAULT max_l3_agents_per_router 3
openstack-config --set /etc/neutron/neutron.conf DEFAULT min_l3_agents_per_router 2
openstack-config --set /etc/neutron/neutron.conf DEFAULT dhcp_agents_per_network 2
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password openstack
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password neutron
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
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:openstack@controller/neutron
```
## 1.5配置ml2
```shell
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,gre,vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan  vni_ranges 1:1000
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks external
```
## 1.6配置nova-api
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
## 1.7软件连接和初始化数据库
```shell
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
neutron-db-manage --config-file /etc/neutron/neutron.conf \
--config-file /etc/neutron/plugins/ml2/ml2_conf.ini \
upgrade head
```
## 1.8重启nova-api启动neutron
```shell
systemctl restart openstack-nova-api.service
systemctl enable neutron-server.service
systemctl start neutron-server.service
```
# 二、网络节点安装

* 注：两台network节点都按如下配置，注意修改桥接网卡名和local_ip

## 2.1配置内核
```shell
cat>>/etc/sysctl.conf<<EOF
net.ipv4.ip_forward=1
net.ipv4.conf.all.rp_filter=0
net.ipv4.conf.default.rp_filter=0
EOF
sysctl -p
```
## 2.2安装相关组件
```shell
yum install -y openstack-neutron openstack-neutron-ml2 openstack-neutron-linuxbridge ebtables
```
## 2.3配置neutron
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT verbose True
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf DEFAULT auth_strategy keystone
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password openstack
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_uri http://controller:5000
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_url http://controller:35357
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken memcached_servers controller:11211
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken auth_type password
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken user_domain_name default
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken project_name service
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken username neutron
openstack-config --set /etc/neutron/neutron.conf keystone_authtoken password neutron
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
openstack-config --set /etc/neutron/neutron.conf database connection mysql+pymysql://neutron:openstack@controller/neutron
```
## 2.4配置ml2
```shell
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 type_drivers flat,vlan,gre,vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 tenant_network_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 mechanism_drivers linuxbridge,l2population
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2 extension_drivers port_security
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_flat flat_networks external
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini ml2_type_vxlan vni_ranges 1:1000
openstack-config --set /etc/neutron/plugins/ml2/ml2_conf.ini securitygroup enable_ipset True
```
## 2.5配置linux_bridge
```shell
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini linux_bridge physical_interface_mappings external:eth1
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.20.30
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan true
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent tunnel_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent prevent_arp_spoofing True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_ipset =True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
## 2.6配置l3
```shell
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT verbose True
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT use_namespaces True
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT external_network_bridge
openstack-config --set /etc/neutron/l3_agent.ini DEFAULT agent_mode legacy
```
### 2.7配置dhcp
```shell
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT interface_driver neutron.agent.linux.interface.BridgeInterfaceDriver
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT dhcp_driver neutron.agent.linux.dhcp.Dnsmasq
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT enable_isolated_metadata True
openstack-config --set /etc/neutron/dhcp_agent.ini DEFAULT verbose True
```
## 2.8配置metadata
```shell
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT 
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT nova_metadata_ip controller
openstack-config --set /etc/neutron/metadata_agent.ini DEFAULT metadata_proxy_shared_secret METADATA_SECRET
```
## 2.9生成软连接
```shell
ln -s /etc/neutron/plugins/ml2/ml2_conf.ini /etc/neutron/plugin.ini
```
## 2.20启动network
```shell
systemctl enable neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-l3-agent.service
systemctl start neutron-linuxbridge-agent.service neutron-dhcp-agent.service neutron-metadata-agent.service neutron-l3-agent.service
```
# 三、compute配置
## 3.1安装
```shell
yum install -y openstack-neutron-linuxbridge ebtables ipset
```
## 3.1配置rabbitmq
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT rpc_backend rabbit
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_host controller
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_userid openstack
openstack-config --set /etc/neutron/neutron.conf oslo_messaging_rabbit rabbit_password openstack
```
## 3.2配置keystone
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
## 3.3配置neutron
```shell
openstack-config --set /etc/neutron/neutron.conf DEFAULT verbose True
openstack-config --set /etc/neutron/neutron.conf DEFAULT core_plugin ml2
openstack-config --set /etc/neutron/neutron.conf DEFAULT service_plugins router
openstack-config --set /etc/neutron/neutron.conf DEFAULT allow_overlapping_ips True
openstack-config --set /etc/neutron/neutron.conf oslo_concurrency lock_path /var/lib/neutron/tmp
```
## 3.4配置Linuxbridge代理
```shell
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent tunnel_types vxlan
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini agent prevent_arp_spoofing True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan enable_vxlan True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan local_ip 10.0.20.20
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini vxlan l2_population True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup enable_security_group True
openstack-config --set /etc/neutron/plugins/ml2/linuxbridge_agent.ini securitygroup firewall_driver neutron.agent.linux.iptables_firewall.IptablesFirewallDriver
```
## 3.5配置nova
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
## 3.6重启nova-api
```
systemctl restart openstack-nova-compute.service
```
## 3.7启动Linuxbridge代理
```
systemctl enable neutron-linuxbridge-agent.service
systemctl start neutron-linuxbridge-agent.service
```
# 四、验证
##  4.1验证neutron-agent
```
root@controller01 ~ 16:38:09 # neutron agent-list
+--------------------------------------+--------------------+-----------+-------------------+-------+----------------+---------------------------+
| id                                   | agent_type         | host      | availability_zone | alive | admin_state_up | binary                    |
+--------------------------------------+--------------------+-----------+-------------------+-------+----------------+---------------------------+
| 03c17f9a-c05d-49db-9f59-54c24fa0dc51 | L3 agent           | network02 | nova              | :-)   | True           | neutron-l3-agent          |
| 1b643095-aeb1-4328-a4eb-350b979d10e0 | Linux bridge agent | network02 |                   | :-)   | True           | neutron-linuxbridge-agent |
| 471e9d70-af6d-460e-910a-587c6f70518d | Linux bridge agent | compute01 |                   | :-)   | True           | neutron-linuxbridge-agent |
| 52f4acc1-7881-4d45-b10c-df46d00a4718 | L3 agent           | network01 | nova              | :-)   | True           | neutron-l3-agent          |
| 6e45ab4a-6435-4b20-af7e-069ee2edb980 | DHCP agent         | network01 | nova              | :-)   | True           | neutron-dhcp-agent        |
| 848dd89e-1b61-421c-9c9c-56ebc88d45f0 | Linux bridge agent | network01 |                   | :-)   | True           | neutron-linuxbridge-agent |
| 8f284cf2-1665-4465-bfe3-f0d3c0194345 | DHCP agent         | network02 | nova              | :-)   | True           | neutron-dhcp-agent        |
| b7b4d967-787b-402a-b1a5-6da2137f1eea | Metadata agent     | network02 |                   | :-)   | True           | neutron-metadata-agent    |
| eeece954-a433-40ef-8dbd-3cde646a5b84 | Metadata agent     | network01 |                   | :-)   | True           | neutron-metadata-agent    |
+--------------------------------------+--------------------+-----------+-------------------+-------+----------------+---------------------------+
```
## 4.2创建网络
```
neutron net-create ext-net --router:external \
--provider:physical_network external \
--provider:network_type flat

neutron subnet-create ext-net 10.0.30.0/24 \
--allocation-pool start=10.0.30.101,end=10.0.30.150 \
--disable-dhcp --gateway 10.0.30.1 \
--name ext-subnet

neutron net-create admin-net

neutron subnet-create admin-net 192.168.1.0/24 \
--gateway 192.168.1.1 \
--dns-nameserver 8.8.8.8 \
--name admin-subnet

neutron router-create admin-router
neutron router-interface-add admin-router admin-subnet
neutron router-gateway-set admin-router ext-net
```
## 4.3验证网络
```
root@controller01 ~ 16:38:28 # neutron net-list
+--------------------------------------+----------------------------------------------------+-------------------------------------------------------+
| id                                   | name                                               | subnets                                               |
+--------------------------------------+----------------------------------------------------+-------------------------------------------------------+
| 8bcf7f20-d7b3-458a-a6e1-e5e45448b9e4 | ext-net                                            | c66a6458-a721-4f3c-9367-3ec5af16375d 10.0.30.0/24     |
| 8b74f24b-3d1a-470d-8a19-cc8df788effa | HA network tenant facece2a46f14816870c43fdb801a218 | 3740a95f-9494-4405-b762-d4de0d20b549 169.254.192.0/18 |
| 6d60a85b-5d53-46ad-9e53-51ef852ef150 | admin-net                                          | afe73d97-683a-4957-affe-509b8708787d 192.168.1.0/24   |
+--------------------------------------+----------------------------------------------------+-------------------------------------------------------+
root@controller01 ~ 16:43:34 # neutron l3-agent-list-hosting-router admin-router
+--------------------------------------+-----------+----------------+-------+----------+
| id                                   | host      | admin_state_up | alive | ha_state |
+--------------------------------------+-----------+----------------+-------+----------+
| 03c17f9a-c05d-49db-9f59-54c24fa0dc51 | network02 | True           | :-)   | standby  | #备用
| 52f4acc1-7881-4d45-b10c-df46d00a4718 | network01 | True           | :-)   | active   | #活动
+--------------------------------------+-----------+----------------+-------+----------+
```

