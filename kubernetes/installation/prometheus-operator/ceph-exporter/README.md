> 目前ceph还是采用二进制方式部署，并没有使用k8s方式跑。目前ceph放k8s里跑还是不太成熟稳定，数据安全大于天，稳定可控才是王道。
# 部署exporter
```bash
docker run -itd -v /etc/ceph:/etc/ceph -p=9128:9128 -it digitalocean/ceph_exporter
curl http://localhost:9128/metrics
```
