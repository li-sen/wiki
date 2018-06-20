# 部署 harbor 私有仓库

本文档介绍使用 docker-compose 部署 harbor 私有仓库的步骤。

## 安装和配置 docker

### 下载最新的 docker 二进制文件

``` bash
$ wget https://get.docker.com/builds/Linux/x86_64/docker-17.04.0-ce.tgz
$ tar -xvf docker-17.04.0-ce.tgz
$ cp docker/docker* /usr/local/bin
$ cp docker/completion/bash/docker /etc/bash_completion.d/
$
```

### 创建 docker 的 systemd unit 文件

``` bash
$ cat docker.service
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io

[Service]
Environment="PATH=/usr/local/bin:/bin:/sbin:/usr/bin:/usr/sbin"
EnvironmentFile=-/run/flannel/docker
ExecStart=/usr/local/bin/dockerd --log-level=error $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Delegate=yes
KillMode=process

[Install]
WantedBy=multi-user.target
```

+ dockerd 运行时会调用其它 docker 命令，如 docker-proxy，所以需要将 docker 命令所在的目录加到 PATH 环境变量中；
+ flanneld 启动时将网络配置写入到 `/run/flannel/docker` 文件中的变量 `DOCKER_NETWORK_OPTIONS`，dockerd 命令行上指定该变量值来设置 docker0 网桥参数；
+ 如果指定了多个 `EnvironmentFile` 选项，则必须将 `/run/flannel/docker` 放在最后(确保 docker0 使用 flanneld 生成的 bip 参数)；
+ 不能关闭默认开启的 `--iptables` 和 `--ip-masq` 选项；
+ 如果内核版本比较新，建议使用 `overlay` 存储驱动；
+ docker 从 1.13 版本开始，可能将 **iptables FORWARD chain的默认策略设置为DROP**，从而导致 ping 其它 Node 上的 Pod IP 失败，遇到这种情况时，需要手动设置策略为 `ACCEPT`：

  ``` bash
  $ sudo iptables -P FORWARD ACCEPT
  $
  ```
  并且把以下命令写入/etc/rc.local文件中，防止节点重启**iptables FORWARD chain的默认策略又还原为DROP**

  ``` bash
  sleep 60 && /sbin/iptables -P FORWARD ACCEPT
  ```


+ 为了加快 pull image 的速度，可以使用国内的仓库镜像服务器，同时增加下载的并发数。(如果 dockerd 已经运行，则需要重启 dockerd 生效。)

    ``` bash
    $ cat /etc/docker/daemon.json
    {
      "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn", "hub-mirror.c.163.com"],
      "max-concurrent-downloads": 10
    }
    ```

