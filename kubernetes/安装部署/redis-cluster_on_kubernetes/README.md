> 此部署是参考 [sanderploegsma/redis-cluster](https://github.com/sanderploegsma/redis-cluster)
# 环境说明
组件|版本
:---:|:---: 
k8s|v1.11.0
存储|ceph version 12.2.6
> 相关环境搭建可以在我wiki中找到
# 自定redis镜像
```bash
docker build -t 172.17.0.91/k8s/redis:v1 .
docker images

# 因为我的环境是镜像仓库的，所以我就直接导出镜像，然后分发到node节点在导入
docker save 172.17.0.91/k8s/redis > /root/redis.tar.gz 
ansible kube-node -m copy -a 'src=/root/redis.tar.gz dest=/root'

[root@by-node01 ~]# docker load < redis.tar.gz

```

# 部署
```bash
# 需要根据你实际环境改两个地方：1. 镜像地址  2. storageClassName
kubectl apply -f redis-cluster.yml

[root@by-deploy01 redis-cluster]# kubectl get pod
NAME              READY     STATUS    RESTARTS   AGE
redis-cluster-0   1/1       Running   0          21m
redis-cluster-1   1/1       Running   0          17m
redis-cluster-2   1/1       Running   0          16m
redis-cluster-3   1/1       Running   0          16m
redis-cluster-4   1/1       Running   0          15m
redis-cluster-5   1/1       Running   0          15m

# redis配置文件说明
redis.conf: |+
    cluster-enabled yes # 开启集群功能
    cluster-require-full-coverage no # 在部分key所在的节点不可用时，如果此参数设置为"yes"(默认值), 则整个集群停止接受操作；如果此参数设置为”no”，则集群依然为可达节点上的key提供读操作。
    cluster-node-timeout 15000 # 集群中的节点能够失联的最大时间，超过这个时间，该节点就会被认为故障，会进行迁移
    cluster-config-file /data/nodes.conf # 名字叫"集群配置文件"，但是此配置文件不能人工编辑，它是集群节点自动维护的文件，主要用于记录集群中有哪些节点、他们的状态以及一些持久化参数等，方便在重启时恢复这些状态。通常是在收到请求之后这个文件就会被更新。
    cluster-migration-barrier 1 # 主节点需要的最小从节点数
    appendonly yes # 开启AOF模式
    protected-mode no # 关闭保护模式，允许远程访问
    
```

# 集群初始化
```bash

kubectl exec -it redis-cluster-0 -- redis-trib create --replicas 1 \
$(kubectl get pods -l app=redis-cluster -o jsonpath='{range.items[*]}{.status.podIP}:6379 ')
# 会有提示 输入yes
# 选项--replicas 1表示我们想为每个master指定一个slave。其余参数是需要加到集群的实例地址。
# 初始化后会得到3主3从
```
# 查看集群状态
```bash
kubectl exec -it redis-cluster-0 /bin/bash
root@redis-cluster-0:/data# redis-cli -h redis-cluster -c
redis-cluster:6379> CLUSTER INFO
redis-cluster:6379> CLUSTER NODES
```
# 集群测试
```bash
root@redis-cluster-0:/data# redis-cli -h redis-cluster -c
redis-cluster:6379> set foo bar
-> Redirected to slot [12182] located at 172.20.8.34:6379
OK
172.20.8.34:6379> set hello world
-> Redirected to slot [866] located at 172.20.236.158:6379
OK
172.20.236.158:6379> get hello
"world"
172.20.236.158:6379> get foo
-> Redirected to slot [12182] located at 172.20.8.34:6379
"bar"
172.20.8.34:6379> del foo
(integer) 1
172.20.8.34:6379> del hello
-> Redirected to slot [866] located at 172.20.236.158:6379
(integer) 1
172.20.236.158:6379> get foo
-> Redirected to slot [12182] located at 172.20.8.34:6379
(nil)
172.20.8.34:6379> get hello
-> Redirected to slot [866] located at 172.20.236.158:6379
(nil)
172.20.236.158:6379> quit
```

# 集群常用操作
> copy原作者；由于时间仓促，自己没有验证，后续有机会再来核实正确性。
## Adding nodes
Adding nodes to the cluster involves a few manual steps. First, let's add two nodes:
``` bash
kubectl scale statefulset redis-cluster --replicas=8
```

Have the first new node join the cluster as master:
``` bash
kubectl exec redis-cluster-0 -- redis-trib add-node \
$(kubectl get pod redis-cluster-6 -o jsonpath='{.status.podIP}'):6379 \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379
```

The second new node should join the cluster as slave. This will automatically bind to the master with the least slaves (in this case, `redis-cluster-6`)
``` bash
kubectl exec redis-cluster-0 -- redis-trib add-node --slave \
$(kubectl get pod redis-cluster-7 -o jsonpath='{.status.podIP}'):6379 \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379
```

Finally, automatically rebalance the masters:
``` bash
kubectl exec redis-cluster-0 -- redis-trib rebalance --use-empty-masters \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379
```

## Removing nodes

### Removing slaves
Slaves can be deleted safely. First, let's get the id of the slave:

``` bash
$ kubectl exec redis-cluster-7 -- redis-cli cluster nodes | grep myself
3f7cbc0a7e0720e37fcb63a81dc6e2bf738c3acf 172.17.0.11:6379 myself,slave 32f250e02451352e561919674b8b705aef4dbdc6 0 0 0 connected
```

Then delete it:
``` bash
kubectl exec redis-cluster-0 -- redis-trib del-node \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379 \
3f7cbc0a7e0720e37fcb63a81dc6e2bf738c3acf
```

### Removing a master
To remove master nodes from the cluster, we first have to move the slots used by them to the rest of the cluster, to avoid data loss.

First, take note of the id of the master node we are removing:
``` bash
$ kubectl exec redis-cluster-6 -- redis-cli cluster nodes | grep myself
27259a4ae75c616bbde2f8e8c6dfab2c173f2a1d 172.17.0.10:6379 myself,master - 0 0 9 connected 0-1364 5461-6826 10923-12287
```

Also note the id of any other master node:
``` bash
$ kubectl exec redis-cluster-6 -- redis-cli cluster nodes | grep master | grep -v myself
32f250e02451352e561919674b8b705aef4dbdc6 172.17.0.4:6379 master - 0 1495120400893 2 connected 6827-10922
2a42aec405aca15ec94a2470eadf1fbdd18e56c9 172.17.0.6:6379 master - 0 1495120398342 8 connected 12288-16383
0990136c9a9d2e48ac7b36cfadcd9dbe657b2a72 172.17.0.2:6379 master - 0 1495120401395 1 connected 1365-5460
```

Then, use the `redis-trib` `reshard` command to move all slots from `redis-cluster-6`:
``` bash
kubectl exec redis-cluster-0 -- redis-trib reshard --yes \
--from 27259a4ae75c616bbde2f8e8c6dfab2c173f2a1d \
--to 32f250e02451352e561919674b8b705aef4dbdc6 \
--slots 16384 \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379
```

After resharding, it is safe to delete the `redis-cluster-6` master node:
``` bash
kubectl exec redis-cluster-0 -- redis-trib del-node \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379 \
27259a4ae75c616bbde2f8e8c6dfab2c173f2a1d
```

Finally, we can rebalance the remaining masters to evenly distribute slots:
``` bash
kubectl exec redis-cluster-0 -- redis-trib rebalance --use-empty-masters \
$(kubectl get pod redis-cluster-0 -o jsonpath='{.status.podIP}'):6379
```

### Scaling down
After the master has been resharded and both nodes are removed from the cluster, it is safe to scale down the statefulset:
``` bash
kubectl scale statefulset redis-cluster --replicas=6
```

## Cleaning up
``` bash
kubectl delete pods,statefulset,svc,configmap,pvc -l app=redis-cluster
```