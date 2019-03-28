此部署参考此部署是参考 [Yolean/kubernetes-mysql-cluster](https://github.com/Yolean/kubernetes-mysql-cluster)，根据项目描述是已经在生产跑了一段时间了，但是负载不高。

> 1. 当集群所有节点都挂掉时，是需要手动进行集群修复，项目作者也是不建议使用自动恢复，因为造成这种问题有很多不确定性，为了避免丢失数据，建议手动进行恢复，并且做好集群监控报警工作；我还是比较认同此观点的，不丢数据才是王道，毕竟集群整个挂掉的几率很小。
> 2. 使用mgc 性能是比较低的，能达到单机的60%就不错了，没办法CAP原则，高一致性，性能肯定要让步的。
> 3. 使用galeracluster方案，必须小心ddl问题，这里我附下链接：[Galera Cluster](https://blog.csdn.net/linuxprobe2017/article/details/76874949?locationNum=4&fps=1)
> 4. 集群需要ceph环境，包括sc、pvc，具体部署参考我之前的ceph搭建文档。

这里我对原项目进行了小幅改动，主要改动点就是podManagementPolicy，将OrderedReady，改成Parallel。改动的原因是在当整个集群节点都挂掉了，需要进行手动数据恢复，为了不丢数据，肯定是数据最完整的先 引导启动，然后其他节点在加入同步，这就不能通过顺序启动了；集群都挂掉，使用--wsrep-recover比对seqno，也需要并发启动。init.sh也进行了相应的改动，请自行查看。


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

#### 集群所有节点down，如何快速恢复
> 会丢失数据，仅用于自己测试环境
```bash
# 到第一个（mariadb-0）节点，找到挂载的数据目录
kubectl delete pod mariadb-0 -n mysql # 一般情况，mariadb-0会起不来，先删掉重新启动，进入init状态
# 重命名 grastate.dat 或者直接干掉
kubectl -n mysql exec -it mariadb-0 bash -c init-config
cd /data/db/
mv grastate.dat grastate.dat.bak
# 重新启动集群
kubectl get pod --all-namespaces -o wide
kubectl delete pod mariadb-0 -n mysql # 一般情况，mariadb-0会起不来，先删掉重新启动，进入init状态
kubectl logs -f mariadb-0  -n mysql -c init-config
kubectl --namespace=mysql exec -c init-config mariadb-0 -- touch /tmp/confirm-new-cluster
# 至此，整个集群会快速拉起来，然后可以进行数据验证。
> 此操作还是尽量避免，可能会丢失事务，我这是测试环境为了可以快速拉起服务，才这么干；正常流程通过检查各节点的事务状态来提取最后的序列号，需要先从最后节点上自举启动，然后再启动其它节点。

#### 非k8s环境，集群挂掉，不丢失数据恢复流程

1. 找到数据最完整的节点。

查看每台服务器上的grastate.dat文件，以查看哪台计算机具有最新数据。seqno最大的节点是具有当前数据的节点。

例如以下三个节点的grastate.dat文件：
```bash
a）Node0：此grastate.dat显示正常关闭。（seqno非负值）
/var/lib/mysql/grastate.dat
version: 2.1
uuid: cbd332a9-f617-11e2-b77d-3ee9fa637069
seqno: 43760

b）Node1：此grastate.dat文件在seqno中显示-1，此节点在事务处理期间崩溃，非正常关闭。
/var/lib/mysql/grastate.dat
version: 2.1
uuid: cbd332a9-f617-11e2-b77d-3ee9fa637069
seqno: -1


c）Node2：此grastate.dat文件seqno为-1 uuid丢失，此节点在DDL期间崩溃。
/var/lib/mysql/grastate.dat
version: 2.1
uuid: 00000000-0000-0000-0000-000000000000
seqno: -1
```
2.没有seqno，要获取seqno，请使用--wsrep-recover选项。
> Mysqld将读取InnoDB头文件并立即关闭，最后一个wsrep位置打印在log文件中。
```bash
恢复seqno：
mysqld --wsrep-recover。

示例：
[Note] WSREP: Recovered position: 38002d82-a0a2-11e8-8ec5-b2c4ecea3ccd:7
```
3. 启动恢复
> 通过对比，Node0 seqno最大，具有完整数据，应首先启动。

> 如果node0 也是非正常关闭，同样用 --wsrep-recover查看seqno，然后进行同样操作，这里关键就是找出最大seqno，保证数据完整性。
```bash
a) 启动Node0节点
方法1：
mv gvwstate.dat gvwstate.dat.bak
然后启动：
启动命令（不同版本的不同启动方式）：
$ service mysql bootstrap # sysvinit
$ service mysql start --wsrep-new-cluster # sysvinit
$ galera_new_cluster # systemd 
$ mysqld_safe --wsrep-new-cluster # command line

方法2:
修改grastate.dat 配置文件 “safe_to_bootstrap: 1”，
systemctl start mariadb 启动此节点即可。


b）然后正常启动Node1和Node2：systemctl start mariadb

c）方法1：一旦所有三个节点都处于启动状态并处于主状态，恢复集群正常状态，请以正常方式重新启动Node0（因此它将作为整个群集的一部分出现，而不仅仅是一个引导程序）。

```
#### 本项目环境，集群挂掉，不丢数据，操作流程
通过非k8s环境的恢复流程，如果集群整体挂掉，对应此项目我们希望不丢数据的恢复流程应该操作如下：
1. 删除statefulsets，此操作不会删除pvc，所以对应存数据的pv也不会有问题
```bash
kubectl delete -f 40mariadb.yml
```

2. 找出数据最完整的节点
```bash
kubectl apply -f 40mariadb.yml
kubectl get pod -n mysql
kubectl logs mariadb-0  -n mysql -c init-config
```

3. 参照init.sh，使非正常关闭的节点进入recover模式，找到数据最完整的节点
```bash
kubectl --namespace=mysql exec -c init-config mariadb-0 -- touch /tmp/confirm-recover
kubectl --namespace=mysql exec -c init-config mariadb-1 -- touch /tmp/confirm-recover
kubectl --namespace=mysql exec -c init-config mariadb-2 -- touch /tmp/confirm-recover

kubectl get pod -n mysql -o wide

kubectl --namespace=mysql logs mariadb-0 -c mariadb
kubectl --namespace=mysql logs mariadb-1 -c mariadb
kubectl --namespace=mysql logs mariadb-2 -c mariadb

查看日志，得到seqno最大节点

# 也可以 先不进入recover模式，直接进入节点查看日志，得到数据最全节点
kubectl --namespace=mysql exec -c init-config mariadb-0 bash
kubectl --namespace=mysql exec -c init-config mariadb-1 bash
kubectl --namespace=mysql exec -c init-config mariadb-2 bash
分别查看对比
tail -200f db/error.log |grep 'WSREP: Recovered position'

```
3. 进行recovery
```bash
# 通过对比发现mariadb-2 seqno最大，故第一个引导启动
kubectl delete -f 40mariadb.yml
kubectl apply -f 40mariadb.yml
kubectl get pod -n mysql

# 到mariadb-2 设置安全启动
kubectl --namespace=mysql exec -c init-config mariadb-2 bash
sed -i 's/safe_to_bootstrap: 0/safe_to_bootstrap: 1/g' grastate.dat

kubectl --namespace=mysql exec -c init-config mariadb-2 -- touch /tmp/confirm-new-cluster
sleep 10
kubectl --namespace=mysql exec -c init-config mariadb-0 -- touch /tmp/confirm-resume
sleep 10
kubectl --namespace=mysql exec -c init-config mariadb-1 -- touch /tmp/confirm-resume
kubectl get pod -n mysql -o wide

至此集群恢复健康状态，可以进行数据验证
```
