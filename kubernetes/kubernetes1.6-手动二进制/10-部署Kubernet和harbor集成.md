<!-- toc -->

tags: kubedns

# 配置kubernet与harbor集成

harbor配置完成之后，登陆https://192.168.5.103，创建一个qibei的项目，然后使用docker构建一个zookeeper的镜像push到仓库。
``` bash
# more Dockerfile
FROM java:7-jre
MAINTAINER Lei Wei <leiwei2094@gmail.com>
ENV REFRESHED_AT 20111-01-24
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo 'Asia/Shanghai' >/etc/timezone && \
    echo "export LC_ALL=en_US.UTF-8" >> /etc/profile && \
    . /etc/profile && \
    curl -fSL http://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz -o zookeeper-3.4.11.tar.gz \
    && tar -xzf zookeeper-3.4.11.tar.gz -C /opt \
    && mv /opt/zookeeper-3.4.11 /opt/zookeeper \
    && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg \
    && mkdir -p /tmp/zookeeper

ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64

EXPOSE 2181 2888 3888

WORKDIR /opt/zookeeper

VOLUME ["/opt/zookeeper/conf", "/tmp/zookeeper"]

ENTRYPOINT ["/opt/zookeeper/bin/zkServer.sh"]
CMD ["start-foreground"]
# docker build -t 192.168.5.103/qibei/zookeeper .
Sending build context to Docker daemon   2.56kB
Step 1/10 : FROM java:7-jre
 ---> b0006d129082
Step 2/10 : MAINTAINER Lei Wei <leiwei2094@gmail.com>
 ---> Running in f4cef1a763bc
 ---> 1d168386dd2b
Removing intermediate container f4cef1a763bc
Step 3/10 : ENV REFRESHED_AT 20111-01-24
 ---> Running in eb5f67d660d6
 ---> f0943cf757a1
Removing intermediate container eb5f67d660d6
Step 4/10 : RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime &&     echo 'Asia/Shanghai' >/etc/timezone &&     echo "export LC_ALL=en_US.UTF-8" >> /etc/profile &&     . /etc/profile &&     curl -fSL http://mirrors.tuna.tsinghua.edu.cn/apache/zookeeper/zookeeper-3.4.11/zookeeper-3.4.11.tar.gz -o zookeeper-3.4.11.tar.gz     && tar -xzf zookeeper-3.4.11.tar.gz -C /opt     && mv /opt/zookeeper-3.4.11 /opt/zookeeper     && cp /opt/zookeeper/conf/zoo_sample.cfg /opt/zookeeper/conf/zoo.cfg     && mkdir -p /tmp/zookeeper
 ---> Running in bf1c15eea94b
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100 34.9M  100 34.9M    0     0  10.8M      0  0:00:03  0:00:03 --:--:-- 11.2M
 ---> e8b68e38b109
Removing intermediate container bf1c15eea94b
Step 5/10 : ENV JAVA_HOME /usr/lib/jvm/java-7-openjdk-amd64
 ---> Running in b4780c09988e
 ---> 4e683c8480a7
Removing intermediate container b4780c09988e
Step 6/10 : EXPOSE 2181 2888 3888
 ---> Running in 73d856b68e1c
 ---> 8d4f490d0c05
Removing intermediate container 73d856b68e1c
Step 7/10 : WORKDIR /opt/zookeeper
 ---> 9120f5251076
Removing intermediate container 6b3f8e395904
Step 8/10 : VOLUME /opt/zookeeper/conf /tmp/zookeeper
 ---> Running in 5901ae003d5c
 ---> 9a9d826ee555
Removing intermediate container 5901ae003d5c
Step 9/10 : ENTRYPOINT /opt/zookeeper/bin/zkServer.sh
 ---> Running in 2ba739988783
 ---> 072975b34b51
Removing intermediate container 2ba739988783
Step 10/10 : CMD start-foreground
 ---> Running in ffa63a99ff0f
 ---> 21facf14d3d5
Removing intermediate container ffa63a99ff0f
Successfully built 21facf14d3d5
Successfully tagged 192.168.5.103/qibei/zookeeper:latest
# docker push 192.168.5.103/qibei/zookeeper
The push refers to a repository [192.168.5.103/qibei/zookeeper]
82e32ab3b583: Pushed
f1d01a184c99: Pushed
72e128c24795: Pushed
782d5215f910: Pushed
0eb22bfb707d: Pushed
a2ae92ffcd29: Pushed
latest: digest: sha256:e600de86363238cbebd38e03cb0bb73d2f1146334fbe79d4820620d87f994fc0 size: 1582
```
但是在使用kubectl创建pod时从harbor仓库拉取image时就失败了，如下
``` bash
# more zookeeper-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: zookeeper
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      containers:
      - name: zookeeper
        image: 192.168.5.103/qibei/zookeeper
        ports:
        - containerPort: 2181
# kubectl create -f zookeeper-deployment.yaml
deployment "zookeeper" created
# kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
zookeeper-3900576732-0sk0l   0/1       ContainerCreating   0          4s

# kubectl get pods
NAME                         READY     STATUS         RESTARTS   AGE
zookeeper-3900576732-0sk0l   0/1       ErrImagePull   0          10s

# kubectl describe pods zookeeper-3900576732-0sk0l
Name:		zookeeper-3900576732-0sk0l
Namespace:	default
Node:		192.168.5.107/192.168.5.107
Start Time:	Thu, 07 Dec 2017 11:17:52 +0800
Labels:		app=zookeeper
		pod-template-hash=3900576732
Annotations:	kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"zookeeper-3900576732","uid":"35cf9530-dafd-11e7-ad05-5452001f15a...
Status:		Pending
IP:		172.30.92.4
Controllers:	ReplicaSet/zookeeper-3900576732
Containers:
  zookeeper:
    Container ID:
    Image:		192.168.5.103:5000/qibei/zookeeper
    Image ID:		
    Port:		2181/TCP
    State:		Waiting
      Reason:		ErrImagePull
    Ready:		False
    Restart Count:	0
    Environment:	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5841m (ro)
Conditions:
  Type		Status
  Initialized 	True
  Ready 	False
  PodScheduled 	True
Volumes:
  default-token-5841m:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-5841m
    Optional:	false
QoS Class:	BestEffort
Node-Selectors:	<none>
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath			Type		Reason		Message
  ---------	--------	-----	----			-------------			--------	------		-------
  1m		1m		1	default-scheduler					Normal		Scheduled	Successfully assigned zookeeper-3900576732-0sk0l to 192.168.5.107
  59s		19s		3	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Pulling		pulling image "192.168.5.103:5000/qibei/zookeeper"
  59s		19s		3	kubelet, 192.168.5.107	spec.containers{zookeeper}	Warning		Failed		Failed to pull image "192.168.5.103:5000/qibei/zookeeper": rpc error: code = 2 desc = Error response from daemon: {"message":"Get https://192.168.5.103:5000/v1/_ping: dial tcp 192.168.5.103:5000: getsockopt: connection refused"}
  59s		19s		3	kubelet, 192.168.5.107					Warning		FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "zookeeper" with ErrImagePull: "rpc error: code = 2 desc = Error response from daemon: {\"message\":\"Get https://192.168.5.103:5000/v1/_ping: dial tcp 192.168.5.103:5000: getsockopt: connection refused\"}"

  58s	5s	3	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal	BackOff		Back-off pulling image "192.168.5.103:5000/qibei/zookeeper"
  58s	5s	3	kubelet, 192.168.5.107					Warning	FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "zookeeper" with ImagePullBackOff: "Back-off pulling image \"192.168.5.103:5000/qibei/zookeeper\""

```
单独使用命令docker pull 192.168.5.103/qibei/zookeeper是可以成功拉取image的。通过查询资料得知需要将harbor与kubernetes集群进行集成。



