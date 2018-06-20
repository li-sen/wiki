# 测试Deployment的rolling-update


## 通过yaml文件来执行Deployment的rolling-update

kubernetes Deployment是一个更高级别的抽象，就像文章开头那幅示意图那样，Deployment会创建一个Replica Set，用来保证Deployment中Pod的副本数。由于kubectl rolling-update仅支持replication controllers，因此要想rolling-updata deployment中的Pod，你需要修改Deployment自己的manifest文件并应用。这个修改会创建一个新的Replica Set，在scale up这个Replica Set的Pod数的同时，减少原先的Replica Set的Pod数，直至zero。而这一切都发生在Server端，并不需要kubectl参与。
我们同样来看一个例子。
我们建立第一个版本的deployment manifest文件：nginx-v0.1.yaml。
```bash
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  minReadySeconds: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 3
      maxUnavailable: 2
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        version: v0.1
    spec:
      containers:
        - name: nginx
          image: 172.16.100.86/efk/nginx:1.10.1
          resources:
            requests:
              cpu: 100m
              memory: 100Mi
          ports:
            - containerPort: 80
              protocol: TCP
          env:
            - name: DEPLOYMENT_DEMO_VER
              value: v0.1
```
创建该deployment：
```bash
[root@docker-deploy duhong-test]# kubectl create -f nginx-v0.1.yaml 
deployment "nginx" created 
[root@docker-deploy duhong-test]# kubectl get deployments
NAME        DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx       4         4         4            4           21s
zookeeper   1         1         1            1           23d
[root@docker-deploy duhong-test]# kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
nginx-896838084        4         4         4         28s
zookeeper-1176509725   1         1         1         23d
[root@docker-deploy duhong-test]# kubectl get pods -o wide
NAME                         READY     STATUS    RESTARTS   AGE       IP            NODE
nginx-896838084-1jc9q        1/1       Running   0          35s       172.30.81.7   172.16.100.82
nginx-896838084-c7nvv        1/1       Running   0          35s       172.30.1.4    172.16.100.81
nginx-896838084-k340b        1/1       Running   0          35s       172.30.59.6   172.16.100.83
nginx-896838084-rngmk        1/1       Running   0          35s       172.30.59.7   172.16.100.83
zookeeper-1176509725-dvjz2   1/1       Running   8          23d       172.30.81.3   172.16.100.82
[root@docker-deploy duhong-test]# kubectl exec nginx-896838084-1jc9q env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx-896838084-1jc9q
DEPLOYMENT_DEMO_VER=v0.1
ZOOKEEPER_SERVICE_HOST=10.254.10.69
ZOOKEEPER_PORT=tcp://10.254.10.69:2181
ZOOKEEPER_PORT_2181_TCP_ADDR=10.254.10.69
KUBERNETES_PORT=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
ZOOKEEPER_PORT_2181_TCP=tcp://10.254.10.69:2181
KUBERNETES_SERVICE_HOST=10.254.0.1
KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
ZOOKEEPER_SERVICE_PORT=2181
ZOOKEEPER_PORT_2181_TCP_PROTO=tcp
ZOOKEEPER_PORT_2181_TCP_PORT=2181
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
NGINX_VERSION=1.10.1-1~jessie
HOME=/root
```
deployment-demo创建了ReplicaSet：nginx-896838084，用于保证Pod的副本数。
我们再来创建使用了该deployment中Pods的Service：
```bash
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx

[root@docker-deploy duhong-test]# kubectl create -f nginx-svc.yaml 
service "nginx" created
[root@docker-deploy duhong-test]# kubectl get svc
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   10.254.0.1      <none>        443/TCP    40d
nginx        10.254.156.74   <none>        80/TCP     4s
zookeeper    10.254.10.69    <none>        2181/TCP   22d
[root@docker-deploy duhong-test]# kubectl describe service/nginx
Name:			nginx
Namespace:		default
Labels:			<none>
Annotations:		<none>
Selector:		app=nginx
Type:			ClusterIP
IP:			10.254.156.74
Port:			<unset>	80/TCP
Endpoints:		172.30.1.4:80,172.30.59.6:80,172.30.59.7:80 + 1 more...
Session Affinity:	None
Events:			<none>    
```
好了，我们看到该service下有四个pods，Service提供的服务也运行正常。
接下来，我们对该Service进行更新。为了方便说明，我们建立了nginx-v0.2.yaml文件，其实你也大可不必另创建文件，直接再上面的nginx-v0.1.yaml文件中修改也行：
```bash
[root@docker-deploy duhong-test]# diff nginx-v0.1.yaml nginx-v0.2.yaml 
20c20
<         version: v0.1
---
>         version: v0.2
24c24
<           image: 172.16.100.86/efk/nginx:1.10.1
---
>           image: 172.16.100.86/efk/nginx:1.11.9
34c34
<               value: v0.1
---
>               value: v0.2
```
我们用nginx-v0.2.yaml文件来更新之前创建的deployments中的Pods：

