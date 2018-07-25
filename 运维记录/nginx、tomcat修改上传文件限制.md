# nginx
上传超过1M大的客户端文件无法正常上传，nginx直接报错，上传文件太大，于是修改了下nginx的配置，就可以了。 
```bash
server {
        listen       80;
        server_name  localhost;
        client_max_body_size 10M;
.....
```
> client_max_body_size 10M 必须要放在server下的server_name下
# tomcat
如果是使用spring cloud 那估计那边设置也要改

当服务器是Tomcat时，通过POST上传的文件大小的最大值为2M（2097152）。

如果想修改该限制，修改方法如下：

tomcat目录下的conf文件夹下，server.xml 文件中以下的位置中添加maxPostSize参数
```bash
<Connector port="8081"  
               maxThreads="150" minSpareThreads="25" maxSpareThreads="75"  
               enableLookups="false" redirectPort="8443" acceptCount="100"  
               debug="0" connectionTimeout="20000"   
               disableUploadTimeout="true" URIEncoding="utf-8"  
               maxPostSize="0"/> 
```
> 当maxPostSize<=0时，POST方式上传的文件大小不会被限制，maxPostSize参数只有当request的Content-Type为“application/x-www-form-urlencoded”时起作用。