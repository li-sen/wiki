> 这是本人参照开源项目kubeasz（[项目地址](https://github.com/gjmzj/kubeasz)），在阿里云vpc环境使用slb搭建的一套3\*master+3\*node的高可用k8s集群，这并不是一份完整的k8s安装文档，这里只是记录使用kubeasz搭建集群关键步骤以及需要注意的点，完整的请参考源项目，命令操作时建议使用screen进行操作，以免网络问题造成操作失败。

> 这里说明下master高可用网络通信：多master使用slb是为了解决api-server的高可用，api-server是为master外部相关应用(kubectl、kubelet等)服务的，master上的cm、scheduler其实是只跟本机的api-server(监听地址一般为127.0.0.1/内网ip)通信，跟slb没有通信需求，并且多master中通过内部ip通信竞选只有一个cm和scheduler是生效可用，其他的节点为备用。
---
# 关于阿里云slb
> 特别说明是slb，因为阿里云的slb 是不支持后端rs访问slb地址，用kubeasz项目部署时，slb后面的master是需要跟slb通信，这里用一台haproxy中转下，考虑高可用也可以用用两台。
kubeasz 新版本将master node角色重合，这样就算有slb，还是得需要haproxy了，如果master可以去掉node角色，基于多master的通信流程，那haproxy只在部署的时候使用下，安装完可以直接去掉，让slb直连master（master就无法使用kubectl），
原理上将是没问题的，只是影不影响kubeasz后续的周边服务安装就不得而知，我是没做测试。

# 集群规划
## 服务器规划

hostname | ip | 配置|用途
---|---|---|---
by-master01 | 172.17.0.85 | 2c4g 40g+20g | master
by-master02 | 172.17.0.86 | 2c4g 40g+20g | master
by-master03 | 172.17.0.87 | 2c4g 40g+20g | master
by-node01 | 172.17.0.88 | 4c8g 40g+40g | node
by-node02 | 172.17.0.89 | 4c8g 40g+40g | node
by-node03 | 172.17.0.90 | 4c8g 40g+40g | node
by-deploy01 | 172.17.0.91 | 2c4g 40g+60g | 发布控制
by-haproxy01 | 172.17.0.94 | 2c4g 40g+60g | lb

## 阿里云slb
名称 | ip | 备注
---|---|---
by-master | 172.17.0.92 | kube-apiserver高可用
by-node | 172.17.0.93 | 后续服务暴露可能用到

## 组件说明
组件 | 版本
---|---
os | CentOS Linux release 7.5.1804 (Core)
kernel | 4.17.3-1.el7.elrepo.x86_64（自行升级）
k8s | v1.10.4
etcd | v3.3.6
docker | 18.03.0-ce
network | calico v3.0
> 建议大家把内核都升级到最新的lt版本，不然会出现各种bug，因为centos的内核版本实在太老了。

## 安装准备
### 各节点安装python
``` bash
# 文档中脚本默认均以root用户执行
# 安装 epel 源并更新
yum install epel-release -y
yum update
# 安装python
yum install python -y
```
### 各节点时间同步
集群时间不同步会造成后续很多问题（譬如etcd不可用），这是集群基本条件；阿里云ecs时间同步ntp服务默认装好，这里就省略，没有的自行安装。

### 在deploy节点安装及准备ansible

``` bash
yum install git python-pip -y
# pip安装ansible(国内如果安装太慢可以直接用pip阿里云加速)
pip install pip --upgrade -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
pip install --no-cache-dir ansible -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
# pip install pip --upgrade
# pip install ansible
```

### 在deploy节点配置免密码登陆

``` bash
ssh-keygen -t rsa -b 2048 -C "deploy@123.com" -P ''
ssh-copy-id $IPs #$IPs为所有节点地址包括自身，按照提示输入yes 和root密码
```
### 在deploy节点编排k8s安装
- 从发布页面 https://github.com/gjmzj/kubeasz/releases 下载源码解压到同样目录

``` bash
  wget https://github.com/gjmzj/kubeasz/archive/0.2.1.zip
  unzip 0.2.1.zip
  mv kubeasz-0.2.1/* /etc/ansible/
```
- 下载二进制文件
请从分享的[百度云链接](https://pan.baidu.com/s/1c4RFaA)，下载解压到/etc/ansible/bin目录，如果你有合适网络环境也可以按照/down/download.sh自行从官网下载各种tar包

``` bash
tar xf k8s.1-10-4.tar.gz
mv bin/* /etc/ansible/bin
```
- [可选]下载离线docker镜像
服务器使用内部yum源/apt源，但是无法访问公网情况下，请下载离线docker镜像完成集群安装；从百度云盘把`basic_images_kubeasz_x.y.tar.gz` 下载解压到`/etc/ansible/down` 目录

``` bash
tar xf basic_images_kubeasz_0.2.tar.gz -C /etc/ansible/down
```

- 4.3 配置集群参数

``` bash
cd /etc/ansible
cp example/hosts.m-masters.example hosts
vim hosts                       # 根据实际情况修改此hosts文件

[root@by-deploy01 ~]# cat /etc/ansible/hosts
# 部署节点：运行这份 ansible 脚本的节点
[deploy]
172.17.0.91
# etcd集群请提供如下NODE_NAME，注意etcd集群必须是1,3,5,7...奇数个节点
[etcd]
172.17.0.85 NODE_NAME=etcd1
172.17.0.86 NODE_NAME=etcd2
172.17.0.87 NODE_NAME=etcd3

[kube-master]
172.17.0.85
172.17.0.86
172.17.0.87

# 负载均衡至少两个节点，安装 haproxy+keepalived
# 如果是公有云环境请优先使用云上负载均衡，lb组留空
# 我这里是自己手动yum装的haproxy
[lb]

[kube-node]
172.17.0.88
172.17.0.89
172.17.0.90

# 如果启用harbor，请配置后面harbor相关参数
[harbor]
#192.168.1.8

[kube-cluster:children]
kube-node
kube-master

# 预留组，后续添加master节点使用
[new-master]
#192.168.1.5

# 预留组，后续添加node节点使用
[new-node]
#192.168.1.xx

[all:vars]
# ---------集群主要参数---------------
#集群部署模式：allinone, single-master, multi-master
DEPLOY_MODE=multi-master

#集群主版本号，目前支持: v1.8, v1.9, v1.10
K8S_VER="v1.11"

# 集群 MASTER IP即 LB节点VIP地址，为区别与默认apiserver端口，设置VIP监听的服务端口8443
# 公有云上请使用云负载均衡内网地址和监听端口
MASTER_IP="172.17.0.92"
KUBE_APISERVER="https://{{ MASTER_IP }}:8443"

#TLS Bootstrapping 使用的 Token，使用 head -c 16 /dev/urandom | od -An -t x | tr -d ' ' 生成
BOOTSTRAP_TOKEN="xxxxxxxxxxxxxxxxxxxx"

# 集群网络插件，目前支持calico, flannel, kube-router
CLUSTER_NETWORK="calico"

# 默认使用kube-proxy, 可选SERVICE_PROXY="IPVS" (前提是网络选择kube-router)
SERVICE_PROXY="kube-proxy"

# 服务网段 (Service CIDR），注意不要与内网已有网段冲突
SERVICE_CIDR="10.68.0.0/16"

# POD 网段 (Cluster CIDR），注意不要与内网已有网段冲突
CLUSTER_CIDR="172.20.0.0/16"

# 服务端口范围 (NodePort Range)
NODE_PORT_RANGE="20000-40000"

# kubernetes 服务 IP (预分配，一般是 SERVICE_CIDR 中第一个IP)
CLUSTER_KUBERNETES_SVC_IP="10.68.0.1"

# 集群 DNS 服务 IP (从 SERVICE_CIDR 中预分配)
CLUSTER_DNS_SVC_IP="10.68.0.2"

# 集群 DNS 域名
CLUSTER_DNS_DOMAIN="cluster.local."

# etcd 集群间通信的IP和端口, 根据etcd组成员自动生成
TMP_NODES="{% for h in groups['etcd'] %}{{ hostvars[h]['NODE_NAME'] }}=https://{{ h }}:2380,{% endfor %}"
ETCD_NODES="{{ TMP_NODES.rstrip(',') }}"

# etcd 集群服务地址列表, 根据etcd组成员自动生成
TMP_ENDPOINTS="{% for h in groups['etcd'] %}https://{{ h }}:2379,{% endfor %}"
ETCD_ENDPOINTS="{{ TMP_ENDPOINTS.rstrip(',') }}"

# 集群basic auth 使用的用户名和密码
BASIC_AUTH_USER="admin"
BASIC_AUTH_PASS="xxxxxx"

# ---------附加参数--------------------
#默认二进制文件目录
bin_dir="/opt/kube/bin"

#证书目录
ca_dir="/etc/kubernetes/ssl"

#部署目录，即 ansible 工作目录，建议不要修改
base_dir="/etc/ansible"

#私有仓库 harbor服务器 (域名或者IP)
#HARBOR_IP="192.168.1.8"
#HARBOR_DOMAIN="harbor.yourdomain.com"
```

- 验证ansible安装，正常能看到每个节点返回 SUCCESS
```bash
ansible all -m ping
```
- 4.4 开始安装
如果你对集群安装流程不熟悉，请阅读项目首页 **安装步骤** 讲解后分步安装，并对 **每步都进行验证**  

``` bash
# 分步安装
cd /etc/ansible/
ansible-playbook 01.prepare.yml
ansible-playbook 02.etcd.yml
ansible-playbook 03.docker.yml
ansible-playbook 04.kube-master.yml
ansible-playbook 05.kube-node.yml
ansible-playbook 06.network.yml
ansible-playbook 07.cluster-addon.yml
# 一步安装
# ansible-playbook 90.setup.yml
```

+ [可选]对集群所有节点进行操作系统层面的安全加固 `ansible-playbook roles/os-harden/os-harden.yml`，详情请参考[os-harden项目](https://github.com/dev-sec/ansible-os-hardening)

# 分步安装说明
## 01.prepare.yml
yum安装增加几个包：
- conntrack
- ipvsadm
- ipset

## 03.docker.yml
更改docker服务存储路径：
```bash
ExecStart=/usr/local/bin/dockerd --graph=/opt/docker
```

## 04.kube-master.yml
- 得先配置好slb+haproxy 不然会报错
```bash
yum install haproxy -y
vim /etc/haproxy/haproxy.cfg

global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/admin.sock mode 660 level admin
        stats timeout 30s
        user haproxy
        group haproxy
        daemon
        nbproc 1

defaults
        log     global
        timeout connect 5000
        timeout client  10m
        timeout server  10m

listen kube-master
        bind 0.0.0.0:6443
        mode tcp
        option tcplog
        balance source
        server master01 172.17.0.85:6443 check inter 2000 fall 2 rise 2 weight 1
        server master01 172.17.0.86:6443 check inter 2000 fall 2 rise 2 weight 1
        server master01 172.17.0.87:6443 check inter 2000 fall 2 rise 2 weight 1
```
- 更改kubelet pod 存储路径
```bash
vim /etc/ansible/roles/kube-node/templates/kubelet.service.j2
  --root-dir=/opt/kubelet \
```
- calico metrics 检查是否开启
```bash
FELIX_PROMETHEUSMETRICSENABLED ture
FELIX_PROMETHEUSMETRICSPORT 9091

ports:
- containerPort: 9091
  hostPort: 9091
  name: http-metrics
```
- 更改kube-proxy负载模式为ipvs
```bash
vim /etc/ansible/roles/kube-node/templates/kube-proxy.service.j2
  --proxy-mode=ipvs \
```
> iptables清理：
```bash
iptables -F &&  iptables -X && iptables -F -t nat &&  iptables -X -t nat
```

# 验证集群功能
- 查看集群状态
```bash
kubectl get node
kubectl cluster-info
kubectl  top node
kubectl get ep --all-namespaces -o yaml
```
- 创建测试文件
```bash
cat > nginx-ds.yml <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-ds
  labels:
    app: nginx-ds
spec:
  type: NodePort
  selector:
    app: nginx-ds
  ports:
  - name: http
    port: 80
    targetPort: 80
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: nginx-ds
  labels:
    addonmanager.kubernetes.io/mode: Reconcile
spec:
  template:
    metadata:
      labels:
        app: nginx-ds
    spec:
      containers:
      - name: my-nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
EOF

kubectl create -f nginx-ds.yml

kubectl get pods  -o wide|grep nginx-ds
kubectl get svc  -o wide|grep nginx-ds

# 检查nodeport可用性
curl 172.17.0.89:36855
# 出现nginx欢迎页
```
- 验证coredns
```bash
kubectl exec  -it nginx-ds-8sd6z /bin/bash

cat /etc/resolv.conf

nameserver 10.68.0.2
search default.svc.cluster.local. svc.cluster.local. cluster.local.
options ndots:5

ping nginx-ds

PING nginx-ds.default.svc.cluster.local (10.68.242.108): 48 data bytes
56 bytes from 10.68.242.108: icmp_seq=0 ttl=64 time=0.049 ms
56 bytes from 10.68.242.108: icmp_seq=1 ttl=64 time=0.063 ms
^C--- nginx-ds.default.svc.cluster.local ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max/stddev = 0.049/0.056/0.063/0.000 ms

# 能成功解析服务到ip证明dns正常
```
- 验证dashboard
```bash
# kube-apiserver 访问 dashboard
kubectl cluster-info
# 也可以通过nodeport 访问
kubectl get svc -n kube-system

# 创建登录 Dashboard 的 token 和 kubeconfig 配置文件
# 上面提到，Dashboard 默认只支持 token 认证，所以如果使用 KubeConfig 文件，需要在该文件中指定 token，不支持使用 client 证书认证。

# 创建登录 token
kubectl create sa dashboard-admin -n kube-system
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
ADMIN_SECRET=$(kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}')
DASHBOARD_LOGIN_TOKEN=$(kubectl describe secret -n kube-system ${ADMIN_SECRET} | grep -E '^token' | awk '{print $2}')
echo ${DASHBOARD_LOGIN_TOKEN}

# 创建使用 token 的 KubeConfig 文件
# 设置集群参数
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=https://172.17.0.92:8443 \
  --kubeconfig=dashboard.kubeconfig

# 设置客户端认证参数，使用上面创建的 Token
kubectl config set-credentials dashboard_user \
  --token=${DASHBOARD_LOGIN_TOKEN} \
  --kubeconfig=dashboard.kubeconfig

# 设置上下文参数
kubectl config set-context default \
  --cluster=kubernetes \
  --user=dashboard_user \
  --kubeconfig=dashboard.kubeconfig

# 设置默认上下文
kubectl config use-context default --kubeconfig=dashboard.kubeconfig
```
用生成的 dashboard.kubeconfig 登录 Dashboard。