```bash
[root@docker-deploy duhong-test]# kubectl apply -f nginx-v0.2.yaml --record
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment "nginx" configured
[root@docker-deploy duhong-test]# kubectl describe deployment nginx
Name:			nginx
Namespace:		default
CreationTimestamp:	Thu, 14 Dec 2017 16:42:36 +0800
Labels:			app=nginx
			version=v0.1
Annotations:		deployment.kubernetes.io/revision=2
			kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"extensions/v1beta1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx","namespace":"default"},"spec":{"minReadySeconds":5,"...
			kubernetes.io/change-cause=kubectl apply --filename=nginx-v0.2.yaml --record=true
Selector:		app=nginx
Replicas:		4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:		RollingUpdate
MinReadySeconds:	5
RollingUpdateStrategy:	2 max unavailable, 3 max surge
Pod Template:
  Labels:	app=nginx
		version=v0.2
  Containers:
   nginx:
    Image:	172.16.100.86/efk/nginx:1.11.9
    Port:	80/TCP
    Requests:
      cpu:	100m
      memory:	100Mi
    Environment:
      DEPLOYMENT_DEMO_VER:	v0.2
    Mounts:			<none>
  Volumes:			<none>
Conditions:
  Type		Status	Reason
  ----		------	------
  Available 	True	MinimumReplicasAvailable
OldReplicaSets:	<none>
NewReplicaSet:	nginx-2286594511 (4/4 replicas created)
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----			-------------	--------	------			-------
  8m		8m		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-896838084 to 4
  34s		34s		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-2286594511 to 3
  34s		34s		1	deployment-controller			Normal		ScalingReplicaSet	Scaled down replica set nginx-896838084 to 2
  34s		34s		1	deployment-controller			Normal		ScalingReplicaSet	Scaled up replica set nginx-2286594511 to 4
  28s		28s		1	deployment-controller			Normal		ScalingReplicaSet	Scaled down replica set nginx-896838084 to 0
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nginx-2286594511-brz32       1/1       Running   0          1m
nginx-2286594511-dsvjg       1/1       Running   0          1m
nginx-2286594511-j7dfj       1/1       Running   0          1m
nginx-2286594511-t2z83       1/1       Running   0          1m
zookeeper-1176509725-dvjz2   1/1       Running   8          23d
[root@docker-deploy duhong-test]# kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
nginx-2286594511       4         4         4         3m
nginx-896838084        0         0         0         11m
zookeeper-1176509725   1         1         1         23d
[root@docker-deploy duhong-test]# kubectl exec nginx-2286594511-brz32 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx-2286594511-brz32
DEPLOYMENT_DEMO_VER=v0.2
KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
ZOOKEEPER_PORT_2181_TCP=tcp://10.254.10.69:2181
NGINX_PORT_80_TCP=tcp://10.254.156.74:80
NGINX_PORT_80_TCP_PROTO=tcp
KUBERNETES_SERVICE_HOST=10.254.0.1
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_PORT=443
NGINX_SERVICE_HOST=10.254.156.74
NGINX_PORT_80_TCP_PORT=80
KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
ZOOKEEPER_PORT_2181_TCP_ADDR=10.254.10.69
NGINX_SERVICE_PORT=80
NGINX_PORT=tcp://10.254.156.74:80
KUBERNETES_PORT=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
ZOOKEEPER_SERVICE_HOST=10.254.10.69
ZOOKEEPER_SERVICE_PORT=2181
ZOOKEEPER_PORT=tcp://10.254.10.69:2181
ZOOKEEPER_PORT_2181_TCP_PROTO=tcp
ZOOKEEPER_PORT_2181_TCP_PORT=2181
NGINX_PORT_80_TCP_ADDR=10.254.156.74
NGINX_VERSION=1.11.9-1~jessie
HOME=/root
```
apply命令是瞬间接收到apiserver返回的Response并结束的。但deployment的rolling-update过程还在进行：
我们可以看到这个update过程中ReplicaSet的变化，同时这个过程中服务并未中断，只是新旧版本短暂地交错提供服务：
最终所有Pod被替换为了v0.2版本：
我们发现deployment的create和apply命令都带有一个–record参数，这是告诉apiserver记录update的历史。通过kubectl rollout history可以查看deployment的update history：

