> 目前ceph安装方式有很多种，譬如docker，ansible，ceph-helm，以及基于ceph封装的rook，不过对于学习的新手还是建议从最传统的ceph-deploy方式手工部署，这样更能熟悉理解集群组件之间的相互关系以及配置文件的作用。我这是测试环境，所以直接拿k8s环境的master用了，大家生产环境还是严格按照官方文档推荐配置来部署，比如硬件要求，lvm、分区要求，以及相关优化。
# 集群环境
|hostname|ip|role|os|
|-------|-------|-----|----|
|by-deploy01|172.17.0.91|admin/deploy|centos_7.5_x64/4.17.3-1.el7.elrepo.x86_64|
|by-master01|172.17.0.85|mon/osd/mgr|centos_7.5_x64/4.17.3-1.el7.elrepo.x86_64|
|by-master02|172.17.0.86|mon/osd/mgr|centos_7.5_x64/4.17.3-1.el7.elrepo.x86_64|
|by-master03|172.17.0.87|mon/osd/mgr|centos_7.5_x64/4.17.3-1.el7.elrepo.x86_64|
# ansilbe相关
```bash
[root@by-deploy01 ~]# cat /etc/ansible/hosts
# 部署节点：运行这份 ansible 脚本的节点
[deploy]
172.17.0.91

[kube-master]
172.17.0.85
172.17.0.86
172.17.0.87

[kube-node]
172.17.0.88
172.17.0.89
172.17.0.90

[kube-cluster:children]
kube-node
kube-master
```
# 配置ssh-key
```bash
# deploy节点操作
ssh-keygen -t rsa -b 2048 -C "your_email@youremail.com" -P ''
ssh-copy-id by-master01
ssh-copy-id by-master02
ssh-copy-id by-master03 (-p33)

# 增加 ~/.ssh/config 文件, 否则 ceph-deploy 创建集群 报端口错误（没改ssh端口可以不用操作此步骤）
vim ~/.ssh/config
Host by-master01
    Hostname by-master01
    Port 33
Host by-master01
    Hostname by-master02
    Port 33
Host by-master01
    Hostname by-master03
    Port 33
```
# 配置hosts
```bash
[root@by-deploy01 ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

# --- Deploy Hosts -----
172.17.0.91 by-deploy01
# --- Deploy Hosts -----

# --- Ceph Hosts -----
172.17.0.85 by-master01
172.17.0.86 by-master02
172.17.0.87 by-master03
# --- Ceph Hosts -----


# --- Node Hosts -----
172.17.0.88 by-node01
172.17.0.89 by-node02
172.17.0.90 by-node03
# --- Node Hosts -----

ansible deploy,kube-cluster -m copy -a 'src=/etc/hosts dest=/etc/hosts'
```

# 配置yum
```bash
ansible deploy,kube-cluster -a 'rpm --import https://download.ceph.com/keys/release.asc'

ansible deploy,kube-cluster -a 'rpm -Uvh --replacepkgs https://download.ceph.com/rpm-luminous/el7/noarch/ceph-release-1-1.el7.noarch.rpm'

ansible deploy,kube-cluster -a "sed -i 's/download\.ceph\.com/mirrors\.aliyun\.com\/ceph/g' /etc/yum.repos.d/ceph.repo"

ansible deploy,kube-cluster -a  'yum makecache'
```
# chrony 时间同步
略，自行百度

