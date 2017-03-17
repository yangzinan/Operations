# 一、环境约定
参见openstack部署文档，在每台计算机添加一块硬盘
# 二、配置ceph部署节点（controller）
## 2.1配置yum源
```shell
ceph-deploy purge
cat>>/etc/yum.repos.d/ceph.repo<<EOF
[ceph]
name=Ceph packages for $basearch
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/$basearch
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc 
[ceph-noarch]
name=Ceph noarch packages
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/noarch
enabled=1
priority=2
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc 
[ceph-source]
name=Ceph source packages
baseurl=http://mirrors.163.com/ceph/rpm-jewel/el7/SRPMS
enabled=0
priority=2
gpgcheck=1
type=rpm-md
gpgkey=http://mirrors.163.com/ceph/keys/release.asc
EOF
```
## 2.2安装ceph部署工具
```shell
yum install -y ceph-deploy
```
## 2.3配置免秘钥访问
```shell
ssh-keygen -t rsa -P ''
ssh-keygen -t rsa -f .ssh/id_rsa -P ''
ssh-copy-id -i .ssh/id_rsa.pub root@controller
ssh-copy-id -i .ssh/id_rsa.pub root@compute
ssh-copy-id -i .ssh/id_rsa.pub root@cinder
```
# 三、安装ceph
## 3.1生成监视器秘钥
```shell
mkdir myceph && cd myceph
ceph-deploy purgedata compute controller cinder
ceph-deploy forgetkeys
```
## 3.2创建集群
```shell
ceph-deploy new controller compute cinder
```
## 3.3安装ceph
```shell
ceph-deploy install controller compute cinder
#注：使用ceph-deploy安装还是会使用国外yum，如果网速不够就在每个节点使用：yum -y install ceph ceph-radosgw 安装
```
## 3.4穿件一个监视集群
```shell
ceph-deploy mon create controller compute cinder
```
## 3.5收集秘钥
```shell
ceph-deploy gatherkeys controller compute cinder
```
## 3.6给各个节点新加的硬盘分区并挂载
```shell
mkfs.xfs /dev/sdb
mkdir -p /ceph/osd/osd-01
echo "/dev/sdb    /ceph/osd/osd-01    xfs    defaults    0 0" >> /etc/fstab
mount -a
```
## 3.7初始化osd节点并激活osd
```shell
ceph-deploy osd prepare controller:/ceph/osd/osd-01 compute:/ceph/osd/osd-01 cinder:/ceph/osd/osd-01
ceph-deploy osd activate controller:/ceph/osd/osd-01 compute:/ceph/osd/osd-01 cinder:/ceph/osd/osd-01
```
## 3.8文件和管理密钥复制到管理节点和你的Ceph节点
```shell
ceph-deploy admin controller compute cinder
```
## 3.9验证安装
```shell
root@controller ~ 11:21:27 # ceph -s                  
    cluster 110c0f7e-51c8-4597-a0aa-047c6909fbd7
     health HEALTH_OK
     monmap e1: 3 mons at {cinder=10.0.10.30:6789/0,compute=10.0.10.20:6789/0,controller=10.0.10.10:6789/0}
            election epoch 28, quorum 0,1,2 controller,compute,cinder
     osdmap e117: 3 osds: 3 up, 3 in
            flags sortbitwise,require_jewel_osds
      pgmap v2432: 448 pgs, 4 pools, 45659 kB data, 23 objects
            15619 MB used, 45790 MB / 61410 MB avail
                 448 active+clean
root@controller ~ 11:21:30 # ceph health
HEALTH_OK
```
# 四集成openstack
# 4.1创建存储卷
```shell
ceph osd pool create volumes 128  #cinder存储卷
ceph osd pool create images 128   #glance存储卷
ceph osd pool create vms 128      #nova存储卷
```
## 4.2配置ceph文件
```shell
ssh controller sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh compute sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
ssh cinder sudo tee /etc/ceph/ceph.conf </etc/ceph/ceph.conf
```
## 4.3配置ceph认证
```shell
ceph auth get-or-create client.cinder mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images'
ceph auth get-or-create client.glance mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=images'
ceph auth get-or-create client.cinder-backup mon 'allow r' osd 'allow class-read object_prefix rbd_children, allow rwx pool=backups'
```
## 4.4配置glance-api(controller)
```shell
ceph auth get-or-create client.glance | ssh controller tee /etc/ceph/ceph.client.glance.keyring
ssh controller chown glance:glance /etc/ceph/ceph.client.glance.keyring
```
## 4.5配置cinder-volume(cinder)
```shell
ceph auth get-or-create client.cinder | ssh cinder tee /etc/ceph/ceph.client.cinder.keyring
ssh cinder chown cinder:cinder /etc/ceph/ceph.client.cinder.keyring
```
## 4.6配置nova-compute（compute）
```shell
ceph auth get-or-create client.cinder | ssh compute tee /etc/ceph/ceph.client.cinder.keyring
ceph auth get-key client.cinder | ssh compute tee client.cinder.key
cat > secret.xml <<EOF
<secret ephemeral='no' private='no'>
  <uuid>457eb676-33da-42ec-9a8c-9293d545c337</uuid>
  <usage type='ceph'>
    <name>client.cinder secret</name>
  </usage>
</secret>

virsh secret-define --file secret.xml
virsh secret-set-value --secret 457eb676-33da-42ec-9a8c-9293d545c337 --base64 $(cat client.cinder.key) && rm client.cinder.key secret.xml
cat>>/etc/ceph/ceph.conf<<EOF
[client]
rbd cache = true
rbd cache writethrough until flush = true
admin socket = /var/run/ceph/guests/$cluster-$type.$id.$pid.$cctid.asok
log file = /var/log/qemu/qemu-guest-$pid.log
rbd concurrent management ops = 20
EOF
mkdir -p /var/run/ceph/guests/ /var/log/qemu/
groupadd libvirtd
chown qemu:libvirtd /var/run/ceph/guests /var/log/qemu/
openstack-config --set /etc/nova/nova.conf libvirt images_type rbd
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_pool vms
openstack-config --set /etc/nova/nova.conf libvirt images_rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/nova/nova.conf libvirt rbd_user cinder
openstack-config --set /etc/nova/nova.conf libvirt rbd_secret_uuid 457eb676-33da-42ec-9a8c-9293d545c337
openstack-config --set /etc/nova/nova.conf libvirt disk_cachemodes "network=writeback"
openstack-config --set /etc/nova/nova.conf libvirt inject_password false
openstack-config --set /etc/nova/nova.conf libvirt inject_key false
openstack-config --set /etc/nova/nova.conf libvirt inject_partition -2
openstack-config --set /etc/nova/nova.conf libvirt hw_disk_discard unmap
```
## 4.6配置glance-api（controller）
```shell
openstack-config --set /etc/glance/glance-api.conf glance_store stores rbd
openstack-config --set /etc/glance/glance-api.conf glance_store default_store rbd
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_pool images
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_user glance
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/glance/glance-api.conf glance_store rbd_store_chunk_size 8
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_multiple_locations True
openstack-config --set /etc/glance/glance-api.conf DEFAULT show_image_direct_url True
openstack-config --set /etc/glance/glance-api.conf paste_deploy flavor keystone
```
## 4.7配置cinder-volume（cinder）
```shell
openstack-config --set /etc/cinder/cinder.conf DEFAULT enabled_backends ceph
openstack-config --set /etc/cinder/cinder.conf ceph volume_driver cinder.volume.drivers.rbd.RBDDriver
openstack-config --set /etc/cinder/cinder.conf ceph volume_backend_name ceph
openstack-config --set /etc/cinder/cinder.conf ceph rbd_pool volumes
openstack-config --set /etc/cinder/cinder.conf ceph rbd_ceph_conf /etc/ceph/ceph.conf
openstack-config --set /etc/cinder/cinder.conf ceph rbd_flatten_volume_from_snapshot false
openstack-config --set /etc/cinder/cinder.conf ceph rbd_max_clone_depth 5
openstack-config --set /etc/cinder/cinder.conf ceph rbd_store_chunk_size 4
openstack-config --set /etc/cinder/cinder.conf ceph rados_connect_timeout -1
openstack-config --set /etc/cinder/cinder.conf ceph glance_api_version 2
openstack-config --set /etc/cinder/cinder.conf ceph rbd_user cinder
openstack-config --set /etc/cinder/cinder.conf ceph rbd_secret_uuid 457eb676-33da-42ec-9a8c-9293d545c337
```
## 4.8在各个相关节点重启以下服务
```shell
systemctl restart openstack-glance-api
systemctl restart openstack-nova-compute
systemctl restart openstack-cinder-volume
```
# 五、验证操作
## 5.1上传镜像并验证
```shell
root@controller ~ 11:43:24 # glance image-list
+--------------------------------------+--------+
| ID                                   | Name   |
+--------------------------------------+--------+
| 47cf7cc9-a906-46c1-98ef-e54a1ee6c82a | cirros |
+--------------------------------------+--------+
root@controller ~ 11:43:41 # rbd ls images
47cf7cc9-a906-46c1-98ef-e54a1ee6c82a
#镜像已经在ceph的images卷中
```
## 5.2创建虚拟机并验证
```shell
root@controller ~ 11:46:28 # nova list 
+--------------------------------------+--------------+--------+------------+-------------+-------------------------+
| ID                                   | Name         | Status | Task State | Power State | Networks                |
+--------------------------------------+--------------+--------+------------+-------------+-------------------------+
| 09e2e138-0322-412a-9c08-254812927248 | admin-node01 | ACTIVE | -          | Running     | admin-net=192.168.1.105 |
+--------------------------------------+--------------+--------+------------+-------------+-------------------------+
root@controller ~ 11:46:33 # rbd ls vms
09e2e138-0322-412a-9c08-254812927248_disk
#虚拟机已经在ceph的vms卷中
```
## 5.2创建云盘并验证
```shell
root@controller ~ 11:46:43 # cinder list
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
|                  ID                  | Status |     Name     | Size | Volume Type | Bootable |             Attached to              |
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
| 2db699f3-dc01-4ed0-8620-36ea3a39f84c | in-use | admin-disk01 |  1   |      -      |  false   | 09e2e138-0322-412a-9c08-254812927248 |
+--------------------------------------+--------+--------------+------+-------------+----------+--------------------------------------+
root@controller ~ 11:47:59 # rbd ls volumes
volume-2db699f3-dc01-4ed0-8620-36ea3a39f84c
#云盘已经在ceph的volmues卷中
```
