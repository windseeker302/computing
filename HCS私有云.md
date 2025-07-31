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