## 配置harbor与kubernetes集成

kubernetes与harbor的集成有多种方式，选中最简单的一种方法：通过kubectl创建一个docker-registry的secret，并在pod描述文件中引用该secret以达到从harbor pull image的目的。

操作之前，可以删除各个node上的~/.docker/config.json

执行kubectl create secret docker-registry时需要提供harbor的访问username和password
``` bash
# kubectl create secret docker-registry harbor-registrykey --docker-server=192.168.5.103 --docker-username=admin --docker-password=qibeitech --docker-email="xxx@xxx.com"
secret "harbor-registrykey" created
[root@docktest-nginx ~/secret]# kubectl get secret
NAME                  TYPE                                  DATA      AGE
default-token-5841m   kubernetes.io/service-account-token   3         1d
harbor-registrykey    kubernetes.io/dockercfg               1         13s
# kubectl get secret harbor-registrykey -o yaml
apiVersion: v1
data:
  .dockercfg: eyIxOTIuMTY4LjUuMTAzIjp7InVzZXJuYW1lIjoiYWRtaW4iLCJwYXNzd29yZCI6InFpYmVpdGVjaCIsImVtYWlsIjoiZHVob25nQHFpYmVpdGVjaC5jb20iLCJhdXRoIjoiWVdSdGFXNDZjV2xpWldsMFpXTm8ifX0=
kind: Secret
metadata:
  creationTimestamp: 2017-12-07T03:13:58Z
  name: harbor-registrykey
  namespace: default
  resourceVersion: "169389"
  selfLink: /api/v1/namespaces/default/secrets/harbor-registrykey
  uid: aa360821-dafc-11e7-9de5-54520009ec1e
type: kubernetes.io/dockercfg

```

