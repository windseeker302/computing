# EFK

## 日志语法

- 任何日志的日志文件都具有语法，日志语法在概念上和语言语法类似，一条日志通常由若干个字段组成，这些字段包括如下几种：
  - 时间戳
  - 日志条目的类型（INFO,ERROR,Debug）
  - 产生该日志的系统或应用
  - 日志的严重性，优先级或重要性
  - 与该日志相关的操作者或用户
  - 日志正文(用户操作行为，程序调用结果等)

## 日志收集框架

### 日志采集

- 对日志进行采集的方法有两种思路：推和拉。推是客户端（日志源设备或应用程序）主动将日志推送到日志分析系统；拉是指日志分析系统主动去客户端拉取日志。
- 常见的日志采集方式:
  - Agent采集
  - Syslog
  - 抓包
  - 接口采集
  - 业务埋点采集（探针）
  - Docker日志采集

#### 日志采集方法-Agent采集

- 在客户端部署一个Agent，由Agent来进行客户端的主动推送日志。

#### 日志采集方式-Syslog

- 它是Linux系统自带的服务。在大多数情况下，Syslog只用于系统日志。
- 使用Syslog发送日志时，需要注意：
  - 默认发送方式是UDP，这种方式有丢失日志的风险。
  - 需要根据日志凉调整发送的缓冲区，如果缓冲区满了，也会丢失日志。
  - 如果每条日志超过4KB，则必须使用TCP方式发送。

#### 日志采集方式-抓包，接口采集

- 通过抓包来收集日志的做法并不常见，因为抓包之后需要解析，此过程需要消耗 CPU 的计算资源，况且解析的是日志内容，日志量本身就比较大。这种方式相对于常规的日志采集（如 Agent 日志采集）方式多了不必要的繁琐过程，抓包的优势体现在网络流量的捕捉上，常见的抓包做法是在交换机端口配置镜像流量，并引流到一个专门解析流量的硬件设备上。
- 在需要获取程序内部信息时，往往采用接口采集方式，或日志并没有进行落地存储，只提供一个接口进行采集。接口采集需要针对采集的内容进行定制化开发，各程序内部运行机制不同，采集方法也有所差异。

#### 日志采集方式-业务埋点采集

- 埋点是在应用特定的流程中注入代码，以便收集该流程的相关信息。例如，在某张图片中埋点，可以收集点击该图片的所有用户信息，这样就能对当天点击该图片的用户进行分析，提取用户特征，以便开展接下来的营销规划。

#### 日志采集方式-Docker日志收集

- Docker 实现原理为’多进程+进程隔离‘，Docker Daemon 父进程会启动一个容器子进程，父进程会收集此子进程产生的所有日志，但子进程下的子进程产生的日志收集不到的。如果容器内只有一个进程，那么可以通过 Docker log driver 来收集子进程日志。
- 当前，使用 Kubernetes 管理容器成为趋势，由于 pod 的存在，通过 Docker log driver 将无法收集业务进程所产生的日志信息。对于解决该问题，有两种主流方案：
  - 通过调用Docker AIP来实现日志采集。
  - 将业务进程的日志文件挂载出来，然后通过采集文件的方式进行日志采集。

### 日志存储方式

- 数据库
  - 关系型数据库
  - 非关系型数据库

- 存储
  - 文件检索系统存储

### 日志分析与可视化

- 对于日志分析而言，首先要对企业内部所有的日志数据做集中管理，解决日志分散问题，然后根据需求定制解析规则，优化日析结果。下面列举一些常用的分析方法：
  - 聚类：日志种类多样，即使同一台设备产生的日志类型也有差别。日志分析涉及各种不同类型的事件，通过聚类可以对这些事件进行自动分类
  - 异常检测：通过聚类或其他方式快速检测出某个错误码，当采用定义关字方式检测业务系统时，如果该关字出现，就意味着系统存在严重错误，在这种情况下，有必要实时监测业务日志。

- 日志可视化，即将日志以更高效、直观、清晰、便捷的可视化方式呈现。日志种类众多，在展示上更加多样化，基于实时的日志数据可更加准确的进行数据分析，可视化还可与钻取、跳转到搜索页结合使用，降低数据交互难度。

## 常用的日志收集方案

一般有ELK，EFK，Filebeat，Loki。

- EFK
  - 采集：Fluentd
  - 存储：Elasticsearch
    - 工作原理-索引数据
      - 采用 Rest 风格 API，其 API 就是一次 http 请求，可以用任何工具发起 http 请求。RestAPI = 动作（PUT,POST）+对象(URL)。
  - 可视化：Kibana
  
- ELK
  - 采集：Logstash
  - 存储：Elasticsearch
  - 可视化：Kibana
  
  

- 企业级“ELFK”

  - 采集：Filebast
  - 数据处理：LogStash
  - 存储：Elasticsearch
  - 可视化：Kibana

  ![image-20240821164625895](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240821164625895.png)

## 实验

部署 EFK （Elasticsearch,filebeat,kinana）。

### 介绍

| 节点  | ip             | 系统版本        | 应用版本                                           |
| ----- | -------------- | --------------- | -------------------------------------------------- |
| efk-1 | 172.129.78.124 | Centos 7.9.2009 | elasticsearch-8.15.0,filebeat-8.15.0,kibana-8.15.0 |
| efk-2 | 172.129.78.37  | Centos 7.9.2009 | elasticsearch-8.15.0,filebeat-8.15.0               |
| efk-3 | 172.129.78.202 | Centos 7.9.2009 | elasticsearch-8.15.0,filebeat-8.15.0               |



