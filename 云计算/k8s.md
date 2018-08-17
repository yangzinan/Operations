# Kubernets 集群部署
## 一、环境约定

|     主机名    | IP   |  安装的软件  |
| --------   | :-----:  | :----:  |
|kube-master|192.168.217.10|etcd、kube-apiserver、kube-controller、kube-scheduler、kubectl|
|kube-node1|192.168.217.20|etcd、kube-node、kube-proxy|
|kube-node2|192.168.217.30|etcd、kube-node、kube-proxy|

## 二、环境准备
### 2.1 创建目录添加PATH(在所有节点执行)
```
mkdir -p /opt/kubernetes/{cfg,bin,ssl,log}
echo "export PATH=/opt/kubernetes/bin:\$PATH" >> /root/.bash_profile  
source /root/.bash_profile  
```
### 2.2 安装docker(在所有节点执行)
```
curl -sSL https://get.daocloud.io/docker | sh
systemctl start docker.service 
systemctl enable docker.service 
```
### 2.3 修改host(在所有节点执行)
```
cat >>/etc/hosts/<<EOF
192.168.217.10  k8s-master
192.168.217.20  k8s-node1
192.168.217.30  k8s-node2
EOF
```
### 2.4拉取测试镜像(可不执行)
```
docker pull daocloud.io/library/nginx:1.13.2
docker pull daocloud.io/library/nginx:1.10.2
```

### 2.5 配置免密码登录(在master执行)
```
ssh-keygen -t rsa
ssh-copy-id k8s-node1
ssh-copy-id k8s-node2
```

## 三、制作CA证书(在master节点操作)
### 3.1 将程序拷贝到相应目录并授权执行
```
chmod +x cfssl*
mv cfssl-certinfo_linux-amd64 /opt/kubernetes/bin/cfssl-certinfo
mv cfssljson_linux-amd64  /opt/kubernetes/bin/cfssljson
mv cfssl_linux-amd64  /opt/kubernetes/bin/cfssl
```

### 3.2 拷贝文件到其他节点
```
scp /opt/kubernetes/bin/cfssl* k8s-node1:/opt/kubernetes/bin
scp /opt/kubernetes/bin/cfssl* k8s-node2:/opt/kubernetes/bin
```

### 3.3 初始化
```
mkdir ssl && cd ssl
cfssl print-defaults config > config.json
cfssl print-defaults csr > csr.json
```

### 3.4 创建用来生成 CA 文件的 JSON 配置文件
```
cat >>ca-config.json<<EOF
{
  "signing": {
    "default": {
      "expiry": "8760h"
    },
    "profiles": {
      "kubernetes": {
        "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ],
        "expiry": "8760h"
      }
    }
  }
}
EOF
```

