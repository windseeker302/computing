# 架构

华为的私有云平台，由产品组件和公共组件组合而成。
## 产品组件

ManagerOne(CMP)
- ManagerOne 在解决方案中承担CMP的职责，通过自研和集成的方式，为企业客户提供对企业自建云资源及企业租用的公有云资源统一管理的能力。
  由服务中心（SC）、运维中心（OC）和运营指挥中心（OCC）三大模块组成。
	- 服务中心（Service Center）是ManagerOne面向租户和运营管理的入口，提供同云服务的运营集成能力，支持多种云服务集成到ManagerOne。通过Console Home集成各云服务Console。
	- 运维中心（Operation Center）是ManageOne运维管理的唯一入口，提供云服务运维管理的能力，实现对云服务端到端的监控能力，包括云服务自身、租户资源和云服务所依赖的基础设施（计算，存储，网络）。
	- 运维指挥中心（Operation Command Center）是华为混合云面向政企客户新推出的混合云大脑，帮助客户构建成本，效率，质量，风险，合规全方位的IT指挥运营分析决策体系。
		- 可选
FusionSphere OpenStack
- FusionSphere CPS
	CloudProvisioningService（CPS）负责laaS的云平台层的部署和升级，是laaS层中真正面向硬件设备，并将其池化软件化的部件。从外部看，CPS的作用就是把laaS层的各种服务给部署好、配置好、升级好。
- Service OM
	ServiceM是资源池（计算、存储、网络）以及基础云服务（ECS、EVS、VPC等）的管理工具，管理员使用ServiceM对资源池及基础云服务进行管控和配置。
	 - web界面需要从OC进入
eSight
- 提供服务器、存储设备和网络设备的统一管理。
- 可选
FusionCare 工具
- 巡检：各云服务向APIGateway注册巡检接口，从而通过FusionCare实现各云服务的统一巡检能力。
- 日志收集： 通过API平面调用openStack中lnfo-collect-server发送巡检请求消息。
CloudNetDebug 工具
- 面向运维人员，实现界面自动化抓包和拨测的运维工具


>问：FustionSphere OpenStack（CPS、ServiceOM）、eSight与ManagerOne的关系？
>ServiceOM负责采集计算、存储、冈网络等软件资源池信息，比如告警、性能数据等等。
>eSight负责采集硬件(服务器、存储、交换机、路由器等)信息，比如告警、性能数据等等。
>ServiceOM和eSight同时将采集到的信息上报到ManageOne的运维中心，并通过统一的界面对用户进行呈现。
## 公共组件

负载均衡
- Lvs
- Nginx
- HAProxy
域名解析
- DNS-OM
- DNS-Tenant
时钟服务
- DMZ_NTP
- OM_NTP
DMK
- DMK（Deploy Managerment Kit）提供同意的部署与安装配置平台。
- 实现云服务的安装部署与补丁升级功能。
- DMK本身也是基于Ansible作为自动化执行引擎来进行相关的自动化运维能力。
API网关
- API网关（API Getaway）配套行业解决方案。
- 提供高性能、高可用、高安全的API托管服务。
- 一款涵盖API运行、管理、分析、安全为一体的端到端API产品。
- 特性 
	- 管理API
	- 流控
	- 认证
	- 安全
- 部署
	- 两台API-LB 主主
	- 三台APIGWZK 主备

组合API 
- 组合AP为弹性云服务器、云硬盘和云硬盘备份等服务提供后台服务，可以理解为console的服务端。组合AP作为公共服务平台，支持云服务的请求、响应、子任务持久化。目前已经托管的服务有：弹性云服务器、云磁盘云硬盘备份、弹性负载均衡、镜像、VPC（创建EIP）、裸金属服务器。
CCS
- CCS（CloudConfiguration Service）在华为云Stack方案中，对云区域、可用分区、云主机规格、云磁盘类型以及外部网络`提供标签管理功能`，并为ECS、EVS、VPC等云服务提供相应标签查询功能。
SDR
- SDR作为公共组件，对云服务提供的资源进行准确计量，生成离线话单文件，供计费系统采集完成计费。
- SDR具备数据采集能力、话单上传能力，各服务提供话单处理逻辑，共同完成离线话单生成。
任务中心
- 任务中心是任务的统一管理界面，运维人员可通过任务中心统一管理注册或托管至任务中心的任务。同时，并支持修改资源采集任务调度周期，用户可根据实际情况配置
GuessDB
- GaussDB是华为自主研发的关系型磁盘数据库管理系统，具备通用的数据库管理功能。基于PostgreSQL9.2开发，在兼容性、性能、安全、可用性和可维护性上做了增强。


