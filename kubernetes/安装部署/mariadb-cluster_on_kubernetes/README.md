此部署参考此部署是参考 [Yolean/kubernetes-mysql-cluster](https://github.com/Yolean/kubernetes-mysql-cluster)，根据项目描述是已经在生产跑了一段时间了，但是负载不高。
> 使用mgc 性能是比较低的，能达到单机的60%就不错了，没办法CAP原则，高一致性，性能肯定要让步的。
> 集群需要ceph环境，参考我之前的ceph搭建文档

# Get started

```bash
kubectl apply -f .
```

### Cluster Health

Readiness and liveness probes will only assert client-level health of individual pods.
Watch logs for "sst" or "Quorum results", or run this quick check:
```
for i in 0 1 2; do kubectl -n mysql exec mariadb-$i -- mysql -e "SHOW STATUS LIKE 'wsrep_cluster_size';" -N; done
kubectl -n mysql exec mariadb-0 -- mysql -e "grant all privileges on *.* to root@'%' identified by 'xxxxx' with grant option;"
```

Port 9104 exposes plaintext metris in [Prometheus](https://prometheus.io/docs/concepts/data_model/) scrape format.
```
# with kubectl -n mysql port-forward mariadb-0 9104:9104
$ curl -s http://localhost:9104/metrics | grep ^mysql_global_status_wsrep_cluster_size
mysql_global_status_wsrep_cluster_size 3
```

A reasonable alert is on `mysql_global_status_wsrep_cluster_size` staying below the desired number of replicas.

### Cluster un-health

We need to assume a couple of things here. First and foremost:
Production clusters are configured so that the statefulset pods do not go down together.

 * Pods are properly spread across nodes.
 * Nodes are spread across multiple availability zones.

Let's also assume that there is monitoring.
Any `wsrep_cluster_size` issue (see above), or absence of `wsrep_cluster_size`
should lead to a human being paged.

Rarity combined with manual attention means that this statefulset can/should avoid
attempts at automatic [recovery](http://galeracluster.com/documentation-webpages/pcrecovery.html).
The reason for that being: we can't test for failure modes properly,
as they depend on the Kubernetes setup.
Automation may appoint the wrong leader - losing writes -
or cause split-brain situations.

We can however support detection in the init script.

It's normal operations to scale down to two instances
- actually one instance, but nodes should be considered ephemeral so don't do that -
and up to any number of replicas.

# 故障记录
1. 整个集群挂掉，如何快速恢复
```bash
# 到第一个（mariadb-0）节点，找到挂载的数据目录
cd /opt/kubelet/plugins/kubernetes.io/rbd/mounts/k8s-image-kubernetes-dynamic-pvc-95b2a503-9561-11e8-bab0-00163e13cf6c/db
# 重命名 grastate.dat 或者直接干掉
mv grastate.dat grastate.dat.bak
# 重新启动集群
kubectl get pod --all-namespaces -o wide
kubectl delete pod mariadb-0 -n mysql # 一般情况，mariadb-0会起不来，先删掉重新启动，进入init状态
kubectl logs -f mariadb-0  -n mysql -c init-config
kubectl --namespace=mysql exec -c init-config mariadb-0 -- touch /tmp/confirm-new-cluster
# 至此，整个集群会快速拉起来，然后可以进行数据验证。
```
> 此操作还是尽量避免，可能会丢失事务，我这是测试环境为了可以快速拉起服务，才这么干；正常流程通过检查各节点的事务状态来提取最后的序列号，需要先从最后节点上自举启动，然后再启动其它节点。

> pc.recovery(默认是启用)：在我们的数据中心停电的情况下，上电后回来，节点将读取启动最后的状态，将尝试恢复主组件一旦所有成员再次开始看到对方。这使得集群自动被关闭而无需任何人工干预恢复。

> 后续再把init.sh进行优化。

2. 集群挂掉，正常回复流程
> 未验证此流程，先记录下。
```bash
# 恢复前设置：
确保在.profile中导出MYSQL_HOME路径。如果MySQL安装位于不同的位置，则将更改更改为MYSQL_HOME。（例如：MYSQL_HOME = / path / to / mysql）
崩溃恢复步骤：

1. 找到有效的seqno。查看每台服务器上的grastate.dat文件，以查看哪台计算机具有最新数据。seqno最大的节点是具有当前数据的节点。

接下来，查看三个grastate.dat文件。

a）Node0：此grastate.dat显示正常关闭。注意seqno。我们正在寻找具有最大seqno的节点。

/var/lib/mysql/grastate.dat
version: 2.1
uuid: cbd332a9-f617-11e2-b77d-3ee9fa637069
seqno: 43760
b）Node1：此grastate.dat文件在seqno中显示-1。此节点在事务处理期间崩溃。使用wsrep-recover选项启动此节点。MySQL将最后提交的GTID存储在InnoDB数据头中。

/var/lib/mysql/grastate.dat
version: 2.1
uuid: cbd332a9-f617-11e2-b77d-3ee9fa637069
seqno: -1
c）Node2：此grastate.dat文件没有seqno或组ID。此节点在DDL期间崩溃。

/var/lib/mysql/grastate.dat
version: 2.1
uuid: 00000000-0000-0000-0000-000000000000
seqno: -1

2. 接下来，使用uuid恢复节点，但没有seqno。要获取seqno，请使用--wsrep-recover选项。要恢复seqno：
/ path / to / mysql / bin / mysqld --wsrep-recover。Mysqld将读取InnoDB头文件并立即关闭。最后一个wsrep位置打印在mysqld.log文件中。

示例：140716 12:55:45 [注意] WSREP：找到保存状态：cbd332a9- f617-11e2-b77d-3ee9fa637069：36742

查看Node0（seqno：43760）和Node1（seqno：-1）的seqno。Node0具有当前数据快照，应首先启动。

在Node0上，发出此命令以启动节点：

a）nohup / path / to / mysql / bin / mysqld_safe - wsrep_cluster_address = gcomm：//＆; 等待此节点联机。

b）然后启动Node1和Node2。这两个节点应该一次启动一个，并且可以像往常一样启动。

c）一旦所有三个节点都处于启动状态并处于主状态，请以正常方式重新启动Node0（因此它将作为整个群集的一部分出现，而不仅仅是一个引导程序）。

如果Node1或Node2具有最高的seqno，那么该节点将作为引导程序启动，并且您将允许其余节点一次启动一个（连接到具有最高seqno的节点）。
```