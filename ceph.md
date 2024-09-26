# Ceph

Ceph是一个对象（object）式存储系统，它把每一个待管理的数据流（如一个文件）切分为一到多个固定大小的对象数据，并以其为原子单元完成数据存储。

对象数据的底层存储服务式由多个主机（host）组成的存储集群，该集群也被称之为`RADOS（Reliable Automatic Distributed Object Store）`存储集群，即可靠，自动化，分布式对象存储系统。

librados 是 RADOS 存储集群的 API，它支持 C、C++、java、Python、Ruby 和 php 等编程语言。

## ceph 架构

一个 Ceph 存储集群至少需要一个 Ceph Monitor（监视器）、Ceph Manager（管理） 和 Ceph OSD（对象存储守护进程）。

**Monitors**

Ceph Monitor ( ceph-mon) 维护集群状态的映射，包括监视器映射、管理器映射、OSD 映射、MDS 映射和 CRUSH 映射。这些映射是 Ceph 守护进程相互协调所需的关键集群状态。监视器还负责管理守护进程和客户端之间的身份验证。冗余和高可用性通常需要至少三个监视器。

**Managers**

Ceph Manager守护进程 ( ceph-mgr) 负责跟踪运行时指标和 Ceph 集群的当前状态，包括存储利用率、当前性能指标和系统负载。高可用性通常需要至少两个管理器。

**Ceph OSD**

一个Ceph OSD（ceph-osd）存储数据、处理数据复制、恢复、重新平衡，并通过检查其他 Ceph OSD 守护进程的心跳来向 Ceph 监视器和管理器提供一些监视信息。冗余和高可用性通常需要至少三个 Ceph OSD。

**MDS**

Ceph 元数据服务器(MDS ceph-mds) 代表Ceph 文件系统存储元数据（即 Ceph 块设备和 Ceph 对象存储不使用 MDS）。Ceph 元数据服务器允许 POSIX 文件系统用户执行基本命令（如 ls、find等），而不会给 Ceph 存储集群带来巨大负担。

> Ceph 将数据作为对象存储在逻辑存储池中。使用 CRUSH算法，Ceph 计算出哪个归置组应该包含该对象，并进一步计算出哪个 Ceph OSD Daemon 应该存储该归置组。CRUSH 算法使 Ceph 存储集群能够动态扩展、重新平衡和恢复。

## 逻辑结构