# 网络平面

External_OM：OpenstackManger，管理openstack的网络平面，控制节点上有External_OM网络平面。
Internal_Base：openstack内部互联的平面，比如neutron找swift，就是用Internal_Base平面互通。
Tunnel_Bearing：隧道方向，用于控制节点，计算机节点，核心交换机与网络节点互通的网络平面，用于访问网络节点所创建的网元设备，但Tunnel_Bearing平面并不是服务器上的真实网卡，是通过vxlan技术得来的。
Inter_Connect： 网元设备互联，是由网络节点创建出的虚拟机网络设备的ip，网络节点创建出的虚拟机成为网元设备，比如br（边界路由,Boundary Router）设备 
DMZ_Service：非军事管理区
Management_Storage_Data：存储管理网络平面
External_Relay_Network：内大网


案例：
> 为什么你能使用 EIP ping 通绑定的实例？
> client icmp 请求流量会流经核心交换机，核心交换机上写有eip的路由，路由指向网络节点上的网元设备br上，因为核心交换机和网络节点的tunnel_Bearing平面互通，所以流量会到br网元设备上，br网元设备又会通过tunnel_Bearing网络平面找到ENAT网元设备上，ENAT会做源目的地址的转换，最终找到实例的私网（VPC）地址。

同VPC同子网同主机内ECS间互访流量走向：
![[Pasted image 20250717001701.png]]


同VPC同子网跨主机ECS间互访流量走向：
![[Pasted image 20250717001722.png]]

同VPC不同子网同主机ECS间互访流量走向：
![[Pasted image 20250717001754.png]]

neutron 网络模型
![[Pasted image 20250717010728.png]]



# 产品

## 1.FusionCompute

FusionCompute是云操作系统软件，主要负责硬件资源的虚拟化，以及对虚拟资源、业务资源、用户资源的集中管理。它采用虚拟计算、虚拟存储、虚拟网络等技术，完成计算资源、存储资源、网络资源的虚拟化。同时通过统一的接口，对这些虚拟资源进行集中调度和管理，从而降低业务的运行成本，保证系统的安全性和可靠性，协助运营商和企业构筑安全、绿色、节能的云数据中心能力。

### 1.1 产品架构

CNA：Compute Node Agent，CNA部署在需要虚拟化的服务器上。  
VRM：Virtual Resource Management，VRM可以部署成VM或者部署在物理服务器上；VRM对外提供网页操作界面供管理维护人员。

| 模块  |                                                                                                           功能                                                                                                           |
| :-: | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
| CNA |                                                                                   提供虚拟计算功能  <br>管理计算节点上的虚拟机  <br>管理计算节点上的计算、存储、网络资源                                                                                    |
| VRM | 管理集群内的块存储资源<br>管理集群内的网络资源(IP/VLAN)，为虚拟机分配IP地址<br>管理集群内虚拟机的生命周期以及虚拟机在计算节点上的分布和迁移<br>管理集群内资源的动态调整<br>通过对虚拟资源、用户数据的统一管理，对外提供弹性计算、存储、IP等服务<br>通过提供统一的操作维护管理接口，操作维护人员通过WebUI远程访问FusionCompute对整个系统进行操作维护，包含资源管理、资源监控、资源报表等。 |


## 2.FusionAccess

