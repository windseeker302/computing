# 	kubernetes 总结

## 概念

### 对象规约和状态

#### 规约（Spec）

"spec"是"规约、规格"的意思，spec是必需的，它表述了对象的期望状态（Desired State）——希望对象所具有的特征，当创建 kubernetes 对象时，必须提供对象的规约，用来描述该对象的期望状态，已经关于对象的一些基本信息（例如名称）。

#### 状态（Status）

表示对象的实际状态，该属性由 k8s 自己维护，k8s 会通过一系列的控制器对对应对象进行管理，让对象尽可能的让实际状态与期望状态保持一致。

### 服务分类

- 有状态

  代表应用：Apache，Nginx

  优点：对客户透明，无依赖关系，可以高效实现扩容，迁移

  缺点：不能存储数据，需要额外的数据服务支撑

- 无状态

  代表应用：MySQL，Redis

  优点：可以独立存储数据，实现数据管理

  缺点：群集环境下需要实现主从，数据同步，备份，水平扩缩容部署比较复杂

  Deployment通常部署无状态的服务，SatefulSet 部署有状态的服务。



### 对象和资源

#### 元数据级

Horizontal Pod Autoscaler (HPA:自动扩缩容)

PodTemplate

LimitRange

#### 集群级

Namespace

Node

ClusterRole

ClusterRoleBinding

#### 命名空间级

##### 控制器

###### 无状态

Deployment

###### 有状态

StatefulSet 是有状态的集合，管理有状态的服务，它所管理的 Pod 的名称不能随意变化。

数据持久化的目录也是不一样，每一个 Pod 都有自己独有的书持久化存储目录。

比如 Mysql 主从，redis 集群等。

###### 功能

> Headless Service

用来定义 pod 网络标识，生成可解释的 DNS 记录。在deployment的使用中，它所创建出来的 pod 是没有顺序的，是随机字符串，而在 statefulset中所管理的 pod 都是要求，有序，且不能变化的，每一个 pod 不能被随意取代，pod 重建后 pod 名称还是一样的，因为 pod ip 是变化的，所以要用生成的 pod 所产生的 DNS 记录来作为每个 pod 的唯一标识，就是访问路径。

StatefulSet 中每个 pod 的 DNS 格式为 statefulSetName-(0..N-1).serviceName.namesapce.svc.cluster.local

- serviceName ：Headless Service 的名字。
- 0..N-1：pod所在的序号，从 0 开始到 N-1。
- statefulSetName：StatefulSet 的名称。
- namespace：服务所在的 namespace ，Headless Service 和 StatefulSet 必须在相同的 namespace。
- cluster.local：是 Cluster Domain 的名称。

~~~shell
# 例如
# 第一个pod名称
mysql-sts-0.nginx-svc.ws.svc.cluster.local
# 第二个pod名称
mysql-sts-1.nginx-svc.ws.svc.cluster.local
# ... 以此类推
~~~



> volumeClaimTemplates

存储卷申请模板，创建 pvc，指定 pvc 名称大小，自动创建 pvc，且 pvc 由存储类供应。

对于有状态应用都会用到持久化存储，比如 mysql 主从，由于主从数据库的数据是不能存放在一 个目录下的，每个 mysql 节点都需要有自己独立的存储空间。而在 deployment 中创建的存储卷 是一个共享的存储卷，多个 pod 使用同一个存储卷，它们数据是同步的，而 statefulset 定义中 的每一个 pod 都不能使用同一个存储卷，这就需要使用 volumeClainTemplate，当在使用 statefulset 创建 pod 时，volumeClainTemplate 会自动生成一个 PVC，从而请求绑定一个 PV，每一个 pod 都有自己专用的存储卷。

##### 守护进程

DaemonSet

保证在每一个 Node 上都运行一个副本容器，通常来部署一些集群的日志，监控或者其他系统管理应用。典型的应用包括：

- 日志收集
- 系统健康，比如 Prometheus Node Exporter
- 系统程序，比如 kube-proxy,kube-dns,ceph

##### 定时任务

job

一次性任务，运行完成后 Pod 自动销毁，不再重新启动新的容器。

Cronjob

实在 job 基础上增加了定时功能。

##### 服务发现

Service（横向流量，东西流量）

实现 k8s 集群内部网络调用，负载均衡（四层负载）。

Ingress（纵向流量，南北流量）

实现将 k8s 内部服务暴漏给外网访问的服务，ingress-nginx（反向代理，负载均衡（七层负载））。



## 使用部署工具安装 Kubernetes

搭建你自己的 Kubernetes 生产集群有许多方法和工具。例如：