# 安装ceph-deploy
```bash
# deploy节点安装
yum -y install ceph-deploy 

# 生成ceph.conf和kering
mkdir -p /opt/ceph-cluster
cd /opt/ceph-cluster
ceph-deploy new by-master01 by-master02 by-master03

# 修改配置文件
[global]
fsid = 686c88bf-cd34-41d1-8990-8c08b2699f61
mon_initial_members = by-master01, by-master02, by-master03
mon_host = 172.17.0.85,172.17.0.86,172.17.0.87
auth_cluster_required = cephx
auth_service_required = cephx
auth_client_required = cephx

osd_pool_default_size = 3           #默认副本数为3
osd_pool_default_min_size = 2       #最小副本数为2，也就是只能坏一个

rbd_default_features = 7

[mgr]
mgr modules = dashboard

[mon]
mon_allow_pool_delete = true

[client]
rbd_cache = true
rbd_cache_max_dirty = 25165824
rbd_cache_max_dirty_age = 5
rbd_cache_size = 268435456

[osd]
journal max write bytes = 1073714824
journal max write entries = 10000
journal queue max bytes = 10485760000
journal queue max ops = 50000
ms_bind_port_max = 7100
osd_client_message_size_cap = 2147483648
osd_crush_update_on_start = true
osd_deep_scrub_stride = 131072
osd_disk_threads = 4
osd_journal_size = 10240
osd_map_cache_bl_size = 128
osd_max_backfills = 4
osd_max_object_name_len = 256
osd_max_object_namespace_len = 64
osd_max_write_size = 512
osd_op_threads = 8
osd_recovery_op_priority = 4
[root@by-deploy01 ceph-cluster]#
```

# 安装ceph
```bash
# deploy节点操作
ceph-deploy install by-master01 by-master02 by-master03
# 我的环境安装时候会替换yum源为ceph官方，从而造成超时
# 如果出现超时，请自行在每个服务器中使用 yum 安装，先卸载pip中安装的urllib3，不然会有失败
# pip freeze|grep urllib3
# pip uninstall urllib3
# yum -y install ceph ceph-radosgw
ansible kube-master -m yum -a 'name=ceph'
ansible kube-master -m yum -a 'name=ceph-radosgw'


# 部署初始化mon 和 准备keys
ceph-deploy mon create-initial  
# 会生成ceph.bootstrap-osd.keyring、ceph.client.admin.keyring等认证文件

# 拷贝管理文件admin key 
# 根据实际情况，拷贝管理文件到设定的管理节点
ceph-deploy admin by-master01 by-master02 by-master03

# 如果配置文件修改了可以统一推送到节点
# ceph-deploy --overwrite-conf config push by-master01 by-master02 by-master03
# 重启mon:
systemctl restart ceph-mon@by-master01.service
systemctl restart ceph-mon@by-master02.service
systemctl restart ceph-mon@by-master03.service
# 重启失败处理：
systemctl reset-failed ceph-mon@by-master01.service
systemctl start ceph-mon@by-master01.service

# 添加mgr
# ceph 12开始，monitor必须添加mgr (luminous以后使用mgr处理统计信息，例如：ceph osd df)
# ceph-mgr 对于当前 ceph 版本十分重要,主要用于管理 pg map 作用 
# 当 ceph-mgr 发生故障, 相当于整个 ceph 集群都会出现严重问题 
# 建议在每个 mon 中都创建独立的 ceph-mgr (至少 3 个 CEPH MON 节点) 
#只需要在每个 mon 节点参考上面方法进行创建即可 (注, 每个 mgr # 需要不同的独立的命名)

ceph-deploy mgr create by-master01 by-master02 by-master03

# 启用dashboard (选一台mgr节点即可)
ceph mgr module enable dashboard
netstat -lnpt|grep 7000
# 设置dashboard的ip和端口 指定一个mon节点即可
ceph config-key put mgr/dashboard/server_addr 172.17.0.85
ceph config-key put mgr/dashboard/server_port 7000
systemctl restart ceph-mgr@by-master01
# 访问dashboard
http://172.17.0.85:7000

# 添加osd
ceph-deploy disk zap by-master01 /dev/vdc
ceph-deploy disk zap by-master02 /dev/vdc
ceph-deploy disk zap by-master03 /dev/vdc
ceph-deploy osd create by-master01 --data /dev/vdc
ceph-deploy osd create by-master02 --data /dev/vdc
ceph-deploy osd create by-master03 --data /dev/vdc

ceph-deploy disk zap by-master01 /dev/vdd
ceph-deploy disk zap by-master02 /dev/vdd
ceph-deploy disk zap by-master03 /dev/vdd
ceph-deploy osd create by-master01 --data /dev/vdd
ceph-deploy osd create by-master02 --data /dev/vdd
ceph-deploy osd create by-master03 --data /dev/vdd

ceph-deploy disk zap by-master01 /dev/vde
ceph-deploy disk zap by-master02 /dev/vde
ceph-deploy disk zap by-master03 /dev/vde
ceph-deploy osd create by-master01 --data /dev/vde
ceph-deploy osd create by-master02 --data /dev/vde
ceph-deploy osd create by-master03 --data /dev/vde
```

