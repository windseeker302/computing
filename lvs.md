# LVS负载均衡

## 基础概念

LVS（Linux Virtual Server）即Linux虚拟服务器，是由张文嵩博士主导的开源负载均衡项目，目前LVS已经被集成到Linux内核模块中（2.6及以上版本内核），LVS本质上是为了解决单台服务器性能处理瓶颈的问题，LVS在Linux内核中实现了基于IP的数据请求负载均衡调度方案，终端互联网用户从外部访问公司的外部负载均衡服务器，终端用户的Web请求会发送给LVS调度器，调度器根据自己预设的算法觉得将请求发送给后端的某台真实的Web服务器，同时如果真实服务器连接的是共享存储，那么提供的服务也是相同的，最终用户不管是访问哪台真实服务器，得到的服务内容都是一样的，因此整个集群对用户而言都是透明的。根据LVS工作模式的不同，LVS工作模式分为NAT模式、TUN模式、以及DR模式，以及阿里自己研发的FULL-NAT模式，不过Full--NAT模式没有被编译进内核，工作中常用的模式是DR模式，也算默认的工作模式。

~~~shell
# 查看内核是否拥有 lvs 模块
grep -i -C 10 ipvs /boot/config-4.19.90-2312.1.0.0255.oe2003sp4.x86_64
~~~



基础术语

- LB (Load Balancer 负载均衡)
- HA (High Available 高可用)
- Failover (失败切换)
- Cluster (集群)
- LVS (Linux Virtual Server Linux 虚拟服务器)
- DS (Director Server)，指的是前端负载均衡器节点
- RS (Real Server)，后端真实的工作服务器
- VIP (Virtual IP)，虚拟的 IP 地址，向外部直接面向用户请求，作为用户请求的目标的 IP 地址
- DIP (Director IP)，主要用于和内部主机通讯的 IP 地址
- RIP (Real Server IP)，后端服务器的 IP 地址
- CIP (Client IP)，访问客户端的 IP 地址



管理工具

- 手动管理：ipvsadm
- 配置文件管理：keepalived

## 四种工作模式

### 1、nat模式

LVS - NAT本质是多目标IP的DNAT，通过将请求报文中的目标地址和目标端口修改为某挑出的RS的RIP和PORT实现转发。

特点

1. RIP和DIP应在同一个IP网络，且应使用私网地址；RS的网关要指向DIP
2. 请求报文和响应报文都必须经由Director转发，Director易于成为系统瓶颈
3. 支持端口映射，可修改请求报文的目标PORT（RealServer的Port端口）
4. VS必须是Linux系统，RS可以是任意OS系统

