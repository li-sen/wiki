# 集群
1. 集群状态
```bash
ceph -s
```
2. 集群实时运行状态
```bash
ceph -w
```
3. 集群健康状态细节
```bash
ceph health detail
```
4. 查看ceph存储空间
```bash
ceph df
```
5. 查看集群的详细配置
```bash
ceph daemon mon.by-master01 config show | more
```
6. 查看ceph log日志所在的目录
```bash
ceph-conf --name mon.node1 --show-config-value log_file
```
7. 查看集群配置
```bash
ceph --show-config
```
# mon
1. 查看mon的状态信息
```bash
ceph mon stat
```
2. 查看mon的选举状态
```bash
ceph quorum_status
```
3. 查看mon的映射信息
```bash
ceph mon dump
```
4. 删除一个mon节点
```bash
ceph mon remove node1
```
5. 获得一个正在运行的mon map，并保存在1.txt文件中
```bash
ceph mon remove node1
```
6. 查看上面获得的map
```bash
monmaptool --print 1.txt 
```
7. 查看mon的详细状态
```bash
ceph daemon mon.node1  mon_status 
```
# 修改配置文件
```bash
# 临时修改所有OSD和恢复相关的选项
ceph tell osd.* injectargs '--osd-max-backfills 1'             # 并发回填操作数
ceph tell osd.* injectargs '--osd-recovery-threads 1'          # 恢复线程数量
ceph tell osd.* injectargs '--osd-recovery-op-priority 1'      # 恢复线程优先级  
ceph tell osd.* injectargs '--osd-client-op-priority 63'       # 客户端线程优先级
ceph tell osd.* injectargs '--osd-recovery-max-active 1'       # 最大活跃的恢复请求数 
```
# osd
1. 查看ceph osd运行状态
```bash
ceph osd stat 
```
2. 查看osd映射信息
```bash
ceph osd dump 
```
3. 查看osd的目录树
```bash
ceph osd tree / ceph osd df tree (osd使用情况)
```
4. down掉一个osd硬盘
```bash
ceph osd down 0   # down掉osd.0节点
```
5. 在集群中删除一个osd硬盘
```bash
ceph osd rm 0
```
6. 在集群中删除一个osd 硬盘 crush map
```bash
ceph osd crush rm osd.0
```
7. 在集群中删除一个osd的host节点
```bash
ceph osd crush rm node1
```
8. 查看最大osd的个数
```bash
ceph osd getmaxosd
```
9. 设置最大的osd的个数（当扩大osd节点的时候必须扩大这个值）
```bash
ceph osd setmaxosd 10
```
10. 设置osd crush的权重
```bash
ceph osd crush reweight osd.3 1.0
```
11. 把一个osd节点逐出集群
```bash
ceph osd out osd.3
```
12. 把逐出的osd加入集群
```bash
ceph osd in osd.3
```
13. 暂停osd （暂停后整个集群不再接收数据）
```bash
ceph osd pause
```
14. 再次开启osd 
```bash
ceph osd unpause
```
15. 把逐出的osd加入集群
```bash
ceph osd in osd.3
```
# pg
1. 查看pg状态
```bash
ceph pg stat
```
2. 查看pg组的映射信息
```bash
ceph pg dump
```
3. 查看一个PG的map
```bash
ceph pg map 0.3f
```
4. 查询一个pg的详细信息
```bash
ceph pg 0.26 query
```
5. 查看pg中stuck的状态
```bash
ceph pg dump_stuck unclean
ceph pg dump_stuck inactive
ceph pg dump_stuck stale
```
6. 恢复一个丢失的pg
```bash
ceph pg {pg-id} mark_unfound_lost revert
```
# pool
1. 查看pool数量
```bash
ceph osd lspools
```
2. 创建一个pool
```bash
ceph osd pool create test 100 #这里的100指的是pg组
```
3. 为一个ceph pool配置配额
```bash
ceph osd pool set-quota data max_objects 10000
```
4. 在集群中删除一个pool
```bash
ceph osd pool delete test  test  --yes-i-really-really-mean-it  #集群名字需要重复两次
```
5. 显示集群中pool的详细信息
```bash
rados df
```
6. 给一个pool创建一个快照
```bash
ceph osd pool mksnap test test-snap 
```
7. 删除pool的快照
```bash
ceph osd pool rmsnap test test-snap
```
8. 查看test池的pg数量
```bash
ceph osd pool set test pg_num 100
```
9. 设置test池的最大存储空间为100T（默认是1T)
```bash
ceph osd pool set test target_max_bytes 100000000000000
```
10. 设置test池的副本数是3
```bash
ceph osd pool set test size 3 
```
11. 设置test池能接受写操作的最小副本为2
```bash
ceph osd pool set test min_size 2 
```
12. 查看集群中所有pool的副本参数
```bash
ceph osd dump | grep 'replicated size' 
```
13. 设置一个pool的pgp数量
```bash
ceph osd pool set test pgp_num 100
```



# ceph 常用运维操作

## ceph full 处理方法
```bash
1. 设置 osd 禁止读写

ceph osd pause

2. 通知 mon 和 osd 修改 full 阈值

ceph tell mon.* injectargs "--mon-osd-full-ratio 0.96"

ceph tell osd.* injectargs "--mon-osd-full-ratio 0.96"



3. 通知 pg 修改 full 阈值

ceph pg set_full_ratio 0.96 (Luminous版本之前)

ceph osd set-full-ratio 0.96 (Luminous版本)



4. 解除 osd 禁止读写

ceph osd unpause



5. 删除相关数据

最好是 nova 或者 glance 删除

也可以在 ceph 层面删除



6. 配置还原

ceph tell mon.* injectargs "--mon-osd-full-ratio 0.95"

ceph tell osd.* injectargs "--mon-osd-full-ratio 0.95"

ceph pg set_full_ratio 0.95 (Luminous版本之前)

ceph osd set-full-ratio 0.95 (Luminous版本)
```