# 查看集群状态
```bash
[root@by-master01 ~]# ceph -s
  cluster:
    id:     686c88bf-cd34-41d1-8990-8c08b2699f61
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum by-master01,by-master02,by-master03
    mgr: by-master03(active), standbys: by-master02, by-master01
    osd: 9 osds: 9 up, 9 in

  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 bytes
    usage:   9285 MB used, 890 GB / 899 GB avail
    pgs:

[root@by-master01 ~]# ceph osd df tree
ID CLASS WEIGHT  REWEIGHT SIZE    USE   AVAIL   %USE VAR  PGS TYPE NAME
-1       0.87918        -    899G 9285M    890G 1.01 1.00   - root default
-3       0.29306        -    299G 3095M    296G 1.01 1.00   -     host by-master01
 0   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.0
 3   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.3
 6   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.6
-5       0.29306        -    299G 3095M    296G 1.01 1.00   -     host by-master02
 1   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.1
 4   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.4
 7   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.7
-7       0.29306        -    299G 3095M    296G 1.01 1.00   -     host by-master03
 2   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.2
 5   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.5
 8   hdd 0.09769  1.00000 102396M 1031M 101364M 1.01 1.00   0         osd.8
                    TOTAL    899G 9285M    890G 1.01
MIN/MAX VAR: 1.00/1.00  STDDEV: 0
[root@by-master01 ~]#

```
# 开机启动
```bash
# 设置所有开机启动 （默认开启）
[root@by-master01 ~]# systemctl list-unit-files |grep ceph
ceph-disk@.service                            static
ceph-mds@.service                             disabled
ceph-mgr@.service                             enabled
ceph-mon@.service                             enabled
ceph-osd@.service                             disabled
ceph-radosgw@.service                         disabled
ceph-volume@.service                          enabled
ceph-mds.target                               enabled
ceph-mgr.target                               enabled
ceph-mon.target                               enabled
ceph-osd.target                               enabled
ceph-radosgw.target                           enabled
ceph.target                                   enabled
```
# 清理安装
有时候我们安装错误，可以执行清理操作：
```bash
# 管理节点执行：
ceph-deploy purge node1
ceph-dpeloy purgedata node1
## ssh到node1
ssh node1
vgs  ## 获取vg信息
vgremove VG ##删除上面获取的vg  ##如果不执行该步骤，则该磁盘无法再次被使用
pvs ##查看pv信息
pvremove PV ##删除pv
parted /dev/vdc
rm 1   ##删除之前block.db的分区
quit
```
# osd测试
> 1. 客户端需要安装ceph-common软件包
> 2. 拷贝配置文件（否则不知道集群在哪）
> 3. 拷贝连接密钥（否则无连接权限
```bash
# 先卸载pip中安装的urllib3，不然会有失败
# pip freeze|grep urllib3
# pip uninstall urllib3
[root@by-deploy01 ~]# ansible kube-node -m yum -a 'name=ceph-common'
[root@by-deploy01 ~]# ansible kube-node -m copy -a 'src=/opt/ceph-cluster/ceph.conf dest=/etc/ceph'
[root@by-deploy01 ~]# ansible kube-node -m copy -a 'src=/opt/ceph-cluster/ceph.client.admin.keyring dest=/etc/ceph'


# 创建存储池
ceph osd pool create test 128
# 创建pool 通常在创建pool之前，需要覆盖默认的pg_num，官方推荐：
# 若少于5个OSD， 设置pg_num为128。
# 5~10个OSD，设置pg_num为512。
# 10~50个OSD，设置pg_num为4096。
# 超过50个OSD，可以参考pgcalc计算
ceph osd lspools

# 创建一个10G块
rbd create --size 10G disk01 --pool test
rbd ls test
rbd info test/disk01
 
# 将10G的块映射到本地
rbd map test/disk01
# 由于内核版本 ceph一些特性无法使用，需要手动禁用才能map （配置文件已关闭不支持特性，进行更改了要更新client、osd节点配置文件）
rbd feature disable test/disk01 object-map fast-diff deep-flatten
# 查看映射
rbd showmapped
mkfs.xfs /dev/rbd0
mount /dev/rbd0 /mnt
df -hT

# 警告处理
# health: HEALTH_WARN
#           application not enabled on 1 pool(s)
ceph health detail
ceph osd pool application enable test rbd

```
> rbd 参考链接
> 1. [rbd无法map （rbd feature disable）](http://www.zphj1987.com/2016/06/07/rbd%E6%97%A0%E6%B3%95map-rbd-feature-disable/)
> 2. [rbd feature说明](https://yq.aliyun.com/articles/73440)

# 性能测试
```bash
###########################
#简单ceph性能测试

#创建测试池mytest
ceph osd pool create mytest 128
rados lspools
# ceph osd pool set mytest size 2 #副本为2
# ceph osd pool delete mytest mytest --yes-i-really-really-mean-it #删除


#Rados性能测试(关注 bandwidth带宽,latency延迟)
rados bench -p mytest 10 write --no-cleanup #写测试10秒
rados bench -p mytest 10 seq  #顺序读
rados bench -p mytest 10 rand #随机读
rados -p mytest cleanup #清理测试数据

#rbd块设备测试
rbd create --size 2G mytest/test1 #创建块设备映像test1
rbd ls mytest
rbd info mytest/test1

rbd map mytest/test1   #映射块设备
#/dev/rbd0
#rbd showmapped         #查看已映射块设备
#挂载
mkfs.xfs /dev/rbd0
mkdir -p /mnt/ceph
mount /dev/rbd0 /mnt/ceph/
df -h /mnt/ceph

#测试
rbd bench-write mytest/test1
#默认参数io 4k,线程数16,总写入1024M, seq顺序写


rbd unmap mytest/test1 #取消块设备映射
rbd rm mytest/test1    #删除块设备映像

#参考
https://www.cnblogs.com/sammyliu/p/5557666.html
###########################
```


# cephfs
## 启用mds服务
```bash
ceph-deploy mds create by-master01 by-master02 by-master03
```
## 创建名为cephfs_data的pool
```bash
ceph osd pool create cephfs_data 128
```
## 创建名为cephfs_metadata的pool
```bash
ceph osd pool create cephfs_metadata 128
```
## 启用pool
```bash
ceph fs new cephfs cephfs_metadata 
ceph fs ls
ceph mds stat 
ceph -s
```
## 客户端
```bash
ansible kube-cluster -m yum -a 'name=ceph-common'
ansible kube-cluster -m shell -a 'ceph-authtool -p /etc/ceph/ceph.client.admin.keyring > /root/admin.key'
ansible kube-cluster -m file -a 'path=/root/admin.key mode=600'

# 挂载测试
mount -t ceph by-master01:6789,by-master02:6789,by-master03:6789:/ /mnt -o name=admin,secretfile=admin.key
df -hT
```