[![img](https://img2020.cnblogs.com/blog/1871335/202011/1871335-20201112181352054-2139978547.png)](https://img2020.cnblogs.com/blog/1871335/202011/1871335-20201112181352054-2139978547.png)

> 在实际的场景中，Clients和LVS之间会跨越互联网以及企业入口防火墙、核心交换机等设备，这里为了简要说明，就模拟Client与LVS直连的情况，如下图所示，当客户端的请求到达LVS调度器之后，LVS会结合系统的netfilter模块对数据包进行判断处理，同时根据LVS自身的调度策略，修改报文目的IP地址和

### 2、TUN模式

TUN模式本质上同DR的方式一样，只是在Director和RealServer之间是通过IP隧道的方式去通信，一般这种方式只在企业需要在多个异地机房做调度时，可以采用这种方式，因此较为不常用。

### 3、DR模式

在DR模型中，LVS服务器只负责请求报文的调度，并且在调度过程中只是修改链路层MAC地址，不修改IP层和传输层头部字段，经过RealServer处理后的返回报文，也是通过VIP和CIP对报文进行封装，因此返回数据包会直接路由到网关，从而经过互联网到达客户端，不会像NAT模式中的返回报文还回到LVS服务器，因此在DR模式中，大大降低了LVS的服务器处理压力，其处理瓶颈基本上只是在LVS硬件本身的性能，下图展示了DR模型下，数据包的大致处理流程，此处忽略了客户端报文到达企业核心的细节过程，因此中间还包括运营商网络、企业入口防火墙等设备。

![img](https://gitee.com/ws203/pic-go-images/raw/master/imgs/1871335-20201112181425448-382930265.png)

### 4、Full-nat模式



### lvs 调度算法

- rr：轮询
- 加权轮询 wrr：某台服务器很好，可以权重大的请求的数量多一点。
- 最小连接 lc：请求最少的访问多一点
- 加权最小连接wlc：给请求少的加权重，访问的多一点
- lblc：基于局部性的最小连接
- lblcr：带复制的基于局部性最小连接 
- dh：目标地址散列调度 
- sh：源地址散列调度 



## lvs DR模式负载+keepalived高可用

lvs配合keepalived实现director调度器的高可靠，keepalived的实现原理就是利用了计算机网络中的VRRP（虚拟网络冗余协议）技术，其本身是为了解决网关单点故障的问题，如下图右上部分，两台Director设备通过心跳来维持一个向外开放虚拟IP，这里也就是192.168.113.100，LVS集群向外提供服务器的地址，这个地址只能同时存在一个设备上，当其中一台设备故障，心跳丢失，虚IP就会漂移到BACKUP设备上，从而接管业务继续调度。

### 实验环境

软件环境：openEuler22.03-TLS-SP3、Keepalived2.2.4、ipvsadm1.31

| 主机名 | IP地址                                   | 角色           |
| ------ | ---------------------------------------- | -------------- |
| db01   | RIP:192.168.113.11                       | 真实服务器     |
| db02   | RIP:192.168.113.142                      | 真实服务器     |
| lvs01  | DIP:192.168.113.149，VIP:192.168.113.100 | keepalived+lvs |
| lvs02  | DIP:192.168.113.148，VIP:192.168.113.100 | keepalived+lvs |
| client |                                          | 客户端         |

### 安装

~~~shell
# keepalived
yum install -y keepalived ipvsadm
# mariadb
yum install -y mariadb-server
systemctl enable --now mariadb
~~~



### keepalived 配置

1、lvs01配置如下，lvs设置为DR模式。

~~~shell
! Configuration File for keepalived

global_defs {
   router_id LVS01
}

vrrp_instance VI_1 {
    state MASTER		# 主
    interface ens160		# 心跳网卡接口
    virtual_router_id 51
    priority 100		# 优先级值设定：MASTER 要比 BACKUP 的值大
    advert_int 1		# 通告时间间隔：单位秒，主备要一致
    authentication {		# 认证机制，主从节点保持一致即可
        auth_type PASS		
        auth_pass 1111
    }
    virtual_ipaddress {	
        192.168.113.100		# VIP，可配置多个
    }	
}

# LB 配置
virtual_server 192.168.113.100 3306 {
    delay_loop 6		# 设置健康状态检查时间
    lb_algo rr		# 调度算法，这里用了 rr 轮询算法
    lb_kind DR			# lvs模式为 DR
    persistence_timeout 50		# 持久连接超时时间
    protocol TCP

    real_server 192.168.113.11 3306 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.113.142 3306 {
        weight 1
        TCP_CHECK{
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}
~~~



2、复制之前的配置文件，修改如下：

~~~shell
! Configuration File for keepalived

global_defs {
   router_id LVS02		# 修改
}

vrrp_instance VI_1 {
    state BACKUP		# 修改
    interface ens160
    virtual_router_id 51
    priority 150		# 修改
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.113.100
    }
}

virtual_server 192.168.113.100 3306 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    real_server 192.168.113.11 3306 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.113.142 3306 {
        weight 1
        TCP_CHECK{
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}
~~~



3、配置完成后，分别重启 Keepalived 服务。

```shell
systemctl restart keepalived
```



### 配置 Realserver

```shell
# 绑定VIP
VIP=192.168.113.100
ifconfig lo:1 $VIP netmask 255.255.255.255 broadcast $VIP
route add -host $VIP dev lo:1
# 抑制ARP广播
echo "1" >/proc/sys/net/ipv4/conf/lo/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/lo/arp_announce
echo "1" >/proc/sys/net/ipv4/conf/all/arp_ignore
echo "2" >/proc/sys/net/ipv4/conf/all/arp_announce
```



### mairadb 配置

~~~shell
# 启动 mariadb后，初始化数据库，db01 和 db02 都需要初始化
[root@mariadb01 ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): 
OK, successfully used password, moving on...

Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.

Set root password? [Y/n] y
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!


By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.

Remove anonymous users? [Y/n] y
 ... Success!

Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? [Y/n] n
 ... skipping.

By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.

Remove test database and access to it? [Y/n] y
 - Dropping test database...
 ... Success!
 - Removing privileges on test database...
 ... Success!

Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.

Reload privilege tables now? [Y/n] y
 ... Success!

Cleaning up...

All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.

Thanks for using MariaDB!
~~~



### 验证

在 lvs 任意节点执行：

~~~shell
[root@lvs01 ~]# ipvsadm -Ln --stats
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port               Conns   InPkts  OutPkts  InBytes OutBytes
  -> RemoteAddress:Port
TCP  192.168.113.100:3306                0        0        0        0        0
  -> 192.168.113.11:3306                 0        0        0        0        0
  -> 192.168.113.142:3306                0        0        0        0        0
~~~



在客户端执行：

~~~shell
[root@test ~]# mysql -uroot -p000000 -h192.168.113.100
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 465
Server version: 10.3.39-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> exit
Bye
~~~







参考文档：[lvs负载简介,原理,常见使用案例及Keepalived高可用 - 常见-youmen - 博客园 (cnblogs.com)](https://www.cnblogs.com/you-men/p/13965366.html)

