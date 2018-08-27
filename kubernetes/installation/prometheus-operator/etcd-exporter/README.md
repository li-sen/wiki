# 先创建etcd secret
```bash
[root@by-master01 system]# kubectl -n monitoring create secret generic etcd-certs --from-file=/etc/kubernetes/ssl/ca.pem  --from-file=/etc/etcd/ssl/etcd.pem --from-file=/etc/etcd/ssl/etcd-key.pem
```

# 使Prometheus Operator接入secret
```bash
vim prometheus-k8s-statefulset.yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: k8s
  namespace: monitoring
  labels:
    prometheus: k8s
spec:
  replicas: 2
  secrets:
  - etcd-certs
  version: v2.0.0

只需加入如下项即可:
  secrets:
  - etcd-certs


kubectl apply -f prometheus-k8s-statefulset.yaml
```

