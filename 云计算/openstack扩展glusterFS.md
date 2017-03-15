# 一、环境约定
参考openstack环境每台主机添加一个硬盘用作glusterfs的存储

# 二、安装配置glustFS
## 2.1配置yum源(所有节点)
```shell
yum install -y centos-release-gluster
sed -i "s#mirror.centos.org#mirrors.aliyun.com#g" /etc/yum.repos.d/CentOS-Gluster-3.10.repo 
```
## 2.2安装glusterFS(所有节点)
```shell
yum install -y glusterfs glusterfs-server glusterfs-fuse
```
## 2.3启动glusterFS服务(所有节点)
```shell
systemctl start glusterd
systemctl enable glusterd
```
## 2.4在任意节点添加集群
```shell
gluster peer probe compute
gluster peer probe cinder
#本例在controller节点执行
```
## 2.5验证集群
```shell
root@controller ~ 11:33:12 # gluster peer status       
Number of Peers: 2

Hostname: cinder
Uuid: 9dc0d94e-efdb-4468-8d50-0b91932eff51
State: Peer in Cluster (Connected)

Hostname: compute
Uuid: 3df787f4-426d-4562-8336-0538d3abeaa5
State: Peer in Cluster (Connected)
root@controller ~ 11:33:15 # 
```
## 2.6创建存数目录(在所有节点执行)
```shell
mkdir -p /openstack/brick1 
mkfs.ext4 /dev/sdb
mount /dev/sdb /openstack/brick1 
```
## 2.7增加开机自动挂载(在所有节点执行)
```shell
echo "/dev/sdb                                  /openstack/brick1       ext4    defaults        0 0" >> /etc/fstab
```
## 2.8创建openstack所需要的卷
```shell
gluster volume create images replica 3 \
controller:/openstack/brick1/image \
compute:/openstack/brick1/image \
cinder:/openstack/brick1/image

gluster volume create instances replica 3 \
controller:/openstack/brick1/instance \
compute:/openstack/brick1/instance \
cinder:/openstack/brick1/instance

gluster volume create volumes replica 3 \
controller:/openstack/brick1/volume \
compute:/openstack/brick1/volume \
cinder:/openstack/brick1/volume
```
## 2.9启动卷
```shell
gluster volume start images
gluster volume start volumes
gluster volume start instances
```
## 2.9验证操作
```shell
root@controller ~ 12:57:50 # gluster volume info  
 
Volume Name: images
Type: Distribute
Volume ID: ff6c32e2-7b1e-49d7-802d-92a7fbb38f85
Status: Started
Snapshot Count: 0
Number of Bricks: 3
Transport-type: tcp
Bricks:
Brick1: controller:/openstack/brick1/image
Brick2: compute:/openstack/brick1/image
Brick3: cinder:/openstack/brick1/image
Options Reconfigured:
auth.allow: 10.0.10.*
transport.address-family: inet
nfs.disable: on
 
Volume Name: instances
Type: Replicate
Volume ID: 582f4978-d136-41cb-99f3-8271b45bdfdf
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: controller:/openstack/brick1/instance
Brick2: compute:/openstack/brick1/instance
Brick3: cinder:/openstack/brick1/instance
Options Reconfigured:
auth.allow: 10.0.10.*
transport.address-family: inet
nfs.disable: on
 
Volume Name: volumes
Type: Replicate
Volume ID: 878f826e-72c2-477b-8175-a737370d01f0
Status: Started
Snapshot Count: 0
Number of Bricks: 1 x 3 = 3
Transport-type: tcp
Bricks:
Brick1: controller:/openstack/brick1/volume
Brick2: compute:/openstack/brick1/volume
Brick3: cinder:/openstack/brick1/volume
Options Reconfigured:
auth.allow: 10.0.10.*
transport.address-family: inet
nfs.disable: on
root@controller ~ 12:57:59 # 
```
# 三、配置openstack使用glusterFS
## 3.1准备
正式环境请先备份虚拟机云硬盘和镜像，测试环境先删除所有正在运行的云主机和云硬盘
## 3.2配置glance服务所在主机挂载glusterFS
### 3.2.1挂载glusterFS
```shell
echo "controller:/images                        /var/lib/glance/images  glusterfs  defaults,_netdev,backupvolfile-server=compute,backupvolfile-server=cinder   0 0" >> /etc/fstab
mount -a
```
### 3.2.2验证
```shell
root@controller ~ 12:21:25 # df -h
Filesystem          Size  Used Avail Use% Mounted on
/dev/sda3           6.8G  3.9G  3.0G  57% /
devtmpfs            1.9G     0  1.9G   0% /dev
tmpfs               1.9G     0  1.9G   0% /dev/shm
tmpfs               1.9G  8.7M  1.9G   1% /run
tmpfs               1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1           197M  102M   96M  52% /boot
tmpfs               378M     0  378M   0% /run/user/0
/dev/sdb            7.8G   37M  7.3G   1% /openstack/brick1
controller:/images   24G  109M   22G   1% /var/lib/glance/images
root@controller ~ 12:21:29 # 
```
### 3.2.3给glance用户授权
```shell
chown -R glance.glance /var/lib/glance/images
```
### 3.2.4重启glance服务
```shell
systemctl restart openstack-glance-api.service \
  openstack-glance-registry.service
```
### 3.2.5重新上传镜像
```shell
source ~/admin.rc
openstack image create "cirros" \
  --file cirros-0.3.4-x86_64-disk.img \
  --disk-format qcow2 --container-format bare \
  --public
```
### 3.2.6验证操作
```shell
root@controller ~ 12:37:14 # glance image-list
+--------------------------------------+--------+
| ID                                   | Name   |
+--------------------------------------+--------+
| 10301f01-0994-41ab-80cd-143f3789e24e | cirros |
+--------------------------------------+--------+
```
## 3.3配置计算节点（compute）
### 3.3.1挂载glusterFS
```shell
echo "ccompute:/instances                        /var/lib/nova/instances glusterfs  defaults,_netdev,backupvolfile-server=controller,backupvolfile-server=cinder   0 0" >> /etc/fstab
mount -a
```
### 3.3.2验证操作
```shell
root@compute ~ 13:33:52 # df -h
Filesystem          Size  Used Avail Use% Mounted on
/dev/sda3           6.8G  2.1G  4.8G  31% /
devtmpfs            1.9G     0  1.9G   0% /dev
tmpfs               1.9G     0  1.9G   0% /dev/shm
tmpfs               1.9G  8.7M  1.9G   1% /run
tmpfs               1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/sda1           197M  103M   95M  53% /boot
tmpfs               378M     0  378M   0% /run/user/0
/dev/sdc            7.8G   49M  7.3G   1% /openstack/brick1
compute:/instances  7.8G   49M  7.3G   1% /var/lib/nova/instances
```
### 3.3.3给nova用户授权
```shell
chown -R nova.nova /var/lib/nova/instances
```
### 3.3.4重启nova-compute服务
```shell
systemctl restart openstack-nova-compute
```
## 3.4配置cinder节点
### 3.4.1备份cinder配置文件
```shell
cp /etc/cinder/cinder.conf{,.bak}
```
### 3.4.2配置cinder使用glusterFS
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT enabled_backends glusterfs
openstack-config --set /etc/cinder/cinder.conf glusterfs volume_driver cinder.volume.drivers.glusterfs.GlusterfsDriver
openstack-config --set /etc/cinder/cinder.conf glusterfs glusterfs_shares_config /etc/cinder/shares.conf
openstack-config --set /etc/cinder/cinder.conf glusterfs glusterfs_mount_point_base /var/lib/cinder/volumes
openstack-config --set /etc/cinder/cinder.conf glusterfs volume_backend_name glusterfs 

