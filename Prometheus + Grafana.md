# Prometheus

## 一、概述

### 1.1 简介

Prometheus 是一个完全开源的系统监控和告警工具包，受 Google 内部 BorgMon 系统启发，自2012年由前 Google 工程师在 SoundCloud 开发以来，已被众多公司采用。它拥有活跃的开发者和用户社区，现为独立开源项目，并于2016年加入云原生计算基金会（CNCF）。Prometheus 的主要特点包括多维数据模型、灵活的查询语言 PromQL、不依赖分布式存储、通过 HTTP 拉取时间序列数据等。其架构简单且功能强大，支持多种图形和仪表盘展示模式。安装和使用 Prometheus 非常简便，可以通过 Docker 快速部署，并与 Grafana 等可。

### 1.2 组件

Prometheus 生态系统由多个组件组成，其中许多组件是可选的：
- 抓取和存储时间序列数据的主 [Prometheus server](https://github.com/prometheus/prometheus)
- 用于检测应用程序代码的[Client libraries](https://prometheus.io/docs/instrumenting/clientlibs/)
- 支持短期作业的[pushgateway](https://github.com/prometheus/pushgateway)
- HAProxy、StatsD、Graphite 等服务的特殊用途[exporters](https://prometheus.io/docs/instrumenting/exporters/) 。
- 用于处理警报的[alertmanager](https://github.com/prometheus/alertmanager)、
- 各种支持工具
大多数 Prometheus 组件都是用 [Go](https://golang.org/) 编写的，因此可以轻松构建和部署为静态二进制文件。

### 1.3 架构

此图说明了 Prometheus 的架构及其一些生态系统组件：

![[Pasted image 20250802001731.png]]
Prometheus 直接或通过中间推送网关从检测作业中抓取指标，以获取短期作业。它将所有抓取的样本存储在本地，并对这些数据运行规则，以从现有数据聚合和记录新时间序列或生成警报。[Grafana](https://grafana.com/) 或其他 API 使用者可用于可视化收集的数据。

### 1.4 prometheus vs zabbix

Prometheus和Zabbix都是非常流行的开源监控系统，但它们的设计理念和适用场景有很大的不同。简单来说，**Prometheus**更偏向于云原生和动态环境，而**Zabbix**则是一个功能全面的企业级监控解决方案。

主要区别：

| 特性         | Prometheus                                                        | Zabbix                                                                 |
| ---------- | ----------------------------------------------------------------- | ---------------------------------------------------------------------- |
| **架构**     | 分布式、去中心化。每个Prometheus Server独立运行，通过拉取（Pull）模型获取数据。                | 集中式、Client-Server架构。Server主动获取（Pull）数据或Agent推送（Push）数据。                |
| **数据模型**   | **多维时序数据**（Time-series data）。数据由指标名、标签（Labels）和时间戳组成，非常灵活。        | 传统关系型数据库（如MySQL）。数据以项目（Item）和主机（Host）为中心。                              |
| **数据采集**   | **拉取（Pull）模型**为主。Prometheus Server定期从被监控端（如服务、应用）暴露的HTTP端口拉取指标数据。 | **拉取（Pull）或推送（Push）模型**。Zabbix Server主动从Agent获取数据，或Agent主动推送数据给Server。 |
| **查询语言**   | **PromQL**。一种功能强大的查询语言，用于对多维数据进行复杂的聚合、计算和预测。                      | SQL或Zabbix内部的简单表达式。查询功能相对较弱，主要通过内置的图形和报表功能展示数据。                        |
| **警报规则**   | **独立**于Server。通过Alertmanager进行统一的警报管理，支持灵活的路由和静默。                 | **集成**在Server中。警报触发器与监控项紧密关联，配置相对直观。                                   |
| **可扩展性**   | 水平扩展性好。可以通过Federation或Thanos等组件实现大规模监控和数据长期存储。                    | 垂直扩展性好。通过Proxy可以实现分布式监控，但Server本身是单点，存在性能瓶颈。                           |
| **生态系统**   | **云原生生态系统**。与Kubernetes、Grafana等工具无缝集成，是云原生监控的事实标准。               | **企业级生态系统**。提供丰富的预置模板、Agent和API，适合监控传统服务器、网络设备等。                       |
| **典型应用场景** | 容器、微服务、动态服务发现等云原生环境。                                              | 传统物理机、虚拟机、网络设备、数据库等企业IT环境。                                             |





## 二、部署

### 2.1 环境准备

环境：当前使用 all in one 部署，全使用一台4c8g的ECS。

| 系统           | Prometheus |
| ------------ | ---------- |
| ubuntu 22.04 | 3.5.0      |
### 2.2 时间同步

因为 Prometheus 使用的时许数据库，时间同步至关重要！！
```bash
root@ecs-prometheus:~# yum install -y ntpdate
root@ecs-prometheus:~# crontab -e
0 12 * * * /usr/sbin/ntpdate ntp.aliyun.com
```

### 2.3 部署方式

参考文档：[Installation | Prometheus](https://prometheus.io/docs/prometheus/latest/installation/)
#### 2.3.1 二进制部署

下载地址：[Download | Prometheus](https://prometheus.io/download/)

~~~shell
root@ecs-prometheus:~# tar -xf prometheus-3.5.0.linux-amd64.tar.gz                     
root@ecs-prometheus:~# mv prometheus-3.5.0.linux-amd64 /opt/                            
root@ecs-prometheus:~# ln -s /opt/prometheus-3.5.0.linux-amd64/ /opt/prometheus
~~~

目录说明：

| 目录说明           | 详解     |
| -------------- | ------ |
| prometheus     | 服务端的命令 |
| prometheus.yml | 配置文件   |
版本检查：

~~~shell
root@ecs-prometheus:/opt/prometheus# ./prometheus --version                             
prometheus, version 3.5.0 (branch: HEAD, revision: 8be3a9560fbdd18a94dedec4b747c35178177202)                                               
  build user:       root@4451b64cb451                                                   
  build date:       20250714-16:15:23                                                   
  go version:       go1.24.5                                                            
  platform:         linux/amd64                                                         
  tags:             netgo,builtinassets
~~~

启动：

~~~shell
root@ecs-prometheus:/opt/prometheus# ./prometheus  --config.file="prometheus.yml"
~~~

#### 2.3.2 容器化部署

~~~shell
docker run \
    -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
~~~


### 2.4 配置
#### 2.4.1 服务端命令行配置

| Prometheus 核心参数                   | 详解                         |
| --------------------------------- | -------------------------- |
| --[no-]version                    | 展示版本号                      |
| --config.file="prometheus.yml"    | Prometheus 配置文件路径          |
| --web.listen-address=0.0.0.0:9090 | 前端web页面，端口和监听地址            |
| --web.max-connections=512         | 并发连接数                      |
| --log.level=info                  | Prometheus 日志默认输出到屏幕（标准输出） |
| --log.format=logfmt               | 日志格式                       |

#### 2.4.2 服务端配置文件配置

Prometheus 配置为 [YAML](https://yaml.org/)。Prometheus 下载在名为 `prometheus.yml` 的文件中附带了一个示例配置，这是一个很好的入门位置。
我们删除了示例文件中的大部分注释，使其更加简洁（注释是以 `#` 为前缀的行）。
~~~yaml
global:
  scrape_interval:     15s
  evaluation_interval: 15s

rule_files:
  # - "first.rules"
  # - "second.rules"

scrape_configs:             
  - job_name: prometheus
    static_configs:
      - targets: ['localhost:9090']
~~~

示例配置文件中有三个配置块：`global`、`rule_files` 和 `scrape_configs`。

`全局`块控制 Prometheus 服务器的全局配置。我们有两个选择。第一个是 `scrape_interval`，控制 Prometheus 抓取目标的频率。您可以为单个目标覆盖此设置。在这种情况下，全局设置为每 15 秒抓取一次。`evaluation_interval` 选项控制 Prometheus 评估规则的频率。Prometheus 使用规则创建新时间序列并生成警报。

`rule_files` 块指定我们希望 Prometheus 服务器加载的任何规则的位置。目前我们还没有规则。

最后一个块 `scrape_configs` 控制 Prometheus 监控哪些资源。由于 Prometheus 还将有关自身的数据公开为 HTTP 端点，因此它可以抓取和监控自己的健康状况。在默认配置中，有一个名为 `prometheus` 的作业，它抓取 Prometheus 服务器公开的时间序列数据。该作业包含单个静态配置的目标，即端口 `9090` 上的`本地主机` 。Prometheus 希望指标在`路径为 /metrics` 的目标上可用。因此，此默认作业是通过 URL 进行抓取：[http://localhost:9090/metrics](http://localhost:9090/metrics)。

返回的时间序列数据将详细说明 Prometheus 服务器的状态和性能。

有关配置选项的完整规范，请参阅 [配置文档](https://prometheus.io/docs/operating/configuration/) 。

### 2.5 systemctl 托管

~~~sh
root@ecs-prometheus:/opt/prometheus# cat /lib/systemd/system/prometheus.service         
[Unit]                                                                                  
Description=prometheus server                                                          
After=network.target auditd.service                                                     
[Service]                                                                               
ExecStart=/opt/prometheus/prometheus --config.file="/opt/prometheus/prometheus.yml"     
KillMode=process                                                                        
Type=simple                                                                             
[Install]                                                                               
WantedBy=multi-user.target
~~~
### 2.6 Exporter




# Grafana

