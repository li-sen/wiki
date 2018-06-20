<!-- toc -->

tags: master, kube-apiserver, kube-scheduler, kube-controller-manager

# 部署 haproxy


+ 上一章介绍部署2个单机 kubernetes master 节点的步骤，本章在2个单机kubernetes master 节点前面加上一个haproxy的负载。

+ 客户端 (kubectl、kubelet、kube-proxy) 使用 haproxy 的 监听地址和端口 来访问 kube-apiserver，从而实现高可用 master 集群。


## 安装haproxy

``` bash
$ yum -y install haproxy
$ 

```


## 配置 安装haproxy

``` bash
# more /etc/haproxy/haproxy.cfg
global
  log 127.0.0.1 local0
  log 127.0.0.1 local1 notice
  maxconn 4096
  chroot /usr/share/haproxy
  user haproxy
  group haproxy
  daemon
defaults
     log global
     mode tcp
     timeout connect 5000ms
     timeout client 50000ms
     timeout server 50000ms
frontend stats-front
  bind *:8088
  mode http
  default_backend stats-back
backend stats-back
  mode http
  balance source
  stats uri /stats
  stats auth admin:passw0rd
listen Kubernetes-Cluster
  bind *:6443
  balance leastconn
  mode tcp
  server master1 192.168.5.104:6443 check inter 2000 fall 3
  server master2 192.168.5.105:6443 check inter 2000 fall 3

  ```

+  将haproxy.cfg按照上述内容修改，开启了2个监听端口，8088为haproxy的状态页面，6443为kubernet master的vip。

## 启动haproxy

``` bash
$ systemctl start haproxy
$ ss -tnlp
State       Recv-Q Send-Q                                                               Local Address:Port                                                                 Peer Address:Port 
LISTEN      0      100                                                                      127.0.0.1:25                                                                              *:*      users:(("master",1157,13))
LISTEN      0      128                                                                              *:6443                                                                            *:*      users:(("haproxy",1770,6))
LISTEN      0      128                                                                              *:22                                                                              *:*      users:(("sshd",769,3))
LISTEN      0      128                                                                              *:8088                                                                            *:*      users:(("haproxy",1770,4))
LISTEN      0      100                                                                            ::1:25                                                                             :::*      users:(("master",1157,14))
LISTEN      0      50                                                                              :::57247                                                                          :::*      users:(("java",11411,175))
LISTEN      0      128                                                                             :::22                                                                             :::*      users:(("sshd",769,4))

$ telnet 192.168.5.109 6443
Trying 192.168.5.109...
Connected to 192.168.5.109.
Escape character is '^]'.
^CConnection closed by foreign host.
```

+ 也可以登陆haproxy的状态管理页面http://192.168.5.109:8088/stats，用户名为admin，密码为passw0rd。