### elasticsearch

#### 概述

- 文档型数据库 mongoDB,elasticsearch 等，

#### 安装

下载地址：https://www.elastic.co/cn/downloads/past-releases/elasticsearch-8-15-0

二进制安装

~~~shell
tar -zxvf https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.0-linux-x86_64.tar.gz
tar xf elasticsearch-8.15.0-linux-x86_64.tar.gz -C /opt/
ln -s /opt/elasticsearch-8.15.0 /opt/elasticsearch
useradd es
mkdir -p /data/elasticsearch/{logs,data}
chown -R es.es /data/elasticsearch /opt/elasticsearch-8.15.0
su es -c "/opt/elasticsearch/bin/elasticsearch -d
~~~



RPM，deb包安装

~~~shell
# RPM
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.0-x86_64.rpm
yum localinstall -y elasticsearch-8.15.0-x86_64.rpm

# deb
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.15.0-amd64.deb
dpkg -i elasticsearch-8.15.0-amd64.deb
~~~



#### 分布式集群部署

从 8.0 开始，ES 简化了安全功能。自管理集群默认启用 Elastic Stack 安全性，配置工作几乎为零（其实8.x 的安全配置更麻烦了，知识默认启用了安全功能而已）。

~~~shell
# 配置文件
egrep -v "^$|^#" /etc/elasticsearch/elasticsearch.yml 
cluster.name: efk
node.name: efk-1		# 各个节点的名称为 efk-1,efk-2,efk-2
path.data: /var/lib/elasticsearch
path.logs: /var/log/elasticsearch
network.host: 0.0.0.0
discovery.seed_hosts: ["efk-1", "efk-2","efk-3"]
xpack.security.enabled: false
xpack.security.enrollment.enabled: false
xpack.security.http.ssl:
  enabled: false
  keystore.path: certs/http.p12
xpack.security.transport.ssl:
  enabled: false
  verification_mode: certificate
  keystore.path: certs/transport.p12
  truststore.path: certs/transport.p12
http.host: 0.0.0.0

# 启动服务
systemctl enable --now elasticsearch
~~~



### kibana



#### 安装

下载地址：[https://www.elastic.co/cn/downloads/past-releases/kibana-8-15-0](https://www.elastic.co/cn/downloads/past-releases#kibana)

安装的版本需要和 elasticsearch 版本一致。

下载 RPM 包，并安装。

~~~shell
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.15.0-x86_64.rpm
yum localinstall kibana-8.15.0-x86_64.rpm
~~~



#### 部署

~~~shell
[root@ela-1 ~]# egrep -v "^$|^#" /etc/kibana/kibana.yml
server.host: "0.0.0.0"
server.name: "kibana-node"
elasticsearch.hosts: ["http://efk-1:9200","http://efk-2:9200","http://efk-3:9200"]
logging:
  appenders:
    file:
      type: file
      fileName: /var/log/kibana/kibana.log
      layout:
        type: json
  root:
    appenders:
      - default
      - file
pid.file: /run/kibana/kibana.pid
i18n.locale: "zh-CN"		# Internationalization 软件的多语言化
~~~



> 集群状态颜色：
>
>  red:
>
>    集群部分主分片无法访问。
>
>  yellow：
>
>    集群的部分副本分片无法访问。
>
>  green：
>
>    集群主分片和副本分片可以访问。



### filebeat



#### 安装

下载地址：https://www.elastic.co/cn/downloads/past-releases/filebeat-8-15-0

下载 RPM 包，并安装。

~~~shell
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-8.15.0-x86_64.rpm
yum localinstall filebeat-8.15.0-x86_64.rpm
~~~







#### input

类型

~~~shell
filebeat.inputs:
- type: stdin
output.console:
  pretty: true
~~~



#### output

##### index

~~~yaml
output.elasticsearch:
  index: "filebeat-efk-%{+yyyy.MM.dd}" 
  hosts: ["http://localhost:9200"]

# 设置索引生命周期，如果 output 使用 index 字段，必须要关闭，否则 index 字段无效。
setup.ilm.enabled: false
# 模板名称
setup.template.name: "filebeat"
# 模板模式
setup.template.pattern: "filebeat*"
~~~



##### indices

~~~shell
output.elasticsearch:
  hosts: ["http://localhost:9200"]
  indices:
    - index: "warning-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        message: "WARN"
    - index: "error-%{[agent.version]}-%{+yyyy.MM.dd}"
      when.contains:
        message: "ERR"
~~~



##### 分片和副本

~~~shell
output.elasticsearch:
  index: "filebeat-efk-%{+yyyy.MM.dd}" 
  hosts: ["http://localhost:9200"]

setup.ilm.enabled: false
setup.template.name: "filebeat"
setup.template.pattern: "filebeat*"
# 覆盖已有的索引模板
setup.template.overwrite: false
# 配置索引模板
setup.template.settings:
  # 设置模板分片数量
  index.number_of_shards: 3
  # 设置模板副本分片数量
  index.number_of_replicas: 2
~~~



> 注意：
>
>   分片数量配置后不可以修改。
>
>   副本数量配置后可以修改。

