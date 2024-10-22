# OpenStack

## 1、介绍

OpenStack是一个开源的云计算平台，旨在构建和管理私有云和公有云基础设施。它由一系列相互关联的项目组成，提供了用于计算、存储和网络的模块化工具集，以及用于身份认证、图像管理和资源编排等关键功能。

OpenStack的优点包括灵活性、可扩展性和可定制性，使用户能够构建适合其需求的定制云解决方案。由于其开源的特性，OpenStack也促进了云计算领域的创新和标准化。

## 2、架构

![image-20230831004336020](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20230831004336020.png)

## 3、组件

一、Nova
Nova是 OpenStack 的计算服务组件，提供虚拟机的创建、启动、停止、重启等功能。Nova采用了模块化架构，将计算节点、调度器、API等模块分离，以实现高可用性和可扩展性。Nova还支持多种虚拟化技术，如 KVM、Xen、VMware 等。

二、Neutron
Neutron是OpenStack的网络服务组件，提供虚拟网络的创建、配置、管理等功能。Neutron支持多种网络模型，如VLAN、GRE、VXLAN等。Neutron还支持网络功能虚拟化（NFV）和软件定义网络（SDN）技术，以支持更灵活的网络配置和管理。

三、Cinder
Cinder是OpenStack的块存储服务组件，提供块存储的创建、挂载、卸载、删除等功能。Cinder支持多种存储后端，如LVM、Ceph、GlusterFS等。Cinder还支持快照、克隆、备份等功能，以实现数据的保护和管理。

四、Swift
Swift是 OpenStack 的对象存储服务组件，提供海量对象的存储和管理。Swift采用了分布式存储、负载均衡、数据冗余、数据分片、对象容器等技术，以支持PB级别的数据存储和管理。Swift还支持多种客户端工具和语言的接入，以支持不同的开发需求。

五、Keystone
Keystone是 OpenStack 的身份认证服务组件，提供用户身份认证、角色管理、权限控制等功能。Keystone支持多种身份认证方式，如用户名密码、LDAP、OpenID 等。Keystone还支持多租户、多域和单点登录等功能，以支持复杂的用户管理和安全策略。

六、Glance
Glance是 OpenStack 的镜像服务组件，提供虚拟机镜像的管理和分发。Glance支持多种镜像格式，如qcow2、vhd、vmdk等。Glance还支持镜像的版本管理、元数据管理和加密等功能，以提高镜像的安全性和可管理性。

七、Horizon
Horizon 是 OpenStack 的 Web 管理界面，提供了一个方便易用的图形化用户界面。Horizon支持用户和管理员的角色切换、虚拟机和网络的管理、监控和报告等功能。Horizon还支持多语言和自定义主题，以适应不同用户的需求。

八、Heat
Heat是OpenStack的编排服务组件，提供基于模板的自动化服务编排。Heat支持模板的定义、参数的配置、资源的创建和依赖关系的管理。Heat还支持多种编排模式，如串行、并行、嵌套等，以支持复杂的应用部署和管理。

九、Ceilometer
Ceilometer是OpenStack的计量服务组件，提供云计算资源的监控和计量功能。Ceilometer支持多种资源类型的监控，如虚拟机、网络、存储等。Ceilometer还支持多种监控指标的收集和分析，以帮助用户了解云计算资源的使用情况和性能瓶颈。

