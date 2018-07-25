> confluence依赖jira，先安装jira，jdk环境略
# 安装jira
1. 数据库配置
```bash
# 数据库参数：
https://confluence.atlassian.com/adminjiraserver/connecting-jira-applications-to-mysql-938846854.html
// remove this if it exists

sql_mode = NO_AUTO_VALUE_ON_ZERO

[mysqld]
default-storage-engine=INNODB
max_allowed_packet=512M
innodb_log_file_size=2GB

# 输入密码后, 登录mysql>命令行, 并创建数据库
create database jira_db default character set utf8 collate utf8_bin;
grant all privileges on jira_db.* to 'jira'@'%' identified by 'XXXXX' with grant option;
grant all privileges on jira_db.* to 'jira'@'localhost' identified by 'XXXXX' with grant option;
flush privileges;
```


2. 开始安装
```bash
# 去官网下载软件
https://www.atlassian.com/software/

# 上传服务器 添加权限 执行
chmod +x *.bin
./atlassian-jira-software-7.8.1-x64.bin 

#按实际情况选择安装选项
Unpacking JRE ...
Starting Installer ...
Apr 01, 2018 11:20:10 AM java.util.prefs.FileSystemPreferences$2 run
INFO: Created system preferences directory in java.home.

This will install JIRA Software 7.8.1 on your computer.
OK [o, Enter], Cancel [c]
o
Choose the appropriate installation or upgrade option.
Please choose one of the following:
Express Install (use default settings) [1], Custom Install (recommended for advanced users) [2, Enter], Upgrade an existing JIRA installation [3]
2

Where should JIRA Software be installed?
[/opt/atlassian/jira]
默认
Default location for JIRA Software data
/opt/atlassian/jira_data
Configure which ports JIRA Software will use.
JIRA requires two TCP ports that are not being used by any other
applications on this machine. The HTTP port is where you will access JIRA
through your browser. The Control port is used to startup and shutdown JIRA.
Use default ports (HTTP: 8080, Control: 8005) - Recommended [1, Enter], Set custom value for HTTP and Control ports [2]
1
JIRA can be run in the background.
You may choose to run JIRA as a service, which means it will start
automatically whenever the computer restarts.
Install JIRA as Service?
Yes [y, Enter], No [n]
y
Details on where JIRA Software will be installed and the settings that will be used.
Installation Directory: /opt/atlassian/jira 
Home Directory: /opt/atlassian/jira_data 
HTTP Port: 8080 
RMI Port: 8005 
Install as service: Yes 
Install [i, Enter], Exit [e]
i

Extracting files ...
                                                                           

Please wait a few moments while JIRA Software is configured.
Installation of JIRA Software 7.8.1 is complete
Start JIRA Software 7.8.1 now?
Yes [y, Enter], No [n]
n 选no 要破解不要启动
Installation of JIRA Software 7.8.1 is complete
Your installation of JIRA Software 7.8.1 is now ready.
Finishing installation ...
```

3. 破解
```bash
# 上传破解包
# 把破解包里面的atlassian-extras-3.2.jar和mysql-connector-java-5.1.42-bin.jar两个文件复制到/opt/atlassian/jira/atlassian-jira/WEB-INF/lib/目录下即可. 其中atlassian-extras-3.2.jar是破解Jira的文件, 另一个mysql-connector-java-5.1.42-bin.jar是连接mysql的驱动包. 
cd /opt/atlassian/jira/bin &&./start-jira.sh
然后安装提示一步步来安装

# 注意点：
安装语言选中文
mode:pvivate

最好有谷歌账号和翻墙，去官网申请30天试用：product选 JIRA Software, Your instance is 选 up and running

激活安装完成
```
# 安装confluence
1. 下安装软件 同jira
2. 数据库配置
```bash
输入密码后, 登录mysql>命令行, 并创建数据库.
create database confluence_db default character set utf8 collate utf8_bin;
grant all privileges on confluence_db.* to 'confluence'@'%' identified by 'XXXXX' with grant option;
grant all privileges on confluence_db.* to 'confluence'@'localhost' identified by 'XXXXX' with grant option;
flush privileges;
数据库参数同jira
```
3. 开始安装
```bash
./atlassian-confluence-6.8.0-x64.bin
按实际情况选择安装选项，跟jira一样不过先启动得到server-id，然后关闭进行破解
```
4. 破解
```bash
记住服务ID，然后暂时关闭程序。

关闭Confluence
cd /opt/atlassian-confluence/bin
./stop-confluence.sh

# 破解文件自行百度，一般csdn都有
然后cd /opt/atlassian-confluence/confluence/WEB-INF/lib目录下
atlassian-extras-decoder-v2-3.3.0.jar 放到windows平台上，改名为atlassian-extras-2.4.jar
备份原有文件 mv  atlassian-extras-decoder-v2-3.3.0.jar atlassian-extras-decoder-v2-3.3.0.jar.bak

下载破解软件confluence5.6.6-crack，进入windows平台
将改名为atlassian-extras-2.4.jar的包放入 jar目录中，然运行confluence_keygen.jar (需要jdk环境)
选择 文件路径 击.gen!生产key值，复制key保留。
将破解后的atlassian-extras-2.4.jar改为atlassian-extras-decoder-v2-3.3.0.jar放到linux上/opt/atlassian-confluence/confluence/WEB-INF/lib目录下
```
5. 启动
```bash
cd /opt/atlassian/confluence/bin
./start-confluence.sh
```
同jira 需要翻墙以及谷歌账号，在配置选项中与jira集成

ok 就完成了！