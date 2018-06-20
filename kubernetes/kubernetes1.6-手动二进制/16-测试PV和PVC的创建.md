# 新建存储
+ 目前阿里云支持kubernetes的存储有2种：第一种是使用NAS数据卷，其实就是NFS，需要安装nfs-utils组件。第二种是阿里云盘，需要安装阿里云盘 driver rpm 包。
阿里云的帮助文档为：https://help.aliyun.com/document_detail/53754.html?spm=5176.doc59379.6.852.G6lt3O


+ 按照阿里云的帮助文档，此次采用NAS方式，后期看日志的时候将所有需要的日志挂载到任意一台机器上。
首先购买NAS数据卷，一次购买6个5G的NAS数据卷
kubernetes内所有需要访问pv的机器和挂载NAS的机器都需要安装nfs-utils。
```bash
[root@node03 ~]# yum install nfs-utils
Loaded plugins: fastestmirror
base                                                                                                                                                                    | 3.6 kB  00:00:00     
epel                                                                                                                                                                    | 4.7 kB  00:00:00     
extras                                                                                                                                                                  | 3.4 kB  00:00:00     
updates                                                                                                                                                                 | 3.4 kB  00:00:00     
(1/3): extras/7/x86_64/primary_db                                                                                                                                       | 145 kB  00:00:00     
(2/3): epel/x86_64/updateinfo                                                                                                                                           | 860 kB  00:00:00     
(3/3): epel/x86_64/primary_db                                                                                                                                           | 6.1 MB  00:00:00     
Loading mirror speeds from cached hostfile
Resolving Dependencies
--> Running transaction check
---> Package nfs-utils.x86_64 1:1.3.0-0.48.el7_4 will be installed
--> Processing Dependency: libtirpc >= 0.2.4-0.7 for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: gssproxy >= 0.7.0-3 for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: rpcbind for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: quota for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: libnfsidmap for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: libevent for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: keyutils for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: libtirpc.so.1()(64bit) for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: libnfsidmap.so.0()(64bit) for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Processing Dependency: libevent-2.0.so.5()(64bit) for package: 1:nfs-utils-1.3.0-0.48.el7_4.x86_64
--> Running transaction check
---> Package gssproxy.x86_64 0:0.7.0-4.el7 will be installed
--> Processing Dependency: libverto-module-base for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libref_array.so.1(REF_ARRAY_0.1.1)(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libini_config.so.3(INI_CONFIG_1.2.0)(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libini_config.so.3(INI_CONFIG_1.1.0)(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libref_array.so.1()(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libini_config.so.3()(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libcollection.so.2()(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
--> Processing Dependency: libbasicobjects.so.0()(64bit) for package: gssproxy-0.7.0-4.el7.x86_64
---> Package keyutils.x86_64 0:1.5.8-3.el7 will be installed
---> Package libevent.x86_64 0:2.0.21-4.el7 will be installed
---> Package libnfsidmap.x86_64 0:0.25-17.el7 will be installed
---> Package libtirpc.x86_64 0:0.2.4-0.10.el7 will be installed
---> Package quota.x86_64 1:4.01-14.el7 will be installed
--> Processing Dependency: quota-nls = 1:4.01-14.el7 for package: 1:quota-4.01-14.el7.x86_64
--> Processing Dependency: tcp_wrappers for package: 1:quota-4.01-14.el7.x86_64
---> Package rpcbind.x86_64 0:0.2.0-42.el7 will be installed
--> Running transaction check
---> Package libbasicobjects.x86_64 0:0.1.1-27.el7 will be installed
---> Package libcollection.x86_64 0:0.6.2-27.el7 will be installed
---> Package libini_config.x86_64 0:1.3.0-27.el7 will be installed
--> Processing Dependency: libpath_utils.so.1(PATH_UTILS_0.2.1)(64bit) for package: libini_config-1.3.0-27.el7.x86_64
--> Processing Dependency: libpath_utils.so.1()(64bit) for package: libini_config-1.3.0-27.el7.x86_64
---> Package libref_array.x86_64 0:0.1.5-27.el7 will be installed
---> Package libverto-libevent.x86_64 0:0.2.5-4.el7 will be installed
---> Package quota-nls.noarch 1:4.01-14.el7 will be installed
---> Package tcp_wrappers.x86_64 0:7.6-77.el7 will be installed
--> Running transaction check
---> Package libpath_utils.x86_64 0:0.2.1-27.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

===============================================================================================================================================================================================
 Package                                            Arch                                    Version                                             Repository                                Size
===============================================================================================================================================================================================
Installing:
 nfs-utils                                          x86_64                                  1:1.3.0-0.48.el7_4                                  updates                                  398 k
Installing for dependencies:
 gssproxy                                           x86_64                                  0.7.0-4.el7                                         base                                     105 k
 keyutils                                           x86_64                                  1.5.8-3.el7                                         base                                      54 k
 libbasicobjects                                    x86_64                                  0.1.1-27.el7                                        base                                      25 k
 libcollection                                      x86_64                                  0.6.2-27.el7                                        base                                      41 k
 libevent                                           x86_64                                  2.0.21-4.el7                                        base                                     214 k
 libini_config                                      x86_64                                  1.3.0-27.el7                                        base                                      63 k
 libnfsidmap                                        x86_64                                  0.25-17.el7                                         base                                      49 k
 libpath_utils                                      x86_64                                  0.2.1-27.el7                                        base                                      27 k
 libref_array                                       x86_64                                  0.1.5-27.el7                                        base                                      26 k
 libtirpc                                           x86_64                                  0.2.4-0.10.el7                                      base                                      88 k
 libverto-libevent                                  x86_64                                  0.2.5-4.el7                                         base                                     8.9 k
 quota                                              x86_64                                  1:4.01-14.el7                                       base                                     179 k
 quota-nls                                          noarch                                  1:4.01-14.el7                                       base                                      90 k
 rpcbind                                            x86_64                                  0.2.0-42.el7                                        base                                      59 k
 tcp_wrappers                                       x86_64                                  7.6-77.el7                                          base                                      78 k

Transaction Summary
===============================================================================================================================================================================================
Install  1 Package (+15 Dependent packages)

Total download size: 1.5 M
Installed size: 4.2 M
Is this ok [y/d/N]: y
Downloading packages:
(1/16): keyutils-1.5.8-3.el7.x86_64.rpm                                                                                                                                 |  54 kB  00:00:00     
(2/16): libbasicobjects-0.1.1-27.el7.x86_64.rpm                                                                                                                         |  25 kB  00:00:00     
(3/16): libcollection-0.6.2-27.el7.x86_64.rpm                                                                                                                           |  41 kB  00:00:00     
(4/16): gssproxy-0.7.0-4.el7.x86_64.rpm                                                                                                                                 | 105 kB  00:00:00     
(5/16): libevent-2.0.21-4.el7.x86_64.rpm                                                                                                                                | 214 kB  00:00:00     
(6/16): libnfsidmap-0.25-17.el7.x86_64.rpm                                                                                                                              |  49 kB  00:00:00     
(7/16): libpath_utils-0.2.1-27.el7.x86_64.rpm                                                                                                                           |  27 kB  00:00:00     
(8/16): libref_array-0.1.5-27.el7.x86_64.rpm                                                                                                                            |  26 kB  00:00:00     
(9/16): libtirpc-0.2.4-0.10.el7.x86_64.rpm                                                                                                                              |  88 kB  00:00:00     
(10/16): libverto-libevent-0.2.5-4.el7.x86_64.rpm                                                                                                                       | 8.9 kB  00:00:00     
(11/16): libini_config-1.3.0-27.el7.x86_64.rpm                                                                                                                          |  63 kB  00:00:00     
(12/16): quota-4.01-14.el7.x86_64.rpm                                                                                                                                   | 179 kB  00:00:00     
(13/16): quota-nls-4.01-14.el7.noarch.rpm                                                                                                                               |  90 kB  00:00:00     
(14/16): rpcbind-0.2.0-42.el7.x86_64.rpm                                                                                                                                |  59 kB  00:00:00     
(15/16): tcp_wrappers-7.6-77.el7.x86_64.rpm                                                                                                                             |  78 kB  00:00:00     
(16/16): nfs-utils-1.3.0-0.48.el7_4.x86_64.rpm                                                                                                                          | 398 kB  00:00:00     
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Total                                                                                                                                                          5.5 MB/s | 1.5 MB  00:00:00     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : libref_array-0.1.5-27.el7.x86_64                                                                                                                                           1/16 


Installing : libcollection-0.6.2-27.el7.x86_64                                                                                                                                          2/16 
  Installing : libevent-2.0.21-4.el7.x86_64                                                                                                                                               3/16 
  Installing : libbasicobjects-0.1.1-27.el7.x86_64                                                                                                                                        4/16 
  Installing : libtirpc-0.2.4-0.10.el7.x86_64                                                                                                                                             5/16 
  Installing : rpcbind-0.2.0-42.el7.x86_64                                                                                                                                                6/16 
  Installing : libverto-libevent-0.2.5-4.el7.x86_64                                                                                                                                       7/16 
  Installing : 1:quota-nls-4.01-14.el7.noarch                                                                                                                                             8/16 
  Installing : libpath_utils-0.2.1-27.el7.x86_64                                                                                                                                          9/16 
  Installing : libini_config-1.3.0-27.el7.x86_64                                                                                                                                         10/16 
  Installing : gssproxy-0.7.0-4.el7.x86_64                                                                                                                                               11/16 
  Installing : tcp_wrappers-7.6-77.el7.x86_64                                                                                                                                            12/16 
  Installing : 1:quota-4.01-14.el7.x86_64                                                                                                                                                13/16 
  Installing : keyutils-1.5.8-3.el7.x86_64                                                                                                                                               14/16 
  Installing : libnfsidmap-0.25-17.el7.x86_64                                                                                                                                            15/16 
  Installing : 1:nfs-utils-1.3.0-0.48.el7_4.x86_64                                                                                                                                       16/16 
  Verifying  : rpcbind-0.2.0-42.el7.x86_64                                                                                                                                                1/16 
  Verifying  : 1:quota-4.01-14.el7.x86_64                                                                                                                                                 2/16 
  Verifying  : libtirpc-0.2.4-0.10.el7.x86_64                                                                                                                                             3/16 
  Verifying  : libnfsidmap-0.25-17.el7.x86_64                                                                                                                                             4/16 
  Verifying  : libini_config-1.3.0-27.el7.x86_64                                                                                                                                          5/16 
  Verifying  : libbasicobjects-0.1.1-27.el7.x86_64                                                                                                                                        6/16 
  Verifying  : libevent-2.0.21-4.el7.x86_64                                                                                                                                               7/16 
  Verifying  : keyutils-1.5.8-3.el7.x86_64                                                                                                                                                8/16 
  Verifying  : libverto-libevent-0.2.5-4.el7.x86_64                                                                                                                                       9/16 
  Verifying  : tcp_wrappers-7.6-77.el7.x86_64                                                                                                                                            10/16 
  Verifying  : libpath_utils-0.2.1-27.el7.x86_64                                                                                                                                         11/16 
  Verifying  : 1:quota-nls-4.01-14.el7.noarch                                                                                                                                            12/16 
  Verifying  : 1:nfs-utils-1.3.0-0.48.el7_4.x86_64                                                                                                                                       13/16 
  Verifying  : gssproxy-0.7.0-4.el7.x86_64                                                                                                                                               14/16 
  Verifying  : libcollection-0.6.2-27.el7.x86_64                                                                                                                                         15/16 
  Verifying  : libref_array-0.1.5-27.el7.x86_64                                                                                                                                          16/16 

Installed:
  nfs-utils.x86_64 1:1.3.0-0.48.el7_4                                                                                                                                                          

Dependency Installed:
  gssproxy.x86_64 0:0.7.0-4.el7            keyutils.x86_64 0:1.5.8-3.el7      libbasicobjects.x86_64 0:0.1.1-27.el7   libcollection.x86_64 0:0.6.2-27.el7   libevent.x86_64 0:2.0.21-4.el7    
  libini_config.x86_64 0:1.3.0-27.el7      libnfsidmap.x86_64 0:0.25-17.el7   libpath_utils.x86_64 0:0.2.1-27.el7     libref_array.x86_64 0:0.1.5-27.el7    libtirpc.x86_64 0:0.2.4-0.10.el7  
  libverto-libevent.x86_64 0:0.2.5-4.el7   quota.x86_64 1:4.01-14.el7         quota-nls.noarch 1:4.01-14.el7          rpcbind.x86_64 0:0.2.0-42.el7         tcp_wrappers.x86_64 0:7.6-77.el7  

Complete!
```

然后创建pv
```bash
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nas
spec:
  capacity:
    storage: 8Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  nfs:
    path: /
    server: 055f84ad83-ixxxx.cn-hangzhou.nas.aliyuncs.com
```
根据上面的文件来创建pv
