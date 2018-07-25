> k8s 版本为v1.11.0
## rbd
1. 新建secret
```bash
kubectl create secret generic ceph-storageclass-secret \
--type="kubernetes.io/rbd" \
--from-literal=key='AQAnpk1bPFWCBRAAfBbc1xBzNNWJVcfyCrhWDA==' \
--namespace=kube-system

kubectl get secret --all-namespaces|grep ceph
```
> --from-literal=key 为你ceph客户端key
2. 新建sc
```bash
[root@by-node01 ~]# cat ceph-storageclass.yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ceph-storageclass
provisioner: kubernetes.io/rbd
parameters:
  monitors: 172.17.0.85:6789,172.17.0.86:6789,172.17.0.87:6789
  adminId: admin
  adminSecretName: ceph-storageclass-secret
  adminSecretNamespace: kube-system
  pool: k8s
  userId: admin
  userSecretName: ceph-storageclass-secret
  fsType: xfs
  imageformat: "2"
  imagefeatures: "layering" #  Currently supported features are layering only. Default is ""

kubectl create -f ceph-storageclass.yml

kubectl get sc --all-namespaces|grep ceph
```
3. 引用sc
```bash
# 测试pvc
[root@by-node01 ~]# cat ceph-pvc-test.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-storageclass
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ceph-storageclass

kubectl create -f ceph-pvc-test.yaml

kubectl get pvc
kubectl get pv

# 查看ceph osd
rbd ls k8s
rbd info k8s/kubernetes-dynamic-pvc-xxxxxx

# 有状态应用引用示例
前面省略...
      volumeMounts:
        - name: datadir
          mountPath: /opt/zookeeper/data
        - name: datalogdir
          mountPath: /opt/zookeeper/data-log
  volumeClaimTemplates:
  - metadata:
      name: datadir
      annotations:
        volume.beta.kubernetes.io/storage-class: ceph-storageclass
    spec:
      accessModes: [ "ReadWriteOnce" ]
      # persistentVolumeReclaimPolicy: Recycle
      resources:
        requests:
          storage: 10Gi
  - metadata:
      name: datalogdir
      annotations:
        volume.beta.kubernetes.io/storage-class: ceph-storageclass
    spec:
      accessModes: [ "ReadWriteOnce" ]
      # persistentVolumeReclaimPolicy: Recycle
      resources:
        requests:
          storage: 10Gi
```
> 备注
> 1. 稳定性在于ceph
> 2. 只能同一node挂载，不能跨node
> 3. 读写只能一个pod，其他pod只能读

rbd官方描述 [rbd官方描述](https://kubernetes.io/docs/user-guide/volumes/#rbd)

k8s rbd pv/pvc官方文档 [参考](https://kubernetes.io/docs/concepts/storage/storage-classes/)

# pv 访问模式
```bash
访问模式包括：
 　 ReadWriteOnce —— 该volume只能被单个节点以读写的方式映射
 　 ReadOnlyMany —— 该volume可以被多个节点以只读方式映射
    ReadWriteMany —— 该volume只能被多个节点以读写的方式映射    
 状态
 　 Available：空闲的资源，未绑定给PVC 
 　 Bound：绑定给了某个PVC 
 　 Released：PVC已经删除了，但是PV还没有被集群回收 
 　 Failed：PV在自动回收中失败了 
 当前的回收策略有：
 　 Retain：手动回收
 　 Recycle：需要擦出后才能再使用 (rm -rf /thevolume/*)，删除pv上的文件，重新使用
 　 Delete：自动删除相关存储，譬如动态生成ceph rbd块
 配置文件加上 persistentVolumeReclaimPolicy: Recycle
 ```
> pv 官方文档说明：[pv 官方文档](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

# cephfs
