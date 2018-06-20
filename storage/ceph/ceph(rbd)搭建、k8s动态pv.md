# 系统环境
|hostname|ip|role|os|
|-------|-------|-----|----|
|deploy|172.16.72.119|admin/deploy|centos_7.4_x64|
|k8s-ceph01|172.16.72.119|mon/osd|centos_7.4_x64|
|k8s-ceph02|172.16.72.132|mon/osd|centos_7.4_x64|
|k8s-ceph03|172.16.72.119|mon/osd|centos_7.4_x64|
>  Ceph.Mon = 2n+1 个 ，3个的情况下只能掉线一个，如果同时2个掉线,集群会出现无法仲裁，集群会一直等待 Ceph.Mon 恢复超过半数。
# 环境准备
```
# 所有节点配置ntp同步，集群必须(ecs自带，省略此步骤)

# 所有节点配置hosts文件
cat << "EOF" >> /etc/hosts
# --- Ceph Hosts -----
172.16.72.132 k8s-ceph01
172.16.72.133 k8s-ceph02
172.16.72.134 k8s-ceph03
# --- Ceph Hosts -----
EOF

# 所有节点都配置yum源
# 配置 官方 的 Ceph 源
rpm --import https://download.ceph.com/keys/release.asc
rpm -Uvh --replacepkgs https://download.ceph.com/rpm-jewel/el7/noarch/ceph-release-1-0.el7.noarch.rpm
# 替换 ceph 源 为 163 源
sed -i 's/download\.ceph\.com/mirrors\.163\.com\/ceph/g' /etc/yum.repos.d/ceph.repo
# 安装 epel 源，ecs自带可省略
yum makecache

# 配置ssh-key
# admin节点
ssh-keygen -t rsa -b 2048 -C "your_email@youremail.com" -P ''
ssh-copy-id k8s-ceph01
ssh-copy-id k8s-ceph02
ssh-copy-id k8s-ceph03 (-p22)

# 增加 ~/.ssh/config 文件, 否则 ceph-deploy 创建集群 报端口错误（没改ssh端口可以不用操作此步骤）
vim ~/.ssh/config
Host k8s-ceph01
    Hostname k8s-ceph01
    Port 33
Host k8s-ceph02
    Hostname k8s-ceph02
    Port 33
Host k8s-ceph03
    Hostname k8s-ceph03
    Port 33
    
# 安装ceph-deploy
# admin节点安装
yum -y install ceph-deploy 
```
# 部署ceph-mon
```
# admin节点操作
# 创建集群目录，用于存放配置文件，证书等信息
mkdir -p /opt/ceph-cluster
cd /opt/ceph-cluster/
# 创建ceph-mon 节点
ceph-deploy new k8s-ceph01 k8s-ceph02 k8s-ceph03
#查看配置文件
cat ceph.conf 
[global]
fsid = 85bd026a-7577-4f35-a048-d27fac700aaa
mon_initial_members = k8s-ceph01, k8s-ceph02, k8s-ceph03
mon_host = 172.16.72.132,172.16.72.133,172.16.72.134
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

# 修改 osd 的副本数，既数据保存N份。
echo 'osd_pool_default_size = 3' >> ./ceph.conf

# 注: 如果文件系统为 ext4 请添加
echo 'osd max object name len = 256' >> ./ceph.conf
echo 'osd max object namespace len = 64' >> ./ceph.conf

echo 'rbd_default_features = 1' >> ./ceph.conf
ceph --show-config|grep rbd|grep features
# rbd image有4个 features，layering, exclusive-lock, object-map, fast-diff, deep-flatten
因为3.10内核仅支持layering，修改默认配置。
```
# 安装ceph
```
# 可通过 ceph-deploy 安装，也可以登陆node 本地安装
ceph-deploy install k8s-ceph01 k8s-ceph02 k8s-ceph03

# 如果出现超时，请自行在每个服务器中使用 yum 安装
# yum -y install ceph ceph-radosgw

#查看 安装信息
ceph --version
ceph version 10.2.10 (5dc1e4c05cb68dbf62ae6fce3f0700e4654fdbbe)
```
# 初始化ceph-mon节点
```
# 请务必在admin节点/opt/ceph-cluster目录下
ceph-deploy mon create-initial
```
# 初始化ceph.osd节点
```
# 首先创建 存储空间, 如果使用分区，可略过
# 每个osd节点
mkdir -p /opt/ceph/osd-1 && chown ceph:ceph /opt/ceph/osd-1
mkdir -p /opt/ceph/osd-2 && chown ceph:ceph /opt/ceph/osd-2
mkdir -p /opt/ceph/osd-3 && chown ceph:ceph /opt/ceph/osd-3

#初始化osd
ceph-deploy osd prepare k8s-ceph01:/opt/ceph/osd-1 k8s-ceph02:/opt/ceph/osd-2 k8s-ceph03:/opt/ceph/osd-3 

# 激活所有osd节点
ceph-deploy osd activate k8s-ceph01:/opt/ceph/osd-1 k8s-ceph02:/opt/ceph/osd-2 k8s-ceph03:/opt/ceph/osd-3

# 把管理节点的配置文件与keyring同步至其它节点
ceph-deploy admin k8s-ceph01 k8s-ceph02 k8s-ceph03
```
# 集群检查
```commandline
# osd或者mon节点
# 集群状态
[root@k8s-k8s-ceph01 ~]# ceph -s
    cluster 85bd026a-7577-4f35-a048-d27fac700aaa
     health HEALTH_OK
     monmap e1: 3 mons at {k8s-k8s-ceph01=172.16.72.132:6789/0,k8s-k8s-ceph02=172.16.72.133:6789/0,k8s-k8s-ceph03=172.16.72.134:6789/0}
            election epoch 22, quorum 0,1,2 k8s-k8s-ceph01,k8s-k8s-ceph02,k8s-k8s-ceph03
     osdmap e49: 3 osds: 3 up, 3 in
            flags sortbitwise,require_jewel_osds
      pgmap v22620: 64 pgs, 1 pools, 2231 MB data, 713 objects
            21857 MB used, 218 GB / 239 GB avail
                  64 active+clean
# 查看 osd tree
[root@k8s-k8s-ceph01 ~]#  ceph osd tree
ID WEIGHT  TYPE NAME           UP/DOWN REWEIGHT PRIMARY-AFFINITY 
-1 0.11696 root default                                          
-2 0.03899     host k8s-k8s-ceph01                                   
 0 0.03899         osd.0            up  1.00000          1.00000 
-3 0.03899     host k8s-k8s-ceph02                                   
 1 0.03899         osd.1            up  1.00000          1.00000 
-4 0.03899     host k8s-k8s-ceph03                                   
 2 0.03899         osd.2            up  1.00000          1.00000 
# 设置所有开机启动
systemctl enable ceph-osd.target
systemctl enable ceph-mon.target           

# 重启系统以后重启 osd
#查看处于down 的 osd
ceph osd tree
# 登陆所在node 启动ceph-osd进程 [id 为 tree 查看的 id]
systemctl start ceph-osd@id
```
# k8s 配置
## node节点ceph-common
```commandline
# 1. 在所有的 k8s node 里面安装 ceph-common
yum -y install ceph-common

# 2. 拷贝 ceph.conf 与 ceph.client.admin.keyring
拷贝到 /etc/ceph/ 目录下
```
## 创建 ceph-secret
```commandline
# 获取 client.admin 的值
ceph auth get-key client.admin
AQASHHZaprbFHxAAN1jM+OQimNaQ0ZZsuDgxBw==

# 转换成 base64 编码
echo "AQASHHZaprbFHxAAN1jM+OQimNaQ0ZZsuDgxBw=="|base64

# 创建ceph-secret.yaml文件
[root@mzzg-jumper zookeeper-ceph]# cat ceph-storageclass-secret.yml 
apiVersion: v1
kind: Secret
metadata:
  name: ceph-storageclass-secret
data:
  key: QVFBU0hIWmFwcmJGSHhBQU4xak0rT1FpbU5hUTBaWnN1RGd4Qnc9PQ==
type: kubernetes.io/rbd

#namespace:kube-system也加下secret
cat ceph-storageclass-secret-system.yml 
apiVersion: v1
kind: Secret
metadata:
  name: ceph-storageclass-secret
  namespace: kube-system
data:
  key: QVFBU0hIWmFwcmJGSHhBQU4xak0rT1FpbU5hUTBaWnN1RGd4Qnc9PQ==
type: kubernetes.io/rbd

# 导入 yaml 文件
kubectl apply -f ceph-secret.yaml
kubectl apply -f ceph-storageclass-secret-system.yml

# 查看 状态
kubectl get secret --all-namespaces|grep ceph
default       ceph-storageclass-secret              kubernetes.io/rbd                     1         3d
kube-system   ceph-storageclass-secret              kubernetes.io/rbd                     1         3d
```
## 创建sc
```commandline
cat ceph-storageclass.yml 
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-storageclass
provisioner: kubernetes.io/rbd
parameters:
  monitors: 172.16.72.132:6789,172.16.72.133:6789,172.16.72.134:6789
  adminId: admin
  adminSecretName: ceph-storageclass-secret
  adminSecretNamespace: kube-system
  pool: rbd
  userId: admin
  userSecretName: ceph-storageclass-secret
  imageformat: "1"
  imagefeatures: "layering"
# 因为内核版本比较老不支持imageformat: "2"

kubectl apply -f ceph-storageclass.yml  

# 查看sc
kubectl get sc
NAME                PROVISIONER         AGE
ceph-storageclass   kubernetes.io/rbd   3d
```
## 引用sc
```commandline
前面省略...
      volumeMounts:
        - name: datadir
          mountPath: /opt/zookeeper/data
        - name: datalogdir
          mountPath: /opt/zookeeper/data-log
  volumeClaimTemplates:
  - metadata:
      name: datadir
      annotations:
        volume.beta.kubernetes.io/storage-class: ceph-storageclass
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
  - metadata:
      name: datalogdir
      annotations:
        volume.beta.kubernetes.io/storage-class: ceph-storageclass
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 5Gi
```
ok,可以查看相关pv，pvc是否成功挂载了已经ceph相关rbd是否建立