完整 unit 见 [docker.service](https://github.com/opsnull/follow-me-install-kubernetes-cluster/blob/master/systemd/docker.service)

### 启动 dockerd

``` bash
$ sudo cp docker.service /etc/systemd/system/docker.service
$ sudo systemctl daemon-reload
$ sudo systemctl stop firewalld
$ sudo systemctl disable firewalld
$ sudo iptables -F && sudo iptables -X && sudo iptables -F -t nat && sudo iptables -X -t nat
$ sudo systemctl enable docker
$ sudo systemctl start docker
$
```

+ 需要关闭 firewalld(centos7)/ufw(ubuntu16.04)，否则可能会重复创建的 iptables 规则；
+ 最好清理旧的 iptables rules 和 chains 规则；

### 检查 docker 服务

``` bash
$ docker version
$
```
## 安装和配置 docker-compose

## 下载文件

从 docker compose [发布页面](https://github.com/docker/compose/releases)下载最新的 `docker-compose` 二进制文件

``` bash
$ wget https://github.com/docker/compose/releases/download/1.12.0/docker-compose-Linux-x86_64
$ mv ~/docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
$ chmod a+x  /usr/local/bin/docker-compose
$
```

从 harbor [发布页面](https://github.com/vmware/harbor/releases)下载最新的 harbor 离线安装包

``` bash
$ wget  --continue https://github.com/vmware/harbor/releases/download/v1.1.0/harbor-offline-installer-v1.1.0.tgz
$ tar -xzvf harbor-offline-installer-v1.1.0.tgz
$ cd harbor
$
```

## 导入 docker images

导入离线安装包中 harbor 相关的 docker images：

``` bash
$ docker load -i harbor.v1.1.0.tar.gz
$
```

## 创建 harbor nginx 服务器使用的 TLS 证书

创建 harbor 证书签名请求：

``` bash
$ cat > harbor-csr.json <<EOF
{
  "CN": "harbor",
  "hosts": [
    "127.0.0.1",
    "$NODE_IP"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

+ hosts 字段指定授权使用该证书的当前部署节点 IP，如果后续使用域名访问 harbor则还需要添加域名；

生成 harbor 证书和私钥：

``` bash
$ cfssl gencert -ca=/etc/kubernetes/ssl/ca.pem \
  -ca-key=/etc/kubernetes/ssl/ca-key.pem \
  -config=/etc/kubernetes/ssl/ca-config.json \
  -profile=kubernetes harbor-csr.json | cfssljson -bare harbor
$ ls harbor*
harbor.csr  harbor-csr.json  harbor-key.pem harbor.pem
$ sudo mkdir -p /etc/harbor/ssl
$ sudo mv harbor*.pem /etc/harbor/ssl
$ rm harbor.csr  harbor-csr.json
```

## 修改 harbor.cfg 文件

``` bash
$ diff harbor.cfg.orig harbor.cfg
5c5
< hostname = reg.mydomain.com
---
> hostname = 192.168.5.103
9c9
< ui_url_protocol = http
---
> ui_url_protocol = https
24,25c24,25
< ssl_cert = /data/cert/server.crt
< ssl_cert_key = /data/cert/server.key
---
> ssl_cert = /etc/harbor/ssl/harbor.pem
> ssl_cert_key = /etc/harbor/ssl/harbor-key.pem
```

## 加载和启动 harbor 镜像

``` bash
[root@k8s-harbor /opt/harbor]# ./install.sh

[Step 0]: checking installation environment ...

Note: docker version: 17.04.0

Note: docker-compose version: 1.12.0

[Step 1]: loading Harbor images ...
Loaded image: vmware/registry:2.6.2-photon
Loaded image: photon:1.0
Loaded image: vmware/notary-photon:signer-0.5.0
Loaded image: vmware/clair:v2.0.1-photon
Loaded image: vmware/harbor-ui:v1.2.0
Loaded image: vmware/harbor-log:v1.2.0
Loaded image: vmware/harbor-db:v1.2.0
Loaded image: vmware/nginx-photon:1.11.13
Loaded image: vmware/postgresql:9.6.4-photon
Loaded image: vmware/harbor-adminserver:v1.2.0
Loaded image: vmware/harbor-jobservice:v1.2.0
Loaded image: vmware/notary-photon:server-0.5.0
Loaded image: vmware/harbor-notary-db:mariadb-10.1.10


[Step 2]: preparing environment ...
loaded secret from file: /data/secretkey
Generated configuration file: ./common/config/nginx/nginx.conf
Generated configuration file: ./common/config/adminserver/env
Generated configuration file: ./common/config/ui/env
Generated configuration file: ./common/config/registry/config.yml
Generated configuration file: ./common/config/db/env
Generated configuration file: ./common/config/jobservice/env
Generated configuration file: ./common/config/jobservice/app.conf
Generated configuration file: ./common/config/ui/app.conf
Generated certificate, key file: ./common/config/ui/private_key.pem, cert file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.


[Step 3]: checking existing instance of Harbor ...


[Step 4]: starting Harbor ...
Creating network "harbor_harbor" with the default driver
Creating harbor-log
Creating harbor-adminserver
Creating registry
Creating harbor-db

ERROR: for registry  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)

ERROR: for adminserver  UnixHTTPConnectionPool(host='localhost', port=None): Read timed out. (read timeout=60)
ERROR: An HTTP request took too long to complete. Retry with --verbose to obtain debug information.
If you encounter this issue regularly because of slow network conditions, consider setting COMPOSE_HTTP_TIMEOUT to a higher value (current value: 60).
本地执行的时候失败了，采取另外的一种方式来运行
[root@k8s-harbor /opt/harbor]# ./prepare
Clearing the configuration file: ./common/config/db/env
Clearing the configuration file: ./common/config/registry/config.yml
Clearing the configuration file: ./common/config/registry/root.crt
Clearing the configuration file: ./common/config/adminserver/env
Clearing the configuration file: ./common/config/jobservice/env
Clearing the configuration file: ./common/config/jobservice/app.conf
Clearing the configuration file: ./common/config/ui/env
Clearing the configuration file: ./common/config/ui/app.conf
Clearing the configuration file: ./common/config/ui/private_key.pem
Clearing the configuration file: ./common/config/nginx/nginx.conf
Clearing the configuration file: ./common/config/nginx/cert/harbor-key.pem
Clearing the configuration file: ./common/config/nginx/cert/harbor.pem
loaded secret from file: /data/secretkey
Generated configuration file: ./common/config/nginx/nginx.conf
Generated configuration file: ./common/config/adminserver/env
Generated configuration file: ./common/config/ui/env
Generated configuration file: ./common/config/registry/config.yml
Generated configuration file: ./common/config/db/env
Generated configuration file: ./common/config/jobservice/env
Generated configuration file: ./common/config/jobservice/app.conf
Generated configuration file: ./common/config/ui/app.conf
Generated certificate, key file: ./common/config/ui/private_key.pem, cert file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.
[root@k8s-harbor /opt/harbor]# docker-compose up -d
harbor-log is up-to-date
Starting registry
Recreating harbor-adminserver
harbor-db is up-to-date
Creating harbor-ui
Creating harbor-jobservice
Creating nginx
[root@k8s-harbor /opt/harbor]# docker-compose ps
       Name                     Command               State                                Ports                               
------------------------------------------------------------------------------------------------------------------------------
harbor-adminserver   /harbor/harbor_adminserver       Up                                                                       
harbor-db            docker-entrypoint.sh mysqld      Up      3306/tcp                                                         
harbor-jobservice    /harbor/harbor_jobservice        Up                                                                       
harbor-log           /bin/sh -c crond && rm -f  ...   Up      127.0.0.1:1514->514/tcp                                          
harbor-ui            /harbor/harbor_ui                Up                                                                       
nginx                nginx -g daemon off;             Up      0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp, 0.0.0.0:80->80/tcp
registry             /entrypoint.sh serve /etc/ ...   Up      5000/tcp                                                        

```

## 访问管理界面

浏览器访问 `https://192.168.5.103`，示例的是 `https://192.168.5.103`

用账号 `admin` 和 harbor.cfg 配置文件中的默认密码 `Harbor12345` 登陆系统：

![harbor](./images/harbo.png)

## harbor 运行时产生的文件、目录

``` bash
$ # 日志目录
$ ls /var/log/harbor/2017-04-19/
adminserver.log  jobservice.log  mysql.log  proxy.log  registry.log  ui.log
$ # 数据目录，包括数据库、镜像仓库
$ ls /data/
ca_download  config  database  job_logs registry  secretkey
```

## docker 客户端登陆

将签署 harbor 证书的 CA 证书拷贝到 `/etc/docker/certs.d/192.168.5.103` 目录下

``` bash
$ sudo mkdir -p /etc/docker/certs.d/192.168.5.103
$ sudo cp /etc/kubernetes/ssl/ca.pem /etc/docker/certs.d/192.168.5.103/ca.crt
$
```

登陆 harbor

``` bash
$ docker login 192.168.5.103
Username: admin
Password:
```

认证信息自动保存到 `~/.docker/config.json` 文件。

## 其它操作

下列操作的工作目录均为 解压离线安装文件后 生成的 harbor 目录。

``` bash
$ # 停止 harbor
$ docker-compose down -v
$ # 修改配置
$ vim harbor.cfg
$ # 更修改的配置更新到 docker-compose.yml 文件
[root@tjwq01-sys-bs003007 harbor]# ./prepare
Clearing the configuration file: ./common/config/ui/app.conf
Clearing the configuration file: ./common/config/ui/env
Clearing the configuration file: ./common/config/ui/private_key.pem
Clearing the configuration file: ./common/config/db/env
Clearing the configuration file: ./common/config/registry/root.crt
Clearing the configuration file: ./common/config/registry/config.yml
Clearing the configuration file: ./common/config/jobservice/app.conf
Clearing the configuration file: ./common/config/jobservice/env
Clearing the configuration file: ./common/config/nginx/cert/admin.pem
Clearing the configuration file: ./common/config/nginx/cert/admin-key.pem
Clearing the configuration file: ./common/config/nginx/nginx.conf
Clearing the configuration file: ./common/config/adminserver/env
loaded secret from file: /data/secretkey
Generated configuration file: ./common/config/nginx/nginx.conf
Generated configuration file: ./common/config/adminserver/env
Generated configuration file: ./common/config/ui/env
Generated configuration file: ./common/config/registry/config.yml
Generated configuration file: ./common/config/db/env
Generated configuration file: ./common/config/jobservice/env
Generated configuration file: ./common/config/jobservice/app.conf
Generated configuration file: ./common/config/ui/app.conf
Generated certificate, key file: ./common/config/ui/private_key.pem, cert file: ./common/config/registry/root.crt
The configuration files are ready, please use docker-compose to start the service.
$ # 启动 harbor
[root@tjwq01-sys-bs003007 harbor]# docker-compose up -d
```
