<!-- toc -->

tags: kubectl

# 部署 kubectl 命令行工具

kubectl 默认从 `~/.kube/config` 配置文件获取访问 kube-apiserver 地址、证书、用户名等信息，如果没有配置该文件，执行命令时出错：

``` bash
$  kubectl get pods
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

本文档介绍下载和配置 kubernetes 集群命令行工具 kubectl 的步骤。

根据前一章的配置kubectl访问kube-apiserver服务将通过haproxy来转发，因此kubectl里配置的kube-apiserver就是haproxy的6443端口。

需要将下载的 kubectl 二进制程序和生成的 `~/.kube/config` 配置文件拷贝到**所有使用 kubectl 命令的机器**。


## 下载 kubectl

``` bash
$ wget https://dl.k8s.io/v1.6.2/kubernetes-client-linux-amd64.tar.gz
$ tar -xzvf kubernetes-client-linux-amd64.tar.gz
$ sudo cp kubernetes/client/bin/kube* /usr/local/bin/
$ chmod a+x /usr/local/bin/kube*
$ export PATH=/usr/local/bin:$PATH
$
```

## 创建 admin 证书

kubectl 与 kube-apiserver 的安全端口通信，需要为安全通信提供 TLS 证书和秘钥。

创建 admin 证书签名请求

``` bash
$ cat admin-csr.json
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
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "system:masters",
      "OU": "System"
    }
  ]
}
```

+ 后续 `kube-apiserver` 使用 `RBAC` 对客户端(如 `kubelet`、`kube-proxy`、`Pod`)请求进行授权；
+ `kube-apiserver` 预定义了一些 `RBAC` 使用的 `RoleBindings`，如 `cluster-admin` 将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` **所有 API**的权限；
+ O 指定该证书的 Group 为 `system:masters`，`kubelet` 使用该证书访问 `kube-apiserver` 时 ，由于证书被 CA 签名，所以认证通过，同时由于证书用户组为经过预授权的 `system:masters`，所以被授予访问所有 API 的权限；
+ hosts 属性值为空列表；

生成 admin 证书和私钥：

``` bash
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=/etc/kubernetes/ssl/ca-config.json \
  -profile=kubernetes admin-csr.json | cfssljson -bare admin
$ ls admin*
admin.csr  admin-csr.json  admin-key.pem  admin.pem
$ sudo mv admin*.pem /etc/kubernetes/ssl/
$ rm admin.csr admin-csr.json
$
```

## 创建 kubectl kubeconfig 文件

``` bash
$ # 设置集群参数
$ kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER}
$ # 设置客户端认证参数
$ kubectl config set-credentials admin \
  --client-certificate=/etc/kubernetes/ssl/admin.pem \
  --embed-certs=true \
  --client-key=/etc/kubernetes/ssl/admin-key.pem
$ # 设置上下文参数
$ kubectl config set-context kubernetes \
  --cluster=kubernetes \
  --user=admin
$ # 设置默认上下文
$ kubectl config use-context kubernetes
```

+ `admin.pem` 证书 O 字段值为 `system:masters`，`kube-apiserver` 预定义的 RoleBinding `cluster-admin` 将 Group `system:masters` 与 Role `cluster-admin` 绑定，该 Role 授予了调用`kube-apiserver` 相关 API 的权限；
+ 生成的 kubeconfig 被保存到 `~/.kube/config` 文件；

## 分发 kubeconfig 文件

将 `~/.kube/config` 文件拷贝到运行 `kubelet` 命令的机器的 `~/.kube/` 目录下。


# 通过 kubectl 命令行查看master的高可用配置

## 查看master内部组件的运行状态
``` bash
$ kubectl get componentstatuses
NAME                 STATUS    MESSAGE              ERROR
scheduler            Healthy   ok                   
controller-manager   Healthy   ok                   
etcd-1               Healthy   {"health": "true"}   
etcd-2               Healthy   {"health": "true"}   
etcd-0               Healthy   {"health": "true"}
```