除了以上这些组件，在openstack官网上还有很多组件供我们使用，[传送门](https://www.openstack.org/software/project-navigator/openstack-components#openstack-services)。

## 4、群集部署

### 4.1、环境

| 节点/主机名 | ip地址                        | 系统      |
| ----------- | ----------------------------- | --------- |
| controller  | 192.168.100.10/192.168.200.10 | centos7.9 |
| compute     | 192.168.100.20/192.168.200.20 | centos7.9 |

注意：controller 节点和 compute 节点都需要两个网卡，而 compute 还需要三块硬盘，也可以是分区。

### 4.2、基础配置

~~~bash
# 1、配置ip
# compute配置ip和controller一样
[root@localhost ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens33
1 TYPE=Ethernet
2 BOOTPROTO=static
3 NAME=ens33
4 DEVICE=ens33
5 ONBOOT=yes
6 IPADDR=192.168.100.10			# compute 设置为 192.168.100.20
7 NETMASK=255.255.255.0

# 2、修改主机名
[root@localhost ~]# hostnamectl set-hostname controller/compute

# 3、配置 /etc/hosts
vi /etc/hosts
192.168.100.10  controller
192.168.100.20  compute

# 4、关闭防火墙
# selinux
[root@controller ~]# sed -i 's/SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
[root@controller ~]# setenforce 0

# firewalld
[root@controller ~]# systemctl stop firewalld
[root@controller ~]# systemctl disable firewalld 

# iptables
[root@controller ~]# yum install  iptables-services  -y
[root@controller ~]# if [ 0  -ne  $? ]; then
        echo -e "\033[31mThe installation source configuration errors\033[0m"
        exit 1
fi
[root@controller ~]# systemctl restart iptables
[root@controller ~]# iptables -F
[root@controller ~]# iptables -X
[root@controller ~]# iptables -Z
[root@controller ~]# /usr/sbin/iptables-save
[root@controller ~]# systemctl stop iptables
[root@controller ~]# systemctl disable iptables


# 5、修改密码,密码设置为 000000
[root@controller ~]# passwd root

# 6、配置 yum 源,配置两个yum源，一个是centos，一个是iaas，centos 不能安装 openstack-iaas-rpm，所以要引用 iaas 这个源，来安装 openstack-iaas-rpm
# 先把 centos 的光盘连接到虚拟机上，另一个 iaas 是用 lrzsz 工具上传镜像。
[root@controller ~]# cd /etc/yum.repos.d/
[root@controller yum.repos.d]# mkdir bak
[root@controller yum.repos.d]# mv CentOS-* bak
[root@controller yum.repos.d]# vi centos.repo
[centos]
name=centos yum
baseurl=file:///opt/centos
enabled=1
gpgcheck=0
[root@controller yum.repos.d]# mkdir /opt/centos && mount /dev/cdrom /opt/centos
[root@controller yum.repos.d]# yum install -y vim bash-completion tree lrzsz 		# 安装常用安装包
#  第一个 yum 配置完毕，配置第二个
[root@controller ~]# rz -E    # 使用 lrzsz 工具上传 chinaskills_cloud_iaas_v2.0.3.iso 
[root@controller ~]# ll
总用量 2590416
-rw-r--r--. 1 root root 2652577792 6月   7 2022 chinaskills_cloud_iaas_v2.0.3.iso		# 上传成功
[root@controller ~]# mount /root/chinaskills_cloud_iaas_v2.0.3.iso /media/
[root@controller ~]# cp -rvf /media/* /opt/
# 配置第二个 yum 配置文件
[root@controller ~]# vim /etc/yum.repos.d/iaas.repo
[iaas]
name=iaas yum
baseurl=file:///opt/iaas-repo
enabled=1
gpgcheck=0
# 测试
[root@controller ~]# yum clean all 		# 清理缓存
[root@controller ~]# yum repolist
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
源标识                                                    源名称                                            状态     
centos                                                   centos yum                                       4,070    # 4070 个包没问题
iaas                                                     iaas yum                                         954      # 954 个包没问题   
repolist: 5,024
~~~

### 4.3、搭建集群

~~~shell
# 1、安装软件包，因为这个软件包里有比赛里所用到的脚本文件。
# 集群的版本是 
[root@controller ~]# yum install -y openstack-iaas

# 2、修改软件包的全局脚本变量文件 controller 和 compute 都要修改。
[root@controller ~]# vim  /etc/openstack/openrc.sh 
     1	#--------------------system Config--------------------##
     2	#Controller Server Manager IP. example:x.x.x.x
     3	HOST_IP=192.168.100.10
     4	
     5	#Controller HOST Password. example:000000 
     6	HOST_PASS=000000
     7	
     8	#Controller Server hostname. example:controller
     9	HOST_NAME=controller
    10	
    11	#Compute Node Manager IP. example:x.x.x.x
    12	HOST_IP_NODE=192.168.100.20
    13	
    14	#Compute HOST Password. example:000000 
    15	HOST_PASS_NODE=000000
    16	
    17	#Compute Node hostname. example:compute
    18	HOST_NAME_NODE=compute
    19	
    20	#--------------------Chrony Config-------------------##
    21	#Controller network segment IP.  example:x.x.0.0/16(x.x.x.0/24)
    22	network_segment_IP=192.168.100.0/24
    23	
    24	#--------------------Rabbit Config ------------------##
    25	#user for rabbit. example:openstack
    26	RABBIT_USER=openstack
    27	
    28	#Password for rabbit user .example:000000
    29	RABBIT_PASS=000000
    30	
    31	#--------------------MySQL Config---------------------##
    32	#Password for MySQL root user . exmaple:000000
    33	DB_PASS=000000
    34	
    35	#--------------------Keystone Config------------------##
    36	#Password for Keystore admin user. exmaple:000000
    37	DOMAIN_NAME=demo
    38	ADMIN_PASS=000000
    39	DEMO_PASS=000000
    40	
    41	#Password for Mysql keystore user. exmaple:000000
    42	KEYSTONE_DBPASS=000000
    43	
    44	#--------------------Glance Config--------------------##
    45	#Password for Mysql glance user. exmaple:000000
    46	GLANCE_DBPASS=000000
    47	
    48	#Password for Keystore glance user. exmaple:000000
    49	GLANCE_PASS=000000
    50	
    51	#--------------------Placement Config----------------------##
    52	#Password for Mysql placement user. exmaple:000000
    53	PLACEMENT_DBPASS=000000
    54	
    55	#Password for Keystore placement user. exmaple:000000
    56	PLACEMENT_PASS=000000
    57	
    58	#--------------------Nova Config----------------------##
    59	#Password for Mysql nova user. exmaple:000000
    60	NOVA_DBPASS=000000
    61	
    62	#Password for Keystore nova user. exmaple:000000
    63	NOVA_PASS=000000
    64	
    65	#--------------------Neutron Config-------------------##
    66	#Password for Mysql neutron user. exmaple:000000
    67	NEUTRON_DBPASS=000000
    68	
    69	#Password for Keystore neutron user. exmaple:000000
    70	NEUTRON_PASS=000000
    71	
    72	#metadata secret for neutron. exmaple:000000
    73	METADATA_SECRET=000000
    74	
    75	#External Network Interface. example:eth1
    76	INTERFACE_NAME=ens34				# 第二块网卡
    77	
    78	#External Network The Physical Adapter. example:provider
    79	Physical_NAME=provider
    80	
    81	#First Vlan ID in VLAN RANGE for VLAN Network. exmaple:101
    82	minvlan=101
    83	
    84	#Last Vlan ID in VLAN RANGE for VLAN Network. example:200
    85	maxvlan=200
    86	
    87	#--------------------Cinder Config--------------------##
    88	#Password for Mysql cinder user. exmaple:000000
    89	CINDER_DBPASS=000000
    90	
    91	#Password for Keystore cinder user. exmaple:000000
    92	CINDER_PASS=000000
    93	
    94	#Cinder Block Disk. example:md126p3
    95	BLOCK_DISK=sdb
    96	
    97	#--------------------Swift Config---------------------##
    98	#Password for Keystore swift user. exmaple:000000
    99	SWIFT_PASS=000000
   100	
   101	#The NODE Object Disk for Swift. example:md126p4.
   102	OBJECT_DISK=sdc
   103	
   104	#The NODE IP for Swift Storage Network. example:x.x.x.x.
   105	STORAGE_LOCAL_NET_IP=192.168.100.20
   106	
   107	#--------------------Trove Config----------------------##
   108	#Password for Mysql trove user. exmaple:000000
   109	TROVE_DBPASS=000000
   110	
   111	#Password for Keystore trove user. exmaple:000000
   112	TROVE_PASS=000000
   113	
   114	#--------------------Heat Config----------------------##
   115	#Password for Mysql heat user. exmaple:000000
   116	HEAT_DBPASS=000000
   117	
   118	#Password for Keystore heat user. exmaple:000000
   119	HEAT_PASS=000000
   120	
   121	#--------------------Ceilometer Config----------------##
   122	#Password for Gnocchi ceilometer user. exmaple:000000
   123	CEILOMETER_DBPASS=000000
   124	
   125	#Password for Keystore ceilometer user. exmaple:000000
   126	CEILOMETER_PASS=000000
   127	
   128	#--------------------AODH Config----------------##
   129	#Password for Mysql AODH user. exmaple:000000
   130	AODH_DBPASS=000000
   131	
   132	#Password for Keystore AODH user. exmaple:000000
   133	AODH_PASS=000000
   134	
   135	#--------------------ZUN Config----------------##
   136	#Password for Mysql ZUN user. exmaple:000000
   137	ZUN_DBPASS=000000
   138	
   139	#Password for Keystore ZUN user. exmaple:000000
   140	ZUN_PASS=000000
   141	
   142	#Password for Keystore KURYR user. exmaple:000000
   143	KURYR_PASS=000000
   144	
   145	#--------------------OCTAVIA Config----------------##
   146	#Password for Mysql OCTAVIA user. exmaple:000000
   147	OCTAVIA_DBPASS=000000
   148	
   149	#Password for Keystore OCTAVIA user. exmaple:000000
   150	OCTAVIA_PASS=000000
   151	
   152	#--------------------Manila Config----------------##
   153	#Password for Mysql Manila user. exmaple:000000
   154	MANILA_DBPASS=000000
   155	
   156	#Password for Keystore Manila user. exmaple:000000
   157	MANILA_PASS=000000
   158	
   159	#The NODE Object Disk for Manila. example:md126p5.
   160	SHARE_DISK=sdd
   161	
   162	#--------------------Cloudkitty Config----------------##
   163	#Password for Mysql Cloudkitty user. exmaple:000000
   164	CLOUDKITTY_DBPASS=000000
   165	
   166	#Password for Keystore Cloudkitty user. exmaple:000000
   167	CLOUDKITTY_PASS=000000
   168	
   169	#--------------------Barbican Config----------------##
   170	#Password for Mysql Barbican user. exmaple:000000
   171	BARBICAN_DBPASS=000000
   172	
   173	#Password for Keystore Barbican user. exmaple:000000
   174	BARBICAN_PASS=000000
   175	###############################################################
   176	#####在vi编辑器中执行:%s/^.\{1\}//  删除每行前1个字符(#号)#####
   177	###############################################################
# 3、执行安装脚本
# controller 需要执行的基础服务脚本
iaas-pre-host.sh		# 执行此脚本后，需重启或重新连接
iaas-install-mysql.sh
iaas-install-keystone.sh
iaas-install-glance.sh
iaas-install-placement.sh
iaas-install-nova-controller.sh
iaas-install-neutron-controller.sh
iaas-install-dashboard.sh
# compute 需要执行的基础服务脚本
iaas-pre-host.sh		# 执行此脚本后，需重启或重新连接
iaas-install-nova-compute.sh
iaas-install-neutron-compute.sh

# 4、搭建完成后，使用以下两条命令，检查组件的状态
openstack-service status
openstack-status

# 5、添加控制节点资源到云平台
# 只需要在控制节点将 iaas-install-nova-compute.sh 脚本中的compute ip 改成自己，再执行一下这个脚本就好。
~~~

## 5、命令行操作

使用命令操作前需要 source keystone admin 文件，这一步骤就像 bashboard 网页，输入账号密码验证你的用户属于的是否合法，密码是否正确等。

~~~shell
[root@controller ~]# source /etc/keystone/admin-openrc.sh 
~~~



### 5.1、网络

- 外部网络

  ~~~shell
  # 创建一个名为 ext-wang1 的外部网络
  openstack network create ext-wang1 \
  --external \
  --project admin \
  --provider-network-type vlan \
  --provider-segment 1000 \
  --share \
  --provider-physical-network  provider
  
  # 查看网络
  openstack network list
  ~~~

  - `--external`：设置该网络为外部网络
  - `--project`：项目的所有者
  - `--provider-network-type`：虚拟网络的物理机制实现的。例如:flat, geneve, gre, local，vxlan vlan。
  - `--provider-segment`：VLAN网络的VLAN ID或Tunnel ID
  - `--share`：在项目之间共享网络

- 内部网络

  ~~~shell
  # 创建一个名为 int-wang1 的内部网络。认网络模式为 vxlan，可以在 /etc/neutron/plugins/ml2/ml2_conf.ini 修改
  openstack network create int-wang1 --internal
  
  # 删除内部网络
  openstack network delete int-wang1
  
  ~~~

  - `--internal`：将此网络设置为内部网络(默认)

- 子网

  ~~~shell
  # 为 int-wang1 的网络添加子网
  openstack subnet create int-wang1-sub \
  --network int-wang1 \
  --subnet-range 10.0.0.0/24 \
  --gateway 10.0.0.1 \
  --allocation-pool start=10.0.0.100,end=10.0.0.200 \
  --dns-nameserver 114.114.114.114
  
  # 删除子网
  openstack subnet delete int-wang1-sub
  ~~~

  - `--network`：子网所属的网络(名称或 ID)
  - `--subnet-range`：CIDR 表示法中的子网范围
  - `--gateway`：为子网指定网关
  - `--allocation-pool`：此子网的分配池IP地址
  - `dns-nameserver`：为子网指定 dns

  

- 路由

  ~~~shell
  # 创建路由
  openstack router create route-wang1
  
  # 为路由器添加子网
  openstack router add subnet route-wang1 int-wang1-sub
  
  # 设置路由器属性
  openstack router set route-wang1 --external-gateway ext-wang1 
  ~~~

  - `--external-gateway`：外部网络用作路由器的网关(名称或ID)

### 5.2、实例规格

~~~shell
# 创建实例规格
openstack flavor create test --ram 2048 --disk 20 --vcpus 2		# 2cpu,2G ram,20G disk

# 查看实例规格
openstack flavor list

# 删除规格
openstack flavor delete test 		# name or id
~~~

### 5.3、镜像

~~~shell
# 创建镜像
openstack image create --disk-format qcow2 --file /opt/images/CentOS-7-x86_64-2009.qcow2  --public centos7.9

# 查看镜像
openstack image list

# 注意：使用 openstack 创建镜像不能显示进度条，只能使用 glance image-create --progress 命令才显示。
~~~

- `--disk-format`：镜像磁盘格式。支持的选项有:ami、Ari、aki、vhd、vmdk、raw、qcow2、vhdx、vdi、iso、ploop。默认格式为:raw
- `--file`：从本地文件上传镜像
- `--public`：镜像对公众开放

### 5.4、实例/云主机

~~~shell
# 创建实例/云主机
openstack server create test-ws1 \
--image centos7.9 \
--flavor test \
--network int-wang1

# 查看
openstack server list
~~~

- `--image`：从这个映像(名称或ID)创建服务器引导磁盘
- `--flavor`：创建服务器指定的实例规格（name or ID）
- `--network`：在服务器上创建网卡，并将网卡接入网络。多次输入option可创建多个网卡。

### 5.5、卷

~~~shell
# 创建名为 disk 的 5G 卷
openstack volume create d1 --size 5

# 查看
openstack volume list

# 给云主机 test-ws2 添加一个名称为 d1 的卷
openstack server add volume test-ws2 d1
~~~

### 5.6、浮动ip

~~~shell
# 浮动 ip 通常指的是一种IP地址，它可以动态地分配给网络中的不同设备或服务。这种IP地址的"浮动"特性允许它在需要时移动到不同的设备，而无需手动更改配置。类似于华为云上的公网ip。
# 创建一个浮动 ip
openstack floating ip create ext-wang1

# 绑定浮动 ip
openstack server add floating ip test-wang1 10.10.220.36

# 卸载浮动 ip
openstack server remove floating ip test-wang1 10.10.220.36

# 查看浮动 ip
openstack floating ip list
~~~



### 5.7、用户，项目，权限

~~~shell
# 查看项目
openstack project list

# 创建一个新项目
[root@controller ~]# openstack project create p-ws
+-------------+----------------------------------+
| Field       | Value                            |
+-------------+----------------------------------+
| description |                                  |
| domain_id   | default                          |
| enabled     | True                             |
| id          | 349f44453d054875aabf5f36f4a03fc7 |
| is_domain   | False                            |
| name        | p-ws                             |
| options     | {}                               |
| parent_id   | default                          |
| tags        | []                               |
+-------------+----------------------------------+

# 查看用户
openstack user list
# 创建用户，基于 ws 这个项目的用户
[root@controller ~]# openstack user create ws --project p-ws --password 123
+---------------------+----------------------------------+
| Field               | Value                            |
+---------------------+----------------------------------+
| default_project_id  | 349f44453d054875aabf5f36f4a03fc7 |
| domain_id           | default                          |
| enabled             | True                             |
| id                  | 4066fc6c861b4894adccac602ec24c93 |
| name                | ws                               |
| options             | {}                               |
| password_expires_at | None                             |
+---------------------+---------------------------------

# 权限
# 在每一个组件的配置文件中，有名称叫 policy.json 的文件，内容可以修改各个组件对用户的权限配置。
~~~



### 5.8、安全组

~~~shell
# 查看安全组
[root@controller ~]# openstack security group list

# 创建安全组
[root@controller ~]# openstack security group create sg01 --tag test --project p-ws

# 查看安全组规则
[root@controller ~]# openstack security group rule list sg01

# 创建安全组规则
[root@controller ~]# openstack security group rule create 996c4db0-d0be-4702-8cfd-ad80245650a3 --remote-ip 0.0.0.0/0 --protocol udp --dst-port 22:80
~~~



### 5.9、其他

~~~shell
# 添加对象属性
# 添加镜像的一个属性
[root@controller ~]# openstack image set --property a=123 d95a6697-a888-43de-b6d5-efd38f1e3908
[root@controller ~]# glance image-update --property a=123 d95a6697-a888-43de-b6d5-efd38f1e3908

# 添加一个 tag 内容为 123 
[root@controller ~]# openstack image set --tag 123 d95a6697-a888-43de-b6d5-efd38f1e3908
~~~



## 6、Openstack Python SDK

实例：

Python运维开发：基于Openstack Python SDK实现云主机创建
使用已建好的OpenStack Python运维开发环境，在/root目录下创建sdk_server_manager.py脚本，使用python-openstacksdk Python模块，完成云主机的创建和查询。创建之前查询是否存在“同名云主机”，如果存在先删除该镜像。
（1）创建1台云主机：云主机信息如下：
云主机名称如下：server001
镜像文件：cirros-0.3.4-x86_64-disk.img
云主机类型：m1.tiny
网络等必要信息自己补充。
（2）查询云主机：查询云主机server001的详细信息，并以json格式文本输出到控制台。
完成后提交OpenStack Python运维开发环境 Controller节点的IP地址，用户名和密码提交。

~~~python
import json
from openstack import connection
server_ip = 'none'
conn = connection.Connection(
    auth_url=f'http://{server_ip}:5000/v3/',
    project_name="admin",
    domain_name="demo",
    username='admin',
    password='000000')

if __name__ == '__main__':
    # 创建 flavor
    conn.create_flavor(name="ws1",ram=1024,vcpus=2,disk=10)
    # 创建镜像
    conn.create_image(name="cirros",filename='./cirros-0.6.2-aarch64-disk.img',container_format='bare',disk_format='qcow2')
    # 创建网络
    conn.create_network(external=True,name='ext1')
    conn.create_subnet('ext1',subnet_name='ext1-subnet',cidr='192.168.150.0/24')
    # 创建实例
    conn.create_server(name='ws',image='cirros',flavor='ws1',network='ext1')
    # 遍历查看实例
    for i in conn.list_servers():
        print(i)
~~~











拆建云主机搭建步骤

每个脚本文件内容

1. source 变量文件
2. mysql 创建用户，修改密码，设置权限
3. 安装需要的 rpm 包
4. 修改组件配置文件
5. 同步数据库
6. 重启服务









日志



