- kubeadm
- [kops](https://kops.sigs.k8s.io/)：自动化集群制备工具。 有关教程、最佳实践、配置选项和社区联系信息，请查阅 [`kOps` 网站](https://kops.sigs.k8s.io/)。
- [kubespray](https://kubespray.io/)： 提供了 [Ansible](https://docs.ansible.com/) Playbook、 [清单（inventory）](https://github.com/kubernetes-sigs/kubespray/blob/master/docs/ansible.md#inventory)、 制备工具和通用 OS/Kubernetes 集群配置管理任务领域的知识。 你可以通过 Slack 频道 [#kubespray](https://kubernetes.slack.com/messages/kubespray/) 联系此社区。
- 二进制安装

### kubeadm

#### 准备开始

- 一台兼容的 Linux 主机。Kubernetes 项目为基于 Debian 和 Red Hat 的 Linux 发行版以及一些不提供包管理器的发行版提供通用的指令。
- 每台机器 2 GB 或更多的 RAM（如果少于这个数字将会影响你应用的运行内存）。
- CPU 2 核心及以上。
- 集群中的所有机器的网络彼此均能相互连接（公网和内网都可以）。
- 节点之中不可以有重复的主机名、MAC 地址或 product_uuid。请参见[这里](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#verify-mac-address)了解更多详细信息。
- 开启机器上的某些端口。请参见[这里](https://kubernetes.io/zh-cn/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#check-required-ports)了解更多详细信息。
- 交换分区的配置。kubelet 的默认行为是在节点上检测到交换内存时无法启动。 kubelet 自 v1.22 起已开始支持交换分区。自 v1.28 起，仅针对 cgroup v2 支持交换分区； kubelet 的 NodeSwap 特性门控处于 Beta 阶段，但默认被禁用。
  - 如果 kubelet 未被正确配置使用交换分区，则你**必须**禁用交换分区。 例如，`sudo swapoff -a` 将暂时禁用交换分区。要使此更改在重启后保持不变，请确保在如 `/etc/fstab`、`systemd.swap` 等配置文件中禁用交换分区，具体取决于你的系统如何配置。

#### 实验环境

- Docker 版本：20+

- kubernetes 版本：1.29.1

- pod 网段：10.244.0.0/16
- service 网段：10.96.0.0/12

> 注意： pod 和 service 网段不可冲突，如果冲突会导致 K8S 集群安装失败。

| 主机名     | ip              | 系统       |
| ---------- | --------------- | ---------- |
| k8s-master | 192.168.113.120 | CentOS 7.9 |
| k8s-node1  | 192.168.113.121 | CentOS 7.9 |
| k8s-node2  | 192.168.113.122 | CentOS 7.9 |

#### 安装步骤

##### 1、初始操作

~~~shell
# 初始操作：每台主机都需要执行
# 1、修改主机名
hostnamectl set-hostname k8s-master
hostnamectl set-hostname k8s-node1
hostnamectl set-hostname k8s-node2

# 2、hosts 解析
cat  >>  /etc/hosts <<EOF
192.168.113.120 k8s-master 
192.168.113.121 k8s-node1 
192.168.113.122 k8s-node2
EOF

# 3、关闭防火墙，selinux,swap 分区
swapoff -a &&  sed -ri 's/.*swap.*/#&/' /etc/fstab && setenforce 0 && sed -i 's/enforcing/disabled/' /etc/selinux/config && systemctl stop firewalld && systemctl disable firewalld 

# 4、配置流量转发
cat <<EOF | tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
# 修改Linux内核参数，添加网桥过滤器和地址转发功能
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv6.conf.all.disable_ipv6=1
EOF
modprobe br_netfilter
sysctl --system   # 生效

# 5、时间同步
vi /etc/chrony.conf
# 添加
server ntp1.aliyun.com iburst		# 并将别的 server 注释掉
# 重启服务
systemctl enable --now chronyd

# 6、配置 ssh 免密
ssh-keygen 
ssh-copy-id 192.168.113.120
ssh-copy-id 192.168.113.121
ssh-copy-id 192.168.113.122
~~~



##### 2、安装基础软件（所有节点）

2.1、所有节点安装 Docker 

~~~shell
# 获取阿里的 yum 源
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
yum makecache

# yum-utils 软件用于提供 yum-config-manager 程序
yum install -y yum-utils

# 使用 yum-config-manager 添加阿里 yum 仓库
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 因为 kubernetes:v1.24 版本，k8s 从 kubelet 中删除 Dockershim
# 安装最新版的 docker 和 docker-engine
yum install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# 启动docker并设置开机自启
systemctl enable docker --now
~~~



2.2、所有节点安装并配置 cri-dockerd 插件

~~~shell
# 下载文件，下载不了，本地下载然后传到服务器中
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.1/cri-dockerd-0.3.1-3.el7.x86_64.rpm
rpm -ivh cri-dockerd-0.3.1-3.el7.x86_64.rpm

# 备份并更新 cri-docker.service 文件
mv /usr/lib/systemd/system/cri-docker.service{,.default}

cat > /usr/lib/systemd/system/cri-docker.service << EOF
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket
[Service]
Type=notify
ExecStart=/usr/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF

# 启动cir-dockerd
systemctl daemon-reload
systemctl enable --now cri-docker.service 
~~~



2.3、所有节点安装 kubeadm，kubelet，kubectl

阿里镜像站：[kubernetes-new-core-stable安装包下载_开源镜像站-阿里云 (aliyun.com)](https://mirrors.aliyun.com/kubernetes-new/core/stable/?spm=a2c6h.25603864.0.0.76a836d2IJIawd)

官方：[在 Linux 系统中安装并设置 kubectl | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/tools/install-kubectl-linux/)

~~~shell
# 本地下载，然后下载到服务器里
[root@k8s-master kubernetes]# ll
total 55036
-rw-r--r--. 1 root root  8604404 Jan 30 17:03 cri-tools-1.29.0-150500.1.1.x86_64.rpm
-rw-r--r--. 1 root root 10197544 Jan 30 17:03 kubeadm-1.29.1-150500.1.1.x86_64.rpm
-rw-r--r--. 1 root root 10604400 Jan 30 17:03 kubectl-1.29.1-150500.1.1.x86_64.rpm
-rw-r--r--. 1 root root 19943436 Jan 30 17:03 kubelet-1.29.1-150500.1.1.x86_64.rpm
-rw-r--r--. 1 root root  6998912 Jan 30 17:03 kubernetes-cni-1.3.0-150500.1.1.x86_64.rpm

[root@k8s-master kubernetes]# yum localinstall -y *

# 启动 kubelet
systemctl enable --now kubelet

# 其他节点也要安装
scp -rp /root/kubernetes/ k8s-node1:/root
scp -rp /root/kubernetes/ k8s-node2:/root
ssh k8s-node1 "yum localinstall -y kubernetes/*"
ssh k8s-node2 "yum localinstall -y kubernetes/*"
~~~



2.4、配置 cgroup

Linux系统每个进程都可以自由竞争系统资源，有时候会导致一些次要进程占用了系统某个资源（如CPU）的绝大部分，主要进程就不能很好地执行，从而影响系统效率，重则在linux资源耗尽时可能会引起错杀进程。因此linux引入了linux cgroups来控制进程资源，让进程更可控。

kubeadm 支持在执行 `kubeadm init` 时，传递一个 `KubeletConfiguration` 结构体。 `KubeletConfiguration` 包含 `cgroupDriver` 字段，可用于控制 kubelet 的 cgroup 驱动。

这是一个最小化的示例，其中显式的配置了此字段：

```yaml
# kubeadm-config.yaml
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
kubernetesVersion: v1.21.0
---
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
cgroupDriver: systemd
```



这样一个配置文件就可以传递给 kubeadm 命令了：

~~~shell
kubeadm init --config kubeadm-config.yaml
~~~



> 警告：你需要确保容器运行时和 kubelet 所使用的是相同的 cgroup 驱动，否则 kubelet 进程会失败。

##### 3、部署 Kubernetes Master

1. 生成初始化默认配置文件

   ~~~shell
   [root@k8s-master ~]# kubeadm config print init-defaults > init.yaml
   ~~~

   根据需求修改配置文件，我修改的配置如下：

   - advertiseAddress：修改为 master 节点的 ip
   - name：hosts 解析里 master 节点的名称
   - criSocket：修改为 cri-dockerd 的 sock
   - imageRepository：配置国内加速源地址
   - kubernetesVersion：集群的版本
   - podSubnet：pod 的网段地址
   - serviceSubnet：service 的网段地址

   最终初始化文件如下：

   ~~~yaml
   apiVersion: kubeadm.k8s.io/v1beta3
   bootstrapTokens:
   - groups:
     - system:bootstrappers:kubeadm:default-node-token
     token: abcdef.0123456789abcdef
     ttl: 24h0m0s
     usages:
     - signing
     - authentication
   kind: InitConfiguration
   localAPIEndpoint:
     advertiseAddress: 192.168.113.120
     bindPort: 6443
   nodeRegistration:
     criSocket: unix:///var/run/cri-dockerd.sock
     imagePullPolicy: IfNotPresent
     name: k8s-master
     taints: null
   ---
   apiServer:
     timeoutForControlPlane: 4m0s
   apiVersion: kubeadm.k8s.io/v1beta3
   certificatesDir: /etc/kubernetes/pki
   clusterName: kubernetes
   controllerManager: {}
   dns: {}
   etcd:
     local:
       dataDir: /var/lib/etcd
   imageRepository: registry.aliyuncs.com/google_containers
   kind: ClusterConfiguration
   kubernetesVersion: 1.29.1
   networking:
     dnsDomain: cluster.local
     podSubnet: 10.244.0.0/16
     serviceSubnet: 10.96.0.0/12
   scheduler: {}
   ~~~

   

2. 初始化

   ~~~shell
   kubeadm init --config=init.yaml --ignore-preflight-errors=all
   
   # 初始化失败，执行这条命令
   kubeadm reset
   ~~~

   初始化成功，输出如下内容：

   ~~~shell
   [root@k8s-master ~]# kubeadm init --config=init.yaml --ignore-preflight-errors=all
   [init] Using Kubernetes version: v1.29.1
   [preflight] Running pre-flight checks
           [WARNING Port-6443]: Port 6443 is in use
           [WARNING Port-10259]: Port 10259 is in use
           [WARNING Port-10257]: Port 10257 is in use
           [WARNING FileAvailable--etc-kubernetes-manifests-kube-apiserver.yaml]: /etc/kubernetes/manifests/kube-apiserver.yaml already exists
           [WARNING FileAvailable--etc-kubernetes-manifests-kube-controller-manager.yaml]: /etc/kubernetes/manifests/kube-controller-manager.yaml already exists
           [WARNING FileAvailable--etc-kubernetes-manifests-kube-scheduler.yaml]: /etc/kubernetes/manifests/kube-scheduler.yaml already exists
           [WARNING FileAvailable--etc-kubernetes-manifests-etcd.yaml]: /etc/kubernetes/manifests/etcd.yaml already exists
           [WARNING Port-10250]: Port 10250 is in use
           [WARNING Port-2379]: Port 2379 is in use
           [WARNING Port-2380]: Port 2380 is in use
           [WARNING DirAvailable--var-lib-etcd]: /var/lib/etcd is not empty
   [preflight] Pulling images required for setting up a Kubernetes cluster
   [preflight] This might take a minute or two, depending on the speed of your internet connection
   [preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
   [certs] Using certificateDir folder "/etc/kubernetes/pki"
   [certs] Using existing ca certificate authority
   [certs] Using existing apiserver certificate and key on disk
   [certs] Using existing apiserver-kubelet-client certificate and key on disk
   [certs] Using existing front-proxy-ca certificate authority
   [certs] Using existing front-proxy-client certificate and key on disk
   [certs] Using existing etcd/ca certificate authority
   [certs] Using existing etcd/server certificate and key on disk
   [certs] Using existing etcd/peer certificate and key on disk
   [certs] Using existing etcd/healthcheck-client certificate and key on disk
   [certs] Using existing apiserver-etcd-client certificate and key on disk
   [certs] Using the existing "sa" key
   [kubeconfig] Using kubeconfig folder "/etc/kubernetes"
   [kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/admin.conf"
   [kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/super-admin.conf"
   [kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/kubelet.conf"
   [kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/controller-manager.conf"
   [kubeconfig] Using existing kubeconfig file: "/etc/kubernetes/scheduler.conf"
   [etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
   [control-plane] Using manifest folder "/etc/kubernetes/manifests"
   [control-plane] Creating static Pod manifest for "kube-apiserver"
   [control-plane] Creating static Pod manifest for "kube-controller-manager"
   [control-plane] Creating static Pod manifest for "kube-scheduler"
   [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
   [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
   [kubelet-start] Starting the kubelet
   [wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
   [apiclient] All control plane components are healthy after 0.020574 seconds
   [upload-config] Storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
   [kubelet] Creating a ConfigMap "kubelet-config" in namespace kube-system with the configuration for the kubelets in the cluster
   [upload-certs] Skipping phase. Please see --upload-certs
   [mark-control-plane] Marking the node k8s-master as control-plane by adding the labels: [node-role.kubernetes.io/control-plane node.kubernetes.io/exclude-from-external-load-balancers]
   [mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/control-plane:NoSchedule]
   [bootstrap-token] Using token: abcdef.0123456789abcdef
   [bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
   [bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to get nodes
   [bootstrap-token] Configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
   [bootstrap-token] Configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
   [bootstrap-token] Configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
   [bootstrap-token] Creating the "cluster-info" ConfigMap in the "kube-public" namespace
   [kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
   [addons] Applied essential addon: CoreDNS
   [addons] Applied essential addon: kube-proxy
   
   Your Kubernetes control-plane has initialized successfully!
   
   To start using your cluster, you need to run the following as a regular user:
   
     mkdir -p $HOME/.kube
     sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
     sudo chown $(id -u):$(id -g) $HOME/.kube/config
   
   Alternatively, if you are the root user, you can run:
   
     export KUBECONFIG=/etc/kubernetes/admin.conf
   
   You should now deploy a pod network to the cluster.
   Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
     https://kubernetes.io/docs/concepts/cluster-administration/addons/
   
   Then you can join any number of worker nodes by running the following on each as root:
   
   kubeadm join 192.168.113.120:6443 --token abcdef.0123456789abcdef \
           --discovery-token-ca-cert-hash sha256:7f290025c2d57e6de1c88c6c3aefa67552aa73a92fd05c0e10104a13cfd0701d 
   ~~~

   

3. 配置kubectl的配置文件config，相当于对kubectl进行授权，这样kubectl命令可以使用这个证书对k8s集群进行管理

   ~~~shell
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ~~~

   

   验证使用可以使用 kubectl 命令。
   
   ```shell
   [root@k8s-master ~]# kubectl get nodes
   NAME         STATUS     ROLES           AGE    VERSION
   k8s-master   NotReady   control-plane   127m   v1.29.1
   ```



##### 4、加入 Kubernetes Node

~~~shell
# 之前的 token 忘了，可以重新生成 token
[root@k8s-master ~]# kubeadm token create --print-join-command

# 在两台node节点执行，注意添加--cri-socket=指定cri-dockerd.sock。
kubeadm join 192.168.113.120:6443 --token abcdef.0123456789abcdef \
        --discovery-token-ca-cert-hash sha256:7f290025c2d57e6de1c88c6c3aefa67552aa73a92fd05c0e10104a13cfd0701d --cri-socket=unix:///var/run/cri-dockerd.sock
~~~

成功加入到集群如下图：

~~~shell
[root@k8s-node1 ~]# kubeadm join 192.168.113.120:6443 --token s8tpf1.5cfm880ckoby8dxc --discovery-token-ca-cert-hash sha256:7f290025c2d57e6de1c88c6c3aefa67552aa73a92fd05c0e10104a13cfd0701d --cri-socket=unix:///var/run/cri-dockerd.sock
[preflight] Running pre-flight checks
        [WARNING Service-Kubelet]: kubelet service is not enabled, please run 'systemctl enable kubelet.service'
[preflight] Reading configuration from the cluster... 
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
~~~



给两台node节点打上标签

~~~shell
[root@k8s-master ~]# kubectl label nodes k8s-node1 node-role.kubernetes.io/work=work
node/k8s-node1 labeled
[root@k8s-master ~]# kubectl label nodes k8s-node2 node-role.kubernetes.io/work=work
node/k8s-node2 labeled

# 查看集群节点
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     ROLES           AGE     VERSION
k8s-master   NotReady   control-plane   138m    v1.29.1
k8s-node1    NotReady   work            5m37s   v1.29.1
k8s-node2    NotReady   work            5m29s   v1.29.1
~~~





##### 5、部署 CNI 网络插件

Calico 在线文档地址：[raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml](https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml)

~~~shell
# 上传到服务器
[root@k8s-master ~]# curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

# 创建 Calico
[root@k8s-master ~]# kubectl apply -f calico.yaml 

# 再次查看群集节点的状态
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS   ROLES           AGE    VERSION
k8s-master   Ready    control-plane   5h6m   v1.29.1
k8s-node1    Ready    work            172m   v1.29.1
k8s-node2    Ready    work            172m   v1.29.1
~~~



##### 6、测试 Kubernetes 集群

1. 下载 busybox:1.28 镜像

   ~~~shell
   docker pull busybox:1.28
   ~~~

   

2. 测试 coredns

   ~~~shell
   kubectl run busybox -it --image busybox:1.28  busybox -- sh
   If you don't see a command prompt, try pressing enter.
   / #  nslookup kubernetes
   Server:    10.96.0.10
   Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local
   
   Name:      kubernetes
   Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
   ~~~

   

> - 注意：busybox 要用指定的 1.28 版本，不能用最新版本，最新版本，nslookup 会解析不到 dns 和 ip。

### minikube

[minikube](https://minikube.sigs.k8s.io/) 是一个轻量级的kubernetes集群环境，可以用来在本地快速搭建一个单节点的kubernetes集群。

#### 安装 minikube

~~~shell
# Linux
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube
~~~



#### 启动 minikube

~~~shell
# 启动minikube
minikube start
~~~



## CRI

CRI：容器接口规范，其主要功能就是启动和停止容器的组件叫做容器运行时（Container Runtime），因为pod中运行的容器，都是要经过kubelet的兼容的，但是要是每一种容器都要自己手动开发kubelet太麻烦，所以kubernetes就加入了容器运行时插件API，即Container Runtime Interface。

Cgroup:Cgrep和namespace类似，也是将进程分组，但是目的与namespace不一样，namespace是为了隔离进程组之前的资源，而Cgroup是为了对一组进程进行统一的资源监控和限制。

## api 访问控制

用户

- 有鉴于此，**Kubernetes 并不包含用来代表普通用户账号的对象**。 普通用户的信息无法通过 API 调用添加到集群中。
- 与此不同，服务账号是 Kubernetes API 所管理的用户。它们被绑定到特定的名字空间， 或者由 API 服务器自动创建，或者通过 API 调用创建。服务账号与一组以 Secret 保存的凭据相关，这些凭据会被挂载到 Pod 中，从而允许集群内的进程访问 Kubernetes API。

身份认证策略

- Kubernetes 通过身份认证插件利用客户端证书、持有者令牌（Bearer Token）或身份认证代理（Proxy） 来认证 API 请求的身份。
- 当集群中启用了多个身份认证模块时，第一个成功地对请求完成身份认证的模块会直接做出评估决定。 API 服务器并不保证身份认证模块的运行顺序。

Kubernetes API 的访问控制

用户使用 `kubectl`、客户端库或构造 REST 请求来访问 [Kubernetes API](https://v1-30.docs.kubernetes.io/zh-cn/docs/concepts/overview/kubernetes-api/)。 人类用户和 [Kubernetes 服务账号](https://v1-30.docs.kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-service-account/)都可以被鉴权访问 API。 当请求到达 API 时，它会经历多个阶段，如下图所示：

![image-20240818160602962](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240818160602962.png)

1. 传输安全：默认情况下，Kubernetes API 服务器在第一个非 localhost 网络接口的 6443 端口上进行监听， 受 TLS 保护。在一个典型的 Kubernetes 生产集群中，API 使用 443 端口。 该端口可以通过 `--secure-port` 进行变更，监听 IP 地址可以通过 `--bind-address` 标志进行变更。

2. 认证：如上图的步骤 **1** 所示，建立 TLS 后， HTTP 请求将进入认证（Authentication）步骤。 集群创建脚本或者集群管理员配置 API 服务器，使之运行一个或多个身份认证组件。 身份认证组件在[认证](https://v1-30.docs.kubernetes.io/zh-cn/docs/reference/access-authn-authz/authentication/)节中有更详细的描述。

3. 鉴权：如上图的步骤 **2** 所示，将请求验证为来自特定的用户后，请求必须被鉴权。请求必须包含请求者的用户名、请求的行为以及受该操作影响的对象。 如果现有策略声明用户有权完成请求的操作，那么该请求被鉴权通过。比如，小明用户只能在 projectCaribou 名称空间中读取 Pod，他如果访问别的命名空间的话，其鉴权请求将被拒绝。

4. 准入控制：这一操作如上图的步骤 **3** 所示。准入控制模块是可以修改或拒绝请求的软件模块。 除鉴权模块可用的属性外，准入控制模块还可以访问正在创建或修改的对象的内容。准入控制器对创建、修改、删除或（通过代理）连接对象的请求进行操作。 准入控制器不会对仅读取对象的请求起作用。 有多个准入控制器被配置时，服务器将依次调用它们。

   与身份认证和鉴权模块不同，如果任何准入控制器模块拒绝某请求，则该请求将立即被拒绝。

   请求通过所有准入控制器后，将使用检验例程检查对应的 API 对象，然后将其写入对象存储（如步骤 **4** 所示）。

5. 审计：Kubernetes 审计提供了一套与安全相关的、按时间顺序排列的记录，其中记录了集群中的操作序列。 集群对用户、使用 Kubernetes API 的应用程序以及控制平面本身产生的活动进行审计。



### 使用 RBAC 鉴权

基于角色（Role）的访问控制（RBAC）是一种基于组织中用户的角色来调节控制对计算机或网络资源的访问的方法。

RBAC 鉴权机制使用 `rbac.authorization.k8s.io` [API 组](https://v1-30.docs.kubernetes.io/zh-cn/docs/concepts/overview/kubernetes-api/#api-groups-and-versioning)来驱动鉴权决定， 允许你通过 Kubernetes API 动态配置策略。

要启用 RBAC，在启动 [API 服务器](https://v1-30.docs.kubernetes.io/zh-cn/docs/concepts/overview/components/#kube-apiserver)时将 `--authorization-mode` 参数设置为一个逗号分隔的列表并确保其中包含 `RBAC`。

```shell
kube-apiserver --authorization-mode=Example,RBAC --<其他选项> --<其他选项>
```



### Webhook 模式

WebHook 是一种 HTTP 回调：某些条件下触发的 HTTP POST 请求；通过 HTTP POST 发送的简单事件通知。一个基于 web 应用实现的 WebHook 会在特定事件发生时把消息发送给特定的 URL。

具体来说，当在判断用户权限时，`Webhook` 模式会使 Kubernetes 查询外部的 REST 服务。

`Webhook` 模式需要一个 HTTP 配置文件，通过 `--authorization-webhook-config-file=SOME_FILENAME` 的参数声明。

使用 HTTPS 客户端认证的配置例子：

```yaml
# Kubernetes API 版本
apiVersion: v1
# API 对象种类
kind: Config
# clusters 代表远程服务。
clusters:
  - name: name-of-remote-authz-service
    cluster:
      # 对远程服务进行身份认证的 CA。
      certificate-authority: /path/to/ca.pem
      # 远程服务的查询 URL。必须使用 'https'。
      server: https://authz.example.com/authorize

# users 代表 API 服务器的 webhook 配置
users:
  - name: name-of-api-server
    user:
      client-certificate: /path/to/cert.pem # webhook plugin 使用 cert
      client-key: /path/to/key.pem          # cert 所对应的 key

# kubeconfig 文件必须有 context。需要提供一个给 API 服务器。
current-context: webhook
contexts:
- context:
    cluster: name-of-remote-authz-service
    user: name-of-api-server
  name: webhook
```







### 准入控制器（Admission Controller）









## 深入了解 pod

### 1>:在 kubernetes 中的 pod 有一个要求

pod中的容器其主程序必须一直在前台运行，如果pod中的容器主程序停止了前台运行，则系统会监控到该pod已经终止，并销毁该pod。这就是kubernetes需要我们自己创建的Docker镜像并以一个前台命令作为启动命令的原因。

### 2>:静态 pod

类似于docker自己启动的容器，kubelet无法对他们进行健康检查，虽然pod是kubelet自己启动的。静态pod始终绑定在某个kubelet,并且始终运行在同一个节点上。
  静态Pod适用于以下场景：
    实验性质的工作负载：比如在集群外部或者边缘节点部署简单的容器应用，不需要Kubernetes的高级功能。
    特殊需求的工作负载：比如有些应用可能需要以特定的方式启动或运行，而无法通过普通Pod的定义文件来满足需求。
    对于kubelet结点的部分监控或管理：可以使用静态Pod在特定节点上运行一些监控或管理的容器应用，用来观察或操作该节点。

### 3>:在容器内获取 pod 信息（Downward API）

对于容器来说，在不与 Kubernetes 过度耦合的情况下，拥有关于自身的信息有时是很有用的。 **Downward API** 允许容器在不使用 Kubernetes 客户端或 API 服务器的情况下获得自己或集群的信息。

在 Kubernetes 中，有两种方法可以将 Pod 和容器字段暴露给运行中的容器：

  环境变量方式：
    spec.containers.env.valueFrom
  Volume挂载方式：
    spec.volumes.downwardAPI

### 4>:pod 生命周期和重启策略

#### 4.1>：pod的状态

-   pending：api server已经创建该pod，但在pod内还有一个或多个容器没有被创建，包括正在下载镜像的过程。
-   running：pod内所有容器均已创建，且至少有一个容器正在运行状态、正在启动状态或正在重启状态。
-   Succeeded：pod内所有容器均成功执行后退出，且不会再重启。
-   Failed：pod内所有容器均已退出，但至少有一个容器退出为失败状态。
-   Unknown：由于某种原因无法获取该pod的状态，可能由于网络通信不畅导致。

#### 4.2>：pod的重启策略

`spec.restartPolicy`

-   Always：pod 一旦终止运行，则无论容器是如何终止的，kubelet都会自动重启该容器。 
-   OnFailure：当容器终止运行且退出码不为0时，运行错误了，kubelet会自动重启该容器。
-   Never：不论容器运行状态如何，kubelet都不会重启该容器，只会讲退出代码报告给 master，不会重启该 pod。

### 5>:pod 健康检查和服务可用性检查

#### 5.1>:探针

StartupProbe（启动探针）：用于判断容器是否启动，如果容器启动完成了，LivenessProbe和ReadinessProbe才可以运行自己的工作，如果没有，则反之。
LivenessProbe（存活探针）：用于判断容器是否存活（Running状态），如果探针探测到容器不健康，则kubelet会根据pod的重启状态把容器删除。
ReadinessProbe（就绪探针）：判断容器服务是否可用（Ready状态），达到Ready状态的pod才可以接受请求，用户才能发送请求。
探测方式：
ExecAction：通过执行命令类如”cat /tmp/health“命令来判断容器是否正常。

~~~shell
# spec.containers.livenessProbe
livenessProbe
  exec:
    command:
    - cat 
    - /health
~~~



TCPSocketAction：通过容器的ip地址和端口号执行TCP检查，如果能建立TCP连接，则表明容器健康。
~~~shell
# spec.containers.livenessProbe
livenessProbe:
  tcpSocket:
    port: 80
~~~



HTTPGetAction：通过容器的IP地址，端口号及路径调用HTTP Get方式，如果相应的状态码大于等于200且小于400，则认为容器健康。

~~~shell
# spec.containers.livenessProbe
livenessProbe:
  httpGet:
    path: /health
    port: 80
    scheme: HTTP
    httpHeaders:           # 请求头
    - name: xxx
      values: xxx
~~~



参数配置：
initialDelaySconds: 60   # 初始化时间，单位为秒
timeoutSeconds: 2   # 超时时间，单位为秒
periodSeconds: 1   # 检测间隔时间
successThreshold: 1   # 检查1次成功就表示成功
failureThreshold: 2   # 检测失败2次就表示失败

#### 5.2>:pod 生命周期

首先，容器环境会执行初始化操作，会有一个或者多个初始化容器进行运行，运行完成，pod内的著容器Main Container 就会开始一系列的开始操作，首先会启动postStart钩子函数，紧接着是Startup启动探针，然后Readiness就绪探针和Liveness探针开始工作，检查启动是否成功，再容器启动的过程中，会检测容器是否存活，如果当前容器挂掉了，Liveness会根据用户设置的重启策略决定是否需要重启，随后preStop钩子函数完成结尾工作，这就是整个pod的生命周期。

StartUp钩子函数：pod主容器启动起来需要做的工作。

~~~shell
# StartUp钩子函数
# spec.containers.lifecycle.portStart
lifecycle:
  postStart:
    exec:
      command:
        - sh
        - -c 
        - "echo 'hello world!' > /file"
~~~



preStop钩子函数：pod主容器结束最后要做的结尾工作。

~~~shell
# preStop钩子函数
# spec.containers.lifecycle.preStop
lifecycle:
  preStop:
    exec:
      command:
        - sh
        - -c 
        - "echo 'sleep 50; echo 'sleep finished...' >> /file"
~~~



### 6>:pod 调度

在k8s里的master节点上的Scheduler服务来实现对pod的调度，在调度过程中通过执行一系列复杂的算法，最终为每个pod都计算出一个最佳的目标节点，这一过程时自动完成的。

#### 6.1>:标签（lable）

~~~shell
# metadata.labels
labels:
  type: xxx 	# 自定义label标签，名字为type，值为xxx
  test: xxx		

# spec.selector.matchLabels
matchLabels:
  app: nginx
  
  
# kubelet 命令
# 临时添加
kubectl label po <资源名称> app=hello -n kube-public
# 修改
kubectl label po <资源名称> app=hello --overwirte
# 查看
kubectl get po -A -l app=hello
kubectl get po -n kube-public --show-labels

# 小知识：在修改deployment,service等控制器时，不像 pod 一样，它们需要在配置文件里面修改，因为它们会随时更新模板，保证可用性。
~~~



#### 6.2>:选择器（Selector）

在各对象的配置sp ec.selector或其他可以写selector的属性中编写。

~~~shell
# kubelet 命令
# 单值匹配，查找 app=hello 的pod
kubectl get po -A -l app=hello

# 匹配多值
kubectl get po -A -l 'k8s-app in (1.2.0,1.2.1)'

# 查找
kubectl get po -l -A type=app --show-labels
kubectl get po -l 'test in (1.0.0,1.0.1)' --show-labels
kubectl get po -l version!=1.2.0
~~~



#### 6.3>:定向调度

在实际情况中，我们可能也需要将pod调度到指定的一些node上，可以通过node的标签和pod的nodeSelector属性相匹配，来达到上述目的。

~~~shell
# 1.给node打上标签
kubectl label nodes k8s-node-1 app=ws
# 2.在pod的定义中加上nodeSelector的设置
spec:
  nodeSelector:
    app: ws
    
# 小知识:如果在群集中，如果指定了pod的nodeSelector设置，且在群集中不存在包含相应标签的node，即使还有其他可供使用的node，这个pod也不会被完成调度。
~~~



#### 6.4>:node 亲和性调度（NodeAffinity）

亲和性和污点的概念正好相反，亲和性是尽可能靠拢，而污点是尽可能的远离，当然亲和性和污点同时使用会出现更细致的调度。

- 从 node 出发，也可以分成亲和性和反亲和性，分别对应 nodeAffinity 和 nodeAntiAffinity

~~~shell
# pod.spec.nodeAffinity 节点亲和性有两种：
# 1.软策略：调度器只有在规则被满足的时候才能执行调度。此功能类似于 nodeSelector， 但其语法表达能力更强。
requiredDuringSchedulingIgnoredDuringExecution
# 2.硬策略：调度器会尝试寻找满足对应规则的节点。如果找不到匹配的节点，调度器仍然会调度该 Pod。
preferredDuringSchedulingIgnoredDuringExecution：
~~~



配置示例

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:		 # 定义亲和性
    nodeAffinity:	# 定义 node 亲和性
      requiredDuringSchedulingIgnoredDuringExecution:		# 硬亲和力
        nodeSelectorTerms:		# 节点选择器术语列表
        - matchExpressions:		# 按节点标签列出的节点选择器需求列表
          - key: topology.kubernetes.io/zone		# 选择器应用于的标签键
            operator: In		# 表示键与一组值的关系。有效的操作符是In，NotIn, Exists, DoesNotExist,Gt和Lt。
            values:
            - antarctica-east1
            - antarctica-west1
      preferredDuringSchedulingIgnoredDuringExecution:		# 软亲和力
      - weight: 1		# 如果有多个软策略选项的话，权重越大，优先级越高
        preference:		# 一个节点选择器项，与相应的权重相关联
          matchExpressions:		
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: registry.k8s.io/pause:2.0
~~~



##### operator（运算符） 匹配类型

- In：满足一个就行
- NotIn：一个都不能满足，反亲和性
- Exists：只要存在，就满足
- DoesNotExist：只有不存在，才满足
- Gt：必须要大于节点的数值才满足
- Lt：必须小于节点的数值才满足

#### 6.5>:pod 亲和性（PodAffinity）和反亲和性（PodAntiAffinity）

Pod 间亲和性与反亲和性使你可以基于已经在节点上运行的 Pod 的标签来约束 Pod 可以调度到的节点，而不是基于节点上的标签。

- 从 pod 出发，可以分成亲和性和反亲和性，分别对应 podAffinity 和 podAntiAffinity

~~~shell
# pod.spec.affinity.podAffinity  pod亲和性有两种：
# 1.软策略：调度器会尝试寻找满足对应规则的 pod。如果找不到匹配的 pod，调度器仍然会调度该 Pod。
preferredDuringSchedulingIgnoredDuringExecution

# 2.硬策略：调度器只有在规则被满足的时候才能执行调度，意思就是如果和别的 pod 的亲和性规则匹配到了，就必须调度到一起。
requiredDuringSchedulingIgnoredDuringExecution 

# pod.spec.affinity.podAntiAffinity pod反亲和性有两种：
# 1.软策略：意思是亲和性匹配到了，尽量不要调度到一起。
preferredDuringSchedulingIgnoredDuringExecution

# 2.硬策略：调度器只有在规则被满足的时候才能执行调度，意思是与别的 pod 的亲和性匹配到一起了，就必须远离被匹配的 pod，然后进行调度。
requiredDuringSchedulingIgnoredDuringExecution
~~~



示例

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:		# 硬亲和力
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: topology.kubernetes.io/zone		# 节点对应的标签
    podAntiAffinity:		# pod 反亲和力
      preferredDuringSchedulingIgnoredDuringExecution:		# 软亲和力
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: topology.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: registry.k8s.io/pause:2.0
~~~



#### 6.6>:污点和容忍

Taint 与 NodeAffinity 相反，它是让 Node 拒绝 Pod 的运行。

##### 污点（Taint）

- NoSchedule：节点不可调度，那么 Pod 就无法调度到该节点上了。

- NoExecute：
  - 如果 Pod 不能容忍这类污点，会马上被驱逐。
  - 如果 Pod 能够容忍这类污点，但是在容忍度定义中没有指定 `tolerationSeconds`， 则 Pod 还会一直在这个节点上运行。
  - 如果 Pod 能够容忍这类污点，而且指定了 `tolerationSeconds`， 则 Pod 还能在这个节点上继续运行这个指定的时间长度。 这段时间过去后，节点生命周期控制器从节点驱除这些 Pod。

~~~shell
# 设置污点
kubectl taint nodes nodes1 key=values:NoSchedule
# 删除污点
kubectl taint nodes nodes1 key=values:NoSchedule-
# 查看污点
kubectl describe nodes nodes1 | grep -i taint
~~~

##### 容忍（Toleration）

- Equal：比较操作类型为 Equal，则意味着必须与污点值做匹配，key/value 都必须相同，才表示能够容忍该污点。
- Exists：容忍与污点的比较，我们只比较 key，不比较 value，不关心 value 是什么东西，只要 key 存在，就表示可以容忍。

~~~shell
# 容忍污点，需要在 pod 上声明 Toleration
pod.spec.tolerations
tolerations:
  - key: "key"
    operator: "Equal"
    value: "value"
    effect: "NoSchedule" 
    
    
# 比如，你可能希望在出现网络分裂事件时，对于一个与节点本地状态有着深度绑定的应用而言， 仍然停留在当前节点上运行一段较长的时间，以等待网络恢复以避免被驱逐。 你为这种 Pod 所设置的容忍度看起来可能是这样：

tolerations:
- key: "node.kubernetes.io/unreachable"
  operator: "Exists"
  effect: "NoExecute"
  tolerationSeconds: 6000			# 该 pod 还能继续在该节点运行 6000 秒，时间到了，还会重新调度
~~~



### 7>:initContainer(初始化容器)

Init 容器的概览：Init 容器是一种特殊容器，在 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 内的应用容器启动之前运行。Init 容器可以包括一些应用镜像中不存在的实用工具和安装脚本。

每个 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 中可以包含多个容器， 应用运行在这些容器里面，同时 Pod 也可以有一个或多个先于应用容器启动的 Init 容器。

Init 容器与普通的容器非常像，除了如下两点：

- 它们总是运行到完成。
- 每个都必须在下一个启动之前成功完成。

如果 Pod 的 Init 容器失败，kubelet 会不断地重启该 Init 容器直到该容器成功为止。 然而，如果 Pod 对应的 `restartPolicy` 值为 "Never"，并且 Pod 的 Init 容器失败， 则 Kubernetes 会将整个 Pod 状态设置为失败。

为 Pod 设置 Init 容器需要在 [Pod 规约](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/pod-v1/#PodSpec)中添加 `initContainers` 字段， 该字段以 [Container](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#container-v1-core) 类型对象数组的形式组织，和应用的 `containers` 数组同级相邻。 参阅 API 参考的[容器](https://kubernetes.io/zh-cn/docs/reference/kubernetes-api/workload-resources/pod-v1/#Container)章节了解详情。

Init 容器的状态在 `status.initContainerStatuses` 字段中以容器状态数组的格式返回 （类似 `status.containerStatuses` 字段）。



示例:

~~~shell
# spec.initContainers
spec:
  initContainers:
    - name: xxx
      image: busybox
      command: 
      - sh
      - -c 
      - 'sleep 10;echo "initd" >> /.init'
~~~



### 8>:pod 安全上下文

安全上下文（Security Context）定义 Pod 或 Container 的特权与访问控制设置。 安全上下文包括但不限于：

- 自主访问控制（Discretionary Access Control）： 基于[用户 ID（UID）和组 ID（GID）](https://wiki.archlinux.org/index.php/users_and_groups) 来判定对对象（例如文件）的访问权限。
- [安全性增强的 Linux（SELinux）](https://zh.wikipedia.org/wiki/安全增强式Linux)： 为对象赋予安全性标签。
- 以特权模式或者非特权模式运行。
- [Linux 权能](https://linux-audit.com/linux-capabilities-hardening-linux-binaries-by-removing-setuid/): 为进程赋予 root 用户的部分特权而非全部特权。

- [AppArmor](https://kubernetes.io/zh-cn/docs/tutorials/security/apparmor/)：使用程序配置来限制个别程序的权能。
- [Seccomp](https://kubernetes.io/zh-cn/docs/tutorials/security/seccomp/)：过滤进程的系统调用。
- `allowPrivilegeEscalation`：控制进程是否可以获得超出其父进程的特权。 此布尔值直接控制是否为容器进程设置 [`no_new_privs`](https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt)标志。 当容器满足一下条件之一时，`allowPrivilegeEscalation` 总是为 true：
  - 以特权模式运行，或者
  - 具有 `CAP_SYS_ADMIN` 权能
- `readOnlyRootFilesystem`：以只读方式加载容器的根文件系统。

以上条目不是安全上下文设置的完整列表 -- 请参阅 [SecurityContext](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#securitycontext-v1-core) 了解其完整列表。



#### 8.1>:为 Pod 配置卷访问权限和属主变更策略

**特性状态：** `Kubernetes v1.23 [stable]`

默认情况下，Kubernetes 在挂载一个卷时，会递归地更改每个卷中的内容的属主和访问权限， 使之与 Pod 的 `securityContext` 中指定的 `fsGroup` 匹配。 对于较大的数据卷，检查和变更属主与访问权限可能会花费很长时间，降低 Pod 启动速度。 你可以在 `securityContext` 中使用 `fsGroupChangePolicy` 字段来控制 Kubernetes 检查和管理卷属主和访问权限的方式。

**fsGroupChangePolicy** - `fsGroupChangePolicy` 定义在卷被暴露给 Pod 内部之前对其 内容的属主和访问许可进行变更的行为。此字段仅适用于那些支持使用 `fsGroup` 来 控制属主与访问权限的卷类型。此字段的取值可以是：

- `OnRootMismatch`：只有根目录的属主与访问权限与卷所期望的权限不一致时， 才改变其中内容的属主和访问权限。这一设置有助于缩短更改卷的属主与访问 权限所需要的时间。
- `Always`：在挂载卷时总是更改卷中内容的属主和访问权限。

例如：

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 3000
  fsGroup: 2000
  fsGroupChangePolicy: "OnRootMismatch"
```

> **说明：** 此字段对于 [`secret`](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#secret)、 [`configMap`](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#configmap) 和 [`emptydir`](https://kubernetes.io/zh-cn/docs/concepts/storage/volumes/#emptydir) 这类临时性存储无效。



为 Pod 设置安全性上下文

pod 和容器的安全策略可以在 pod 的 spec 或 container 的 securityContext 字段中进行设置，如果在 pod 和 conatiner 级别都设置了相同的安全类型字段，容器将使用 container 级别的设置。

要为 Pod 设置安全性设置，可在 Pod 规约中包含 `securityContext` 字段。`securityContext` 字段值是一个 [PodSecurityContext](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#podsecuritycontext-v1-core) 对象。你为 Pod 所设置的安全性配置会应用到 Pod 中所有 Container 上。 下面是一个 Pod 的配置文件，该 Pod 定义了 `securityContext` 和一个 `emptyDir` 卷：

~~~shell
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsUser: 1000		# 容器内的进程都使用用户 ID 1000 来运行
    runAsGroup: 3000	# 所有容器中的进程都以主组 ID 3000 来运行
    fsGroup: 2000		# 容器中所有进程也会是附组 ID 2000 的一部分,卷 /data/demo 及在该卷中创建的任何文件的属主都会是组 ID 2000
  volumes:
  - name: sec-ctx-vol
    emptyDir: {}
  containers:
  - name: sec-ctx-demo
    image: busybox:1.28
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      privileged: true		# 是否 pod 以特权模式运行
    volumeMounts:
    - name: sec-ctx-vol
      mountPath: /data/demo
    securityContext:
      allowPrivilegeEscalation: false
~~~



### 9>:配置文件

~~~yaml
apiVersion: v1		# api 文档版本
kind: Pod		# 资源对象类型
metadata:		# pod 相关的元数据，用于
  labels:		# 定义 pod 的标签
    app: nginx-deploy		# 具体的 key/values，key=app,value=nginx-deploy
    test: test1
  name: nginx		# pod 的名称
  namespace: ws			# 所在的命名空间
spec:		# 期望 pod 按照这里的描述进行创建
  containers:		# 对于 pod 中容器的描述
  - image: nginx:latest		# 容器镜像
    imagePullPolicy: IfNotPresent		# 定义镜像拉取策略
    name: nginx			# 容器的名称
    command:		# 指定容器启动时执行的命令
    - nginx
    - -g
    - 'daemon off;'  # nginx -g 'daemon off';
    workingDir: /usr/share/nginx/html		# 定义容器启动后的工作目录
    ports:		# 指定容器需要用到的端口列表
    - name: port1		# 指定端口的名称
      containerPort: 80		# 指定容器需要监听的端口号
      hostPort: 50080 		# 指定容器所在主机需要监听的端口号；默认跟上面 containerPort 相同，注意设置了 hostPort 同一台主机无法启动改容器的想通副本（因为主机的端口号不能相同，这样会冲突）
      protocol: TCP		# 指定端口协议，支持 TCP和 UDP，默认值为 TCP
    env:		# 定义容器运行前设置的环境变量列表
    - name: ws_test		# 变量名称
      value: 'test'		# 变量值
    resources:		# 定义资源限制
      requests:		# 最小需要多少资源
        cpu: 100m		# 限制 cpu 最少使用 0.1 个核心，1000m=1核心
        memory: 128Mi		# 限制内存最少使用 128 兆
      limits:		# 最多能使用多少资源
        cpu: 200m 		# 限制 cpu 最多使用 0.2 个核心
        memory: 300Mi		# 限制内存最多使用 300 兆
    terminationMessagePath: /dev/termination-log		# 日志所在的 path
    terminationMessagePolicy: File			# 日志策略为文件格式
    volumeMounts:		# 定义容器内部的存储卷配置
    - mountPath: /var/run/secrets/kubernetes.io/serviceaccount		# 指定可以被容器挂载的存储卷的路径
      name: kube-api-access-x5rlm		# 指定容器存储卷的名称
      readOnly: true		# 设置存储卷的读写模式，true 或者 false，默认是读写模式
  nodeSelect: 			# 定义 Node 的 label 过滤标签，以 key:value 格式指定
    app: ws
  imagePullSecrets: 		# pull 镜像时使用 secret 名称，以 name: secretkey 格式指定
  - name: images-secret
  restartPolicy: Always		# 重启策略，pod 一旦终止运行，kubelet 都会重启该容器
  volumes:		# 定义存储卷
  - name: kube-api-access-x5rlm
    projected:
      defaultMode: 420
      sources:
      - serviceAccountToken:
          expirationSeconds: 3607
          path: token
      - configMap:
          items:
          - key: ca.crt
            path: ca.crt
          name: kube-root-ca.crt
      - downwardAPI:
          items:
          - fieldRef:
              apiVersion: v1
              fieldPath: metadata.namespace
            path: namespace
~~~



## 资源调度（controller）

Kubernetes Controller是Kubernetes系统中的一种控制器，它负责管理和控制系统中的特定资源对象，确保系统的状态与期望状态一致。这些控制器运行在kube-controller-manager中，是Kubernetes中的核心组件之一。

### Deployment

[Deployment](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/deployment/) （也间接包括 [ReplicaSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/replicaset/)） 是在集群上运行应用的最常见方式。Deployment 适合在集群上管理无状态应用工作负载， 其中 Deployment 中的任何 Pod 都是可互换的，可以在需要时进行替换。 （Deployment 替代原来的 [ReplicationController](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-replication-controller) API）。

#### 功能

##### 创建

~~~shell
# 在 ns ws 中创建一个用 nginx 镜像 名为 nginx-deploy 的 deployment。
kubectl create deployment nginx-deploy --image=nginx -n ws

# 查看 deploy
kubectl get deployments.apps -n ws 
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           173m

# 查看 repolicaset
kubectl get replicaset -n ws 
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-846d6f46b7   1         1         1       173m

# 查看 pod
kubectl get pod -n ws 
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-846d6f46b7-8jcz7   1/1     Running   0          174m

# 一次性查看所有
kubectl get deploy,rs,pod -n ws
NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deploy   1/1     1            1           3h24m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deploy-846d6f46b7   1         1         1       3h24m

NAME                                READY   STATUS    RESTARTS   AGE
pod/nginx-deploy-846d6f46b7-8jcz7   1/1     Running   0          3h24m
~~~



##### 滚动更新

只有修改了 Deployment 配置文件中的 template 中的属性后，才会触发更新操作。

~~~shell
# 查看滚动更新的过程
kubectl -n ws rollout status deploy nginx-deploy

# 改修副本数，执行 rollout status 会发现里面什么内容都没
kubectl -n ws edit deployment nginx-deploy
spec:
  replicas: 2		# 修改成 2

# 使用 set 修改镜像，执行 rollout status，会发现里面的内容一条一条蹦出来，里面记录着滚动更新的过程
kubectl -n ws set image deployment/nginx-deploy nginx=nginx:latest

# 查看 deployment，在更新的过程中，重复执行，你会发现 READY 的值一直都是2，READY表示副本数，UP-TO-DATE会反映出有多少个副本已经应用了这次变更，确保系统中运行的副本与你的期望状态一致，AVAILABLE表示可用的副本数。
kubectl -n ws get deployment
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2/2     2            2           4h14m
~~~

![image-20240106171849538](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240106171849538.png)

##### 回滚

在 Kubernetes 中，Deployment 的回滚是指将应用程序的部署版本还原到先前的状态，以应对不良更新或问题。

可以通过`.spec.revisionHistoryLimit` 来指定 deployment 保留多少 revision,如果设置成 0，则表示不允许 deployment 回退了。

~~~shell
# 查看 rs 并且显示标签
kubectl get rs -n ws --show-labels
NAME                      DESIRED   CURRENT   READY   AGE     LABELS
nginx-deploy-7b68b887bc   0         0         0       17m     app=nginx-deploy,pod-template-hash=7b68b887bc
nginx-deploy-846d6f46b7   0         0         0       5h13m   app=nginx-deploy,pod-template-hash=846d6f46b7
nginx-deploy-dd79d5576    2         2         2       63m     app=nginx-deploy,pod-template-hash=dd79d5576

# 查看历史回滚信息
kubectl rollout history deployment -n ws nginx-deploy
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>

# 查看版本 revision 2 的信息
kubectl rollout history deployment -n ws nginx-deploy --revision=2 

# 回退到版本 revision 2 
kubectl rollout undo deployment -n ws nginx-deploy --to-revision=2
~~~



##### 扩缩容

~~~shell
kubectl scale --replicas=6 -n ws deployment nginx-deploy
~~~



##### 暂停和恢复

由于每次对 pod template 中的信息发生修改后，都会触发更新操作，那么此时如果频繁修改信息，就会产生多次更新，而实际上只需要执行最后一次更新即可，当出现此类情况时我们就可以暂停 deployment 的 rollout。

~~~shell
# 暂停
kubectl rollout pause deployment -n ws nginx-deploy

# 然后再对容器进修改
kubectl edit deployment -n ws nginx-deploy
# 比如添加一个资源的限制
   spec:
     containers:
     - image: nginx:latest
       resources:
         requests:
           cpu: 100m			# 最小 cpu
           memory: 128Mi			# 最小内存
         limits:
           cpu: 500m		# 最大 cpu
           memory: 512Mi 		# 最大内存

# 恢复，在停止期间，无论做了多少操作，恢复完成后，只会创建一个 rs、rollout版本。
kubectl rollout resume deploy -n ws nginx-deploy
~~~



#### 配置文件

~~~yaml
apiVersion: apps/v1		# deployment api 版本
kind: Deployment		# 资源类型为 Deployment
metadata:			# 元信息
#  annotations:			# annotations 和 label 类似，也是用来描述对象的数据，但是它比 label 更强大，annotations 里可以描述更多更细节的内容，而 label 里只是一个简易的标签而已。
#    deployment.kubernetes.io/revision: "1"
  labels:		# 标签
    app: nginx-deploy		# 具体的 key:valu 配置形式
  name: nginx-deploy	# deployment 的名字
  namespace: ws		# 所在的命名空间
spec:
  replicas: 1	# 期望的副本数
  revisionHistoryLimit: 10		# 进行滚动更新后，保留的历史版本个数,如果设置成 0，则表示不允许 deployment 回退了
  selector:		# 选择器，用于找到配置的 RS
    matchLabels:		# 按照标签匹配
      app: nginx-deploy		# 匹配标签 key/value 
  strategy:			# 更新策略
    rollingUpdate:		# 滚动更新配置
      maxSurge: 25%			# 进行滚动更新时，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25%		# 新型滚动更新时，最大不可用更新比例，表示在所有副本数中，最多可以有多少个不更新成功
    type: RollingUpdate		# 更新类型，采用滚动更新
  template:		# pod 模版
    metadata:		# pod 元信息
      labels:		# 标签
        app: nginx-deploy
    spec:		# pod 规约、规范（期望信息）
      containers:		# pod 的容器
      - image: nginx		# 容器的镜像
        imagePullPolicy: Always		# 镜像拉取策略
        name: nginx		# 容器的名字
      restartPolicy: Always		# 重启策略
      terminationGracePeriodSeconds: 30		# 删除操作最多宽限多长时间
~~~



### StatefulSet

[StatefulSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/statefulset/) 允许你管理一个或多个运行相同应用代码、但具有不同身份标识的 Pod。 StatefulSet 与 Deployment 不同。Deployment 中的 Pod 预期是可互换的。 StatefulSet 最常见的用途是能够建立其 Pod 与其持久化存储之间的关联。 例如，你可以运行一个将每个 Pod 关联到 [PersistentVolume](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/) 的 StatefulSet。如果该 StatefulSet 中的一个 Pod 失败了，Kubernetes 将创建一个新的 Pod， 并连接到相同的 PersistentVolume。

#### 创建

~~~shell
# 把下面的配置文件来来用最初的创建
kubectl apply -f statfulset-test.yaml
kubectl get svc,sts
NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   234d
service/nginx        ClusterIP   None         <none>        80/TCP    2m22s

NAME                   READY   AGE
statefulset.apps/web   2/2     2m22s

# 创建使用busybox镜像的 pod 测试
kubectl run -it --image busybox:1.28.4 dns-test /bin/sh
If you don't see a command prompt, try pressing enter.
/ # nslookup web-0.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local		# 完成的映射路径，DNS格式 statefulSetName-(0..N-1).serviceName.namespace.svc.cluster.local

Name:      web-0.nginx
Address 1: 198.18.87.183
/ # nslookup web-1.nginx
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 198.18.88.219
~~~



#### 扩缩容

~~~shell
# 和 deployment 使用方法一样
kubectl scale statefulset --replicas=5 web

# 查看详细信息
kubectl describe statefulsets.apps web
Name:               web
Namespace:          default
CreationTimestamp:  Tue, 09 Jan 2024 19:39:06 +0800
Selector:           app=nginx
Labels:             <none>
Annotations:        <none>
Replicas:           5 desired | 5 total
Update Strategy:    RollingUpdate
  Partition:        0
Pods Status:        5 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:latest
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Volume Claims:    <none>
Events:
  Type    Reason            Age   From                    Message
  ----    ------            ----  ----                    -------
  Normal  SuccessfulCreate  13m   statefulset-controller  create Pod web-0 in StatefulSet web successful	# 每一个扩容或缩容都有顺序性
  Normal  SuccessfulCreate  13m   statefulset-controller  create Pod web-1 in StatefulSet web successful
  Normal  SuccessfulCreate  13s   statefulset-controller  create Pod web-2 in StatefulSet web successful
  Normal  SuccessfulCreate  12s   statefulset-controller  create Pod web-3 in StatefulSet web successful
  Normal  SuccessfulCreate  11s   statefulset-controller  create Pod web-4 in StatefulSet web successful
~~~



#### 镜像更新

~~~shell
# 使用 set 命令进行更新
kubectl set image sts web nginx=nginx:latest

# 过程
candidate@node01:~$ kubectl get po -w		# -w：在列出/获得所请求的对象之后，注意变化。
NAME                       READY   STATUS              RESTARTS      AGE
dns-test                   1/1     Running             1 (40m ago)   45m
redis123-c69bfccd9-grt94   1/1     Running             4 (56m ago)   232d
web-0                      1/1     Running             0             2m10s
web-1                      0/1     ContainerCreating   0             7s
web-1                      1/1     Running             0             62s
web-0                      1/1     Terminating         0             3m5s
web-0                      1/1     Terminating         0             3m5s
web-0                      0/1     Terminating         0             3m5s
web-0                      0/1     Terminating         0             3m6s
web-0                      0/1     Terminating         0             3m6s
web-0                      0/1     Terminating         0             3m6s
web-0                      0/1     Pending             0             0s
web-0                      0/1     Pending             0             0s
web-0                      0/1     ContainerCreating   0             0s
web-0                      0/1     ContainerCreating   0             0s
web-0                      1/1     Running             0             1s
~~~



##### RollingUpdata

Statefulset 也可以采用滚动更新，同样是修改 pod template 属性后会触发更新，但是由于 pod 有序的，在 Statefulset 中更新时是基于 pod 的顺序倒序更新的。

- 金丝雀发布（Canary release）是一种渐进式的软件发布策略，用于逐步将新版本的软件引入生产环境，以降低风险并及早发现潜在问题。这种发布方法得名于矿工在矿井中使用金丝雀来检测有毒气体。类比于软件开发，金丝雀发布首先将新版本引入一小部分用户或流量，监测其性能和稳定性。如果没有问题，逐渐扩大覆盖范围，最终将新版本发布给所有用户。

- 灰度发布/金丝雀发布：利用滚动更新中的 partition 属性，可以实现简易的灰度发布的效果。假如我们有 5 个 pod，如果当前 partition 设置为 3，那么此时滚动更新时，只会更那些序号 >=3 的pod，利用该机制，我们可以通过控制 partition 的值，来决定只更新其中一部分 pod，确认没有问题后再逐渐增大更新的 pod 数量，最终实现全部 pod 更新。

  ~~~shell
  # 扩容
  kubectl scale statefulset --replicas=5 web
  
  # 设置 partition 值;statefulset.spec.updateStrategy.rollingUpdate.partition
    updateStrategy:
      rollingUpdate:
        partition: 3
        
  # 测试
  # 查看 web-03 原来的镜像
  kubectl get pod  web-3 -o yaml | grep -i image:
    - image: nginx:1.9.1		# 镜像为 nginx:1.9.1
      image: docker.io/library/nginx:1.9.1
  
  # 更新镜像
  kubectl set image sts web nginx=nginx:latest
  
  # 查看 
  candidate@node01:~$ kubectl get pod  web-4 -o yaml | grep -i image:
    - image: nginx:latest
      image: docker.io/library/nginx:latest
  
  candidate@node01:~$ kubectl get pod  web-2 -o yaml | grep -i image:
    - image: nginx:1.9.1		# 没变
      image: docker.io/library/nginx:1.9.1
  ~~~

  

##### OnDelete

触发遗留行为,禁用滚动重启;这意味着只有在删除现有pod时才应用更新。删除现有的pod后，将使用更新后的配置创建新的pod。

pod 是从 StatefulSetSpec 重新创建的.手动删除时。执行缩放操作时此策略、规范版本由 statfulset 指示 currentRevision。

~~~shell
# 修改类型
kubectl edit statefulset web
# 修改到只剩这两行
  updateStrategy:
    type: OnDelete
~~~

> 注意：只有 Deployment 是和 rs 关联在一起的，statefulset 是直接和 pod 管理的。

##### 删除

kubectl 在删除资源的时候，默认是级联删除的。

~~~shell
# 非级联删除
kubectl delete sts web --cascade=false
~~~



#### 配置文件

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet		# StatefulSet 类型的资源
metadata:
  name: web		# StatefulSet 对象的名称
spec:		# 定义 StatefulSet 的规约
  serviceName: "nginx"		# 指定名称为 nginx 的 service 来管理 web
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:		# 定义 pod 的规约
      containers:
      - name: nginx
        image: nginx:latest
        imagePullPolicy: IfNotPresent
        ports:
        - containerPort: 80		# 容器暴露的端口号
          name: web		
        volumeMounts:		# 定义加载数据卷
        - name: www		# 定义加载数据卷的名称
          mountPath: /usr/share/nginx/html		# 定义加载指定位置
  volumeClaimTemplates:		# 数据卷模版
  - metadata:		# 数据卷描述
      name: www			# 数据卷的名称
    spec:		# 数据卷的规约
      accessModes: [ "ReadWriteOnce" ]		# 存储卷的访问模式
      resources:
        requests:
          storage: 1Gi		# 需要 1G 的存储资源
~~~



### DaemonSet

[DaemonSet](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/daemonset/) 定义了在特定[节点](https://kubernetes.io/zh-cn/docs/concepts/architecture/nodes/)上提供本地设施的 Pod， 例如允许该节点上的容器访问存储系统的驱动。当必须在合适的节点上运行某种驱动或其他节点级别的服务时， 你可以使用 DaemonSet。DaemonSet 中的每个 Pod 执行类似于经典 Unix / POSIX 服务器上的系统守护进程的角色。DaemonSet 可能对集群的操作至关重要， 例如l作为插件让该节点访问[集群网络](https://kubernetes.io/zh-cn/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-network-model)， 也可能帮助你管理节点，或者提供增强正在运行的容器平台所需的、不太重要的设施。 你可以在集群的每个节点上运行 DaemonSets（及其 Pod），或者仅在某个子集上运行 （例如，只在安装了 GPU 的节点上安装 GPU 加速驱动）。

#### 指定 node 节点

三种方式

- nodeSelector：只调度匹配指定 label 的 node 上。

  ~~~shell
  # 修改 daemonset.spec.template.spec.nodeSelector
  kubectl edit daemonsets.apps fluentd-elasticsearch
        nodeSelector:
          type: ws
  ~~~

  

- nodeAffinity：功能更丰富的 Node 选择器，比如支持集合操作。

- podAffinity： 调度到满足条件的 pod 所在的 node 上。

#### 滚动更新

在 DaemonSet 中，当使用 `updateStrategy` 并设置 `type: OnDelete` 时，表明采用手动删除旧的 DaemonSet Pod 的升级策略。以下是适用于这种场景的情况：

1. **手动升级：** DaemonSet 使用 `OnDelete` 升级策略，需要手动删除旧的 DaemonSet Pod，新的 Pod 才会根据更新后的 DaemonSet 模板重新创建。这种方式允许操作者在合适的时机进行升级，以确保新的配置生效。
2. **灵活性和控制：** 采用手动删除的方式使得升级过程更为灵活，操作者可以在确保没有中断的情况下逐步进行升级，有更多的控制权。
3. **保留旧 Pod：** 在升级时，旧的 DaemonSet Pod 不会被自动删除，而是需要手动触发删除操作。这对于需要保留历史 Pod 以便进一步分析或回滚的情况很有用。

~~~shell
# 修改 daemonset.spec.updateStrategy.OnDelete
kubectl edit daemonsets.apps fluentd-elasticsearch
  updateStrategy:
    type: OnDelete
~~~



#### 配置文件

~~~yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
  labels:
    k8s-app: fluentd-logging
spec:
  selector:
    matchLabels:		# 和 pod 标签作匹配
      name: fluentd-elasticsearch     
  template:			# pod 的模板
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:		# pod 的约束
      nodeSelector:		# 和 node 标签作匹配
        type: ws
      tolerations:
      # 这些容忍度设置是为了让该守护进程集在控制平面节点上运行
      # 如果你不希望自己的控制平面节点运行 Pod，可以删除它们
      - key: node-role.kubernetes.io/control-plane
        operator: Exists
        effect: NoSchedule
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      containers:
      - name: fluentd-elasticsearch
        image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:		# 加载数据卷
        - name: varlog
          mountPath: /var/log		# 定义数据卷挂在哪个路径
      # 可能需要设置较高的优先级类以确保 DaemonSet Pod 可以抢占正在运行的 Pod
      # priorityClassName: important
      terminationGracePeriodSeconds: 30
      volumes:		# 定义数据卷
      - name: varlog
        hostPath:		# 类型，主机路径的模式，也就是与 node 共享目录
          path: /var/log		# node 中的共享目录
~~~



### CronJob 计划任务

**CronJob** 创建基于时隔重复调度的 [Job](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/job/)。

CronJob 用于执行排期操作，例如备份、生成报告等。 一个 CronJob 对象就像 Unix 系统上的 **crontab**（cron table）文件中的一行。 它用 [Cron](https://zh.wikipedia.org/wiki/Cron) 格式进行编写， 并周期性地在给定的调度时间执行 Job。

CronJob 有所限制，也比较特殊。 例如在某些情况下，单个 CronJob 可以创建多个并发任务。 请参阅下面的[限制](https://kubernetes.io/zh-cn/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations)。

当控制平面为 CronJob 创建新的 Job 和（间接）Pod 时，CronJob 的 `.metadata.name` 是命名这些 Pod 的部分基础。 CronJob 的名称必须是一个合法的 [DNS 子域](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names)值， 但这会对 Pod 的主机名产生意外的结果。为获得最佳兼容性，名称应遵循更严格的 [DNS 标签](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-label-names)规则。 即使名称是一个 DNS 子域，它也不能超过 52 个字符。这是因为 CronJob 控制器将自动在你所提供的 Job 名称后附加 11 个字符，并且存在 Job 名称的最大长度不能超过 63 个字符的限制。

#### cron 表达式

~~~shell
# .---------------- 分钟 (0 - 59)
# |  .------------- 小时 (0 - 23)
# |  |  .---------- 日 (1 - 31)
# |  |  |  .------- 月份 (1 - 12) 
# |  |  |  |  .---- 周 (0 - 6)
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
~~~

#### 配置示例

~~~yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
# startingDeadlineSeconds: 30           # 间隔时间多长时间检测失败的任务并重新执行，时间不能小于 10
  concurrencyPolicy: Allow		# 可选字段，并发调度策略，默认 Allow：允许并发调度，Forbid：不允许并发执行，Replace： 如果之前的任务还没执行完，就直接执行新的，放弃上一个任务
  successfulJobsHistoryLimit: 3			# 保留多少个成功的任务
  failedJobsHistoryLimit: 1			# 保留多少个失败的任务
  suspend: false		# 调度挂起，是否挂起任务，若为 true 则该任务不会被执行
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure		# 重启策略，针对 .status.phase 等于 Pending 或 Running 的 Pod，计算其中所有容器的重试次数。
~~~



### HPA

pod 自动扩容；可以根据 CPU 使用率或自定义指标（metrics）自动对 pod 进行扩/缩容。

HPA 只能作用于 deployment、statefulset 这中类型的控制器，daemonset 是在匹配上的 node 启动 pod，所以 daemonset 不能使用 HPA。



#### cpu、内存指标监控

实现 cpu 或内存的监控，首先有个前提条件是该对象必须配置了 resources.requests.cpu 或 resources.requests.memory 才可以，可以配置当 cpu/memory 达到上述配置的百分比后进行扩容或缩容。

##### 创建 HPA

~~~shell
# 先准备好一个有做资源限制的 deployment/statefulset
# 执行命令
kubectl autoscale deployment -n ws nginx-deploy --cpu-percent=20 --min=2 --max=5

# 测试 HPA，当 cpu 使用率到 20%，就会自动扩容到 5 个副本。
while true ; do wget -q -O- http://<ip>:<port> > /dev/null ;done
~~~



#### 自定义 metrics

~~~shell
# 查看 pod/node 的 cpu 和内存的使用率
kubectl top pod 
# 如果出现 error，就下载指标服务
error: Metrics API not available

# 安装 Metrics
# 下载 metrics-server 组件配置文件
wget  https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.6.4/components.yaml -O metrics-server-components.yaml
# 替换成国内的源
sed -i 's/registry.k8s.io\/metrics-server/registry.cn-hangzhou.aliyuncs.com\/google_containers/g' metrics-server-components.yaml
# 修改容器的 tls 配置，不验证tls，在 containers 的 args 参数中增加 --kubelet-insecure-tls 参数
        - --kubelet-insecure-tls
# 部署组件
kubectl apply -f metrics-server-components.yaml


# 之后使用 top 命令
kubectl top po -n ws
NAME                            CPU(cores)   MEMORY(bytes)   
nginx-deploy-6594d85888-7p2nl   0m           4Mi             
nginx-deploy-6594d85888-8htz9   0m           4Mi             
~~~



## 服务发布

### Service

Service是kubernetes 的核心概念，通过创建service，可以为一组具有相同功能的容器应用提供统一的入口地址，并将请求负载分发到后端的各个容器上。

转发流量的过程：master 节点通过 kubectl 命令执行，会请求到 api-server 服务上，api-server 会通过 ip 找到对应的 Service，Service 找到自己的 Endpoint，通过 iptables 的转发找到目标的服务器上的 kube-proxy 找到自身节点上的容器，然后执行命令。

Service,Endpoint,pod流量转发示意图：

![image-20231025213205631](C:\ws\syncdisk\大一\picture\image-20231025213205631.png)



#### 代理

##### 代理 k8s 内部服务

~~~shell
# 使用命令创建出一个 svc
kubectl expose deploy nginx-deploy --port 80 --name=ws-svc -n ws

# 利用同一个命名空间下的 pod 访问 ws-svc 就是 nginx-deployment 里 pod 的内容,前提是没有网络策略
# 比如使用一个叫 mysql-test 的 pod 来访问
kubectl exec -it mysql-test -- sh
wget http://ws-svc		# 如果不是在同一个命名空间下就用 ws-svc.default 的方式来访问
wget http://ws-svc.ws    # <svc的名称>.<命名空间的名称>
~~~



##### 代理 k8s 外部服务 

通过 service 不指定 selector 就不自动创建 endpoint 的这个机制

我们定义一个 svc

~~~yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-external
  labels:	
    app: nginx		# service 自己的标签
spec:
#  selector: 		# 匹配哪些 pod 会被该 service 代理
#    app: nginx-deploy		# 所有匹配使用这些标签的 pod 都可以通过该 service 进行访问
  ports: 		# 映射端口
  - port: 80		# service 自己的端口，在使用内网 ip 访问时使用
    targetPort: 80		# 目标 pod 的端口
    name: web 		# 给端口起名字
  type: ClusterIP 	# Server 的类型默认是 clusterIP
~~~

我们把 selector 给注释了，该 service 就不会自动给我们创建 endpoint ，我们自己创建 endpoint

~~~yaml
apiVersion: v1
kind: Endpoints
metadata:
  name: nginx-svc-external		# 必须要和 service 一致
  labels:
    app: nginx		# 必须要和 service 一致
  namespace: default		# 必须要和 service 一致
subsets:
- addresses:
  - ip: 10.64.150.10   # 目标的 ip 地址，需要代理的 ip,这里就是我服务器的 web 网站
  ports:
  - port: 80
    name: web
    protocol: TCP
~~~



##### 反向代理外部域名

这个比上一种方式更简单

定义一个 service 即可

~~~yaml
apiVersion: v1
kind: Service
metadata: 
  name: baidu-svc-external
  labels:
    app: baidu-svc-external
spec: 
  type: externalName
  externalName: www.baidu.com
~~~



#### service 类型

##### ClusterIP

ClusterIP 是默认的 Service 类型。它将创建一个虚拟的 IP 地址，用于连接客户端和 Pod。这个 IP 地址只能在集群内部使用，无法从集群外访问。这种类型通常用于后端服务，如数据库或缓存服务。

##### ExternalName

ExternalName Service 允许 Service 对外暴露一个外部名称，这个名称可以被解析为外部服务的 DNS 名称。该类型的 Service 不会在集群中创建任何负载均衡器或 IP，而是将请求直接转发到指定的外部服务。

##### NodePort

会在所有安装了 kube-proxy 的节点都绑定一个端口，此端口可以代理至对应的 pod，集群外部可以使用任意节点 IP + NodePort 的端口号访问到集群中对应 pod 中的服务。

当类型设置为 NodePort 后，可以在 ports 配置中增加 nodePort 配置定义端口，需要在下方的端口范围内，如果不指定会随机指定端口。

端口范围：30000~32767

端口范围配置在 /usr/lib/systemd/system/kube-apiserver.service 文件中，不一定会在这个，如果你用二进制部署的集群，会在你配置的apiserver配置文件中，当然你都部署二进制了，你肯定知道在哪里。

##### LoadBalancer

LoadBalancer 可以在云环境中自动创建外部负载均衡器，并将客户端请求路由到 Pod。该类型的 Service 通常用于公共云或私有云环境中，可以将流量平衡到多个集群节点上，从而提高服务的可靠性和可用性。

#### 配置文件

~~~shell
apiVersion: v1
kind: Service		# 资源类型为 Service
metadata:
  name: nginx-svc		# Service 名字
  labels:	
    app: nginx		# service 自己的标签
  namespace: ws		# 所在命名空间的名称
spec:
#  selector: 		# 匹配哪些 pod 会被该 service 代理，如果不设置的话，svc 就不会自动创建 endpoint，需要手动创建 endpoint
#    app: nginx-deploy		# 所有匹配使用这些标签的 pod 都可以通过该 service 进行访问
  ports: 		# 映射端口
  - port: 80		# service 自己的端口，在使用内网 ip 访问时使用
    targetPort: 80		# 目标 pod 的端口
    name: web 		# 给端口起名字
  type: NodePort 	# Server 的类型能随机创建一个端口（300000~32767）,映射到 ports 中的端口，该端口是直接绑定在 node 上的，且群集中的每一个 node 都会绑定这个端口；也可以当服务暴露给外部服务，这种方式的暴露是一种轮训的算法，当我们的节点数越多，它的效率越差，暴露服务使用建议使用 ingress；
~~~



### Ingress

[Ingress](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.29/#ingress-v1-networking-k8s-io) 提供从集群外部到集群内[服务](https://kubernetes.io/zh-cn/docs/concepts/services-networking/service/)的 HTTP 和 HTTPS 路由。 流量路由由 Ingress 资源所定义的规则来控制;Ingress 是对 service 的更高层次的抽象，service 是工作在 tcp/ip 层，基于ip 和 port 的，那么 ingress 是针对 http 七层路由机制，将客户端的请求直接转发到 service 对应的后端 pod 服务上

#### 安装 ingress 控制器

如果要使用 ingress，首先需要有一个 ingress 控制器才能满足 ingress 的需求，我们可以根据自己的需求来安装。

官网 ingress-nginx 控制器：[Installation Guide - Ingress-Nginx Controller (kubernetes.github.io)](https://kubernetes.github.io/ingress-nginx/deploy/)

Helm 下载地址：[Releases · helm/helm (github.com)](https://github.com/helm/helm/releases)

1. 方法一：

   ~~~shell
   # 安装 Helm，添加 helm 仓库，当前环境是内网，内网环境手动下载安装，上方有下载地址
   wget https://get.helm.sh/helm-v3.14.0-rc.1-linux-amd64.tar.gz
   
   # 解压文件
   tar -zxvf helm-v3.14.0-rc.1-linux-amd64.tar.gz
   sudo mv linux-amd64/helm /usr/local/bin/helm
   
   # 安装完成，查看版本
   helm version
   
   # 添加仓库
   helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
   # 查看仓库
   helm repo list
   
   # 查找 ingress-nginx
   helm search repo ingress-nginx
   # 下载 ingress-nginx 软件包
   helm pull ingress-nginx/ingress-nginx
   tar -xf ingress-nginx-4.9.0.tgz
   cd ingress-nginx
   # 修改配置
   vim values.yaml
     24     registry: registry.cn-hangzhou.aliyuncs.com
     25     image: google_containers/nginx-ingress-controllers
     30     #digest: sha256:b3aba22b1da80e7acfc52b115cae1d4c687172cbf2b742d5b502419c25ff340e
     31     #digestChroot: sha256:9a8d7b25a846a6461cd044b9aea9cf6cad972bcf2e64d9fd246c0279979aad2d
     
     73   dnsPolicy: ClusterFirstWithHostNet
   
     94   hostNetwork: true
   
    298     nodeSelector:
    299       kubernetes.io/os: linux
    300       ingress: "true"
    
    458     type: ClusterIP
   
    778       image:
    779         registry: registry.cn-hangzhou.aliyuncs.com
    780         image: google_containers/kube-webhook-certgen
    784         #tag: v20231011-8b53cabe0
    785         tag: v1.3.0
    786         #digest: sha256:a7943503b45d552785aa3b5e457f169a5661fb94d82b8a3373bcd9ebaf9aac80
    787         #pullPolicy: IfNotPresent
    
   # 打标签
   kubectl label node node01 ingress=true
   
   # 安装 ingress-nginx
   helm install ingress-nginx -n ingress-nginx .
   ~~~

   

2. 方法二：

   ~~~shell
   # 如果没有 Helm，或者更喜欢使用 YAML 清单，可以改为运行以下命令：
   kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.8.2/deploy/static/provider/cloud/deploy.yaml
   ~~~

   

#### 基本使用

##### 创建

一个最小的 Ingress 资源示例：

~~~yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx-example
  rules:		# ingress 规则配置，可以配置多个
  - http:		# http 协议
      paths:			
      - path: /testpath		# 前缀匹配
        pathType: Prefix		# 一共三种配置，ImplementationSpecific：对于这种路径类型，匹配方法取决于 IngressClass。 具体实现可以将其作为单独的 pathType 处理或者作与 Prefix 或 Exact 类型相同的处理；Exact：精确匹配 URL 路径，且区分大小写；Prefix：基于以 / 分隔的 URL 路径前缀匹配。匹配区分大小写， 并且对路径中各个元素逐个执行匹配操作。 路径元素指的是由 / 分隔符分隔的路径中的标签列表。 如果每个 p 都是请求路径 p 的元素前缀，则请求与路径 p 匹配。
        backend:
          service:		# 代理到哪个 service
            name: test
            port:		# Service 的端口
              number: 80
~~~



## 配置与存储

### 配置管理

#### ConfigMap

ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。

> ConfigMap 并不提供保密或者加密功能。 如果你想存储的数据是机密的，请使用 [Secret](https://kubernetes.io/zh-cn/docs/concepts/configuration/secret/)， 或者使用其他第三方工具来保证你的数据的私密性，而不是用 ConfigMap。

##### 创建

~~~shell
# 使用 kubectl create 创建
# 基于指定目录的形式，进行创建
kubectl create configmap my-dir-config --from-file=path/to/bar

# 基于指定文件创建
kubectl create configmap my-file-config --from-file=key1=/path/to/bar/file1.txt --from-file=key2=/path/to/bar/file2.txt

# 基于 key=vaule 的形式创建
kubectl create configmap my-key-value-config --from-literal=username=root --from-literal=password=admin
~~~



##### 使用 ConfigMap

~~~shell
# 编写一个 yml 文件，这种方法是直接指定 env 的方法，还有一种方法是基于 volume 使用
vim test-env-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-test
spec: 
  containers:
  - name: env-test
    image: alpine
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "env;sleep 3600"]
    env: 
    - name: WS_USER
      valueFrom:
        configMapKeyRef:
          name: my-key-value-config
          key: username
    - name: WS_PASSWD
      valueFrom:
        configMapKeyRef:
          name: my-key-value-config
          key: password

# 创建一个 pod 
kubectl create -f test-env-pod.yaml
# 查看 logs 
kubectl logs -f env-test

# 基于 volume 挂载的形式使用 ConfigMap
vim file-test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-configfile-pod
spec: 
  containers:
  - name: config-test
    image: nginx
    imagePullPolicy: IfNotPresent
    command: ["/bin/sh", "-c", "env;sleep 3600"]
    volumeMounts:               # 加载数据卷
    - name: ws-config
      mountPath: "/config"
      readOnly: true            # 是否只读，默认是 false
  volumes:              # 数据卷挂载 configMap、secret
  - name: ws-config
    configMap:
      name: tset-dir-config             # 指定 configMap 文件
      items:            # 对ConfigMap 中的 key 进行映射，如果不指定，默认会将 configmap 中所有 key 全部转换为一个个同名的文件
      - key: "1.txt"		# 
        path: "ws"
# 创建
kubectl create -f file-test-pod.yaml
# 测试
kubectl exec -it  test-configfile-pod -- sh
# cat /config/ws
username=root
password=admin
~~~



#### 加密数据配置 Secret

与 ConfigMap 类似，用于存储配置信息，但是主要用于存储敏感信息，需要加密的信息，Secret 可以提供数据加密，解密功能。

在创建 Secret 时，要注意如果要加密的字符中，包含了有特殊字符，需要使用转义符转移，例如 $ 转移后为 \\$,也可以对特殊字符使用单引号描述。



~~~shell
# 通用生成
kubectl create secret generic ws-secret --from-literal=username=admin --from-literal=password='123@_/23'

# docker-registry
kubectl create secret docker-registry harbor-secret --docker-username=admin --docker-password=ws --docker-email=wangshuo@ws.com --docker-server=192.168.11.100:8848

# 在本地仓库中拉取镜像
# 添加 pod.spec.imagePullSecrets 字段
apiVersion: v1
kind: Pod
metadata:
  name: ws
spec:
  imagePullSecrets:
  - name: harbor-secret		# 指定 secret
  containers:
  - name: config-test
    image: 192.168.11.100:8848/ws/nginx:latest
    imagePullPolicy: IfNotPresent
    
# 为 sa 创建 secret
apiVersion: v1
kind: Secret
metadata:
  name: jenkins
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: "jenkins"		# serveraccount 的名称
type: kubernetes.io/service-account-token
~~~

> 注意：在 1.25 之后创建 serviceaccount 不会自动生成 Secret。



#### SubPath 的使用

当你在使用 configMap 的时候，你会发现，configMap 指定的目录会把容器内的这个路径的所有文件都会被覆盖掉，SubPath 就是用来解决 ConfigMap 覆盖文件的问题

1. 定义 volumes 时需要添加 items 属性，配置 key 和 path，且 path 的值不能从 / 开始。
2. 在容器内的 volumeMounts 中添加 subPath 属性，该值与 valumes 中 items.path 的值相同。

~~~~yaml
...
spec:
  containers:
    volumeMounts:
    - name: config
      mountPath: "/etc/nginx/nginx.conf"
      subPath: nginx.conf
  volumes:
  - name: config
    configMap:
      name: nginx-conf-cm
      items:
      - key: 1.txt
        path: nginx.conf
~~~~



#### 配置的热更新

我们通常会将项目的配置文件作为 ConfigMap 然后挂载到 pod，那么如果更新 ConfigMap 中的配置，会不会更新到 pod 中呢？

这得分成几中情况：

- 默认方式：会更新，更新周期是更新时间 + 缓存时间
- SubPath：不会更新
- 变量形式：如果 pod 中的一个变量是从 ConfigMap 或 Secret 中得到，同样也是不会更新

##### 通过 edit 修改 ConfigMap

~~~shell
kubectl edit cm nginx-conf-cm
# 修改 configMap 之后，里面的容器读到这配置，容器里的 configMap 会自动发生变动
~~~



##### 通过 replace 替换

~~~shell
# 先把 1.txt 改成我们想要的文件，然后再执行下面这段命令
kubectl create cm txt.cm --from-file=1.txt --dry-run -o yaml | kubectl replace -f txt.cm
~~~



#### 不可变的 Secret 和 ConfigMap

Kubernetes 特性 *Immutable Secret 和 ConfigMaps* 提供了一种将各个 Secret 和 ConfigMap 设置为不可变更的选项。对于大量使用 ConfigMap 的集群 （至少有数万个各不相同的 ConfigMap 给 Pod 挂载）而言，禁止更改 ConfigMap 的数据有以下好处：

- 保护应用，使之免受意外（不想要的）更新所带来的负面影响。
- 通过大幅降低对 kube-apiserver 的压力提升集群性能， 这是因为系统会关闭对已标记为不可变更的 ConfigMap 的监视操作。

你可以通过将 `immutable` 字段设置为 `true` 创建不可变更的 ConfigMap。 例如：

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  ...
data:
  ...
immutable: true
```

一旦某 ConfigMap 被标记为不可变更，则 *无法* 逆转这一变化，，也无法更改 `data` 或 `binaryData` 字段的内容。你只能删除并重建 ConfigMap。 因为现有的 Pod 会维护一个已被删除的 ConfigMap 的挂载点，建议重新创建这些 Pods

### 持久化存储

#### volumes

Kubernetes 支持很多类型的卷。 [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 可以同时使用任意数目的卷类型。 [临时卷](https://kubernetes.io/zh-cn/docs/concepts/storage/ephemeral-volumes/)类型的生命周期与 Pod 相同， 但[持久卷](https://kubernetes.io/zh-cn/docs/concepts/storage/persistent-volumes/)可以比 Pod 的存活期长。 当 Pod 不再存在时，Kubernetes 也会销毁临时卷；不过 Kubernetes 不会销毁持久卷。 对于给定 Pod 中任何类型的卷，在容器重启期间数据都不会丢失。

##### HostPath

`hostPath` 卷能将主机节点文件系统上的文件或目录挂载到你的 Pod 中。 虽然这不是大多数 Pod 需要的，但是它为一些应用程序提供了强大的逃生舱。

###### hostPath 配置示例

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx
    imagePullPolicy: IfNotPresent
    name: test-container
    volumeMounts:
    - mountPath: /test-pd
      name: test-volume
      readOnly: true
  volumes:
  - name: test-volume
    # 与主机共享目录，加载主机中的指定目录到文件中。
    hostPath:		 
      # 宿主机上目录位置
      path: /data
      # 此字段为可选
      type: Directory
~~~

hostPath 支持的 `type` 值如下：

|                   | 空字符串（默认）用于向后兼容，这意味着在安装 hostPath 卷之前不会执行任何检查。 |
| ----------------- | ------------------------------------------------------------ |
| DirectoryOrCreate | 如果在给定路径上什么都不存在，那么将根据需要创建空目录，权限设置为 0755，具有与 kubelet 相同的组和属主信息。 |
| Directory         | 在给定路径上必须存在的目录。                                 |
| FileOrCreate      | 如果在给定路径上什么都不存在，那么将在那里根据需要创建空文件，权限设置为 0644，具有与 kubelet 相同的组和所有权。 |
| File              | 在给定路径上必须存在的文件。                                 |
| Socket            | 在给定路径上必须存在的 UNIX 套接字。                         |
| CharDevice        | 在给定路径上必须存在的字符设备。                             |
| BlockDevice       | 在给定路径上必须存在的块设备。                               |

> **警告：**
>
> HostPath 卷存在许多安全风险，最佳做法是尽可能避免使用 HostPath。 当必须使用 HostPath 卷时，它的范围应仅限于所需的文件或目录，并以只读方式挂载。
>
> 如果通过 AdmissionPolicy 限制 HostPath 对特定目录的访问，则必须要求 `volumeMounts` 使用 `readOnly` 挂载以使策略生效。

##### EmptyDir

EmptyDir 主要用于一个 pod 中不同的 Container 共享数据使用，由于只是在 pod 内使用，因此与其他 volume 比较大的区别是，当 pod 如果被删除了，那么 EmptyDir 也会被删除，但是容器崩溃并**不**会导致 Pod 被从节点上移除，因此容器崩溃期间 `emptyDir` 卷中的数据是安全的。

`emptyDir.medium` 字段用来控制 `emptyDir` 卷的存储位置。 默认情况下，`emptyDir` 卷存储在该节点所使用的介质上； 此处的介质可以是磁盘、SSD 或网络存储，这取决于你的环境。 你可以将 `emptyDir.medium` 字段设置为 `"Memory"`， 以告诉 Kubernetes 为你挂载 tmpfs（基于 RAM 的文件系统）。 虽然 tmpfs 速度非常快，但是要注意它与磁盘不同， 并且你所写入的所有文件都会计入容器的内存消耗，受容器内存限制约束。

你可以通过为默认介质指定大小限制，来限制 `emptyDir` 卷的存储容量。 此存储是从[节点临时存储](https://kubernetes.io/zh-cn/docs/concepts/configuration/manage-resources-containers/#setting-requests-and-limits-for-local-ephemeral-storage)中分配的。 如果来自其他来源（如日志文件或镜像分层数据）的数据占满了存储，`emptyDir` 可能会在达到此限制之前发生存储容量不足的问题。

###### emptyDir 配置示例

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-emptydir
spec:
  containers:
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: test-podvolume-1
    command: ["/bin/sh","-c","sleep 3600;"]
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  - image: alpine
    imagePullPolicy: IfNotPresent
    name: test-podvolume-2
    command: ["/bin/sh","-c","sleep 3600;"]
    volumeMounts:
    - mountPath: /opt
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}
```

#### NFS 挂载

`nfs` 卷能将 NFS (网络文件系统) 挂载到你的 Pod 中。 不像 `emptyDir` 那样会在删除 Pod 的同时也会被删除，`nfs` 卷的内容在删除 Pod 时会被保存，卷只是被卸载。 这意味着 `nfs` 卷可以被预先填充数据，并且这些数据可以在 Pod 之间共享。

##### nfs 配置示例

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pod
spec:
  containers:
  - image: nginx
    name: test-container
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: test-volume
  volumes:
  - name: test-volume
    nfs:
      # ip 或域名都可以
      server: 172.129.78.132			
      # 网络路径
      path: /nfs/
      # 可选，是否只读
      readOnly: true
~~~



> **说明：**
>
> 在使用 NFS 卷之前，你必须运行自己的 NFS 服务器并将目标 share 导出备用。
>
> 还需要注意，不能在 Pod spec 中指定 NFS 挂载可选项。 可以选择设置服务端的挂载可选项，或者使用 [/etc/nfsmount.conf](https://man7.org/linux/man-pages/man5/nfsmount.conf.5.html)。 此外，还可以通过允许设置挂载可选项的持久卷挂载 NFS 卷。



#### PV 与 PVC

##### 生命周期

- 构建

  - 静态构建

    集群管理员创建若干 PV卷，这些卷对象带有真实存储的细节信息，并且对集群用户可用（可见），PV 卷对象存在与 Kubernetes API 中，可供用户消费（使用）。

  - 动态构建

    如果集群中已经有的 PV 无法满足 PVC 的需求，那么集群会根据 PVC 自动构建一个 PV，该操作是通过 StorageClass 是现实的。
    
    想要实现这个操作，前提是 PVC 必须设置 StorageClass，否则会无法动态构建该 PV，可以通过弃用 DefaultStorageClass 来实现 PV 的构建。

- 绑定

  当用户创建了一个 PVC 对象后，主节点会检测新的 PVC 对象，并且寻找与之匹配的 PV 卷，找到 PV 卷后将二者绑定在一起。

  如果找不到对象的 PV，则需要看 PVC 是否设置 StorageClass 来决定是否动态创建 PV，若没有配置，PVC 就会一直处于未绑定状态，直到有与之匹配的 PV 后才会申领绑定关系。

- 使用

  Pod 将 PVC 当作存储卷来使用，群集会通过 PVC 找到绑定的 PV，并为 Pod 挂在该卷。

  Pod 一旦使用 PVC 绑定 PV 后，为了保护数据，避免数据丢失问题，PV 对象会受到保护，在系统中无法被删除。

- 回收策略

  1. Retain -- 手动回收

     当 PVC 对象被删除时，PV 卷仍然存在，对应的数据卷被视为“已释放（released）”。由于卷上仍然存在这前一申领的人的数据，该卷还不能用于其他申领。

  2. Recycle -- 简单擦除 (`rm -rf /thevolume/*`)

  3. Delete -- 删除关联存储资产

     默认就是 Delete

##### PV

###### 状态

- Available: 空闲，未被绑定
- Bound：已经被 PVC 绑定
- Released：PVC 被删除，资源已回收，但是 PV 未被重新使用
- Failed：自动回收失败

###### 配置文件

~~~yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv001
spec:
  capacity:		#  容量配置
    storage: 5Gi		# PV 的容量
  volumeMode: Filesystem		# 存储类型为文件系统
  accessModes:		# 访问模式
    - ReadWriteOnce		# 可被单节点读写
  persistentVolumeReclaimPolicy: Retain		# 回收策略
  storageClassName: slow		# 创建 PV 的存储类名，需要与 PVC 相同
  mountOptions:		# 加载配置
    - hard
    - nfsvers=4.1
  nfs:		# 连接到 nfs
    path: /data/
    server: my-share.nfs.server
~~~

accessMode 访问模式

- ReadWriteOnce

  卷可以被一个节点以读写方式挂载。 ReadWriteOnce 访问模式也允许运行在同一节点上的多个 Pod 访问卷。

- ReadOnlyMany

  卷可以被多个节点以只读方式挂载。

- ReadWriteMany

  卷可以被多个节点以读写方式挂载。

- ReadWriteOncePod

  特性状态： Kubernetes v1.27 [beta]

  卷可以被单个 Pod 以读写方式挂载。 如果你想确保整个集群中只有一个 Pod 可以读取或写入该 PVC， 请使用 ReadWriteOncePod 访问模式。这只支持 CSI 卷以及需要 Kubernetes 1.22 以上版本。

##### PVC

###### Pod 绑定 PVC

~~~yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: nginx
    name: test-container
    imagePullPolicy: IfNotPresent
    volumeMounts:
    - mountPath: /usr/share/nginx/html
      name: test-volume
  volumes:
  - name: test-volume
    persistentVolumeClaim:		# PVC 配置
      claimName: nfs-pvc		# 要关联到哪个 PVC
~~~

###### 配置文件

~~~yaml
apiVersion: v1
kind: PersistentVolumeClaim
medatada:
  name: nfs-pvc
spec:
  accessModes:		# 如果你想指定 PV，这个类型必须要和你指定的 PV 的类型一样
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 5Gi		# 资源可以小于 PV 的，但是不能大于，如果大于就会匹配不到 pv
  storageClassName: slow		# 名字需要与对应的 pv 相同
#  selector: 		# 使用选择器选择对应的 PV
#    matchLabels:
#      release: "stable"
#    matchExpressions:
#      - {key: environment, operator: In, values: [dev,ws]}
~~~



#### StorageClassNFS

Kubernetes提供了一套可以自动创建PV的机制,即:Dynamic Provisioning.而这个机制的核心在于:StorageClass这个API对象.

StorageClass对象会定义下面两部分内容:
1,PV的属性.比如,存储类型,Volume的大小等.
2,创建这种PV需要用到的存储插件
有了这两个信息之后,Kubernetes就能够根据用户提交的PVC,找到一个对应的StorageClass,之后Kubernetes就会调用该StorageClass声明的存储插件,进而创建出需要的PV.
但是其实使用起来是一件很简单的事情,你只需要根据自己的需求,编写YAML文件即可,然后使用kubectl create命令执行即可

~~~shell
mkdir storageclass
cd storageclass

# 编辑rbac.yaml
vim rbac.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
    # replace with namespace where provisioner is deployed
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    # replace with namespace where provisioner is deployed
    namespace: default
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
  
# 编辑 nfs-provisioner.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
  # replace with namespace where provisioner is deployed
  namespace: default  #与RBAC文件中的namespace保持一致
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nfs-client-provisioner
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          image: quay.io/alberthua/nfs-client-provisioner
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: ws-nfs-storage  #provisioner名称,请确保该名称与 nfs-StorageClass.yaml文件中的provisioner名称保持一致
            - name: NFS_SERVER
              value: 172.129.78.101    #NFS Server IP地址
            - name: NFS_PATH  
              value: /share    #NFS挂载卷
      volumes:
        - name: nfs-client-root
          nfs:
            server: 172.129.78.101  #NFS Server IP地址
            path: /share     #NFS 挂载卷
            
# 编辑 nfs-storageclass.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-nfs-storage
  annotations:
    "storageclass.kubernetes.io/is-default-class": "true"   #添加此注释
provisioner: ws-nfs-storage #or choose another name, must match deployment's env PROVISIONER_NAME'
parameters:
  archiveOnDelete: "false"

[root@k8s-master storageclass]# ll
total 16
-rw-r--r-- 1 root root 1295 Apr 15 13:02 nfs-provisioner.yaml
-rw-r--r-- 1 root root  317 Apr 15 12:22 nfs-storageclass.yaml
-rw-r--r-- 1 root root 1744 Apr 15 08:31 rbac.yaml
-rw-r--r-- 1 root root  242 Apr 15 12:17 test-pvc.yaml
~~~



## 身份验证和权限

### 认证

所有 Kubernetes 集群中有两类用户：由 Kubernetes 管理的 ServiceAccount （服务账号）和（Users Accounts） 普通账户。

普通账户是假定被外部或独立服务管理的，由管理员分配 keys，用户像使用 Keystone 或 google 账户一样，被存储在包含 usernames 和passwords 的 list 文件里。

需要注意：在 Kubernetes 中不能通过 API 调用将普通用户添加到集群中。

普通账户和服务账户的区别

- 普通账户时针对（人）用户的，而服务账户针对 pod 进程。
- 普通账户是全局性的，在集群中所有 namespaces 中，名字具有唯一性
- 通常，集群的普通用户可以于企业数据库同步。服务账号创建目的是更轻量化，允许集群用户为特性任务创建服务账户。
- 普通账户和服务账户的深刻注意事项不同。
- 对于复杂系统的配置包，可以包括对该系统的各种组件的服务账号进行定义。

#### User Accounts

用户账户

#### Service Accounts

当用户访问集群（例如使用kubectl命令）时，apiserver 会将您认证为一个特定的 User Account（目前通常是admin，除非您的系统管理员自定义了集群配置）。
Pod 容器中的进程也可以与 apiserver 联系。 当它们在联系 apiserver 的时候，它们会被认证为一个特定的 Service Account。

因为kubernetes是高度模块化，所有认证方式和授权方式都可以通过插件的方式让客户自定义的，可以支持很多种。客户端请求的时候首先需要进行认证，
认证通过后再进行授权检查，因有些操作需要级联到其他资源或者环境，但是级联环境是否有授权权限，这时候需要准入控制。

##### Service Account 自动化

- Service Account Admission Controller

  通过 Admission Controller 插件来实现对 pod 修改，它是 apiserver 的一部分，创建或更新 pod 时会同步进行修改 pod。当插件处于激活状态（在大多数发行版中默认激活）创建或者修改时，会按一下操作执行：

  1. 如果 pod 没有设置 ServiceAccount，则将 ServiceAccount 设置为 default。
  2. 确保 pod 引用的 ServiceAccount 存在，否则将会拒绝请求。
  3. 如果 pod 不包含任何 imagePullSecrets,则将 ServiceAccount 中的 ImagePullSecret 添加到 pod 中。
  4. 为包含 API 访问的 Token  的 pod 添加一个 volume，同时会把 volume Source 添加到 pod 的每个容器中，默认挂载在 /var/run/secrets/kubernetes.io/serviceaccount。

  每一个 pod 都需要绑定 imagePullSecrets 很麻烦，我们只需要把 imagePullSecrets 绑定到 Service Account上，再把需要这个 imagePullSecrets 的 pod 绑定上这个 Service Account 上，就方便很多了。

  假设一个 pod 要访问一个 API，而 API 的认证形式是 token 认证形式，我们创建的 Service Account，他都会关联到一个 token，这个 pod 想要访问对应的 API 的时候，我们必须要有token，来证明这个 pod 是有这个权限的，这个 token 就是放在 volume 里的，它会自动的把 volume 放在这个pod里，不需要我们创建 pod 的时候，再手动配置。

- Token Controller

  当用 kubectl 命令执行命令时，kubectl 会根据执行的命令来找对应的 apiserver 里的资源，在访问 apiserver 时，apiserver 会让你出示你的身份，身份通过才会让访问 apiserver 里的资源，这个身份就是 token，每个用户都由一个 token，只能忍着成功，你的kubectl命令才有效。

- Service Account Controller

  Service Account Controller 在 namespace 里管理 ServiceAccount，并确保每个有效的 namespaces 中都存在一个名为 “default”的 ServiceAccount。

##### 创建service account

创建一个名为 backend-sa 的新 ServiceAccount， 确保此 ServiceAccount 不自动挂载 API 凭据。

~~~yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: backend-sa
automountServiceAccountToken: false
~~~

pod 绑定 Service Account

~~~yaml
...
spec:
  serviceAccountName: backend-sa			# 指定 sa 的名称就能绑定上 Service Account，没指定默认是 default。
~~~



### 授权（RBAC）

RBAC：Role-Based Access Control（基于角色的访问控制）

用户权限的配置是基于角色来进行访问控制的，我们多个不同的用户可能有相同的权限，如果我们一个一个绑定权限，配置起来是不是很不方便，我们如果有一个角色，这个角色有我们用户想用的权限、

#### Role && ClusterRole

代表一个角色，会包含一组权限，没有拒绝规则，只是附加允许。它是 Namespace 级别的资源，只能作用与 Namespace 之内。

ClusterRole 功能与 Role 一样，区别时资源类型为群集类型，而 Role 只能定义 namespace 级别的资源的权限。

##### 配置文件

~~~yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" 标明 core API 组
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
~~~



#### RoleBinding && ClusterRoleBinding

Role 或 ClusterRole 只是用于制定权限集合，具体作用于什么对象上，需要使用 RoleBinding 来进行绑定。

作用于 Namespace 内，可以将 Role 或者 ClusterRole 绑定到 User,Group,Service Account 上。

##### 示例：

~~~yaml
piVersion: rbac.authorization.k8s.io/v1
# 此角色绑定允许 "jane" 读取 "default" 名字空间中的 Pod
# 你需要在该名字空间中有一个名为 “pod-reader” 的 Role
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# 你可以指定不止一个“subject（主体）”
- kind: User
  name: jane # "name" 是区分大小写的
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" 指定与某 Role 或 ClusterRole 的绑定关系
  kind: Role        # 此字段必须是 Role 或 ClusterRole
  name: pod-reader  # 此字段必须与你要绑定的 Role 或 ClusterRole 的名称匹配
  apiGroup: rbac.authorization.k8s.io
~~~



## 网络原理

### CNI

CNI 是 Container Network Interface（容器网络接口）的缩写，它是一个用于定义和实现容器网络的规范，旨在使容器运行时能够与网络插件进行通信，并配置容器的网络连接。

在 Kubernetes 中，CNI 扮演着非常重要的角色，它定义了如何创建和管理容器的网络连接，包括分配 IP 地址、配置路由、设置网络策略等。CNI 的设计使得 Kubernetes 可以支持各种网络插件，例如 Calico、Flannel、Weave Net 等，这些网络插件可以根据 CNI 的规范来实现网络功能，并与 Kubernetes 集成。

Kubernetes 它需要网络插件来提供集群内部和集群外部的网络通信。

以下是一些常用的 k8s CNI 网络插件：

- Flannel：Flannel 是最常用的 k8s 网络插件之一，它使用了虚拟网络技术来实现容器之间的通信，支持多种网络后端，如 VXLAN、UDP 和 Host-GW。
- Calico：Calico 是一种基于 BGP 的网络插件，它使用路由表来路由容器之间的流量，支持多种网络拓扑结构，并提供了安全性和网络策略功能。
- Canal：Canal 是一个组合了 Flannel 和 Calico 的网络插件，它使用 Flannel 来提供容器之间的通信，同时使用 Calico 来提供网络策略和安全性功能。
- Weave Net：Weave Net 是一种轻量级的网络插件，它使用虚拟网络技术来为容器提供 IP 地址，并支持多种网络后端，如 VXLAN、UDP 和 TCP/IP，同时还提供了网络策略和安全性功能。
- Cilium：Cilium 是一种基于 eBPF (Extended Berkeley Packet Filter) 技术的网络插件，它使用 Linux 内核的动态插件来提供网络功能，如路由、负载均衡、安全性和网络策略等。
- Contiv：Contiv 是一种基于 SDN 技术的网络插件，它提供了多种网络功能，如虚拟网络、网络隔离、负载均衡和安全策略等。
- Antrea：Antrea 是一种基于 OVS (Open vSwitch) 技术的网络插件，它提供了容器之间的通信、网络策略和安全性等功能，还支持多种网络拓扑结构。

CNI 网络插件使用封装网络模型（例如 Virtual Extensible Lan，缩写是 [VXLAN](https://github.com/flannel-io/flannel/blob/master/Documentation/backends.md?spm=a2c6h.12873639.article-detail.14.3708741936bTgG#vxlan)）或非封装网络模型（例如 Border Gateway Protocol，缩写是 [BGP](https://en.wikipedia.org/wiki/Border_Gateway_Protocol)）来实现网络结构。

#### calico

##### 模式

calico默认使用的是`IPIP`

- #### **IPIP**

  - 从字面上理解，就是把一个IP数据包又套在一个IP包里，即把IP层封装到IP层的一个Tunnel。它的作用其实基本上就相当于一个基于IP层的网桥。一般来说，普通的网桥是基于MAC层的，不需要IP，而这个IP则是**通过两端的路由做一个Tunnel，把两个本来不通的网络通过点对点连接起来**。

- **BGP**

  - BGP，通俗的讲就是讲接入到机房的多条线路（如电信、联通、移动等）融合为一体，实现多线单IP，BGP 机房的优点：服务器只需要设置一个IP地址，最佳访问路由是由网络上的骨干路由器根据路由跳数与其它技术指标来确定的，不会占用服务器的任何系统。

参考网站：[kubernetes网络组件calico详解 - yuhaohao - 博客园 (cnblogs.com)](https://www.cnblogs.com/yuhaohao/p/14103844.html)

##### 部署 calico

calico/node：每个节点服务器上的代理，提供felix、bird4、bird6和confd等守护进程,可以在每个节点上独立与k8s集群部署，也可以以deamonset，运行在k8s集群上（后面实例也才有这种方式）

calico/kube-controller：calico与k8s协同的插件

~~~shell
# 获取清单文件
curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.26.1/manifests/calico.yaml

# 清单中的重要配置
4770             # Enable IPIP
4771             - name: CALICO_IPV4POOL_IPIP
4772               value: "Always"
4773             # Enable or Disable VXLAN on the default IP pool.
4774             - name: CALICO_IPV4POOL_VXLAN
4775               value: "Never"

###
CALICO_IPV4POOL_IPIP：启用IPIP隧道配置
Always：全流量使用IPIP隧道转发
Cross-SubNet：跨网段节点使用IPIP隧道转发，同一网段节点使用BGP路由直接转发
Never：不启用IPIP
CALICO_IPV4POOL_VXLAN:：启用VXLAN配置
Always：全流量使用VXLAN隧道转发
Cross-SubNet：跨网段节点使用VXLAN隧道转发，同一网段节点使用BGP路由直接转发
Never：不启用VXLAN
###

# 部署
kubectl apply -f calico.yaml
~~~

#### Flannel





### Networkpolicy

如果你希望针对 TCP、UDP 和 SCTP 协议在 IP 地址或端口层面控制网络流量， 则你可以考虑为集群中特定应用使用 Kubernetes 网络策略（NetworkPolicy）。前提条件是如果你想使用 networkpolicy 必须需用一个 CNI （容器网络接口）插件。

说明：flannel不支持Networkpolicy，仅只有 calico 支持。

要了解容器运行时如何管理 CNI 插件的具体信息，可参见对应容器运行时的文档

containerd：https://github.com/containerd/containerd/blob/main/script/setup/install-cni

CRI-O：[cri-o/contrib/cni/README.md at main · cri-o/cri-o (github.com)](https://github.com/cri-o/cri-o/blob/main/contrib/cni/README.md)

#### pod 隔离

pod有两种隔离（类型），一种是出口（`egress`），一种是入口(`ingress`)。

默认情况下，一个 Pod 的出口是非隔离的，即所有外向连接都是被允许的。如果有任何的 NetworkPolicy 选择该 Pod 并在其 `policyTypes` 中包含 “Egress”，则该 Pod 是出口隔离的， 我们称这样的策略适用于该 Pod 的出口。当一个 Pod 的出口被隔离时， 唯一允许的来自 Pod 的连接是适用于出口的 Pod 的某个 NetworkPolicy 的 `egress` 列表所允许的连接。 这些 `egress` 列表的效果是相加的。

默认情况下，一个 Pod 对入口是非隔离的，即所有入站连接都是被允许的。如果有任何的 NetworkPolicy 选择该 Pod 并在其 `policyTypes` 中包含 “Ingress”，则该 Pod 被隔离入口， 我们称这种策略适用于该 Pod 的入口。当一个 Pod 的入口被隔离时，唯一允许进入该 Pod 的连接是来自该 Pod 节点的连接和适用于入口的 Pod 的某个 NetworkPolicy 的 `ingress` 列表所允许的连接。这些 `ingress` 列表的效果是相加的。

网络策略不冲突;它们是累加的。如果任何策略应用于给定方向的给定容器，则允许从该容器向该方向建立的连接是适用策略允许的并集。因此，评估顺序不会影响策略结果。

要允许从源 Pod 到目标 Pod 的连接，源 Pod 上的出口策略和目标 Pod 上的入口策略都需要允许连接。如果任何一方不允许连接，则不会发生。

官方文档：[Network Policies | Kubernetes --- 网络策略 ](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

##### 策略详解

**必需字段**：与所有其他的 Kubernetes 配置一样，NetworkPolicy 需要 `apiVersion`、 `kind` 和 `metadata` 字段。关于配置文件操作的一般信息， 请参考[配置 Pod 以使用 ConfigMap](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/configure-pod-configmap/) 和[对象管理](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/object-management)。

**spec**：NetworkPolicy [规约](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-architecture/api-conventions.md#spec-and-status) 中包含了在一个名字空间中定义特定网络策略所需的所有信息。

**podSelector**：每个 NetworkPolicy 都包括一个 `podSelector`， 它对该策略所适用的一组 Pod 进行选择。示例中的策略选择带有 "role=db" 标签的 Pod。 空的 `podSelector` 选择名字空间下的所有 Pod。

**policyTypes**：每个 NetworkPolicy 都包含一个 `policyTypes` 列表，其中包含 `Ingress` 或 `Egress` 或两者兼具。`policyTypes` 字段表示给定的策略是应用于进入所选 Pod 的入站流量还是来自所选 Pod 的出站流量，或两者兼有。 如果 NetworkPolicy 未指定 `policyTypes` 则默认情况下始终设置 `Ingress`； 如果 NetworkPolicy 有任何出口规则的话则设置 `Egress`。

**ingress**：每个 NetworkPolicy 可包含一个 `ingress` 规则的白名单列表。 每个规则都允许同时匹配 `from` 和 `ports` 部分的流量。示例策略中包含一条简单的规则： 它匹配某个特定端口，来自三个来源中的一个，第一个通过 `ipBlock` 指定，第二个通过 `namespaceSelector` 指定，第三个通过 `podSelector` 指定。

**egress**：每个 NetworkPolicy 可包含一个 `egress` 规则的白名单列表。 每个规则都允许匹配 `to` 和 `ports` 部分的流量。该示例策略包含一条规则， 该规则将指定端口上的流量匹配到 `10.0.0.0/24` 中的任何目的地。

所以，该网络策略示例:

1. 隔离 `default` 名字空间下 `role=db` 的 Pod （如果它们不是已经被隔离的话）。
2. （Ingress 规则）允许以下 Pod 连接到 `default` 名字空间下的带有 `role=db` 标签的所有 Pod 的 6379 TCP 端口：
   - `default` 名字空间下带有 `role=frontend` 标签的所有 Pod
   - 带有 `project=myproject` 标签的所有名字空间中的 Pod
   - IP 地址范围为 172.17.0.0–172.17.0.255 和 172.17.2.0–172.17.255.255 （即，除了 172.17.1.0/24 之外的所有 172.17.0.0/16）
3. （Egress 规则）允许 `default` 名字空间中任何带有标签 `role=db` 的 Pod 到 CIDR 10.0.0.0/24 下 5978 TCP 端口的连接。

~~~yaml
# yaml 解释
apiVersion: v1
kind: NetworkPolicy
metadata:
  name: ws
  namespace: ws			# 定义的是属于命名空间 ws 下的 pod，下面写的ingress，和egress，都是围绕这个命名空间来定义的。
spec: 
  podSelector:			# 定义 pod 策略。
    matchLabls:				# 命名空间 ws 下的 pod 标签来进行匹配。
      role: db			# 如果 pod 中有 role=db 这个标签的 pod，就会被匹配到，那就是这个网络策略是这些有 role=db 的标签来进行定义的。
  policyTypes:			# 表示给定的策略是应用于进入所选 Pod 的入站流量还是来自所选 Pod 的出站流量，或两者兼有。 如果 NetworkPolicy 未指定 policyTypes 则默认情况下始终设置 Ingress； 如果 NetworkPolicy 有任何出口规则的话则设置 Egress。
    - Ingress			# ingress 入站流量的规则。
    - Egress			# egress 出站流量的规则，如果 policyTypes 字段设置了这个规则，表示这个规则生效。
  ingress:			# 定义入站流量的规则的白名单，每个规则都允许同时匹配 `from` 和 `ports` 部分的流量。
    - from:
        - ipBlock: 		# 白名单，匹配 ip 的规则
            cidr: 11.1.0.0/16 # 在 11.1.0.0/16 范围内的所有 ip，除了11.1.2.0/24，都会被这条策略所匹配上。
            except:
              - 11.1.2.0/24
        - namespaceSelector: 	# 白名单，匹配命名空间的规则
            matchLabels:
              project: ws
        - podSelector:		  # 白名单，匹配 pod 的规则
            matchLabels:
              role: frontend
    - ports:	
        - protocol: TCP		# 协议类型
          port: 6379	# 端口号
  egress: 			# 定义入站流量的规则的白名单，每个规则都允许匹配 `to` 和 `ports` 部分的流量
    - to: 
        - ipBlock:
            cidr: 10.0.0.0/24
    - ports: 
        - portocol: TCP
          port: 5678
# 默认拒绝所有入站流量
# 只在 policyTypes 设置了 Ingress，而没有定义 ingress 的白名单，表示拒绝所有入站流量，policyTypes 里没有设置 egress 表示允许所有出站流量。
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}			# podSelector 字段内无内容表示匹配所有命名空间以及 pod。
  policyTypes:
  - Ingress

# 允许所有入站流量
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-all-ingress
spec:
  podSelector: {}
  ingress:			# ingress 白名单
  - {}				# 表示所有
  policyTypes:
  - Ingress
~~~



#### 练习

##### 示例1

Context 

一个默认拒绝（default-deny）的 NetworkPolicy 可避免在未定义任何其他 NetworkPolicy 的 namespace 中意外公开 Pod。 

Task 

为所有类型为 Ingress+Egress 的流量在 namespace testing 中创建一个名为 denypolicy 的新默认拒绝 NetworkPolicy。 

此新的 NetworkPolicy 必须拒绝 namespace testing 中的所有的 Ingress + Egress 流量。 

将新创建的默认拒绝 NetworkPolicy 应用与在 namespace testing 中运行的所有 Pod。

~~~yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: denypolicy
  namespace: testing
spec:
  podSelector: {}
  policyTypes:
    - Ingress
    - Egress
~~~



##### 示例2

创建一个名为 pod-restriction 的 NetworkPolicy 来限制对在 namespace dev-team 中运行的 Pod products-service 的访问。 

只允许以下 Pod 连接到 Pod products-service

- namespace qaqa 中的 Pod。
- 只允许位于 namespace ws 中，带有标签 environment: testing 的 Pod。

~~~yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: 
  name: pod-restriction
  namespace: dev-team
spec: 
  podSelector:
     matchLabels:
       environment: testing
  policyTypes:
    - Ingress
  ingress: 
    - from:
        - namespaceSelector:		# 第一个限制条件
            matchLabels:
              name: qaqa
        - namespaceSelector:		# 第二个限制条件
            matchLabels:
              name: ws
          podSelector:
            matchLabels:
              environment: testing
~~~



## Helm

helm是k8s的包管理工具，类似于centos的yum，k8s将管理的资源都抽象成api，并且推荐使用声明方式创建，修改，删除这些对象，每个 API 对象都通过一个 yaml 格式或者 json 格式的文本来声明。这带来的一个问题就是这些 API 对象声明文本的管理成本，每当我需要创建一个应用，都需要去编写一堆这样的声明文件，helm就是管理这些api对象的工具，它把创建一个应用所需的所有 Kubernetes API 对象声明文件组合并打包在一起。并提供了仓库的机制便于分发共享，还支持模版变量替换，同时还有版本的概念，使之能够对一个应用进行版本的管理。

### 安装 Helm 客户端

Helm 下载地址：[Releases · helm/helm (github.com)](https://github.com/helm/helm/releases)

~~~shell
# 安装 Helm，添加 helm 仓库，当前环境是内网，内网环境手动下载安装，上方有下载地址
wget https://get.helm.sh/helm-v3.14.0-rc.1-linux-amd64.tar.gz

# 解压文件
tar -zxvf helm-v3.14.0-rc.1-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/local/bin/helm

# 安装完成，查看版本
helm version
~~~



### Helm 概念

- `Chart`：表示 helm 包，包含在 Kubernetes 集群内部运行应用程序，工具或服务所需的所有资源定义。
- `Reposityroy（仓库）`：用来存放和共享 charts 的地方。
- `Release`：运行在 Kubernetes 集群中的 chart 的实例，一个 chart 通常在同一个集群中安装多次，每一次安装都会创建一个新的 release。

### Helm 命令

- helm repo list：列出，增加，删除 chart 仓库
- helm search：使用关键字搜索 
- helm pull：拉取远程仓库中的 chart 到本地
- helm create：在本地创建新的 chart
- helm dependency：管理chart 依赖
- helm install：安装 chart
- helm list：列出所有 release
- helm lint：检查 chart 配置是否有误
- helm package：打包本地 chart
- helm rollback：回滚 release 到历史版本
- helm uninstall：卸载 release
- helm upgrade：升级 release



配置 helm 仓库

~~~shell
# 使用 helm repo add 命令来创建仓库
Usage: helm repo add [NAME] [URL] [flags]

# 阿里云仓库
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

# 微软仓库
helm repo add stable http://mirror.azure.cn/kubernetes/charts

# bitnami 仓库
helm repo add my-repo https://charts.bitnami.com/bitnami

# 更新删除 仓库
helm repo update aliyun 
helm repo remove aliyun 
~~~



创建一个 chart

~~~shell
# 在当前目录创建一个名为 nginx 的chart 
helm create nginx		

# 配置 chart
# 删除默认配置
rm -rf nginx/templates/*

# 配置 deployment
vim nginx/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: {{ .Values.nginx.replica }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
# 配置 service
vim nginx/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: service-nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30007
# 配置变量
vim nginx/values.yaml
nginx:
  replica: 5
  
# 运行刚才编写的 chart
helm install nginx ./nginx/

# 查看
helm list
NAME    NAMESPACE       REVISION        UPDATED                                 STATUS          CHART   APP VERSION
nginx   default         2               2024-03-26 11:53:22.731032139 +0000 UTC deployed        test-v2    
~~~



更新

~~~shell
vim nginx/values.yaml
nginx:
  replica: 2

# 使用 helm upgrade 更新 chart
helm upgrade nginx nginx/ -f ./nginx/values.yaml
~~~



回滚

~~~shell
# 使用 helm rollback 命令进行回滚
helm rollback nginx 1		# 1 就是我们之前的版本，回滚之后副本数就会变成 5
~~~



## 普罗米修斯（Prometheus）

普罗米修斯（Prometheus）是一种开源的监控和警报工具，用于记录和查看系统、服务和应用程序的性能指标数据。它最初由 SoundCloud 开发，并于2012年作为开源项目发布。后来，Prometheus 成为了 Cloud Native Computing Foundation（CNCF）的一部分，成为了云原生监控领域的标准之一。

### 使用 kube-prometheus 安装

#### 下载 kube-prometheus

~~~shell
git clone https://github.com/prometheus-operator/kube-prometheus.git
~~~



#### **kube-prometheus 部署**

~~~shell
# 修改 service 的类型为 NodePort，我现在的版本是现在最新版的，到时候你们下载的肯定不跟我的一样。
vim manifests/prometheus-service.yaml
  1 apiVersion: v1
  2 kind: Service
  3 metadata:
  4   labels:
  5     app.kubernetes.io/component: prometheus
  6     app.kubernetes.io/instance: k8s
  7     app.kubernetes.io/name: prometheus
  8     app.kubernetes.io/part-of: kube-prometheus
  9     app.kubernetes.io/version: 2.50.1
 10   name: prometheus-k8s
 11   namespace: monitoring
 12 spec:
 13   type: NodePort			# 添加
 14   ports:
 15   - name: web
 16     port: 9090
 17     targetPort: web
 18   - name: reloader-web
 19     port: 8080
 20     targetPort: reloader-web
 21   selector:
 22     app.kubernetes.io/component: prometheus
 23     app.kubernetes.io/instance: k8s
 24     app.kubernetes.io/name: prometheus
 25     app.kubernetes.io/part-of: kube-prometheus
 26   sessionAffinity: ClientIP
vim manifests/alertmanager-service.yaml
  type: NodePort
vim manifests/grafana-service.yaml
  type: NodePort
  
# 更改镜像源
# 查找
grep -rn 'quay.io' *
# 批量替换
sed -i 's/quay.io/quay.mirrors.ustc.edu.cn/g' `grep "quay.io" -rl *`
# 再查找
grep -rn 'quay.io' *
grep -rn 'image: ' *

# 使用中科大的镜像源也是有两个镜像拉取不了，我们需要手动拉取
docker pull bitnami/kube-state-metrics:v2.11.0
docker tag bitnami/kube-state-metrics:2.11.0 registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.11.0
docker pull v5cn/prometheus-adapter:v0.10.0
docker tag v5cn/prometheus-adapter:v0.10.0 registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.10.0

# 拉取好镜像文件后，便可以开始部署kube-prometheus了，以此执行以下命令：
 
kubectl apply --server-side -f manifests/setup
kubectl wait \
	--for condition=Established \
	--all CustomResourceDefinition \
	--namespace=monitoring
kubectl apply -f manifests/
~~~



#### 访问测试

~~~shell
# 现在还无法访问grafan、prometheus以及alertmanger，因为prometheus operator内部默认配置了NetworkPolicy，需要删除其对应的资源，才可以通过外网访问：
kubectl delete -f manifests/prometheus-networkPolicy.yaml
kubectl delete -f manifests/grafana-networkPolicy.yaml
kubectl delete -f manifests/alertmanager-networkPolicy.yaml
# 删除后，通过服务器ip:服务端口的形式，即可访问对应的服务了，在此，kube-prometheus的部署彻底完成。
~~~

参考文档：

https://blog.csdn.net/weixin_45112997/article/details/125968773?ops_request_misc=&request_id=&biz_id=102&utm_term=kube-prometheus&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduweb~default-4-125968773.nonecase&spm=1018.2226.3001.4187

https://blog.csdn.net/slc09/article/details/132571091



## Kubernetes 可视化界面

### kubernetes dashboard

官方文档：[部署和访问 Kubernetes 仪表板（Dashboard） | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/access-application-cluster/web-ui-dashboard/)

~~~shell
# 下载 Kubernetes 仪表板，我当前是 1.29 版本，所以下载 2.7.0，你们下载的时候要看官方文档部署。
wget https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml

vim recommended.yaml
kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort			# 类型设置为 NodePort，新加一行
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
   
# 创建 dashboard
kubectl apply -f recommended.yaml

# 再创建一个账户，用处为登录 dashboard 页面所使用的账号
vim auth.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dashboard-admin
subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io

kubectl apply -f auth.yaml

# 找到登录的 token
kubectl get secret -n kube-system|grep admin|awk '{print $1}'

# 找不到的话，可以用以下命令生成
kubectl create token -n kube-system dashboard-admin
~~~



### kubesphere

官方文档：[在 Kubernetes 上最小化安装 KubeSphere](https://www.kubesphere.io/zh/docs/v3.4/quick-start/minimal-kubesphere-on-k8s/)

安装 kubesphere 之前需要一个 sc(storageclass) 资源，可以参考前面持久化存储。

## EFK

主流的 ELK ([Elasticsearch](https://cloud.tencent.com/product/es?from_column=20065&from=20065), `Logstash`, Kibana) 目前已经转变为 EFK (Elasticsearch, `Filebeat` or `Fluentd`, Kibana) 比较重，对于容器云的日志方案业内也普遍推荐采用 Fluentd。

- Elasticsearch 是实时全文搜索和分析引擎，提供搜集、分析、存储数据三大功能。
- Filebeat 是一个轻量级的日志数据收集引擎，用Go语言编写，能高效地从大量日志文件中读取数据。
- Kibana 是一个基于 Web 的图形界面，用于搜索、分析和可视化存储在 Elasticsearch 指标中的日志数据。

Logstash 的主要作用是收集分布在各处的 log 并进行处理；Elasticsearch 则是一个集中存储 log 的地方，更重要的是它是一个全文检索以及分析的引擎，它能让用户以近乎实时的方式来查看、分析海量的数据。Kibana 则是为 Elasticsearch 开发的前端 GUI，让用户可以很方便的以图形化的接口查询 Elasticsearch 中存储的数据，同时也提供了各种分析的模块，比如构建 dashboard 的功能。

### Elasticsearch 



### Filebeat 



### Kibana 





























