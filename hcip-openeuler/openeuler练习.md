# Keepalive配置实践

## 1. Keepalived实现Nginx的高可用集群

### 1.1 实验组网介绍

本实验由三台虚拟机组成，Nginx1和Nginx2可以和上实验复用，其上运行Nginx服务，并通过Keepalived虚拟出一台主机供客户端访问。Nginx1和Nginx2之间通过10网段进行心跳监控，并通过浮动IP 192.168.1.20对外提供服务，如果其中一台主机出现故障或Nginx进程出现故障，业务自动切换到另外一台主机上。

### 1.2 实验步骤

安装 keepalive

```shell
# 在Nginx1和Nginx2上执行以下命令，进行Keepalived的安装： 
yum install -y keepalived
```



Keepalived配置

```shell
# 此次实验将Nginx1设置为主节点，Nginx2设置为备节点，因此将Nginx1上的keepalived配置文件（/etc/keepalived/keepalived.conf）修改为以下内容 !  ! Configuration File for keepalived

global_defs {
   router_id Nginx1
}

vrrp_instance Nginx {
    state MASTER
    interface ens192
    virtual_router_id 51
    priority 225
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass Huawei@1
    }
    virtual_ipaddress {
        192.168.1.20/24
    }
}

# 将Nginx2的keepalived配置文件修改为以下内容
! Configuration File for keepalived

global_defs {
   router_id Nginx2
}

vrrp_instance Nginx {
    state BACKUP
    interface ens192
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass Huawei@1
    }
    virtual_ipaddress {
        192.168.1.20/24
    }
}
```



配置完成后，分别在Nginx1和Nginx2上使用以下命令重启Keepalived服务：

```shell
systemctl restart keepalived
```



### 1.3 健康检查配置

目前Keepalived仅能通过主机是否宕机来进行业务切换，如果仅Nginx业务出现故障是无法切换的，需在Keepalive中添加健康检查来达到检测Nginx是否可用的目的。

首先开启Nginx1和nginx2，并分别在Nginx1和Nginx2的Keepalived配置文件中添加健康检查的相关配置，具体如下：

```shell
global_defs {
   router_id Nginx1
}
vrrp_script nginx_check {
  script “/etc/keepalived/check.sh”
  interval 1
  weight -5
  fail 3
} 
vrrp_instance Nginx {
……
track_script {
  nginx_check
}
}
```



## 2. Keepalived+LVS实现Nginx集群

### 2.1 实验介绍

本实验将使用Keepalived为LVS提供高可用配置，同时LVS为后端的Nginx1和Nginx2提供负载均衡，最终通过Keepalived+LVS实现Nginx集群。

### 2.2 实验步骤

在LVS1和LVS2虚拟机上安装keepalived和ipvs

```shell
yum install -y keepalived ipvsadm
```

修改LVS1的配置文件

```shell
# 将LVS1的配置keepalived配置文件修改为以下内容 ! Configuration File for keepalived

global_defs {
   router_id Cluster1
}

vrrp_instance Nginx {
    state MASTER
    interface ens192
    mcast_src_ip 20.0.0.1
    virtual_router_id 51
    priority 255
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.20/24
    }
}

virtual_server 192.168.1.20 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    real_server 192.168.1.14 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.1.15 80 {
        weight 2
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}
```



修改LVS2的配置文件

```shell
# 将LVS2的配置keepalived配置文件修改为以下内容：
! Configuration File for keepalived

global_defs {
   router_id Cluster2
}

vrrp_instance Nginx {
    state BACKUP
    interface ens192
    mcast_src_ip 20.0.0.2
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.20/24
    }
}

virtual_server 192.168.1.20 80 {
    delay_loop 6
    lb_algo rr
    lb_kind DR
    persistence_timeout 50
    protocol TCP

    real_server 192.168.1.14 80 {
        weight 1
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
    real_server 192.168.1.15 80 {
        weight 2
        TCP_CHECK {
            connect_timeout 3
            retry 3
            delay_before_retry 3
        }
    }
}
```

测试完成就会发现为什么每次都是有Nginx2响应服务？

通过LVS的链接默认开启了持久连接，且超时时间设置为了50，因此，每次连接都转发至第一次分配的服务器上，将Keepalived配置文件中的相关配置注释掉即可实现负载均衡

```shell
...
virtual_server 192.168.1.20 80 {   
	delay_loop 6   
	lb_algo rr   
	lb_kind DR 
	# persistence_timeout 50   
	protocol TCP 
...	
```

重启完成后，就会轮训到 nginx1和nginx2上了。