```bash
[root@docker-deploy duhong-test]# kubectl rollout history deployment nginx
deployments "nginx"
REVISION	CHANGE-CAUSE
1		<none>
2		kubectl apply --filename=nginx-v0.2.yaml --record=true
```
同时，我们会看到old ReplicaSet并未被删除：
```bash
[root@docker-deploy duhong-test]# kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
nginx-2286594511       4         4         4         3m
nginx-896838084        0         0         0         11m
zookeeper-1176509725   1         1         1         23d
```
这些信息都存储在server端，方便回退！
Deployment下Pod的回退操作异常简单，通过rollout undo即可完成。rollout undo会将Deployment回退到record中的上一个revision（见上面rollout history的输出中有revision列）：
```bash
[root@docker-deploy duhong-test]# kubectl rollout undo deployment nginx
deployment "nginx" rolled back
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nginx-2286594511-brz32       1/1       Running   0          6m
nginx-2286594511-j7dfj       1/1       Running   0          6m
nginx-896838084-3cvxl        1/1       Running   0          5s
nginx-896838084-pwz7n        1/1       Running   0          5s
nginx-896838084-qj2g8        1/1       Running   0          5s
nginx-896838084-vgvlh        1/1       Running   0          5s
zookeeper-1176509725-dvjz2   1/1       Running   8          23d
[root@docker-deploy duhong-test]# kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
nginx-2286594511       0         0         0         6m
nginx-896838084        4         4         4         14m
zookeeper-1176509725   1         1         1         23d
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS        RESTARTS   AGE
nginx-2286594511-brz32       0/1       Terminating   0          6m
nginx-896838084-3cvxl        1/1       Running       0          16s
nginx-896838084-pwz7n        1/1       Running       0          16s
nginx-896838084-qj2g8        1/1       Running       0          16s
nginx-896838084-vgvlh        1/1       Running       0          16s
zookeeper-1176509725-dvjz2   1/1       Running       8          23d
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nginx-896838084-3cvxl        1/1       Running   0          18s
nginx-896838084-pwz7n        1/1       Running   0          18s
nginx-896838084-qj2g8        1/1       Running   0          18s
nginx-896838084-vgvlh        1/1       Running   0          18s
zookeeper-1176509725-dvjz2   1/1       Running   8          23d
```
回滚后rs的状态又颠倒回来：

查看update历史:
```bash
[root@docker-deploy duhong-test]# kubectl rollout history deployment nginx
deployments "nginx"
REVISION	CHANGE-CAUSE
2		kubectl apply --filename=nginx-v0.2.yaml --record=true
3		<none>
```
可以看到history中最多保存了两个revision记录（这个Revision保存的数量应该可以设置）。

