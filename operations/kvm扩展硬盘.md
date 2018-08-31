1. 调整镜像文件大小
virsh shutdown smb-app01-5.46 
qemu-img resize /var/lib/libvirt/images/k8s-192-168-5-121.img +5G
virsh start smb-app01-5.46

2. 分区重建
查看当前分区状况，可以看到磁盘/dev/vda大小变化变为15G
```bash
[root@localhost ~]# fdisk -l

Disk /dev/vda: 16.1 GB, 16106127360 bytes
255 heads, 63 sectors/track, 1958 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           1          25      200781   83  Linux
/dev/vda2              26         156     1052257+  82  Linux swap / Solaris
/dev/vda3             157        1305     9229342+  83  Linux      #注意这里起始cylinder 是157，起始cylinder 绝对不可以改，否则会破坏原分区的数据。
```

3. 扩展根分区
```bash

[root@localhost ~]# fdisk /dev/vda

The number of cylinders for this disk is set to 1958.
There is nothing wrong with that, but this is larger than 1024,
and could in certain setups cause problems with:
1) software that runs at boot time (e.g., old versions of LILO)
2) booting and partitioning software from other OSs
   (e.g., DOS FDISK, OS/2 FDISK)

Command (m for help): p

Disk /dev/vda: 16.1 GB, 16106127360 bytes
255 heads, 63 sectors/track, 1958 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           1          25      200781   83  Linux
/dev/vda2              26         156     1052257+  82  Linux swap / Solaris
/dev/vda3             157        1305     9229342+  83  Linux

Command (m for help): d
Partition number (1-4): 3                    #根/分区是多少就输入几，这里是/dev/vda3

Command (m for help): p

Disk /dev/vda: 16.1 GB, 16106127360 bytes
255 heads, 63 sectors/track, 1958 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *           1          25      200781   83  Linux
/dev/vda2              26         156     1052257+  82  Linux swap / Solaris

Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 3                  #之前是多少，这里还输入多少
First cylinder (157-1958, default 157):  #之前是多少，这里还输入多少  一般都为默认
Using default value 157
Last cylinder or +size or +sizeM or +sizeK (157-1958, default 1958):    #一般都为默认         
Using default value 1958

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

The kernel still uses the old table.
The new table will be used at the next reboot.
Syncing disks.
```
4. 重启服务器
```bash
reboot

resize2fs /dev/vda1
root@smb-app01 ~]# df -h 查看扩展结果
```