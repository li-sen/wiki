# 环境说明：
- 软件版本

组件|版本
:---:|:---:
centos|7.5.1804 x86_64
kernel|4.17.3
ceph|12.2.6 
fio|3.1

- 测试对象

名称|说明
:---:|:---:
rbd |100g 云盘 （2c4g 9osd 副本数3）
云盘(yp) |  100g
ssd 云盘(ssd)|100g

- 测试命令
```bash
fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=randwrite -size=1G -filename=/dev/xxx -iodepth=32 -runtime=120
fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=write -size=1G -filename=/dev/xxx -iodepth=32 -runtime=120
fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=randread -size=1G -filename=/dev/xxx -iodepth=32 -runtime=120
fio -ioengine=libaio -bs=4k -direct=1 -thread -rw=read -size=1G -filename=/dev/xxx -iodepth=32 -runtime=120
```
- 测试结果

```bash
随机写：
ssd  IOPS=4845, BW=18.9MiB/s (19.8MB/s)(1024MiB/54104msec)
rbd  IOPS=2899, BW=11.3MiB/s (11.9MB/s)(1024MiB/90419msec)
yp   IOPS=2615, BW=10.2MiB/s (10.7MB/s)(1024MiB/100227msec)
顺序写：
ssd  IOPS=5942, BW=23.2MiB/s (24.3MB/s)(1024MiB/44115msec)
rbd  IOPS=3042, BW=11.9MiB/s (12.5MB/s)(1024MiB/86168msec)
yp   IOPS=4835, BW=18.9MiB/s (19.8MB/s)(1024MiB/54212msec)

随机读：
ssd  IOPS=4844, BW=18.9MiB/s (19.8MB/s)(1024MiB/54113msec)
rbd  IOPS=14.2k, BW=55.4MiB/s (58.1MB/s)(1024MiB/18484msec)
yp   IOPS=2621, BW=10.2MiB/s (10.7MB/s)(1024MiB/100005msec)

顺序读：
ssd  IOPS=5550, BW=21.7MiB/s (22.7MB/s)(1024MiB/47226msec)
rbd  IOPS=16.0k, BW=62.6MiB/s (65.6MB/s)(1024MiB/16361msec)
yp   IOPS=2621, BW=10.2MiB/s (10.7MB/s)(1024MiB/100000msec)
```


> 我这是在阿里云环境，在测试的过程中，ceph服务器cpu负载立马到极限，条件不够严谨，测试结果你们做个基本参考吧