+ 因为kube-apiserver是无状态的服务，kube-controller-manager和kube-scheduler为有状态服务，同一时间只有一台当选，会在三台master机之间进行选举，由其中一台担任leader的角色。

## 查看kube-controller-manager和kube-scheduler组件的选举情况

``` bash
$ kubectl get endpoints -n kube-system  kube-controller-manager -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master2","leaseDurationSeconds":15,"acquireTime":"2017-12-06T02:29:37Z","renewTime":"2017-12-07T01:24:27Z","leaderTransitions":0}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "160718"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-controller-manager
  uid: 4e03d332-da2d-11e7-a4f0-54520009ec1e
subsets: []
```
+ 从输出结果可以看出kube-controller-manager的leader在k8s-master2上。

``` bash
$ kubectl get endpoints -n kube-system  kube-scheduler -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master2","leaseDurationSeconds":15,"acquireTime":"2017-12-06T02:29:37Z","renewTime":"2017-12-07T01:26:02Z","leaderTransitions":0}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-scheduler  namespace: kube-system
  resourceVersion: "160841"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-scheduler
  uid: 4e2d414a-da2d-11e7-a4f0-54520009ec1e
subsets: []
```
+ 从输出结果可以看出kube-scheduler的leader在k8s-master2上。

## 测试kube-apiserver断开kube-controller-manager和kube-scheduler组件的重新选举和切换情况
从上面的输出可以看出kube-controller-manager和kube-scheduler的leader都在k8s-master2上，现在去k8s-master2上停止kube-apiserver服务。

``` bash
$ systemctl stop kube-apiserver
$ kubectl get endpoints -n kube-system  kube-controller-manager -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master2","leaseDurationSeconds":15,"acquireTime":"2017-12-06T02:29:37Z","renewTime":"2017-12-07T01:29:05Z","leaderTransitions":0}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "161084"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-controller-manager
  uid: 4e03d332-da2d-11e7-a4f0-54520009ec1e
subsets: []
$ kubectl get endpoints -n kube-system  kube-scheduler -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master2","leaseDurationSeconds":15,"acquireTime":"2017-12-06T02:29:37Z","renewTime":"2017-12-07T01:29:06Z","leaderTransitions":0}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-scheduler
  namespace: kube-system
  resourceVersion: "161086"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-scheduler
  uid: 4e2d414a-da2d-11e7-a4f0-54520009ec1e
subsets: []
$ kubectl get endpoints -n kube-system  kube-controller-manager -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master1","leaseDurationSeconds":15,"acquireTime":"2017-12-07T01:29:25Z","renewTime":"2017-12-07T01:29:45Z","leaderTransitions":1}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-controller-manager
  namespace: kube-system
  resourceVersion: "161125"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-controller-manager
  uid: 4e03d332-da2d-11e7-a4f0-54520009ec1e
subsets: []
$ kubectl get endpoints -n kube-system  kube-scheduler -o yaml
apiVersion: v1
kind: Endpoints
metadata:
  annotations:
    control-plane.alpha.kubernetes.io/leader: '{"holderIdentity":"k8s-master1","leaseDurationSeconds":15,"acquireTime":"2017-12-07T01:29:24Z","renewTime":"2017-12-07T01:29:56Z","leaderTransitions":1}'
  creationTimestamp: 2017-12-06T02:29:37Z
  name: kube-scheduler
  namespace: kube-system
  resourceVersion: "161139"
  selfLink: /api/v1/namespaces/kube-system/endpoints/kube-scheduler
  uid: 4e2d414a-da2d-11e7-a4f0-54520009ec1e
subsets: []
```

停掉k8s-master2上的kube-apiserver服务后，立即查看kube-controller-manager和kube-scheduler的leader还是在k8s-master2上，稍等5-10秒左右再次查看就发现
kube-controller-manager和kube-scheduler的leader都切换到k8s-master1上。