docker-registry创建成功后，还需要修改pod的yaml文件，在yaml文件里引用这个secret对象，如下

``` bash
# more zookeeper-deployment.yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: zookeeper
spec:
  replicas: 1
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      containers:
      - name: zookeeper
        image: 192.168.5.103/qibei/zookeeper
        ports:
        - containerPort: 2181
      imagePullSecrets:
      - name: harbor-registrykey

# kubectl delete -f zookeeper-deployment.yaml
deployment "zookeeper" deleted
# kubectl create -f zookeeper-deployment.yaml
deployment "zookeeper" created
# kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
zookeeper-3308723936-dgnj7   0/1       ContainerCreating   0          4s
# kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
zookeeper-3308723936-dgnj7   0/1       ContainerCreating   0          6s
# kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
zookeeper-3308723936-dgnj7   0/1       ContainerCreating   0          9s
# kubectl get pods
NAME                         READY     STATUS              RESTARTS   AGE
zookeeper-3308723936-dgnj7   0/1       ContainerCreating   0          10s
# kubectl describe pods zookeeper-3308723936-dgnj7
Name:		zookeeper-3308723936-dgnj7
Namespace:	default
Node:		192.168.5.107/192.168.5.107
Start Time:	Thu, 07 Dec 2017 11:55:01 +0800
Labels:		app=zookeeper
		pod-template-hash=3308723936
Annotations:	kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"zookeeper-3308723936","uid":"66cab57e-db02-11e7-ad05-5452001f15a...
Status:		Pending
IP:		
Controllers:	ReplicaSet/zookeeper-3308723936
Containers:
  zookeeper:
    Container ID:
    Image:		192.168.5.103/qibei/zookeeper
    Image ID:		
    Port:		2181/TCP
    State:		Waiting
      Reason:		ContainerCreating
    Ready:		False
    Restart Count:	0
    Environment:	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5841m (ro)
Conditions:
  Type		Status
  Initialized 	True
  Ready 	False
  PodScheduled 	True
Volumes:
  default-token-5841m:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-5841m
    Optional:	false
QoS Class:	BestEffort
Node-Selectors:	<none>
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath			Type		Reason		Message
  ---------	--------	-----	----			-------------			--------	------		-------
  23s		23s		1	default-scheduler					Normal		Scheduled	Successfully assigned zookeeper-3308723936-dgnj7 to 192.168.5.107
  20s		20s		1	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Pulling		pulling image "192.168.5.103/qibei/zookeeper"
# kubectl describe pods zookeeper-3308723936-dgnj7
Name:		zookeeper-3308723936-dgnj7
Namespace:	default
Node:		192.168.5.107/192.168.5.107
Start Time:	Thu, 07 Dec 2017 11:55:01 +0800
Labels:		app=zookeeper
		pod-template-hash=3308723936
Annotations:	kubernetes.io/created-by={"kind":"SerializedReference","apiVersion":"v1","reference":{"kind":"ReplicaSet","namespace":"default","name":"zookeeper-3308723936","uid":"66cab57e-db02-11e7-ad05-5452001f15a...
Status:		Running
IP:		172.30.92.4
Controllers:	ReplicaSet/zookeeper-3308723936
Containers:
  zookeeper:
    Container ID:	docker://8947ae1c4b1b287e813a428763c7ad34222987104e0d3d40f40d0adcf34cbd1e
    Image:		192.168.5.103/qibei/zookeeper
    Image ID:		docker-pullable://192.168.5.103/qibei/zookeeper@sha256:e600de86363238cbebd38e03cb0bb73d2f1146334fbe79d4820620d87f994fc0
    Port:		2181/TCP
    State:		Running
      Started:		Thu, 07 Dec 2017 11:55:36 +0800
    Ready:		True
    Restart Count:	0
    Environment:	<none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5841m (ro)
Conditions:
  Type		Status
  Initialized 	True
  Ready 	True
  PodScheduled 	True
Volumes:
  default-token-5841m:
    Type:	Secret (a volume populated by a Secret)
    SecretName:	default-token-5841m
    Optional:	false
QoS Class:	BestEffort
Node-Selectors:	<none>
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath			Type		Reason		Message
  ---------	--------	-----	----			-------------			--------	------		-------
  59s		59s		1	default-scheduler					Normal		Scheduled	Successfully assigned zookeeper-3308723936-dgnj7 to 192.168.5.107
  56s		56s		1	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Pulling		pulling image "192.168.5.103/qibei/zookeeper"
  32s		32s		1	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Pulled		Successfully pulled image "192.168.5.103/qibei/zookeeper"
  25s		25s		1	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Created		Created container with id 8947ae1c4b1b287e813a428763c7ad34222987104e0d3d40f40d0adcf34cbd1e
  25s		25s		1	kubelet, 192.168.5.107	spec.containers{zookeeper}	Normal		Started		Started container with id 8947ae1c4b1b287e813a428763c7ad34222987104e0d3d40f40d0adcf34cbd1e
# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
zookeeper-3308723936-dgnj7   1/1       Running   0          1m

```
这次pod的创建就是正常的。

## 拉取kubernetes插件需要的镜像

+ kubernetes的插件image都在官方docker仓库，安装插件的时候拉取image太痛苦了，提前手动将image pull到本地，然后打个tag，push到harbor中。
+ 在harbor仓库新建一个公开的k8s项目，然后将kubernetes查看对应的image全部push过去。harbor上的公开项目可以直接pull不需要docker-register。

``` bash
docker pull xuejipeng/k8s-dns-kube-dns-amd64:v1.14.1
docker pull xuejipeng/k8s-dns-dnsmasq-nanny-amd64:v1.14.1
docker pull xuejipeng/k8s-dns-sidecar-amd64:v1.14.1
docker pull lvanneo/heapster-grafana-amd64:v4.0.2
docker pull lvanneo/heapster-amd64:v1.3.0-beta.1
docker pull lvanneo/heapster-influxdb-amd64:v1.1.1
docker pull cokabug/kubernetes-dashboard-amd64:v1.6.0
```
