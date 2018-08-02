此部署参考此部署是参考 [Yolean/kubernetes-mysql-cluster](https://github.com/Yolean/kubernetes-mysql-cluster)，具项目描述是已经在生产跑了一段时间了，但是负载不高。
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
