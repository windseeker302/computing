# Kubeedge

## 概念

#### 云原生

云原生技术使组织能够在现代的动态环境（如公共云、私有云和混合云）中构建和运行可扩展的应用程序。
容器、服务网格、微服务、不可变基础设施和声明性api都是这种方法的例子。

#### 云原生特性容器

![image-20241118102445414](kubeedge.assets/image-20241118102445414.png)

服务网格

- 传统

使用 service deployment 创建微服务

微服务 bookinfo  服务通过service

- 使用 istio  创建微服务



#### kubeedge 

类似于章鱼，章鱼40%的神经元分布在大脑上，60%的神经元分布在腕上，多个小脑和一个大脑组成的一个分布式计算，边缘计算也是一种分布式计算，好处就是大部分重复，简易的操作由边缘侧来完成。



#### 云边通信方式

云端和边缘端通过websocket建立实时且双方通信的连接。



#### 什么是WebSocket

WebSocket是一种协议，用于在Web应用程序和服务器之间建立实时、[双向的](https://so.csdn.net/so/search?q=双向的&spm=1001.2101.3001.7020)通信连接。它通过一个单一的TCP连接提供了持久化连接，这使得Web应用程序可以更加实时地传递数据。WebSocket协议最初由W3C开发，并于2011年成为标准。

WebSocket的优势和劣势

**WebSocket的优势包括：**

- **实时性：** 由于WebSocket的持久化连接，它可以实现实时的数据传输，避免了Web应用程序需要不断地发送请求以获取最新数据的情况。
- **双向通信：** WebSocket协议支持双向通信，这意味着服务器可以主动向客户端发送数据，而不需要客户端发送请求。
- **减少网络负载：** 由于WebSocket的持久化连接，它可以减少HTTP请求的数量，从而减少了网络负载。

**WebSocket的劣势包括：**

- **需要浏览器和服务器都支持：** WebSocket是一种相对新的技术，需要浏览器和服务器都支持。一些旧的浏览器和服务器可能不支持WebSocket。
- **需要额外的开销：** WebSocket需要在服务器上维护长时间的连接，这需要额外的开销，包括内存和CPU。
- **安全问题：** 由于WebSocket允许服务器主动向客户端发送数据，可能会存在安全问题。服务器必须保证只向合法的客户端发送数据。



#### 云边架构图

![image-20241120102350402](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20241120102350402.png)

云端工作原理，分发任务

- EdgeController

![image-20241120103339255](kubeedge.assets/image-20241120103339255.png)

- DeviceController

![image-20241120105234072](kubeedge.assets/image-20241120105234072.png)

边端工作原理

边端通过metaManager（管理pod）和DeviceTwin（管理crd device，接入设备）两个组件进行工作。

![image-20241120132256384](kubeedge.assets/image-20241120132256384.png)

![image-20241120132241879](kubeedge.assets/image-20241120132241879.png)











## 部署

### 一、云端

#### 1、基础环境检查与配置

```shell
# master
kubectl get nodes,pod -A
hostnamectl
cat >> /etc/hosts <<EOF
121.36.219.145 master
1.94.42.213 kubeedge-node1
172.129.78.58 kubeedge-node2
EOF
```

#### 2、上传 kubernetes_kubeedge.tar.gz

```shell
# master
wget kubernetes_kubeedge_allinone.tar.gz	#路径不一
tar -zxvf kubernetes_kubeedge_allinone.tar.gz -C /opt/
mv /etc/yum.repos.d/* /media/
cat > /etc/yum.repos.d/local.repo <<EOF
[docker]
name=docker
baseurl=file:///opt/yum
gpgcheck=0
enabled=1
EOF
yum -y install vsftpd
echo anon_root=/opt >> /etc/vsftpd/vsftpd.conf
systemctl enable vsftpd --now
```

#### 3、配置docker本地镜像

```shell
#master
cat >> /etc/docker/daemon.json << EOF
{
 "log-driver": "json-file",
 "log-opts": {
 "max-size": "200m",
 "max-file": "5"
 },
 "default-ulimits": {
 "nofile": {
 "Name": "nofile",
 "Hard": 655360,
 "Soft": 655360
 },
 "nproc": {
 "Name": "nproc",
 "Hard": 655360,
 "Soft": 655360
 }
 },
 "live-restore": true,
 "oom-score-adjust": -1000,
 "max-concurrent-downloads": 10,
 "max-concurrent-uploads": 10,
 "insecure-registries": ["0.0.0.0/0"]
}
EOF
systemctl daemon-reload
systemctl enable docker --now #设置开机自启
```

#### 4、部署Harbor仓库

```shell
#master
cd /opt
mv docker-compose-Linux-x86_64 /usr/bin/docker-compose
tar -zxvf harbor-offline-installer-v2.5.0.tgz 
cd harbor && cp harbor.yml.tmpl harbor.yml
sed -i 's/hostname: reg.mydomain.com/hostname: 121.36.219.145/g' /opt/harbor/harbor.yml
./install.sh
docker login -u admin -p Harbor12345 master
cd /opt/k8simage/ && sh load.sh	#如下图创建好在执行push.sh
sh push.sh	#需要控制节点IP
```

![image-20240710100101819](../../../Users/admin/Documents/WeChat Files/wxid_ccirxal4w0gm22/FileStorage/File/2024-11/assets/image-20240710100101819.png)

#### 5、配置节点亲和性（手搓版）

```shell
#master
kubectl edit daemonset -n kube-system kube-flannel-ds	#在41-44之间内容
32     spec:
33       affinity:
34         nodeAffinity:
35           requiredDuringSchedulingIgnoredDuringExecution:
36             nodeSelectorTerms:
37             - matchExpressions:
38               - key: kubernetes.io/os
39                 operator: In
40                 values:
41                 - linux
42               -key: node-role.kubernetes.io/kubeedge
43                 operator: DoesNotExist
44       containers:
45       - args:
46         - --ip-masq
47         - --kube-subnet-mgr
48         command:
49         - /opt/bin/flanneld

kubectl edit daemonset -n kube-system kube-proxy	#28-36之间内容

24     metadata:
25       creationTimestamp: null
26       labels:
27         k8s-app: kube-proxy
28     spec:
29       affinity:
30         nodeAffinity:
31           requiredDuringSchedulingIgnoredDuringExecution:
32             nodeSelectorTerms:
33             - matchExpressions:
34               - key: node-role.kubernetes.io/edge
35                 operator: DoesNotExist
36       containers:
37       - command:
38         - /usr/local/bin/kube-proxy
39         - --config=/var/lib/kube-proxy/config.conf
40         - --hostname-override=$(NODE_NAME)
41         env:
42         - name: NODE_NAME
43           valueFrom:
44             fieldRef:
```

#### 6、查看修改后的pod

```shell
#master
kubectl get pod -n kube-system
NAME                              READY   STATUS    RESTARTS   AGE
coredns-78fcd69978-jc2b6          1/1     Running   0          42m
coredns-78fcd69978-jrkl7          1/1     Running   0          42m
etcd-master                       1/1     Running   0          42m
kube-apiserver-master             1/1     Running   0          42m
kube-controller-manager-master    1/1     Running   0          42m
kube-flannel-ds-fjqjq             1/1     Running   0          17m
kube-proxy-q4k2k                  1/1     Running   0          10m
kube-scheduler-master             1/1     Running   0          42m
metrics-server-77564bc84d-n8lbg   1/1     Running   0          42m
```

#### 7、配置云端安装包

```shell
#master
cd /opt/kubeedge/
mv keadm /usr/bin/
mkdir /etc/kubeedge
tar -zxf kubeedge-1.11.1.tar.gz
cp -rf kubeedge-1.11.1/build/tools/* /etc/kubeedge/
cp -rf kubeedge-1.11.1/build/crds/ /etc/kubeedge/
tar -zxf kubeedge-v1.11.1-linux-amd64.tar.gz
cp -rf * /etc/kubeedge/
```

#### 8、启动云端

```shell
#master
cd /etc/kubeedge
keadm deprecated init --kubeedge-version=1.11.1 --advertise-address=121.36.219.145
```

#### 9、检查云端状态

```shell
#master
netstat -ntpl |grep cloudcore
tcp6       0      0 :::10000                :::*                    LISTEN      5672/cloudcore 
tcp6       0      0 :::10002                :::*                    LISTEN      5672/cloudcore 
```

#### 10、获取加入token

```shell
#master
keadm gettoken
cfcb4c262b1727d9e3a909f1da8cd13301801e5e6793fe73494daf9edf8a4998.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjA2NjY0MTZ9.o3lqE3uQEcNGGHdxfitkh1av6NRWh4UylP_tLDZ6rIc
```

#### 11、查看云端状态

```shell
#master
kubectl get no
NAME             STATUS   ROLES                  AGE     VERSION
kubeedge-node1   Ready    agent,edge             14m     v1.22.6-kubeedge-v1.11.1
kubeedge-node2   Ready    agent,edge             65s     v1.22.6-kubeedge-v1.11.1
master           Ready    control-plane,master   5h11m   v1.22.1
```

#### 12、云端监控服务

```shell
#master
cat >> /etc/kubeedge/top.so << EOF
export CLOUDCOREIPS="121.36.219.145"
EOF
source /etc/kubeedge/top.so
cd /etc/kubeedge/
 ./certgen.sh stream
```

#### 13、修改云端配置

```shell
#master
sed -i '44,44s/false/true/g' /etc/kubeedge/config/cloudcore.yaml
sed -i '103,103s/false/true/g' /etc/kubeedge/config/cloudcore.yaml
```

#### 14、重启云端服务

```shell
#master
kill -9 $(netstat -lntup |grep cloudcore |awk 'NR==1 {print $7}' |cut -d '/' -f 1)
cp -rfv /etc/kubeedge/cloudcore.service /usr/lib/systemd/system/
systemctl start cloudcore.service
```

#### 15、查看云端端口状态

```shell
#master
netstat -lntup |grep 10003
```

### 二、边端

#### 1、基础环境检查与配置

```shell
#kubeedge
hostnamectl set-hostname kubeedge-node2
rm -rf /etc/yum.repos.d/*
cat >> /etc/yum.repos.d/http.repo << EOF
[centos]
name=centos
baseurl=http://192.168.77.200:30000/source1/centos
gpgcheck=0
enabled=1
EOF
yum -y install bash-com vim wget lrzsz net-tools unzip
cat >> /etc/hosts << EOF
172.129.78.34 kubeedge-node2
1.94.42.213 kubeedge-node
121.36.219.145 master
EOF
```

#### 2、上传kubernetes_kubeedge_allinone.tar.gz

```shell
#kubeedge
wget kubernetes_kubeedge_allinone.tar.gz
tar zxvf kubernetes_kubeedge_allinone.tar.gz -C /opt/
```

#### 3、安装docker，配置本地镜像

```shell
#kubeedge
cat >> /etc/yum.repos.d/dcoker.repo << EOF
[docker]
name=docker
baseurl=ftp://master/yum
gpgcheck=0
enabled=1
EOF
yum -y install docker-ce
cat >> /etc/docker/daemon.json << EOF
{
 "log-driver": "json-file",
 "log-opts": {
 "max-size": "200m",
 "max-file": "5"
 },
 "default-ulimits": {
 "nofile": {
 "Name": "nofile",
 "Hard": 655360,
 "Soft": 655360
 },
 "nproc": {
 "Name": "nproc",
 "Hard": 655360,
 "Soft": 655360
 }
 },
 "live-restore": true,
 "oom-score-adjust": -1000,
 "max-concurrent-downloads": 10,
 "max-concurrent-uploads": 10,
 "insecure-registries": ["0.0.0.0/0"]
}
EOF
systemctl daemon-reload
systemctl enable docker --now #开机自启
docker ps
```

#### 4、配置边端环境

```shell
#kubeedge
scp root@master:/usr/bin/keadm /usr/local/bin/	#需要输入密码
mkdir /etc/kubeedge
cd /etc/kubeedge/
scp -r root@master:/etc/kubeedge/* /etc/kubeedge/	#需要输入密码
```

#### 5、加入云端

```shell
#kubeedge
keadm deprecated join --cloudcore-ipport=121.36.219.145:10000 --kubeedge-version=1.11.1 --token=cfcb4c262b1727d9e3a909f1da8cd13301801e5e6793fe73494daf9edf8a4998.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MjA2NzE0MDB9.ry4baygoQLt6fvk-lJfdqrzqXvwIjwlo87EnryPyGHg
```

#### 6、查看边端状态

```shell
#kubeedge
systemctl status edgecore
● edgecore.service
   Loaded: loaded (/etc/systemd/system/edgecore.service; enabled; vendor preset: disabled)
   Active: active (running) since Wed 2024-07-10 14:36:12 CST; 16min ago
 Main PID: 9418 (edgecore)
   CGroup: /system.slice/edgecore.service
           └─9418 /usr/local/bin/edgecore

Jul 10 14:51:14 kubeedge-node1 edgecore[9418]: W0710 14:51:14.474534    9418 context_channel.go:159] Get bad anonName:f13e9d6f-8efa-4210-8f3... nothing
Jul 10 14:51:24 kubeedge-node1 edgecore[9418]: W0710 14:51:24.493382    9418 context_channel.go:159] Get bad anonName:7c57035d-822c-4bbc-83f... nothing
Jul 10 14:51:34 kubeedge-node1 edgecore[9418]: W0710 14:51:34.514152    9418 context_channel.go:159] Get bad anonName:d30754b3-5322-4e5e-888... nothing
Jul 10 14:51:44 kubeedge-node1 edgecore[9418]: W0710 14:51:44.533968    9418 context_channel.go:159] Get bad anonName:89166e40-1fa0-4819-b67... nothing
Jul 10 14:51:54 kubeedge-node1 edgecore[9418]: W0710 14:51:54.554774    9418 context_channel.go:159] Get bad anonName:6f9327d8-bbdb-44dd-977... nothing
Jul 10 14:52:04 kubeedge-node1 edgecore[9418]: W0710 14:52:04.576824    9418 context_channel.go:159] Get bad anonName:e3e66e58-6452-49f4-91c... nothing
Jul 10 14:52:14 kubeedge-node1 edgecore[9418]: W0710 14:52:14.594857    9418 context_channel.go:159] Get bad anonName:95459aa5-cfc5-4cb4-acd... nothing
Jul 10 14:52:24 kubeedge-node1 edgecore[9418]: W0710 14:52:24.614323    9418 context_channel.go:159] Get bad anonName:eb7ceb52-c779-461b-b97... nothing
Jul 10 14:52:34 kubeedge-node1 edgecore[9418]: W0710 14:52:34.634790    9418 context_channel.go:159] Get bad anonName:cb1f5a87-3776-4eb4-a18... nothing
Jul 10 14:52:44 kubeedge-node1 edgecore[9418]: W0710 14:52:44.655888    9418 context_channel.go:159] Get bad anonName:01d9d4a3-7478-47bd-8da... nothing
Hint: Some lines were ellipsized, use -l to show in full.
```

#### 7、边端监控服务

```shell
#kubeedge
sed -i '37,37s/false/true/g' /etc/kubeedge/config/edgecore.yaml
sed -i '103,103s/false/true/g' /etc/kubeedge/config/edgecore.yaml
systemctl restart edgecore.service
```



