# 虚拟机配置注意点
- 网卡  
  有两张网卡，一张为host-only，一张为nat
  virtualbo全局设置先新建一张host-only网卡，建议取消dhcp
- cpu  
  开启cpu虚拟化支持

# centos7安装注意点
- 统一网卡命名
  系统启动时  
  光标选中install centos7 --> 按tab ——> 进入内核选项 ——> net.ifnames=0 biosdevname=0
> 这个目的是为了让虚拟机的网卡统一为eth0 eht1格式，不加的话网卡名会根据规则命名，无法统一。
- 建议全屏操作
- 时区：上海
- 语言支持：英文、简体中文
- 键盘：US
- 软件选择：minimal
- 分区：不建议使用lvm以及swap  
  手动分：  
  /boot 512m  
  swap 1034m 后续会禁用删掉  
  / 剩余所有空间

# 安装后配置
- ip配置：
  - eth0(host-only)
  BOOTPROTO=static
  ONBOOT=yes
  IPADDR=192.168.56.11
  NETMASK=255.255.255.0
  - eth1(nat)
  ONBOOT=yes
- dns：  
```
vi /etc/resolv.conf
nameserver 223.5.5.5
nameserver 223.6.6.6
```
```
systemctl restart network
ping www.baidu.com
```
- hostname
```
vi /etc/hostname
node01.example.com

vi /etc/sysconfig/network
node01.example.com
```
- hosts
```
vi /etc/hosts
192.168.56.11 node01 node01.example.com
```
- 关闭selinux、防火墙
```
getenforce
sed -i 's/enforcing/disabled/g' /etc/sysconfig/selinux
setenforce 0

systemctl disable firewalld
systemctl stop firewalld
```
- 禁用swap
```
swapoff -a && sysctl -w vm.swappiness=0
sed -i '/swap/d' /etc/fstab
```

- 安装常用包
```
yum install wget vim screen net-tools lrzsz glances -y
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum groupinstall "Development Tools" -y
yum update -y
```
- 升级内核
```
# 载入公钥
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
# 安装ELRepo
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
# 载入elrepo-kernel元数据
yum --disablerepo=\* --enablerepo=elrepo-kernel repolist
# 查看可用的rpm包
yum --disablerepo=\* --enablerepo=elrepo-kernel list kernel*
# 安装最新版本的kernel
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-ml.x86_64

#查看默认启动顺序
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (4.17.1-1.el7.elrepo.x86_64) 7 (Core)
CentOS Linux (3.10.0-862.3.3.el7.x86_64) 7 (Core)
CentOS Linux (3.10.0-862.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-14cd5fb0ee114fa88d6c747daee61c31) 7 (Core)

#默认启动的顺序是从0开始，新内核是从头插入（目前位置在0，而4.4.4的是在1），所以需要选择0。

grub2-set-default 0

reboot
uname -a

# 删除老内核以及内核工具
rpm -qa|grep kernel|grep 3.10
rpm -qa|grep kernel|grep 3.10|xargs yum remove -y

# 安装新版本工具包
yum --disablerepo=\* --enablerepo=elrepo-kernel install -y kernel-ml-tools.x86_64

rpm -qa|grep kernel

至此内核升级完毕！
```
# 虚机复制克隆
- 删掉所有网卡uuid
```
sed -i '/UUID/d' /etc/sysconfig/network-scripts/ifcfg-eth0
```
- 删除udev rules记录
```
rm -f /etc/udev/rules.d/70-persistent-*
```
- 关机克隆
```
shutdown -h now
```
然后就可以复制克隆了

# 克隆模板注意点
- 修改网卡ip
- 修改/etc/hostname
- 修改/etc/sysconfig/network
- 修改/etc/hosts
```
shutdown -h now
```
**然后所有虚拟机做快照！！**