![在这里插入图片描述](https://ask.qcloudimg.com/http-save/7847908/15700597a2c747b49fbaa0c9a6b853d5.png)

**LIBRADOS：** 一个允许应用程序直接访问 RADO 的库，支持 C、C++、Java、Python、Ruby 和 PHP。

**RADOSGW：** 基于存储桶的 REST网关，兼容s3和Swift。

**RBD：** 一个负责任的，完全-分布式块设备，使用Linux内核cliont和QEMU/KVM驱动程序。

**CEPHFS：** 符合POSIX标准的分发文件系统，具有Linux内核客户端和对FUSE的支持。

**RADOS：** 由自我修复、自我管理、智能存储节点组成的可靠、自主、分布式对象存储。

## 三大存储

### RADOSGW（对象网关）

Ceph 对象网关是一个构建在librados之上的对象存储接口，Ceph 对象存储支持两个接口：

- 兼容 S3 ：通过与 Amazon S3 RESTful API 的大部分子集兼容的接口提供对象存储功能。
- 兼容 Swift：提供对象存储功能，其接口与 OpenStack Swift API 的大部分子集兼容。 Ceph 对象存储使用 Ceph 对象网关守护进程 ( radosgw)，它是用于与 Ceph 存储集群交互的 HTTP 服务器。由于它提供了与 OpenStack Swift 和 Amazon S3 兼容的接口，因此 Ceph 对象网关有自己的用户管理。

### BRD（块存储）

块是字节序列（通常为 512）。基于块的存储接口是在 HDD、SSD、CD、软盘甚至磁带等介质上存储数据的成熟且常见的方式。

Ceph 块设备是精简配置的、可调整大小的，并将数据分条存储在多个 OSD 上。Ceph 块设备利用 RADOS功能，包括快照、复制和强一致性。Ceph 块存储客户端通过内核模块或librbd库与 Ceph 集群通信。

### CEPHFS（文件存储）

Ceph 文件系统或CephFS是一个符合 POSIX 的文件系统，构建在 Ceph 的分布式对象存储RADOS之上。

文件元数据与文件[数据存储](https://cloud.tencent.com/product/cdcs?from_column=20065&from=20065)在单独的 RADOS 池中，并通过可调整大小的元数据服务器集群或MDS提供服务，该集群可以扩展以支持更高吞吐量的元数据工作负载。文件系统的客户端可以直接访问 RADOS 以读取和写入文件数据块。

对数据的访问是通过 MDS 集群来协调的，MDS 集群作为由客户端和 MDS 共同维护的分布式元数据缓存状态的权限。元数据的突变由每个 MDS 聚合成一系列有效的写入 RADOS 上的日志；MDS 没有在本地存储元数据状态。该模型允许在 POSIX 文件系统的上下文中客户端之间进行一致和快速的协作。



## Ceph 部署

### 基础环境配置

0.环境

| 主机名      | IP             | 系统            |
| :---------- | :------------- | :-------------- |
| ceph-1      | 192.168.200.43 | OpenEuler-22.03 |
| ceph-2      | 192.168.200.44 | OpenEuler-22.03 |
| ceph-3      | 192.168.200.45 | OpenEuler-22.03 |
| ceph-client | 192.168.200.45 | OpenEuler-22.03 |



1.修改主机名（三节点操作）

~~~shell
[root@localhost ~]# hostnamectl set-hostname ceph-1
[root@ceph-1 ~]# bash
[root@localhost ~]# hostnamectl set-hostname ceph-2
[root@ceph-2 ~]# bash
[root@localhost ~]# hostnamectl set-hostname ceph-3
[root@ceph-3 ~]# bash
[root@localhost ~]# hostnamectl set-hostname ceph-client
[root@ceph-clinet ~]# bash
~~~

2.配置 hosts 文件解析（三节点操作）

~~~shell
$ cat >> /etc/hosts << EOF
192.168.200.43 ceph-1
192.168.200.44 ceph-2
192.168.200.45 ceph-3
192.168.200.46 ceph-client
EOF
~~~



3.关闭防火墙（三节点操作）

~~~shell
$ systemctl stop firewalld && systemctl disable firewalld
$ setenforce 0
$ sed -i 's/SELINUX=.*/SELINUX=permissive/g' /etc/selinux/config
~~~



4.配置 ssh 免密

~~~shell
[root@ceph-1 ~]# ssh-keygen
[root@ceph-1 ~]# ssh-copy-id ceph-1
[root@ceph-1 ~]# ssh-copy-id ceph-2
[root@ceph-1 ~]# ssh-copy-id ceph-3
[root@ceph-1 ~]# ssh-copy-id ceph-client
[root@ceph-1 ~]# for i in 1 2 3 client; do ssh ceph-$i hostname ; done
ceph-1
ceph-2
ceph-3
ceph-clinet
~~~



5.配置 yum 源

~~~shell
# 使用默认源即可，因为我是华为云买的 ECS。使用的是华为云的源。
~~~



6.配置 chrony 时间同步

- ceph-1节点执行

> 注意：chronyc sources命令查看时间同步是否成功，左下角 ^* 表示成功， ^? 表示不成功。
>
> 这一步很重要确保完成，否则安装ceph集群时会报一些奇奇怪怪的错误。

~~~shell
$ yum install chrony -y
$ sed -i '3,6s/^/#/g' /etc/chrony.conf
$ sed -i '7s/^/server ceph-1 iburst/g' /etc/chrony.conf
$ echo "allow 192.168.200.0/24" >> /etc/chrony.conf
$ echo "local stratum 10" >> /etc/chrony.conf
$ systemctl restart chronyd && systemctl enable chronyd

$ chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ceph-1                       10   6     7     2    -25ns[-6354ns] +/-   10us
[root@ceph-1 ~]# date
Fri Jun  3 17:45:29 CST 2022
~~~



- ceph-2、ceph-3、ceph-client节点执行

~~~shell
$ yum install chrony -y
$ sed -i '3,6s/^/#/g' /etc/chrony.conf
$ sed -i '7s/^/server ceph-1 iburst/g' /etc/chrony.conf
$ systemctl restart chronyd && systemctl enable chronyd

$ chronyc sources
210 Number of sources = 1
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* ceph-1                       11   6     7     1  -2624ns[ -411ms] +/- 1195us
[root@ceph-2 ~]# date
Fri Jun  3 17:45:29 CST 2022
~~~



### 初始化ceph集群

方法一：

- cephadm

  ~~~shell
  # 1.初始化ceph集群：
  $ cephadm bootstrap --mon-ip 192.168.0.21 --allow-fqdn-hostname --initial-dashboard-user admin --initial-dashboard-password Huawei@123 --dashboard-password-noupdate
  
  # 2.为集群添加node节点
  $ cephadm shell
  
  # 3.将全部主机添加到集群内
  $ ceph orch host add ceph02.novalocal
  $ ceph orch host add ceph03.novalocal
  
  # 查看当前集群中主机状态
  $ ceph orch host ls
  
  # 查看 ceph 集群的状态
  $ ceph -s
  ~~~

方法二：

- ceph-deploy

  ~~~shell
  # 1.执行 ceph-deploy install 命令初始化osd节点
  $ mkdir /opt/ceph-ws/ && cd /opt/ceph-ws
  $ ceph-deploy install --no-adjust-repos --nogpgcheck ceph-{1..3}
  ...
  
  ...
  [ceph-node3][DEBUG ] 完毕！
  [ceph-node3][INFO  ] Running command: sudo ceph --version
  [ceph-node3][DEBUG ] ceph version 13.2.10 (564bdc4ae87418a232fc901524470e1a0f76d641) mimic (stable)
  
  # 2.执行ceph-deploy new 命令执行 ceph 集群初始化
  # 由于我是将 mon 也放到 osd 节点上，所以这里就是 ceph-1、ceph-2、ceph-3了
  # 生产环境，建议将 mon 单独服务器节点。
  $ ceph-deploy new --cluster-network 192.168.6.0/24 --public-network 192.168.2.0/24  ceph-1 ceph-2 ceph-3
  
  # 查看当前目录，可以发现生产了一些文件
  $ ll
  总用量 16
  -rw-rw-r-- 1 ceph ceph  292 12月 22 12:10 ceph.conf  # conf是ceph集群的配置文件
  -rw-rw-r-- 1 ceph ceph 5083 12月 22 12:10 ceph-deploy-ceph.log # 日志
  -rw------- 1 ceph ceph   73 12月 22 12:10 ceph.mon.keyring # 这个是ceph集群的密钥
  
  # 3.配置 mon 节点
  # 如果单独部署 mon 节点，记得安装 ceph-mon。
  yum install -y ceph-mon
  # 初始化 mon
  $ ceph-deploy mon create-initial
  # 执行完成后回发现又多了一些配置文件，这些文件都是非常重要的，类似k8s中的kubeconfig，不要随意泄露。
  $ ll
  总用量 476
  -rw------- 1 ceph ceph    113 12月 22 13:11 ceph.bootstrap-mds.keyring
  -rw------- 1 ceph ceph    113 12月 22 13:11 ceph.bootstrap-mgr.keyring
  -rw------- 1 ceph ceph    113 12月 22 13:11 ceph.bootstrap-osd.keyring
  -rw------- 1 ceph ceph    113 12月 22 13:11 ceph.bootstrap-rgw.keyring
  -rw------- 1 ceph ceph    151 12月 22 13:11 ceph.client.admin.keyring
  -rw-rw-r-- 1 ceph ceph    292 12月 22 12:11 ceph.conf
  -rw-rw-r-- 1 ceph ceph 207826 12月 22 13:17 ceph-deploy-ceph.log
  -rw------- 1 ceph ceph     73 12月 22 12:11 ceph.mon.keyring
  
  # 推送管理密钥到节点
  $ ceph-deploy admin ceph-node{1..3}
  
  # 4.配置 mgr 节点
  # 如果是独立的mgr节点服务器，记得检查是否安装了ceph-mgr包
  yum install -y ceph-mgr
  # 添加 mgr 节点
  ceph-deploy mgr create ceph-node1 ceph-node2
  
  # 5.检测
  $ ceph -s
    cluster:
      id:     f1da3a2e-b8df-46ba-9c6b-0030da25c73e
      health: HEALTH_WARN
              OSD count 0 < osd_pool_default_size 3
   
    services:
      mon: 3 daemons, quorum ceph-node1,ceph-node2,ceph-node3
      mgr: ceph-node1(active), standbys: ceph-node2
      osd: 0 osds: 0 up, 0 in
   
    data:
      pools:   0 pools, 0 pgs
      objects: 0  objects, 0 B
      usage:   0 B used, 0 B / 0 B avail
      pgs:     
  ~~~

  



#### 管理 osd

> 注意：在使用方面，cephadm 提供了许多命令，可用于部署和管理 Ceph 集群。其中一些最常用的命令包括：
>
> ceph orch apply：此命令用于将配置应用于 Ceph 集群。它可用于部署新服务、更新现有服务或删除不再需要的服务。
> ceph orch ls：此命令用于列出当前在 Ceph 集群中运行的服务。
> ceph orch ps：此命令用于列出当前在 Ceph 集群中运行的服务的状态。
> ceph orch rm：此命令用于从 Ceph 集群中删除服务。
> ceph orch upgrade：此命令用于将 Ceph 集群升级到新版本。

~~~shell
# 将所有主机上的硬盘添加为OSD
$ ceph orch apply osd --all-available-devices
# 使用 ceph-deploy 添加 OSD,示例：
$ ceph-deploy osd create ceph1 --data /dev/sdb

# 通过ceph osd stat命令检查
$ ceph osd stat
9 osds: 9 up, 9 in; epoch: e37

# 使用 ceph osd status 查看
- `id`: OSD的唯一标识符。
- `host`: OSD所在的主机名
- `used`: OSD已使用的存储容量。
- `avail`: OSD可用的存储容量。
- `wr ops`: OSD每秒写入操作的数量。
- `wr data`: OSD每秒写入数据的数量。
- `rd ops`: OSD每秒读取操作的数量。
- `rd data`: OSD每秒读取数据的数量。
- `state`: OSD的状态，"exists"表示OSD存在，"up"表示OSD正常运行。
$ ceph osd status
+----+------------+-------+-------+--------+---------+--------+---------+-----------+
| id |    host    |  used | avail | wr ops | wr data | rd ops | rd data |   state   |
+----+------------+-------+-------+--------+---------+--------+---------+-----------+
| 0  | ceph-node1 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 1  | ceph-node1 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 2  | ceph-node1 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 3  | ceph-node2 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 4  | ceph-node2 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 5  | ceph-node2 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 6  | ceph-node3 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 7  | ceph-node3 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
| 8  | ceph-node3 | 1028M | 4087M |    0   |     0   |    0   |     0   | exists,up |
+----+------------+-------+-------+--------+---------+--------+---------+-----------+

# ceph osd tree 也可以
$ ceph osd tree
ID CLASS WEIGHT  TYPE NAME           STATUS REWEIGHT PRI-AFF 
-1       0.04408 root default                                
-3       0.01469     host ceph-node1                         
 0   hdd 0.00490         osd.0           up  1.00000 1.00000 
 1   hdd 0.00490         osd.1           up  1.00000 1.00000 
 2   hdd 0.00490         osd.2           up  1.00000 1.00000 
-5       0.01469     host ceph-node2                         
 3   hdd 0.00490         osd.3           up  1.00000 1.00000 
 4   hdd 0.00490         osd.4           up  1.00000 1.00000 
 5   hdd 0.00490         osd.5           up  1.00000 1.00000 
-7       0.01469     host ceph-node3                         
 6   hdd 0.00490         osd.6           up  1.00000 1.00000 
 7   hdd 0.00490         osd.7           up  1.00000 1.00000 
 8   hdd 0.00490         osd.8           up  1.00000 1.00000 
 
# 这个是用来查看osd的disk free，类似linux的df
- `ID`: OSD的唯一标识符。
- `CLASS`: OSD的存储类别。
- `WEIGHT`: OSD的权重。
- `REWEIGHT`: OSD的重新加权比例。
- `SIZE`: OSD的总存储容量。
- `RAW USE`: OSD当前使用的原始存储容量。
- `DATA`: OSD数据存储使用量。
- `OMAP`: OSD的OMAP（Object Map）数据存储使用量。
- `META`: OSD元数据存储使用量。
- `AVAIL`: OSD可用的存储容量。
- `%USE`: OSD使用率百分比。
- `VAR`: OSD使用率方差。
- `PGS`: OSD分布的PG（Placement Group）数量。
- `STATUS`: OSD的状态，"up"表示OSD正常运行。
$ ceph osd df
ID CLASS WEIGHT  REWEIGHT SIZE    USE     DATA    OMAP META  AVAIL   %USE  VAR  PGS 
 0   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 1   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 2   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 3   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 4   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 5   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 6   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 7   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
 8   hdd 0.00490  1.00000 5.0 GiB 1.0 GiB 4.7 MiB  0 B 1 GiB 4.0 GiB 20.11 1.00   0 
                    TOTAL  45 GiB 9.0 GiB  42 MiB  0 B 9 GiB  36 GiB 20.11          
MIN/MAX VAR: 1.00/1.00  STDDEV: 0


# 移除 osd
# 1.停止osd：
$ ceph orch daemon stop osd.0

# 2.待所有osd停止后，使用以下命令移除osd：
$ ceph osd rm 0

# 3.擦除对应磁盘的数据：
$ ceph orch device zap ceph-node-2 /dev/sdb --force

# 4.删除CURSH的osd映射：
$ ceph osd crush rm osd.0

# 5.等待CRUSH中的osd映射删除完成后，关闭自动osd部署功能，命令为：
$ ceph orch apply osd --all-available-devices --unmanaged=true
~~~



#### 移除服务并移除节点

~~~shell
# 在删除节点之前需要先删除 osd，以及移除所有服务，
# 先执行上面删除 osd 步骤后，再移除节点。
# 查看服务
$ ceph orch ps

# 删除前，要关闭集群组件的自动扩展功能
$ ceph orch apply mon --unmanaged=true
$ ceph orch apply mgr --unmanaged=true
$ ceph orch apply node-exporter --unmanaged=true
$ ceph orch apply crash --unmanaged=true
$ ceph orch apply prometheus --unmanaged=true

# 移除需要移除节点的所有服务
$ ceph orch daemon rm <names>

# 移除节点
$ ceph orch host rm ceph02.novalocal
~~~



## 副本池创建及配置

- 创建一个副本池

  ~~~shell
  $ ceph osd pool create test
  
  # 查看资源池的信息
  $ ceph osd pool ls [detail] 	# 更详细的信息
  
  # 查看到更详细的资源池信息
  $ ceph osd pool get test all
  ~~~

  

- 修改资源池

  ~~~shell
  $ ceph osd pool set test pg_num 256
  
  # 修改 min_size 它决定了在Ceph集群中存储对象时最小的副本数量。默认为2
  $ ceph osd pool set test min_size 1
  ~~~

  在创建Pool资源池的时候一定要指定pg_num和pgp_num参数，因为Ceph集群不能自动计算PG的数量。

  **官方建议的PG使用数量：**

  - 集群中小于5个OSD，则设置PG的数量为128。
  - 集群有5-10个OSD时，设置PG的数量为512。
  - 集群中有10-50个OSD时，设置PG的数量为1024.

  

- 使用 rados 命令进行简单对象操作

  ~~~shell
  $ rados -p test_rbd put hosts /etc/hosts
  $ rados -p test_rbd ls
  ~~~

  

- 为资源池设置应用模式

  其实就是为资源池设置一个分类，有rbd、rgw、cephfs三种。

  命令格式：`ceph osd pool application enable {pool_name} rbd`

  ~~~shell
  ceph osd pool application enable ceph-pool-1 rbd
  ~~~

  

- 为资源池创建快照

  ~~~shell
  $ ceph osd pool mksnap test snapshot1
  
  # 把 test 池中的 hosts 文件删除
  $ rados -p test rm hosts
  
  # 使用快照还原对象
  $ rados -p test -n snapshot1 get hosts hosts-ws
  ~~~

  

- 资源池配额设置

  ~~~shell
  # 使用以下命令设置test_rbd最多能上传5个对象
  $ ceph osd pool set-quota test_rbd max_objects 5
  
  # 如果上传的对象超过五个
  $ ceph health detail 		# 会发现一个报错
  ~~~



- 获取池值

  ~~~shell
  ceph osd pool get {pool-name} {key}
  ~~~

  

- 删除资源池

  ~~~shell
  # 开启资源池允许删除选项
  $ ceph config set mon mon_allow_pool_delete true
  
  # 删除资源池
  $ ceph osd pool rm test test --yes-i-really-really-mean-it		# 输入两遍资源池名称
  ~~~



- 纠删码池

  ~~~shell
  # 创建纠删码池规则
  $ ceph osd erasure-code-profile set test k=4 m=2 crush-failure-domin=osd
  
  # 查看规则
  $ ceph osd erasure-code-profile get test
  
  # 利用创建好的纠删码规则创建纠删码池
  $ ceph osd pool create e_test erasure test
  ~~~



## ceph 用户管理实践

~~~shell
# 创建用户
$ ceph auth add client.test

# 为新用户赋权
$ ceph auth caps client.test mon "allow rw"  osd "allow *"

# 导出认证文件，然后将 keyring 文件导入到客户端的 /etc/ceph 目录下
$ ceph auth get client.test -o client.test.keyring

# 客户端测试命令
$ ceph -s --id test --keyring /etc/ceph/client.test.keyring
~~~





## ceph 块存储 RBD 实践

### 简介

- Ceph 块存储共享基于 RADOS 构建，经过精简配置，大小调整后，在多个 OSD 上实现存储数据条带化。Ceph 块存储可实现快照，复制和存储的强一致性。
- 应用访问块存储通过两种方式实现：
  - Librbd 用户态接口：librados.so 库作为客户端连接 RADOS 集群。
  - Krbd 内核态接口，通过 rbd 命令将块设备映射为主机设备，可挂载至指定目录。

### 创建块设备

~~~shell
# 创建资源池block
$ ceph osd pool create block
$ ceph osd pool application enable block rbd

# 创建两个rbd设备
$ rbd create --size 3G block/rbd1
$ rbd create --size 5G block/rbd2

# 创建用户block，使其对所有的rbd设备有读写权限
$ ceph auth add client.block mon ‘allow r’ osd 'allow rwx pool=block'

# 将用户block的keyring文件导出为ceph.client.block.keyring
$ ceph auth get client.block -o ceph.client.block.keyring

# 将该文件发送到客户端的/etc/ceph目录中
$ scp ceph.client.block.keyring 192.168.0.4:/etc/ceph
~~~



### 客户端连接

~~~shell
# 客户端安装客户端软件
$ yum install -y ceph-common

# 使用rbd命令将创建好的rbd设备映射到Client，--name 表明身份，/etc/ceph/ceph.client.block.keyring
$ rbd map block/rbd1 --name client.block
$ rbd map block/rbd2 --name client.block

# 将映射后的设备格式化，并按照规划，挂载到指定目录下
$ mkdir -p /mnt/rbd{1,2}
$ mkfs.xfs /dev/rbd0
$ mkfs.xfs /dev/rbd1
$ mount /dev/rbd0 /mnt/rbd1
$ mount /dev/rbd1 /mnt/rbd2

# rbd设 备设置为自动挂载
$ vim /etc/fstab
/dev/rbd/block/rbd1	/mnt/rbd1	xfs	defaults,_netdev 0 0
/dev/rbd/block/rbd2	/mnt/rbd2	xfs	defaults,_netdev 0 0

# 编辑/etc/ceph/rbdmap文件
block/rbd1	id=block,keyring=/etc/ceph/ceph.client.block.keyring
block/rbd2	id=block,keyring=/etc/ceph/ceph.client.block.keyring

# 将rbdmap服务设置为开机自启动
$ systemctl enable rbdmap
~~~



### RBD 设备扩容

~~~shell
$ # 扩容到 5G
$ rbd --pool block resize rbd1 --size 5G
$ rbd -p block du rbd1

# 客户端扩容,执行分区扩容
$ xfs_growfs /dev/rbd0
~~~



### 创建快照

~~~shell
# 创建快照
$ rbd snap create block/rbd2@snap1

# 回滚快照
# 首先要取消客户端 rbd 设备的映射和挂载
$ umount /mnt/rbd2
$ rbdmap unmap block/rbd2

# 在ceph管理节点上，使用以下命令回滚快照
$ rbd snap rollback  block/rbd2@snap1

# 将快照设为保护模式
$ rbd snap protect block/rbd2@snap1

# 使用快照创建克隆卷并挂载
$ rbd clone  block/rbd2@snap1 block/rbd3

# 将克隆卷转为独立rbd设备
$ rbd flatten block/rbd3

# 在客户端将生成的rbd3映射到Client中
$ rbd map block/rbd3 --name client.block

# 然后将该克隆卷的母卷卸载，最后再将克隆卷挂载到新创建的目录/mnt/rbd3中，因为母卷被格式化为xfs格式，其uuid和克隆卷是一致的
$ umount /mnt/rbd2
$ mkidr /mnt/rbd3
$ mount /dev/rbd2 /mnt/rbd3
~~~



### Ceph 块存储所涉其他命令

- **列出资源池内所有rbd设备**：rbd [--pool pool-name] ls
- **查询指定rbd设备的详细信息**：rbd info [poo-name/]image-name
- **查询指定rbd设备的状态信息**：rbd status [poo-name/]image-name
- **查询指定rbd设备的容量大小**：rbd du [poo-name/]image-name
- **查询指定rbd设备是否已被加锁**：rbd lock ls [poo-name/]image-name
- **删除指定rbd设备的锁**：rbd [--pool pool-name] lock rm image-name lock-id locker

### RBD 设备特性

- 在创建 RBD 设备时，使用参数--image-feature" 可开启对应特性， RBD 设备支持的特性有:
  - layering：克隆的分层
  - Strping：可提高性能的条带 v2 版本
  - exclusive-lock：独占锁
  - object-map：对象映射，依赖 exclusive-lock 特性
  - fast-diff：快速差异计算，依赖object-map 特性
  - deep-flatten：快照的扁平化
  - journaling：日志功能，依赖 exclusive-lock特性
  - data-pool：纠删码池支持

### RBD 设备的导入导出

- RBD 设备支持全量导入导出：

  ~~~shell
  # 将 rbd 设备导出为文件 bd-export
  $ rbd -p block export rbd1 /tmp/rbd-export
  
  # 借助导出文件，将 rbd 设备导入到资源池 target 中
  $ rbd -p target import /tmp/rbd-export rbd-export
  $ rbd -p target list
  ~~~

  

- 除了全量导入导出外，RBD 设备还可以借助快照，进行增量导入导出：

  ~~~shell
  $ rbd snap create block/rbd1@snap3
  
  # 基于快照，将增量数据导出为文件 /tmp/snap3
  $ rbd export-diff block/rbd1@snap3 /tmp/snap3
  
  # 基于导出的文件，将增量数据导入到 rbd-export
  $ rbd import-diff target/rbd-export
  ~~~

  

## ceph 文件存储 Cephfs 实践

### 简介

ceph 文件系统(CephFS)是基于 ceph 分布式存储 DOS 构建的兼容 POS 以的文件系统。

cephFS 致力于提供先进的、多用户、高可用、高性能的文件存储。

CephFS 支持的场景包括共享主目录、高性能计算暂存空间及工作流共享存储等。

CephFS 需要运行在集群中运行 MDS 服务，其守护进程为 ceph-mds ，用于管理 CephFS 中所存储文件的元数据，及协调对 ceph 集群的访问。

CephFS 通过 ceph-mds 将元数据与数据分开，可降低复杂性并提高可靠性。

### 创建 cephfs 所需的资源

~~~shell
# 创建资源池
# 创建两个副本池，分别命名为data_cephfs和metadata_cephfs，其中data_cephfs作为cephFS的数据池，metadata_cephfs作为cephFS的元数据池，具体命令如下：
$ ceph osd pool create metadata_cephfs
$ ceph osd pool create data_cephfs

# 格式化资源池
$ ceph fs new fs01 metadata_cephfs data_cephfs

# 在ceph集群中部署两个MDS
$ ceph orch apply mds fs01 --placement="2"
# 查看MDS的状态
$ ceph orch ls

# 创建用户
# 创建用户user01，使其可对fs01有读写权限
$ ceph fs authorize fs01 client.user01 / rwps -o ceph.client.user01.keyring

# 查看 MDS 所在节点的地址
$ ceph fs status
fs01 - 0 clients
====
RANK  STATE             MDS               ACTIVITY     DNS    INOS   DIRS   CAPS  
 0    active  fs01.ceph-node-2.qkvtut  Reqs:    0 /s    10     13     12      0   
      POOL         TYPE     USED  AVAIL  
metadata_cephfs  metadata  96.0k  37.9G  
  data_cephfs      data       0   37.9G  
      STANDBY MDS        
fs01.ceph-node-3.cncuju  
MDS version: ceph version 16.2.13 (5378749ba6be3a0868b51803968ee9cde4833a3e) pacific (stable)
~~~



### 客户端挂载

将创建cephfs挂载到客户端的/mnt/cephfs目录中，并需实现开机自动挂载。

~~~shell
# 步骤 1	拷贝user01的keyring到客户端
$ scp ceph.client.user01.keyring root@172.16.0.233:/etc/ceph/
# 创建认证文件user.txt，将用户信息和秘钥信息保存其中
$ echo "AQC7rSNlBesFBxAAeBhPod/SfnJxJNdrLo66YA==" > user.txt

# 步骤 3	修改/etc/fstab文件
# 在/etc/fstab文件中加入以下内容，内容里的两个ip就是 MDS 所有节点的 ip
192.168.0.22:6789,192.168.0.23:6789:/          /mnt/cephfs     ceph     defaults,_netdev,name=user01,secretfile=/root/user.txt 0 0 

# 挂载
$ mount -a
~~~