crudini --set /etc/cinder/cinder.conf glusterfs volume_driver "cinder.volume.drivers.glusterfs.GlusterfsDriver"
crudini --set /etc/cinder/cinder.conf glusterfs glusterfs_shares_config "/etc/cinder/glusterfs_shares"
crudini --set /etc/cinder/cinder.conf glusterfs glusterfs_mount_point_base "/var/lib/cinder/glusterfs"
crudini --set /etc/cinder/cinder.conf glusterfs nas_volume_prov_type thin
crudini --set /etc/cinder/cinder.conf glusterfs glusterfs_disk_util df
crudini --set /etc/cinder/cinder.conf glusterfs glusterfs_qcow2_volumes True
crudini --set /etc/cinder/cinder.conf glusterfs volume_backend_name GLUSTERFS         
```
### 3.4.3创建配置文件
```shell
cat>>/etc/cinder/shares.conf<<EOF 
cinder:/volumes
controller:/volumes
compute:/volumes
EOF
```
### 3.4.4配置权限
```shell
chown -R cinder.cinder /etc/cinder/shares.conf
chown -R cinder:cinder /var/lib/cinder/*
```
### 3.4.5重启cinder服务
```shell
systemctl start openstack-cinder-volume
```
# 四、验证操作
## 4.1创建云主机



## 4.2创建云盘