[华为云计算（6）——FusionAccess - 知乎](https://zhuanlan.zhihu.com/p/332509273)

FusionAccess是华为推出的桌面云产品，是一种虚拟桌面应用，它主要通过在硬件上部署FusionAccess配套的软件基础上，虚拟化出相互隔离的桌面，用户通过瘦客户端等方式连接到云桌面来使用此桌面，目前华为最新的FusionAccess软件架构已经更新到8.0版本。

### 2.1 产品架构

**1.TC/SC（Thin Client、Software Client）**：TC是硬件的接入终端盒子，SC则是软件浏览器的接入。

**2.WI（Web Interface）**：WI为最终用户提供Web登录界面，与liteAS、[HDC](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=HDC&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJIREMiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjIwMjE3MDAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.jtr6pV0wDwZKXkSanfBEF_cxAAUWbPPf1uV151Kbwgk&zhida_source=entity)进行数据交互等。

**3.UNS（Unified Name Service）**：为桌面云提供统一的域名，减少用户在不同的域名之间转换。

**4.vAG（Virtual Access Gateway）**：虚拟接入网关，可选软件，vAG主要负责用户登入桌面时流量的转发和当用户桌面出现故障时的维护。

**5.[vLB](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=vLB&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJ2TEIiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjIwMjE3MDAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.qCrlMOolkXVgn7lJxuLrygwHk4CMRlgD0OrEGA5hrUo&zhida_source=entity)（Virtual Loadbalance）**：虚拟负载均衡器，vLB负责对WI进行负载，防止用户大量的访问请求都集中到一个WI上。将多个WI地址绑定到一个域名上，当用户请求访问WI时，vLB将多个请求依次解析到WI上，也就意味着vLB是轮询负载模式。

**6.HDC（Huawei Desktop Controller）**：华为桌面控制器，HDC是桌面云的核心组件，几乎所有的核心信息都是由HDC来完成。例如与DB、[HDA](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=HDA&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJIREEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjIwMjE3MDAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.4i_YS8fvm_HkF1ldpJNRIaP9pJzqh7GT_5gGYS_-i6Q&zhida_source=entity)的通信等。

**7.[ITA](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=ITA&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJJVEEiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjIwMjE3MDAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.C_VRucU-D2iD2LESxWcGJRIJUs_JpL56UOacsHWchr0&zhida_source=entity)（IT Adaptor）**：IT适配器，桌面云的创建、管理等操作指令都由ITA来下发。

**8.[GaussDB](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=GaussDB&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJHYXVzc0RCIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MTYyMDIxNzAwLCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.ANN2G4JLk7P0WlFvBiBHNzpqQbz8jGjGqnfUWvpcKsk&zhida_source=entity)**：华为自研的一款数据库软件，在桌面云架构中负责记录桌面的信息，例如注册信息、用户所属虚拟机列表、用户桌面配置策略信息等。

**9.HDA（Huawei Desktop Agent）**：华为桌面代理，这款软件主要是安装在用户桌面上，HDA会和HDC进行通信（心跳、注册信息等），一些HDC的配置信息由HDC下发到HDA，由HDA完成。

**10.[LiteAS](https://zhida.zhihu.com/search?content_id=162021700&content_type=Article&match_order=1&q=LiteAS&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NTQ2NDIxMTEsInEiOiJMaXRlQVMiLCJ6aGlkYV9zb3VyY2UiOiJlbnRpdHkiLCJjb250ZW50X2lkIjoxNjIwMjE3MDAsImNvbnRlbnRfdHlwZSI6IkFydGljbGUiLCJtYXRjaF9vcmRlciI6MSwiemRfdG9rZW4iOm51bGx9.VRbpDoI1Q_Q9o3_p8U6pJY68_fCVfNluvx0C3DfxbOg&zhida_source=entity)**：统一鉴权服务器，用户登录桌面云时的用户名和密码的验证都是由LiteAS来完成。本质上可以理解为是AD做的验证与鉴权。

**11.Lincese**：Lincese服务器，主要负责用户数量、FusionAccess软件规模的限制等。

**12.TCM：**TCM是终端管理，当有大量的终端TC时，为了方便管理TC，例如统一升级等，所设置的一款终端管理设备。

### 2.2桌面云协议

HDP
是华为自研的新一代云接入桌面协议
特点：
- 最大支持64虚拟通道，每个虚拟通道可承载不同的上层应用协议。
- 可根据不同的应用类型采用不同的压缩算法，灵活使用服务器渲染及本地加速渲染
- 视频播放更清晰流畅
- 无损压缩算法
- 还原声音细节
- 丰富协议管理策略

其他常见的桌面协议
ICA/HDX：ICA是目前应用较多的虚拟桌面协议之一，HDX是ICA的增强版
PCoIP：最大特点就是，将用户的会话以图像的方式进行压缩传输，对于用户的操作，只传输变化部分，保证在低带宽下也能高效的使用
SPICE：一款开源虚拟桌面协议，最初是由Qumranet开发，后来被RedHat收购并开源
RDP/RemoteFX：微软的远程桌面协议，主要用在windows环境中。RemoteFX是RDP的增强版

## 3.FusionStorage

[华为云计算（5）——FusionStorage - 知乎](https://zhuanlan.zhihu.com/p/332315648)

FusionStorage是华为推出的一款可大规模横向扩展的分布式存储系统，其本质是将通用的X86、ARM服务器的本地已有的HHD磁盘，SATA盘，SAS盘，SSD盘等介质，利用分布式技术组成大规模存储资源池。利用软件系统模拟出SCSI和iSCSI接口向上层应用提供块存储服务，以满足云资源池及数据库等场景的存储需求。

### 3.1 版本

FusionStorage存储系统分为了三个不同类型的版本：

1. FusionStorage Block（块存储）
2. 2.FusionStorage File（文件存储）
3. 3.FusionStorage OBS（分布式对象存储）通常我们说FusionStorage是指Block版本，本文重点也是关于块存储。

### 3.2 创造思想：

1. 数据写入是随机的
2. 存储的数据是均匀的分布在存储介质上

首先在创建存储池时，FusionStorage默认是在一块磁盘上部署一个OSD（进程），多个OSD接管上来的磁盘组成了一个地址空间，这就是存储池。

FusionStorage利用OSD所接管的底层磁盘提供存储的空间后，VM虚拟机等要想使用存储服务，首先就是能找到存储的地址。FusionStorage利用DHT分布式hash算法，将OSD提供上来的磁盘分成了2的32次方个地址块。每一个地址块用LBA ID作唯一标识，由于此地址空间过大，FusionStorage存储系统为了更好的管理与查找LBA ID，便将多个LBA ID组成的连续地址空间划分为一个Partition（区），DHT分布式hash算法的作用就是保存LBA ID与Patition的对应的关系（即LBA ID属于哪个一个Partition），与Patition对应OSD的关系（即Partition用到了哪个OSD提供上来的地址空间）。

Patition分区的个数是固定3600个。

## 4.eBackup

华为备份软件open-eBackupt正式开源
- 随着数据的数量、种类和增长速度呈现指数级变化，同时，由于人为错误、病毒、自然灾害以及其他网络安全威胁等原因，企业面临越来越多的数据丢失的风险，数据保护的重要性日益明显。
- open-eBackup为主流数据库、虚拟化、文件系统、大数据等应用提供E2E包含数据备份、数据恢复能力，帮助用户实现关键数据高效备份，节省数据保护投资。

open-eBackup 开源项目地址： [项目首页 - open-eBackup:open-eBackup是一款开源备份软件，采用集群高扩展架构，通过应用备份通用框架、并行备份等技术，为主流数据库、虚拟化、文件系统、大数据等应用提供E2E的数据备份、恢复等能力，帮助用户实现关键数据高效保护。 - GitCode](https://gitcode.com/eBackup/open-eBackup)
## 5.OceanStor 100D

[OceanStor 100D智能分布式存储-FusionStorage-华为企业业务](https://e.huawei.com/cn/products/storage/scale-out-storage/oceanstor-100d)

OceanStor 100D(原FusionStorage) 是一款可大规模横向扩展的全自研智能分布式存储产品，可为上层应用提供文件存储、大数据存储、对象存储等工业界标准接口，消除烟囱式存储系统构建导致的运营复杂问题，帮助企业实现复杂业务承载更稳、多样性数据使用效率更高、海量数据储存成本更优。

1. 文件存储：兼容原生NFS、SMB协议，兼容POSIX和MPI，适用于HPC（High-Performance Computing，高性能计算）等高性能场景。
2. 大数据存储：提供基于原生HDFS的大数据存算分离方案，实现存储与计算资源按需配置，提供一致用户体验的同时，助您降低总拥有成本；支持与原有计算存储一体化架构共存。广泛应用于金融大数据、互联网日志留存大数据、政务大数据和智慧城市大数据等场景。
3. 对象存储：最大支持单桶1000亿对象承载且性能不降，消除大型应用分桶改造麻烦；自动分级到蓝光介质，免数据迁移省空间。广泛应用于金融电子票据影像和双录（录音/录像）、医疗影像、政企电子文档和车联网场景生产存储、备份或归档。