## 通过set image来执行Deployment的rolling-update
上面的方法是通过yaml文件的方式来执行rolling-update，还有一种直接通过set image的方式来rolling-update
```bash
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nginx-896838084-3cvxl        1/1       Running   0          3m
nginx-896838084-pwz7n        1/1       Running   0          3m
nginx-896838084-qj2g8        1/1       Running   0          3m
nginx-896838084-vgvlh        1/1       Running   0          3m
zookeeper-1176509725-dvjz2   1/1       Running   8          23d
[root@docker-deploy duhong-test]# kubectl exec nginx-896838084-3cvxl  env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx-896838084-3cvxl
DEPLOYMENT_DEMO_VER=v0.1
NGINX_SERVICE_HOST=10.254.156.74
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
ZOOKEEPER_SERVICE_HOST=10.254.10.69
ZOOKEEPER_PORT_2181_TCP_PROTO=tcp
NGINX_PORT=tcp://10.254.156.74:80
NGINX_PORT_80_TCP_PORT=80
KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
ZOOKEEPER_PORT_2181_TCP=tcp://10.254.10.69:2181
ZOOKEEPER_PORT_2181_TCP_ADDR=10.254.10.69
NGINX_PORT_80_TCP=tcp://10.254.156.74:80
KUBERNETES_SERVICE_PORT=443
KUBERNETES_PORT=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
ZOOKEEPER_SERVICE_PORT=2181
ZOOKEEPER_PORT=tcp://10.254.10.69:2181
NGINX_SERVICE_PORT=80
NGINX_PORT_80_TCP_PROTO=tcp
NGINX_PORT_80_TCP_ADDR=10.254.156.74
KUBERNETES_SERVICE_HOST=10.254.0.1
ZOOKEEPER_PORT_2181_TCP_PORT=2181
NGINX_VERSION=1.10.1-1~jessie
HOME=/root
```
可以看出当前nginx pod的nginx版本为1.10.1，通过如下命令来将nginx pod中nginx的版本更新为1.11.9
```bash
[root@docker-deploy duhong-test]# kubectl set image deploy nginx nginx=172.16.100.86/efk/nginx:1.11.9 --record
deployment "nginx" image updated
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS        RESTARTS   AGE
nginx-2033560013-030r6       1/1       Running       0          5s
nginx-2033560013-1j5kv       1/1       Running       0          5s
nginx-2033560013-913cz       1/1       Running       0          5s
nginx-2033560013-n37pd       1/1       Running       0          5s
nginx-896838084-3cvxl        0/1       Terminating   0          7m
nginx-896838084-pwz7n        1/1       Running       0          7m
nginx-896838084-qj2g8        0/1       Terminating   0          7m
nginx-896838084-vgvlh        1/1       Running       0          7m
zookeeper-1176509725-dvjz2   1/1       Running       8          23d
[root@docker-deploy duhong-test]# kubectl get rs
NAME                   DESIRED   CURRENT   READY     AGE
nginx-2033560013       4         4         4         8s
nginx-2286594511       0         0         0         13m
nginx-896838084        0         0         0         21m
zookeeper-1176509725   1         1         1         23d
```
更新过程和通过yaml完全一致。

```bash
[root@docker-deploy duhong-test]# kubectl get pods
NAME                         READY     STATUS    RESTARTS   AGE
nginx-2033560013-030r6       1/1       Running   0          2m
nginx-2033560013-1j5kv       1/1       Running   0          2m
nginx-2033560013-913cz       1/1       Running   0          2m
nginx-2033560013-n37pd       1/1       Running   0          2m
zookeeper-1176509725-dvjz2   1/1       Running   8          23d
[root@docker-deploy duhong-test]# kubectl exec nginx-2033560013-030r6 env
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
HOSTNAME=nginx-2033560013-030r6
DEPLOYMENT_DEMO_VER=v0.1
NGINX_PORT=tcp://10.254.156.74:80
NGINX_PORT_80_TCP_ADDR=10.254.156.74
KUBERNETES_SERVICE_PORT=443
KUBERNETES_SERVICE_PORT_HTTPS=443
ZOOKEEPER_SERVICE_HOST=10.254.10.69
ZOOKEEPER_PORT=tcp://10.254.10.69:2181
ZOOKEEPER_PORT_2181_TCP_ADDR=10.254.10.69
NGINX_SERVICE_HOST=10.254.156.74
KUBERNETES_PORT=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP=tcp://10.254.0.1:443
KUBERNETES_PORT_443_TCP_PROTO=tcp
NGINX_SERVICE_PORT=80
KUBERNETES_PORT_443_TCP_PORT=443
KUBERNETES_PORT_443_TCP_ADDR=10.254.0.1
NGINX_PORT_80_TCP_PORT=80
KUBERNETES_SERVICE_HOST=10.254.0.1
ZOOKEEPER_SERVICE_PORT=2181
ZOOKEEPER_PORT_2181_TCP_PROTO=tcp
ZOOKEEPER_PORT_2181_TCP_PORT=2181
NGINX_PORT_80_TCP_PROTO=tcp
ZOOKEEPER_PORT_2181_TCP=tcp://10.254.10.69:2181
NGINX_PORT_80_TCP=tcp://10.254.156.74:80
NGINX_VERSION=1.11.9-1~jessie
HOME=/root
[root@docker-deploy duhong-test]# kubectl rollout history deployment nginx
deployments "nginx"
REVISION	CHANGE-CAUSE
2		kubectl apply --filename=nginx-v0.2.yaml --record=true
3		<none>
4		kubectl set image deploy nginx nginx=172.16.100.86/efk/nginx:1.11.9 --record=true
```
可以看出nginx的镜像已经成功更新为1.11.9，并且rollout history也有记录。