### 3.5 创建用来生成 CA 证书签名请求（CSR）的 JSON 配置文件
```
cat >>ca-csr.json<<EOF
{
  "CN": "kubernetes",
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

### 3.6生成CA证书（ca.pem）和密钥（ca-key.pem）
```
[root@k8s-master ssl]# cfssl gencert -initca ca-csr.json | cfssljson -bare ca
2018/06/07 09:30:16 [INFO] generating a new CA key and certificate from CSR
2018/06/07 09:30:16 [INFO] generate received request
2018/06/07 09:30:16 [INFO] received CSR
2018/06/07 09:30:16 [INFO] generating key: rsa-2048
2018/06/07 09:30:17 [INFO] encoded CSR
2018/06/07 09:30:17 [INFO] signed certificate with serial number 442587652045208440377444170631446076181330771637
[root@k8s-master ssl]# ls -l ca*
-rw-r--r--. 1 root root  290 6月   7 09:11 ca-config.json
-rw-r--r--. 1 root root 1001 6月   7 09:30 ca.csr
-rw-r--r--. 1 root root  208 6月   7 09:12 ca-csr.json
-rw-------. 1 root root 1675 6月   7 09:30 ca-key.pem
-rw-r--r--. 1 root root 1359 6月   7 09:30 ca.pem
```

### 3.7 颁发证书
```
cp ca.csr ca.pem ca-key.pem ca-config.json /opt/kubernetes/ssl
scp ca.csr ca.pem ca-key.pem ca-config.json k8s-node1:/opt/kubernetes/ssl 
scp ca.csr ca.pem ca-key.pem ca-config.json k8s-node2:/opt/kubernetes/ssl
```

## 四、部署ETCD集群

### 4.1安装ETCD(在master执行)
```
tar zxf etcd-v3.3.6-linux-amd64.tar.gz 
cd etcd-v3.3.6-linux-amd64
cp etcd etcdctl /opt/kubernetes/bin/ 
scp etcd etcdctl k8s-node1:/opt/kubernetes/bin/
scp etcd etcdctl k8s-node2:/opt/kubernetes/bin/
```

### 4.2 创建etcd证书签名请求(在master执行)
```
cat >>etcd-csr.json<<EOF
{
  "CN": "etcd",
  "hosts": [
    "127.0.0.1",
	"192.168.217.10",
	"192.168.217.20",
	"192.168.217.30"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

### 4.3 生成 etcd 证书和私钥(在master执行)
```
[root@k8s-master /usr/local/src/ssl 09:40:24]# cfssl gencert -ca=/opt/kubernetes/ssl/ca.pem \
   -ca-key=/opt/kubernetes/ssl/ca-key.pem \
   -config=/opt/kubernetes/ssl/ca-config.json \
   -profile=kubernetes etcd-csr.json | cfssljson -bare etcd
2018/06/07 09:41:02 [INFO] generate received request
2018/06/07 09:41:02 [INFO] received CSR
2018/06/07 09:41:02 [INFO] generating key: rsa-2048
2018/06/07 09:41:02 [INFO] encoded CSR
2018/06/07 09:41:02 [INFO] signed certificate with serial number 572998794032319495782221835898884368761935481424
2018/06/07 09:41:02 [WARNING] This certificate lacks a "hosts" field. This makes it unsuitable for
websites. For more information see the Baseline Requirements for the Issuance and Management
of Publicly-Trusted Certificates, v.1.1.6, from the CA/Browser Forum (https://cabforum.org);
specifically, section 10.2.3 ("Information Requirements").
[root@k8s-master /usr/local/src/ssl 09:41:02]# ls -l etcd*
-rw-r--r--. 1 root root 1086 6月   7 09:41 etcd.csr
-rw-r--r--. 1 root root  288 6月   7 09:38 etcd-csr.json
-rw-------. 1 root root 1675 6月   7 09:41 etcd-key.pem
-rw-r--r--. 1 root root 1456 6月   7 09:41 etcd.pem
```

### 4.4 将证书移动到/opt/kubernetes/ssl目录下(在master执行)
```
cp etcd*.pem /opt/kubernetes/ssl
scp etcd*.pem k8s-node1:/opt/kubernetes/ssl
scp etcd*.pem k8s-node2:/opt/kubernetes/ssl
rm -f etcd.csr etcd-csr.json
```
### 4.5 设置ETCD配置文件
#### master配置
```
cat >>/opt/kubernetes/cfg/etcd.conf<<EOF
#[member]
ETCD_NAME="etcd-node1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_SNAPSHOT_COUNTER="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
ETCD_LISTEN_PEER_URLS="https://192.168.217.10:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.217.10:2379,https://127.0.0.1:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.217.10:2380"
# if you use different ETCD_NAME (e.g. test),
# set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
ETCD_INITIAL_CLUSTER="etcd-node1=https://192.168.217.10:2380,etcd-node2=https://192.168.217.20:2380,etcd-node3=https://192.168.217.30:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.217.10:2379"
#[security]
CLIENT_CERT_AUTH="true"
ETCD_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
PEER_CLIENT_CERT_AUTH="true"
ETCD_PEER_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_PEER_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
EOF
```
#### node1而配置
```
cat >>/opt/kubernetes/cfg/etcd.conf<<EOF
#[member]
ETCD_NAME="etcd-node2"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_SNAPSHOT_COUNTER="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
ETCD_LISTEN_PEER_URLS="https://192.168.217.20:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.217.20:2379,https://127.0.0.1:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.217.20:2380"
# if you use different ETCD_NAME (e.g. test),
# set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
ETCD_INITIAL_CLUSTER="etcd-node1=https://192.168.217.10:2380,etcd-node2=https://192.168.217.20:2380,etcd-node3=https://192.168.217.30:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.217.20:2379"
#[security]
CLIENT_CERT_AUTH="true"
ETCD_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
PEER_CLIENT_CERT_AUTH="true"
ETCD_PEER_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_PEER_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
EOF
```

#### node2配置
```
cat >>/opt/kubernetes/cfg/etcd.conf<<EOF
#[member]
ETCD_NAME="etcd-node3"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
#ETCD_SNAPSHOT_COUNTER="10000"
#ETCD_HEARTBEAT_INTERVAL="100"
#ETCD_ELECTION_TIMEOUT="1000"
ETCD_LISTEN_PEER_URLS="https://192.168.217.30:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.217.30:2379,https://127.0.0.1:2379"
#ETCD_MAX_SNAPSHOTS="5"
#ETCD_MAX_WALS="5"
#ETCD_CORS=""
#[cluster]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.217.30:2380"
# if you use different ETCD_NAME (e.g. test),
# set ETCD_INITIAL_CLUSTER value for this name, i.e. "test=http://..."
ETCD_INITIAL_CLUSTER="etcd-node1=https://192.168.217.10:2380,etcd-node2=https://192.168.217.20:2380,etcd-node3=https://192.168.217.30:2380"
ETCD_INITIAL_CLUSTER_STATE="new"
ETCD_INITIAL_CLUSTER_TOKEN="k8s-etcd-cluster"
ETCD_ADVERTISE_CLIENT_URLS="https:/192.168.217.30:2379"
#[security]
CLIENT_CERT_AUTH="true"
ETCD_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
PEER_CLIENT_CERT_AUTH="true"
ETCD_PEER_CA_FILE="/opt/kubernetes/ssl/ca.pem"
ETCD_PEER_CERT_FILE="/opt/kubernetes/ssl/etcd.pem"
ETCD_PEER_KEY_FILE="/opt/kubernetes/ssl/etcd-key.pem"
EOF
```

### 4.4 创建etcd服务(在所有节点执行)
```
cat >>/etc/systemd/system/etcd.service<<EOF
[Unit]
Description=Etcd Server
After=network.target

[Service]
Type=simple
WorkingDirectory=/var/lib/etcd
EnvironmentFile=-/opt/kubernetes/cfg/etcd.conf
# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /opt/kubernetes/bin/etcd"
Type=notify

[Install]
WantedBy=multi-user.target
EOF
```

### 4.5 启动ectd集群(在所有节点执行)
```
systemctl daemon-reload
systemctl enable etcd
mkdir /var/lib/etcd
systemctl start etcd
```

### 4.6 检查集群状态(在master执行)
```
[root@k8s-master /usr/local/src/ssl 10:56:14]# etcdctl --endpoints=https://192.168.217.10:2379 \
   --ca-file=/opt/kubernetes/ssl/ca.pem \
   --cert-file=/opt/kubernetes/ssl/etcd.pem \
   --key-file=/opt/kubernetes/ssl/etcd-key.pem cluster-health
member 3c012a87c4e193ac is healthy: got healthy result from https://192.168.217.20:2379
member 3cb279d5dff870db is healthy: got healthy result from https://192.168.217.30:2379
member 92b780d24fec7c16 is healthy: got healthy result from https://192.168.217.10:2379
cluster is healthy
[root@k8s-master /usr/local/src/ssl 10:56:26]# 
```

## 五、安装Master

### 5.1 安装软件
```
tar zxf kubernetes-server-linux-amd64.tar.gz.tar
cd kubernetes
cp server/bin/kube-apiserver /opt/kubernetes/bin/
cp server/bin/kube-controller-manager /opt/kubernetes/bin/
cp server/bin/kube-scheduler /opt/kubernetes/bin/
```

### 5.2 创建生成CSR的 JSON 配置文件
```
cat >>kubernetes-csr.json<<EOF
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "192.168.217.10",
    "10.1.0.1",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

### 5.3 生成 kubernetes 证书和私钥
```
cfssl gencert -ca=/opt/kubernetes/ssl/ca.pem \
   -ca-key=/opt/kubernetes/ssl/ca-key.pem \
   -config=/opt/kubernetes/ssl/ca-config.json \
   -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
cp kubernetes*.pem /opt/kubernetes/ssl/
scp kubernetes*.pem k8s-node1:/opt/kubernetes/ssl/
scp kubernetes*.pem k8s-node2:/opt/kubernetes/ssl/
```

### 5.4 创建 kube-apiserver 使用的客户端 token 文件
```
token=$(head -c 16 /dev/urandom | od -An -t x | tr -d ' ') 
echo "$token,kubelet-bootstrap,10001,\"system:kubelet-bootstrap\"" > /opt/kubernetes/ssl/bootstrap-token.csv
```

### 5.5 创建基础用户名/密码认证配置
```
cat >/opt/kubernetes/ssl/basic-auth.csv<<EOF
admin,admin,1
readonly,readonly,2
EOF
```

### 5.6 创建API Server服务
```
cat >/usr/lib/systemd/system/kube-apiserver.service<<EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
ExecStart=/opt/kubernetes/bin/kube-apiserver \
  --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,NodeRestriction \
  --bind-address=192.168.217.10 \
  --insecure-bind-address=127.0.0.1 \
  --authorization-mode=Node,RBAC \
  --runtime-config=rbac.authorization.k8s.io/v1 \
  --kubelet-https=true \
  --anonymous-auth=false \
  --basic-auth-file=/opt/kubernetes/ssl/basic-auth.csv \
  --enable-bootstrap-token-auth \
  --token-auth-file=/opt/kubernetes/ssl/bootstrap-token.csv \
  --service-cluster-ip-range=10.1.0.0/16 \
  --service-node-port-range=20000-40000 \
  --tls-cert-file=/opt/kubernetes/ssl/kubernetes.pem \
  --tls-private-key-file=/opt/kubernetes/ssl/kubernetes-key.pem \
  --client-ca-file=/opt/kubernetes/ssl/ca.pem \
  --service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \
  --etcd-cafile=/opt/kubernetes/ssl/ca.pem \
  --etcd-certfile=/opt/kubernetes/ssl/kubernetes.pem \
  --etcd-keyfile=/opt/kubernetes/ssl/kubernetes-key.pem \
  --etcd-servers=https://192.168.217.10:2379,https://192.168.217.20:2379,https://192.168.217.30:2379 \
  --enable-swagger-ui=true \
  --allow-privileged=true \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/opt/kubernetes/log/api-audit.log \
  --event-ttl=1h \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log
Restart=on-failure
RestartSec=5
Type=notify
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### 5.7 启动API Server服务
```
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
```

### 5.8 验证API Server
```
[root@k8s-master /usr/local/src/kubernetes 11:19:42]# systemctl status kube-apiserver             
● kube-apiserver.service - Kubernetes API Server
   Loaded: loaded (/usr/lib/systemd/system/kube-apiserver.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2018-06-07 11:19:42 CST; 8s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 22837 (kube-apiserver)
    Tasks: 7
   Memory: 303.5M
   CGroup: /system.slice/kube-apiserver.service
           └─22837 /opt/kubernetes/bin/kube-apiserver --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota,NodeRestriction --bind-address=192.168.217.10 --insecure-bind-address=127.0...

6月 07 11:19:35 k8s-master systemd[1]: kube-apiserver.service holdoff time over, scheduling restart.
6月 07 11:19:35 k8s-master systemd[1]: Starting Kubernetes API Server...
6月 07 11:19:36 k8s-master kube-apiserver[22837]: Flag --admission-control has been deprecated, Use --enable-admission-plugins or --disable-admission-plugins instead. Will be removed in a future version.
6月 07 11:19:36 k8s-master kube-apiserver[22837]: Flag --insecure-bind-address has been deprecated, This flag will be removed in a future version.
6月 07 11:19:38 k8s-master kube-apiserver[22837]: [restful] 2018/06/07 11:19:38 log.go:33: [restful/swagger] listing is available at https://192.168.217.10:6443/swaggerapi
6月 07 11:19:38 k8s-master kube-apiserver[22837]: [restful] 2018/06/07 11:19:38 log.go:33: [restful/swagger] https://192.168.217.10:6443/swaggerui/ is mapped to folder /swagger-ui/
6月 07 11:19:39 k8s-master kube-apiserver[22837]: [restful] 2018/06/07 11:19:39 log.go:33: [restful/swagger] listing is available at https://192.168.217.10:6443/swaggerapi
6月 07 11:19:39 k8s-master kube-apiserver[22837]: [restful] 2018/06/07 11:19:39 log.go:33: [restful/swagger] https://192.168.217.10:6443/swaggerui/ is mapped to folder /swagger-ui/
6月 07 11:19:42 k8s-master systemd[1]: Started Kubernetes API Server.
[root@k8s-master /usr/local/src/kubernetes 11:19:50]# 
```

### 5.9 部署Controller Manager服务
```
cat >/usr/lib/systemd/system/kube-controller-manager.service<<EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/opt/kubernetes/bin/kube-controller-manager \
  --address=127.0.0.1 \
  --master=http://127.0.0.1:8080 \
  --allocate-node-cidrs=true \
  --service-cluster-ip-range=10.1.0.0/16 \
  --cluster-cidr=10.2.0.0/16 \
  --cluster-name=kubernetes \
  --cluster-signing-cert-file=/opt/kubernetes/ssl/ca.pem \
  --cluster-signing-key-file=/opt/kubernetes/ssl/ca-key.pem \
  --service-account-private-key-file=/opt/kubernetes/ssl/ca-key.pem \
  --root-ca-file=/opt/kubernetes/ssl/ca.pem \
  --leader-elect=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 5.10 启动Controller Manager服务
```
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
```

### 5.11 验证Controller Manager服务
```
[root@k8s-master /usr/local/src/kubernetes 11:45:24]# systemctl status kube-controller-manager
● kube-controller-manager.service - Kubernetes Controller Manager
   Loaded: loaded (/usr/lib/systemd/system/kube-controller-manager.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2018-06-07 11:45:24 CST; 8s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 22894 (kube-controller)
    Tasks: 7
   Memory: 10.1M
   CGroup: /system.slice/kube-controller-manager.service
           └─22894 /opt/kubernetes/bin/kube-controller-manager --address=127.0.0.1 --master=http://127.0.0.1:8080 --allocate-node-cidrs=true --service-cluster-ip-range=10.1.0.0/16 --cluster-cidr=10.2.0.0/16 --cluster-name=kuberne...

6月 07 11:45:24 k8s-master systemd[1]: Started Kubernetes Controller Manager.
6月 07 11:45:24 k8s-master systemd[1]: Starting Kubernetes Controller Manager...
6月 07 11:45:26 k8s-master kube-controller-manager[22894]: E0607 11:45:26.105012   22894 core.go:75] Failed to start service controller: WARNING: no cloud provider provided, services of type LoadBalancer will fail
```

### 5.12 部署Kubernetes Scheduler
```
cat >/usr/lib/systemd/system/kube-scheduler.service<<EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes

[Service]
ExecStart=/opt/kubernetes/bin/kube-scheduler \
  --address=127.0.0.1 \
  --master=http://127.0.0.1:8080 \
  --leader-elect=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log

Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```
### 5.13 启动Kubernetes Scheduler
```
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
```

### 5.14 验证Kubernetes Scheduler
```
[root@k8s-master /usr/local/src/kubernetes 11:48:35]# systemctl status kube-scheduler
● kube-scheduler.service - Kubernetes Scheduler
   Loaded: loaded (/usr/lib/systemd/system/kube-scheduler.service; enabled; vendor preset: disabled)
   Active: active (running) since 四 2018-06-07 11:48:35 CST; 7s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 22942 (kube-scheduler)
    Tasks: 6
   Memory: 8.2M
   CGroup: /system.slice/kube-scheduler.service
           └─22942 /opt/kubernetes/bin/kube-scheduler --address=127.0.0.1 --master=http://127.0.0.1:8080 --leader-elect=true --v=2 --logtostderr=false --log-dir=/opt/kubernetes/log

6月 07 11:48:35 k8s-master systemd[1]: Started Kubernetes Scheduler.
6月 07 11:48:35 k8s-master systemd[1]: Starting Kubernetes Scheduler...
[root@k8s-master /usr/local/src/kubernetes 11:48:42]# 
```

### 5.15 部署kubectl命令
```
tar zxf kubernetes-client-linux-amd64.tar.gz.tar 
cd /usr/local/src/kubernetes/client/bin
cp kubectl /opt/kubernetes/bin/
```

### 5.16 创建 admin 证书签名请求
```
cd /usr/local/src/ssl/
cat > admin-csr.json<<EOF
{
  "CN": "admin",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
EOF
```

### 5.17 生成admin证书、私钥
```
cfssl gencert -ca=/opt/kubernetes/ssl/ca.pem \
   -ca-key=/opt/kubernetes/ssl/ca-key.pem \
   -config=/opt/kubernetes/ssl/ca-config.json \
   -profile=kubernetes admin-csr.json | cfssljson -bare admin
mv admin*.pem /opt/kubernetes/ssl/
```

### 5.18 设置集群参数
```
kubectl config set-cluster kubernetes \
   --certificate-authority=/opt/kubernetes/ssl/ca.pem \
   --embed-certs=true \
   --server=https://192.168.217.10:6443
```

### 5.19 设置客户端认证参数
```
kubectl config set-credentials admin \
   --client-certificate=/opt/kubernetes/ssl/admin.pem \
   --embed-certs=true \
   --client-key=/opt/kubernetes/ssl/admin-key.pem
```

### 5.20 设置上下文参数
```
kubectl config set-context kubernetes \
   --cluster=kubernetes \
   --user=admin
```

### 5.21 设置默认上下文
```
kubectl config use-context kubernetes
```

### 5.22 使用kubectl命令验证
```
[root@k8s-master /usr/local/src/ssl 11:58:48]# kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
etcd-1               Healthy   {"health":"true"}   
etcd-2               Healthy   {"health":"true"}   
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
[root@k8s-master /usr/local/src/ssl 11:58:58]# 
```

## 六、部署node节点

### 6.1 安装软件(在master执行)
```
tar zxf kubernetes-node-linux-amd64.tar.gz.tar 
cd /usr/local/src/kubernetes/server/bin/
scp kubelet kube-proxy k8s-node1:/opt/kubernetes/bin/
scp kubelet kube-proxy k8s-node2:/opt/kubernetes/bin/
```

### 6.2 创建角色(在master执行)
```
kubectl create clusterrolebinding kubelet-bootstrap --clusterrole=system:node-bootstrapper --user=kubelet-bootstrap
```

### 6.3 创建 kubelet bootstrapping kubeconfig 文件 设置集群参数(在master执行)
```
kubectl config set-cluster kubernetes \
   --certificate-authority=/opt/kubernetes/ssl/ca.pem \
   --embed-certs=true \
   --server=https://192.168.217.10:6443 \
   --kubeconfig=bootstrap.kubeconfig
```

### 6.4 设置客户端认证参数(在master执行)
```
kubectl config set-credentials kubelet-bootstrap \
   --token=$token\
   --kubeconfig=bootstrap.kubeconfig   
```

### 6.5 设置上下文参数(在master执行)
```
kubectl config set-context default \
   --cluster=kubernetes \
   --user=kubelet-bootstrap \
   --kubeconfig=bootstrap.kubeconfig
```

### 6.6 默认选择上下文(在master执行)
```
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig
scp bootstrap.kubeconfig k8s-node1:/opt/kubernetes/cfg
scp bootstrap.kubeconfig k8s-node2:/opt/kubernetes/cfg
```

### 6.7 设置CNI支持(在所有node执行)
```
mkdir -p /etc/cni/net.d
cat >/etc/cni/net.d/10-default.conf<<EOF
{
        "name": "flannel",
        "type": "flannel",
        "delegate": {
            "bridge": "docker0",
            "isDefaultGateway": true,
            "mtu": 1400
        }
}
EOF
```

### 6.8 创建kubelet目录(在所有node执行)
```
mkdir /var/lib/kubelet
```

### 6.9 创建kubelet服务配置
#### node1
```
cat >/usr/lib/systemd/system/kubelet.service<<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/opt/kubernetes/bin/kubelet \
  --address=192.168.217.20 \
  --hostname-override=192.168.217.20 \
  --pod-infra-container-image=mirrorgooglecontainers/pause-amd64:3.0 \
  --experimental-bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \
  --kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \
  --cert-dir=/opt/kubernetes/ssl \
  --network-plugin=cni \
  --cni-conf-dir=/etc/cni/net.d \
  --cni-bin-dir=/opt/kubernetes/bin/cni \
  --cluster-dns=10.1.0.2 \
  --cluster-domain=cluster.local. \
  --hairpin-mode hairpin-veth \
  --allow-privileged=true \
  --fail-swap-on=false \
  --logtostderr=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log
Restart=on-failure
RestartSec=5
EOF
```
#### node2
```
cat >/usr/lib/systemd/system/kubelet.service<<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service

[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/opt/kubernetes/bin/kubelet \
  --address=192.168.217.30 \
  --hostname-override=192.168.217.30 \
  --pod-infra-container-image=mirrorgooglecontainers/pause-amd64:3.0 \
  --experimental-bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \
  --kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \
  --cert-dir=/opt/kubernetes/ssl \
  --network-plugin=cni \
  --cni-conf-dir=/etc/cni/net.d \
  --cni-bin-dir=/opt/kubernetes/bin/cni \
  --cluster-dns=10.1.0.2 \
  --cluster-domain=cluster.local. \
  --hairpin-mode hairpin-veth \
  --allow-privileged=true \
  --fail-swap-on=false \
  --logtostderr=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
```

### 6.10 启动kubelet(在所有node执行)
```
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
```

### 6.11 验证kubelet(在所有node执行)
```
[root@k8s-node2 ~]# systemctl status kubelet
● kubelet.service - Kubernetes Kubelet
   Loaded: loaded (/usr/lib/systemd/system/kubelet.service; static; vendor preset: disabled)
   Active: active (running) since 四 2018-06-07 13:34:07 CST; 32s ago
     Docs: https://github.com/GoogleCloudPlatform/kubernetes
 Main PID: 25392 (kubelet)
    Tasks: 6
   Memory: 12.6M
   CGroup: /system.slice/kubelet.service
           └─25392 /opt/kubernetes/bin/kubelet --address=192.168.217.30 --hostname-override=192.168.217.30 --pod-infra-container-image=mirrorgooglecontainers/pause-amd64:3.0 --experimental-bootstrap-kubeconfig=/opt/kubernetes/cfg...

6月 07 13:34:07 k8s-node2 systemd[1]: kubelet.service holdoff time over, scheduling restart.
6月 07 13:34:07 k8s-node2 systemd[1]: Started Kubernetes Kubelet.
6月 07 13:34:07 k8s-node2 systemd[1]: Starting Kubernetes Kubelet...
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --address has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/admini...ore information.
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --cluster-dns has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/ad...ore information.
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --cluster-domain has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks...ore information.
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --hairpin-mode has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/a...ore information.
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --allow-privileged has been deprecated, will be removed in a future version
6月 07 13:34:08 k8s-node2 kubelet[25392]: Flag --fail-swap-on has been deprecated, This parameter should be set via the config file specified by the Kubelet's --config flag. See https://kubernetes.io/docs/tasks/a...ore information.
Hint: Some lines were ellipsized, use -l to show in full.
[root@k8s-node2 ~]# 
```

### 6.12 查看csr请求(在master执行)
```
[root@k8s-master /usr/local/src/kubernetes/server/bin 13:34:07]# kubectl get csr                         
NAME                                                   AGE       REQUESTOR           CONDITION
node-csr-VKSZrqXu0FOwgkcrWKgb2LsTngcCJfV8xJ4GjLC65r4   48s       kubelet-bootstrap   Pending
node-csr-nzkrupg8ngY7em9qSpSJ5QRJCcwcdk0alyNDuRnyCNE   52s       kubelet-bootstrap   Pending
```

### 6.13 批准kubelet 的 TLS 证书请求(在master执行)
```
kubectl get csr|grep 'Pending' | awk 'NR>0{print $1}'| xargs kubectl certificate approve
```

### 6.14 安装lvs(在所有node执行)
```
yum install -y ipvsadm ipset conntrack
```

### 6.15 创建 kube-proxy 证书请求(在master执行)
```
cd /usr/local/src/ssl/
cat >kube-proxy-csr.json<<EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

### 6.16 生成kube-proxy证书(在master执行)
```
cfssl gencert -ca=/opt/kubernetes/ssl/ca.pem \
   -ca-key=/opt/kubernetes/ssl/ca-key.pem \
   -config=/opt/kubernetes/ssl/ca-config.json \
   -profile=kubernetes  kube-proxy-csr.json | cfssljson -bare kube-proxy
cp kube-proxy*.pem /opt/kubernetes/ssl/
scp kube-proxy*.pem k8s-node1:/opt/kubernetes/ssl/
scp kube-proxy*.pem k8s-node2:/opt/kubernetes/ssl/
```

### 6.17 创建kube-proxy配置文件(在master执行)
```
kubectl config set-cluster kubernetes \
   --certificate-authority=/opt/kubernetes/ssl/ca.pem \
   --embed-certs=true \
   --server=https://192.168.217.10:6443 \
   --kubeconfig=kube-proxy.kubeconfig
kubectl config set-credentials kube-proxy \
   --client-certificate=/opt/kubernetes/ssl/kube-proxy.pem \
   --client-key=/opt/kubernetes/ssl/kube-proxy-key.pem \
   --embed-certs=true \
   --kubeconfig=kube-proxy.kubeconfig
kubectl config set-context default \
   --cluster=kubernetes \
   --user=kube-proxy \
   --kubeconfig=kube-proxy.kubeconfig
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
cp kube-proxy.kubeconfig /opt/kubernetes/cfg/
scp kube-proxy.kubeconfig k8s-node1:/opt/kubernetes/cfg/
scp kube-proxy.kubeconfig k8s-node2:/opt/kubernetes/cfg/
```

### 6.18 创建kube-proxy服务配置
#### node1
```
mkdir /var/lib/kube-proxy
cat >/usr/lib/systemd/system/kube-proxy.service<<EOF
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/opt/kubernetes/bin/kube-proxy \
  --bind-address=192.168.217.20 \
  --hostname-override=192.168.217.20 \
  --kubeconfig=/opt/kubernetes/cfg/kube-proxy.kubeconfig \
--masquerade-all \
  --feature-gates=SupportIPVSProxyMode=true \
  --proxy-mode=ipvs \
  --ipvs-min-sync-period=5s \
  --ipvs-sync-period=5s \
  --ipvs-scheduler=rr \
  --logtostderr=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log

Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### node2
```
mkdir /var/lib/kube-proxy
cat >/usr/lib/systemd/system/kube-proxy.service<<EOF
[Unit]
Description=Kubernetes Kube-Proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target

[Service]
WorkingDirectory=/var/lib/kube-proxy
ExecStart=/opt/kubernetes/bin/kube-proxy \
  --bind-address=192.168.217.30 \
  --hostname-override=192.168.217.30 \
  --kubeconfig=/opt/kubernetes/cfg/kube-proxy.kubeconfig \
--masquerade-all \
  --feature-gates=SupportIPVSProxyMode=true \
  --proxy-mode=ipvs \
  --ipvs-min-sync-period=5s \
  --ipvs-sync-period=5s \
  --ipvs-scheduler=rr \
  --logtostderr=true \
  --v=2 \
  --logtostderr=false \
  --log-dir=/opt/kubernetes/log

Restart=on-failure
RestartSec=5
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### 6.19 启动kube-proxy(在所有node执行)
```
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
```

### 6.20 验证(在master执行)
```
[root@k8s-master /usr/local/src/ssl 13:46:34]# kubectl get node
NAME             STATUS    ROLES     AGE       VERSION
192.168.217.20   Ready     <none>    12m       v1.10.3
192.168.217.30   Ready     <none>    12m       v1.10.3
```

## 七、部署Flannel

### 7.1 为Flannel生成证书
```
cd /usr/local/src/ssl
cat >flanneld-csr.json<<EOF
{
  "CN": "flanneld",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "TianJin",
      "L": "TianJin",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

### 7.2 生成证书
```
cfssl gencert -ca=/opt/kubernetes/ssl/ca.pem \
   -ca-key=/opt/kubernetes/ssl/ca-key.pem \
   -config=/opt/kubernetes/ssl/ca-config.json \
   -profile=kubernetes flanneld-csr.json | cfssljson -bare flanneld
```

### 7.3 分发证书
```
cp flanneld*.pem /opt/kubernetes/ssl/
scp flanneld*.pem k8s-node1:/opt/kubernetes/ssl/
scp flanneld*.pem k8s-node2:/opt/kubernetes/ssl/
```

### 7.4安装Flannel
```
tar zxf flannel-v0.10.0-linux-amd64.tar.gz
cp flanneld mk-docker-opts.sh /opt/kubernetes/bin/
scp flanneld mk-docker-opts.sh k8s-node1:/opt/kubernetes/bin/
scp flanneld mk-docker-opts.sh k8s-node2:/opt/kubernetes/bin/
cd /usr/local/src/kubernetes/cluster/centos/node/bin/
cp remove-docker0.sh /opt/kubernetes/bin/
scp remove-docker0.sh k8s-node1:/opt/kubernetes/bin/
scp remove-docker0.sh k8s-node2:/opt/kubernetes/bin/
```

### 7.5 配置Flannel
```
cat >/opt/kubernetes/cfg/flannel<<EOF
FLANNEL_ETCD="-etcd-endpoints=https://192.168.217.10:2379,https://192.168.217.20:2379,https://192.168.217.30:2379"
FLANNEL_ETCD_KEY="-etcd-prefix=/kubernetes/network"
FLANNEL_ETCD_CAFILE="--etcd-cafile=/opt/kubernetes/ssl/ca.pem"
FLANNEL_ETCD_CERTFILE="--etcd-certfile=/opt/kubernetes/ssl/flanneld.pem"
FLANNEL_ETCD_KEYFILE="--etcd-keyfile=/opt/kubernetes/ssl/flanneld-key.pem"
EOF
scp /opt/kubernetes/cfg/flannel k8s-node1:/opt/kubernetes/cfg/
scp /opt/kubernetes/cfg/flannel k8s-node2:/opt/kubernetes/cfg/
```

### 7.6 设置Flannel系统服务
```
cat >/usr/lib/systemd/system/flannel.service<<EOF
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
Before=docker.service

[Service]
EnvironmentFile=-/opt/kubernetes/cfg/flannel
ExecStartPre=/opt/kubernetes/bin/remove-docker0.sh
ExecStart=/opt/kubernetes/bin/flanneld \${FLANNEL_ETCD} \${FLANNEL_ETCD_KEY} \${FLANNEL_ETCD_CAFILE} \${FLANNEL_ETCD_CERTFILE} \${FLANNEL_ETCD_KEYFILE}
ExecStartPost=/opt/kubernetes/bin/mk-docker-opts.sh -d /run/flannel/docker

Type=notify

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service
EOF
scp /usr/lib/systemd/system/flannel.service k8s-node1:/usr/lib/systemd/system/
scp /usr/lib/systemd/system/flannel.service k8s-node2:/usr/lib/systemd/system/
```

### 7.7 Flannel CNI集成
```
cd /usr/local/src/
mkdir /opt/kubernetes/bin/cni
tar zxf cni-plugins-amd64-v0.7.1.tgz -C /opt/kubernetes/bin/cni
scp -r /opt/kubernetes/bin/cni/* k8s-node1:/opt/kubernetes/bin/cni/
scp -r /opt/kubernetes/bin/cni/* k8s-node2:/opt/kubernetes/bin/cni/
```

### 7.8 创建Etcd的key
```
/opt/kubernetes/bin/etcdctl --ca-file /opt/kubernetes/ssl/ca.pem --cert-file /opt/kubernetes/ssl/flanneld.pem --key-file /opt/kubernetes/ssl/flanneld-key.pem \
      --no-sync -C https://192.168.217.10:2379,https://192.168.217.20:2379,https://192.168.217.30:2379 \
mk /kubernetes/network/config '{ "Network": "10.2.0.0/16", "Backend": { "Type": "vxlan", "VNI": 1 }}' >/dev/null 2>&1
```

### 7.9 修改docker启动文件(再所有节点执行)
```
[root@k8s-master ~]# vim /usr/lib/systemd/system/docker.service
[Unit] #在Unit下面修改After和增加Requires
After=network-online.target firewalld.service flannel.service
Wants=network-online.target
Requires=flannel.service

[Service] #增加EnvironmentFile=-/run/flannel/docker
Type=notify
EnvironmentFile=-/run/flannel/docker
ExecStart=/usr/bin/dockerd $DOCKER_OPTS
```

### 7.10 分发docker启动文件
```
scp /usr/lib/systemd/system/docker.service k8s-node1:/usr/lib/systemd/system/
scp /usr/lib/systemd/system/docker.service k8s-node2:/usr/lib/systemd/system/
```

### 7.11重启docker(再所有节点执行)
```
systemctl daemon-reload
systemctl restart docker
```