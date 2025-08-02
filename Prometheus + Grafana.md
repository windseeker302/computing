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
#### 1) 二进制部署

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

#### 2) 容器化部署

容器镜像同步站：[渡渡鸟](https://docker.aityp.com/)

~~~shell
docker run \
    -d -p 9090:9090 \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
~~~


### 2.4 配置
#### 1) 服务端命令行配置

| Prometheus 核心参数                   | 详解                         |
| --------------------------------- | -------------------------- |
| --[no-]version                    | 展示版本号                      |
| --config.file="prometheus.yml"    | Prometheus 配置文件路径          |
| --web.listen-address=0.0.0.0:9090 | 前端web页面，端口和监听地址            |
| --web.max-connections=512         | 并发连接数                      |
| --log.level=info                  | Prometheus 日志默认输出到屏幕（标准输出） |
| --log.format=logfmt               | 日志格式                       |

#### 2) 服务端配置文件配置

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

最后一个块 `scrape_configs` 控制 Prometheus 监控哪些资源。由于 Prometheus 还将有关自身的数据公开为 HTTP 端点，因此它可以抓取和监控自己的健康状况。在默认配置中，有一个名为 `prometheus` 的作业，它抓取 Prometheus 服务器公开的时间序列数据。该作业包含单个静态配置的目标，即端口 `9090` 上的`本地主机` 。Prometheus 希望指标在`路径为 /metrics` 的目标上可用。因此，此默认作业是通过 URL 进行抓取：http://localhost:9090/metrics。

返回的时间序列数据将详细说明 Prometheus 服务器的状态和性能。

有关配置选项的完整规范，请参阅 [配置文档](https://prometheus.io/docs/operating/configuration/) 。

> 注意：
> 	在一个抓取配置指定单个作业时，指定 `static_configs` 块，而需要保持同步情况时，请使用服务发现形式，比如 `file_sd_config` 块，基于文件的服务发现提供了一种更通用的方式来配置静态目标，并充当插入自定义服务发现机制的接口。


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
## 三、Exporter

有许多库和服务器有助于将现有指标从第三方系统导出为 Prometheus 指标。这对于无法直接使用 Prometheus 指标（例如 HAProxy 或 Linux 系统统计信息）检测给定系统的情况非常有用。

文档：[Exporters and integrations | Prometheus](https://prometheus.io/docs/instrumenting/exporters/)

Third-party exporters，其中一些导出器作为官方 [Prometheus GitHub 组织](https://github.com/prometheus)的一部分进行维护，那些被标记为官方，另一些则由外部贡献和维护。举例：

- node_exporter
	- 下载地址：[node_exporter](https://github.com/prometheus/node_exporter)


某些第三方软件以 Prometheus 格式公开指标，因此不需要单独的导出器，举例：

 - docker daemon
	 - 配置文档：[dockerd |Docker 文档 --- dockerd | 守护进程指标](https://docs.docker.com/reference/cli/dockerd/#daemon-metrics)

## 四、PromQL 过滤语句

### 4.1 基本过滤

在 Prometheus web 界面的 Query 上直接输入键值

![[Pasted image 20250803025052.png]]

### 4.2 条件过滤

![[Pasted image 20250803030046.png]]

格式：{} 内的内容为过滤内容

符号：
- =：等于
- !=：不等于
- =~：支持正则，匹配
- !~：支持正则，不匹配

### 4.3 函数过滤

参考文档：[Query functions | Prometheus](https://prometheus.io/docs/prometheus/latest/querying/functions/)


### 4.4 复杂查询

查询内存的可用率

~~~proql
((node_memory_MemTotal_bytes - node_memory_MemFree_bytes) / node_memory_MemTotal_bytes ) * 100
~~~

![[Pasted image 20250803035617.png]]


## 五、Pushgateway

应用场景：自定义监控项，短时作业（任何运行时间很短，并且在 Prometheus 下一次拉取之前就会结束的任务）

官方文档： [pushgateway/README.md 在 master ·普罗米修斯/推送网关 --- pushgateway/README.md at master · prometheus/pushgateway](https://github.com/prometheus/pushgateway/blob/master/README.md)


案例：获取主机的磁盘可用率，exporter中没提供这样的键值。

### 5.1 下载

下载地址：[Releases · prometheus/pushgateway](https://github.com/prometheus/pushgateway/releases/)

### 5.2 安装

~~~shell
root@ecs-prometheus:~# tar -xf pushgateway-1.11.1.linux-amd64.tar.gz -C /opt/
root@ecs-prometheus:~# ln -s /opt/pushgateway-1.11.1.linux-amd64/ /opt/pushgateway
~~~

### 5.3 启动

~~~shell
# 可用使用 systemctl 托管
root@ecs-prometheus:~# cd /opt/pushgateway
root@ecs-prometheus:/opt/pushgateway# ./pushgateway &>>/var/log/pushgateway.log &
~~~


### 5.3 服务端配置

~~~shell
  - job_name: pushgateway
    static_configs:
      - targets: 
        - 172.17.0.1:9091
~~~


### 5.4 客户端传参

官方文档：[推送指标 |普罗 米修斯 --- Pushing metrics | Prometheus](https://prometheus.io/docs/instrumenting/pushing/)

~~~shell
# 编写脚本
#!/bin/bash

# 定义 Pushgateway 的变量
JOB="disk_metrics"
INSTANCE="172.17.0.1"
PUSHGATEWAY_URL="http://172.17.0.1:9091"

# 从 df 命令获取总磁盘空间和可用空间 (KB)
# 使用 grep -E "^/dev/" 过滤出实际的设备行，避免获取到表头
total=$(df -k / | grep -E "^/dev/" | awk '{print $2}')
free=$(df -k / | grep -E "^/dev/" | awk '{print $4}')

# 检查 total 和 free 是否成功获取且为数字
if ! [[ "$total" =~ ^[0-9]+$ ]] || ! [[ "$free" =~ ^[0-9]+$ ]]; then
  echo "错误：无法从 'df' 获取有效的磁盘空间数字。"
  exit 1
fi

# 使用 printf 严格按照 Prometheus 格式构建指标数据
METRICS_DATA=$(printf "# HELP disk_total_kilobytes Total disk space in kilobytes.
# TYPE disk_total_kilobytes gauge
disk_total_kilobytes{instance=\"%s\", job=\"%s\"} %s

# HELP disk_free_kilobytes Free disk space in kilobytes.
# TYPE disk_free_kilobytes gauge
disk_free_kilobytes{instance=\"%s\", job=\"%s\"} %s
" "$INSTANCE" "$JOB" "$total" "$INSTANCE" "$JOB" "$free")

# 使用单个 curl 命令将所有指标一次性推送到 Pushgateway
# --data-binary @- 表示从标准输入读取数据
echo "$METRICS_DATA" | curl -sS -X POST --data-binary @- "$PUSHGATEWAY_URL/metrics/job/$JOB/instance/$INSTANCE"

# 根据 curl 命令的退出码判断是否成功
if [ $? -eq 0 ]; then
  echo "指标已成功推送到 Pushgateway，时间为 $(date)。"
else
  echo "错误：无法将指标推送到 Pushgateway，时间为 $(date)。"
fi
~~~


## 六、




# Grafana

## 一、概述

_Grafana 开源软件 （OSS）_ 使您能够查询、可视化、警报和探索指标、日志和跟踪，无论它们存储在何处。Grafana 数据源插件使您能够查询数据源，包括 Prometheus 和 CloudWatch 等时间序列数据库、Loki 和 Elasticsearch 等日志记录工具、Postgres 等 NoSQL/SQL 数据库、GitHub 等 CI/CD 工具等等。Grafana OSS 为您提供了在实时控制面板上显示该数据的工具，并具有富有洞察力的图表和可视化效果。

## 二、部署

### 2.1 安装

```bash
docker run -d -p 3000:3000 --name=grafana grafana/grafana-enterprise
```

### 2.2 配置

官方文档：[配置 Grafana Enterprise |Grafana 文档 --- Configure Grafana Enterprise | Grafana documentation](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/enterprise-configuration/)

2.3 登录

 Steps  步骤 [直达官网](https://grafana.com/docs/grafana/latest/setup-grafana/sign-in-to-grafana/#steps)

To sign in to Grafana for the first time, follow these steps:  
若要首次登录 Grafana，请执行以下步骤：

1. Open your web browser and go to root URL specified in [Grafana configuration file](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/).  
    打开 Web 浏览器并转到 [Grafana 配置文件](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)中指定的根 URL。
    
    Unless you have configured Grafana differently, it is set to use `http://localhost:3000` by default.  
    除非您以不同的方式配置了 Grafana，否则默认情况下它设置为使用 `http://localhost:3000`。
    
2. On the signin page, enter `admin` for username and password.  
    在登录页面上，输入 `admin` 作为用户名和密码。
    
3. Click **Sign in**.  点击**登录。**
    
    If successful, you will see a prompt to change the password.  
    如果成功，您将看到更改密码的提示。
    
4. Click **OK** on the prompt and change your password.  
    单击提示中的 **“确定”** 并更改您的密码。
## 三、仪表板

添加 Prometheus 数据源

官方文档：[Prometheus 数据源 |Grafana 文档 --- Prometheus data source | Grafana documentation](https://grafana.com/docs/grafana/latest/datasources/prometheus/)
在配置 Prometheus 数据源时添加示例。

![[Pasted image 20250803065214.png]]
![[Pasted image 20250803065248.png]]
![[Pasted image 20250803065302.png]]
![[Pasted image 20250803065330.png]]
![[Pasted image 20250803065348.png]]
![[Pasted image 20250803065408.png]]
![[Pasted image 20250803065505.png]]
![[Pasted image 20250803065511.png]]


## 四、Grafana Loki

### 4.1 概述

与其他日志记录系统不同，Loki 是围绕仅索引日志标签的元数据（就像 Prometheus 标签一样）的想法构建的。然后，日志数据本身被压缩并分块存储在对象存储中，例如 Amazon Simple Storage Service （S3） 或 Google Cloud Storage （GCS），甚至存储在本地文件系统上。

### 4.2 安装

官网：[使用 Docker 或 Docker Compose 安装 Loki |Grafana Loki 文档 --- Install Loki with Docker or Docker Compose | Grafana Loki documentation](https://grafana.com/docs/loki/latest/setup/install/docker/#install-with-docker-on-linux)

1. Create a directory called `loki`. Make `loki` your current working directory:  
    创建一个名为 `loki` 的目录。将 `loki` 设为您当前的工作目录：
    
    bash  抨击![Copy code to clipboard](https://grafana.com/media/images/icons/icon-copy-small-2.svg)Copy  复制
    
    ```bash
    mkdir loki
    cd loki
    ```
    
2. Copy and paste the following commands into your command line to download `loki-local-config.yaml` and `promtail-docker-config.yaml` to your `loki` directory.  
    将以下命令复制并粘贴到命令行中，将 `loki-local-config.yaml` 和 `promtail-docker-config.yaml` 下载到 `loki` 目录。
    
    bash  抨击![Copy code to clipboard](https://grafana.com/media/images/icons/icon-copy-small-2.svg)Copy  复制
    
    ```bash
    wget https://raw.githubusercontent.com/grafana/loki/v3.4.1/cmd/loki/loki-local-config.yaml -O loki-config.yaml
    wget https://raw.githubusercontent.com/grafana/loki/v3.4.1/clients/cmd/promtail/promtail-docker-config.yaml -O promtail-config.yaml
    ```
    
3. Copy and paste the following commands into your command line to start the Docker containers using the configuration files you downloaded in the previous step.  
    将以下命令复制并粘贴到命令行中，以使用您在上一步中下载的配置文件启动 Docker 容器。
    
    bash  抨击![Copy code to clipboard](https://grafana.com/media/images/icons/icon-copy-small-2.svg)Copy  复制
    
    ```bash
    docker run --name loki -d -v $(pwd):/mnt/config -p 3100:3100 grafana/loki:3.4.1 -config.file=/mnt/config/loki-config.yaml
    docker run --name promtail -d -v $(pwd):/mnt/config -v /var/log:/var/log --link loki grafana/promtail:3.4.1 -config.file=/mnt/config/promtail-config.yaml
    ```
    
    > Note  注意
    > 
    > The image is configured to run by default as user `loki` with UID `10001` and GID `10001`. You can use a different user, specially if you are using bind mounts, by specifying the UID with a `docker run` command and using `--user=UID` with a numeric UID suited to your needs.  
    > 默认情况下，该映像配置为以用户 `loki` 身份运行，UID `为 10001` 和 GID `10001`。您可以使用不同的用户，特别是如果您使用绑定挂载，方法是使用 `docker run` 命令指定 UID 并使用 `--user=UID` 和适合您需求的数字 UID。
    
4. Verify that your containers are running:  
    验证容器是否正在运行：
    
    bash  抨击![Copy code to clipboard](https://grafana.com/media/images/icons/icon-copy-small-2.svg)Copy  复制
    
    ```bash
root@ecs-prometheus:~/loki# docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED          STATUS          PORTS                                         NAMES
f142ba812e25   grafana/promtail:3.5.1                 "/usr/bin/promtail -…"   4 seconds ago    Up 2 seconds                                                  promtail
6bd4f6a49ed5   grafana/loki:3.5.3                     "/usr/bin/loki -conf…"   51 seconds ago   Up 51 seconds   0.0.0.0:3100->3100/tcp, [::]:3100->3100/tcp   loki
78b205edb6bf   quay.io/prometheus/prometheus:v3.5.0   "/bin/prometheus --c…"   3 hours ago      Up 33 minutes   0.0.0.0:9090->9090/tcp, [::]:9090->9090/tcp   prometheus
    ```
    
5. Verify that Loki is up and running.  
    验证 Loki 是否已启动并正在运行。
    
    - To view readiness, navigate to http://localhost:3100/ready.  
        若要查看就绪情况，请导航到 http://localhost:3100/ready。
    - To view metrics, navigate to http://localhost:3100/metrics.  
        要查看指标，请导航到 http://localhost:3100/metrics。