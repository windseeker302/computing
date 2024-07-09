# Nginx

## Nginx 是什么	

Nginx（读作engine x）是一个`轻量级开源Web服务器软件`，由俄罗斯人Igor Sysoev在Rambler.ru任职期间开发，最初是为了解决高性能和可扩展性的问题而设计的。Nginx的历史可以追溯到2002年8月6日，当时Igor Sysoev发布了第一个公开版本0.1.0。

在Nginx出现之前，Apache是最流行的Web服务器软件之一，但它的问题在于`处理高并发连接时`的性能瓶颈。Nginx采用了一种不同的设计思路，使用了异步事件驱动的方式处理请求，使得它能够更好地应对高并发场景。

Nginx的发展得到了广泛的关注和支持。从nginx-0.5.x版本开始，它开始受到国内互联网人士的关注，并在国内得到了广泛的应用和推广。随着Nginx的不断发展和完善，它逐渐被应用于更多的场景，如反向代理、负载均衡、缓存等。

在2011年，阿里巴巴旗下的淘宝网技术团队开源了一个基于Nginx的变种，名为`Tengine`。Tengine在Nginx的基础上增加了一些新的功能和优化，以满足淘宝等高并发场景的需求。

随着时间的推移，Nginx逐渐成为了Web服务器市场的重要力量。根据2020年3月的数据，Nginx在全球Web服务器市场的占有率超过了老牌Web服务器Apache，成为了市场份额最大的Web服务器软件之一。

Nginx之所以受到广泛的欢迎和应用，主要是因为它具有`高性能`、`稳定性好`、`结构模块化`、`配置简单`以及`资源消耗低`等优点。此外，Nginx还提供了丰富的功能和扩展，如SSL、GZIP、虚拟主机、URL重写等，可以满足各种Web应用的需求。

总之，Nginx的历史可以追溯到2002年，经过十多年的发展和完善，它已经成为了Web服务器市场的重要力量，并在高并发场景下发挥着重要的作用。

## HTTP 工作机制

一次请求发送和对应的请求响应组成一次 HTTP 事务。

HTTP 请求访问的完整过程：

1. 建立连接：接受一个客户端请求，或者如果不希望于这个客户端建立连接，就将其关闭。
2. 建立请求：从网络中读取一条 HTTP 请求报文。
3. 处理请求：对请求报文进行解释，并采取行动。
4. 访问资源：从数据存储中读取报文中指定的资源。
5. 构建响应：创建带有正确头部的 HTTP 响应报文。
6. 发送响应：将响应回送给客户端。
7. 记录事务处理过程：将与已完成事务相关的内容记录到日志中。

HTTP处理请求的方法：

- GET：向服务器请求资源

- POST：向服务器输入数据

- PUT：向服务器写入文档

- DELETE：删除URL中所指定的资源

- HEAD：向服务器请求资源后，服务器仅返回资源首部，不返回主体

- TRACE：客户端发起请求时如果穿过防火墙、代理等应用时，原始的HTTP请求会被修改，使用TRACE后，客户端可以在最终请求发送给服务器时，看到其形态

- OPTIONS：请求web服务器告知其支持的各种功能

## 网络架构模式

### B/S 架构

B/S即“Browser/Server”，是“浏览器/服务器”模式。在这种模式中，客户是浏览器，服务器是Web服务器。客户向服务器发出信息浏览请求，服务器向客户送回客户所要的万维网文档，以页面的形式显示在客户的屏幕上。B/S模式的一个重要特点是平台无关性，Browser、Web Server及主流语言Java和HTML等都可以做到与软硬件平台无关。另外，B/S模式的客户端变瘦，其功能主要是一个多媒体浏览器。

### C/S 架构

C/S即“Client/Server”，是“客户端/服务器”模式。在这种模式中，客户（Client）和服务器（Server）分别是两个应用进程，可以位于互联网的两台不同主机上。服务器被动地等待服务请求，客户向服务器主动发出服务请求，服务器做出响应，并返回服务结果给客户。C/S模式中的客户端一般泛指客户端应用程序EXE，程序需要先安装后，才能运行在用户的电脑上，对用户的电脑操作系统环境依赖较大。

> 技术架构
>
> 收费技术栈：
>
> redhad + jqurey + js + svn + oracle + tomcat+java + apache
>
> 
>
> 省钱，走向开源技术栈：
>
> centos + jquery + js + git + mysql + java+tomcat + python+perl + nginx

### 其他web服务器

除了Nginx之外，还有一些其他常用的Web服务器软件，包括但不限于：

1. Apache：Apache是最流行的Web服务器之一，具有跨平台、高可靠性、高性能、模块化设计、开源等优势。它支持几乎所有的Unix、Windows、Linux系统平台，尤其对Linux的支持相当完美。
2. IIS（Internet Information Services）：IIS是微软公司开发的Web服务器，具有良好的Windows系统集成性、易于管理、可靠性高等优势。它适用于Windows系统，可用于监视配置和控制Internet服务。
3. Tomcat：Tomcat是一个开放源代码、运行servlet和JSP Web应用软件、并基于Java的Web应用软件容器。它具有跨平台、开源、易于扩展等优势，适用于Java应用程序的Web服务器。
4. Lighttpd：Lighttpd是由德国人Jan Kneschke领导开发的，基于BSD许可的开源Web服务器软件。它的根本目的是提供一个专门针对高性能网站，安全、快速、兼容性好并且灵活的Web服务器环境。

除此之外，还有其他的Web服务器软件，如 Kangle、Tengin 等。这些服务器软件各有特点和优势，可以根据具体的需求和应用场景选择合适的 Web 服务器来提高Web应用的性能和可靠性。

### Web服务器响应码

- 100-199：信息性状态码
- 200-299：成功性状态码
- 300-399：重定向状态码
- 400-499：客户端错误状态码
- 500-599：服务器错误状态码

常见的状态码：

- 200：OK，表示连接正常。
- 403：Forbidden，表示访问错误，大多数是由于客户端对被请求的文件没有权限，或者由于被访问资源对应的文件或目录损坏。
- 404：Not Found，表示无法找到任何资源。
- 503：Server Unavailable，表示服务器当前无法处理客户端的请求，可以尝试刷新重新发送新的请求。
- 504：Gateway Timeout，表示未接受到服务器的及时响应。

## Nginx特点

### 成本低

nginx 强大在于其方向代理的功能，软件负载均衡，还存在硬件负载均衡（f5，netscaler）,但是由于价格昂贵，小型公司不会使用，利用 nginx 搭建高可用性的负载均衡的站点

由于 nginx 支持 BSD 开源许可协议，BSD 就是可以给用户更自己的使用权限，可以自由试用，修改源码，如果你修改后发布，还得遵顼 BSD 协议。

利用软件技术，可以实现硬件负载均衡一样的实惠，经济实惠。

### 几大优势

- nginx 配置文件更加易懂
- nginx 支持网站 url 地址重写（网站更换域名），还能够根据 url 的特点，进行请求转发判断（7层负载均衡，比如判断来自于移动端请求，发给移动端服务器，判断是来自于PC端流量，用户请求就发给运行着PC端代码的服务器）。
- nginx 支持高可用性配置（防止单节点故障，服务器崩溃），nginx 非常稳定，宕机异常退出的几率很小。
- nginx 能够节省忘了带宽，支持静态文件压缩后传输，支持 gzip 压缩功能。
- nginx 还能热部署，可以在不停止 nginx 情况下更新代码（reload），并且 nginx 支持 7*24h 运转。

## 网络模型

### 网络 IO 模型概念

常见的 io 模型有：

- 阻塞模型
- 非阻塞模型
- IO 多路复用
- 异步 IO

网络 IO 指的就是在网络中进行数据的读，写操作，本质上就是一个 socket 套接字读取，socket 套接字在 linux 系统中被抽象为流的概念，网络 IO 就是对数据流的处理。



## 安装

nginx 提供商业版和开源版，且都支持 windows 和 linux 平台。

商业版：https://nginx.com

开源版：https://nginx.org

### 环境准备

准备 Nginx 需要的一些第三方系统库的支持，安装编译工具

检查防火墙是否关闭，selinux关闭，yum 仓库的配置，网络环境等等。

~~~shell
# 配置 aliyun yum 源
cd /etc/yum.repos.d/
mkdir bak
mv * bak
# 安装 centos 7 阿里的 yum 源，不同的版本可以安装不同的源
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
# 安装 epel 源
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo

# 清除之前的缓存
yum clean all
# 缓存 aliyun 的 yum 源，加速下载
yum makecache

# 安装常用的软件包
yum install -y vim bash-completion wget net-tools lrzsz

# 安装如下的编译工具
yum install -y gcc gcc-c++ autoconf automake make

# 安装 nginx 所需要的第三方系统库
yum install -y zlib zlib-devel openssl openssl-devel pcre pcre-devel httpd-tools 

# 关闭防火墙
systemctl disable --now firewalld
setenforce 0
~~~



### 编译安装 nginx

~~~shell
# 1.下载 nignx 源代码
wget https://nginx.org/download/nginx-1.25.4.tar.gz
# 或者下载 tengine 
wget https://tengine.taobao.org/download/tengine-3.1.0.tar.gz

# 2.压缩 nginx
tar -zxvf nginx-1.25.4.tar.gz -C /opt/
cd /opt/nginx-1.25.4/
[root@nginx-server nginx-1.25.4]# ll
total 824
drwxr-xr-x. 6 502 games   4096 Mar  6 14:57 auto		# 检测系统模块依赖信息
-rw-r--r--. 1 502 games 326027 Feb 15 00:03 CHANGES		# 存放 nginx 的变化记录日志
-rw-r--r--. 1 502 games 498741 Feb 15 00:03 CHANGES.ru
drwxr-xr-x. 2 502 games    168 Mar  6 14:57 conf		# 存放 nginx 主配置文件的目录
-rwxr-xr-x. 1 502 games   2611 Feb 15 00:03 configure		# 可执行的脚步，用于释放编译文件的定制脚步
drwxr-xr-x. 4 502 games     72 Mar  6 14:57 contrib		# 提供了 vim 插件，让配置文件颜色区分
drwxr-xr-x. 2 502 games     40 Mar  6 14:57 html		# 存放了标准的 html 页面文件
-rw-r--r--. 1 502 games   1397 Feb 15 00:03 LICENSE
drwxr-xr-x. 2 502 games     21 Mar  6 14:57 man
-rw-r--r--. 1 502 games     49 Feb 15 00:03 README
drwxr-xr-x. 9 502 games     91 Feb 15 00:03 src		# 存放了 nginx 源代码的目录

# 3.拷贝 nginx 的配置文件语法高亮，发给 vim 的插件目录
[root@nginx-server ~]# mkdir ~/.vim
[root@nginx-server ~]# cp -r /opt/nginx-1.25.4/contrib/vim/* ~/.vim

# 4.安装编译所需的软件包和依赖项
[root@nginx-server nginx-1.25.4]# sudo yum install -y gcc gcc-c++ make zlib-devel pcre-devel openssl-devel

# 5.开始编译安装三部曲
# 1>: ./configure --prefix=/opt/nginx --with-http_ssl_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-threads --with-file-aio

# 2>: make（如同开始下一步安装）

# 3>: make install（如同点击开始安装）

# 6.配置 nginx 环境变量
[root@nginx-server ~]# vim /etc/profile.d/nginx.sh
export PATH="$PATH:/opt/nginx/sbin"
[root@nginx-server ~]# source /etc/profile.d/nginx.sh
[root@nginx-server ~]# nginx -V
nginx version: nginx/1.25.4
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/opt/nginx --with-http_ssl_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-threads --with-file-aio
~~~



## 概念

### nginx 配置文件语法详解

- nginx.conf 是由指令和指令块组成
- 每行语句都得有分号结束，指令和参数之间是有空格分割的
- 指令块可以由大括号{}组织多语句
- nginx.conf 使用 # 号表示注释符
- nginx 支持用 $ 变量名支持该语法
- nginx 支持 include 语句，组合多个配置文件
- nginx 部分指令支持正则表达式，如 rewrite 重写指令
- nginx 核心功能都在 http{}指令块中，http{}中还包含了以下指令
  - server{}，对应一个站点配置，反向代理，静态资源
  - location{}，对应一个 url
  - upstream{}，定义上流服务，负载均衡池

~~~shell
########## 全局指令
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

########## 局部指令
events {
    worker_connections  1024;
}

########## http{}语句快，核心功能配置
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

########## 此处是 server{}语句块，是在 http{}语句块里的
    server {
        listen       80;
        server_name  localhost;
        
        location / {
            root   html;
            index  index.html index.htm;
        }
        
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
~~~

### 主配置文件详解 

#### 全局配置

- Master进程的用户为root，worker进程的用户可在全局配置中通过user调整
- 默认情况worker进程数量和服务器CPU核心数一致，也可以在全局配置中通过worker_processes调整

~~~shell
user nginx;                                                      # worker进程所属用户
worker_processes auto;                                    # worker进程数量
error_log /var/log/nginx/error.log;                    # 错误日志存放路径
pid /run/nginx.pid;                                          # 进程文件对应路径
include /usr/share/nginx/modules/*.conf;          # 功能模块对应路径
~~~

#### HTTP配置

~~~shell
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      ‘“$http_user_agent” “$http_x_forwarded_for”’;                  # 错误日志格式
    access_log  /var/log/nginx/access.log  main;                           # 接入日志存放路径
    sendfile            on;               # 高效文件传输模式状态，默认为开启
    tcp_nopush          on;            # 性能优化参数，对于数据是否立刻发送
    tcp_nodelay         on;               # 性能优化参数，对于小数据包是否延时发送
    keepalive_timeout   65;                     # 持久连接时间或超时时间
    types_hash_max_size 4096;                        # 性能优化参数，影响散列表的冲突率
    include             /etc/nginx/mime.types;                         # 可解析的静态资源类型 
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;                          # 子配置文件存放路径
    server {
        listen       80;                      # 监听的IPv4端口
        listen       [::]:80;                  # 监听的IPv6端口
        server_name  _;            
        root         /usr/share/nginx/html;                   # 主页存放路径
        include /etc/nginx/default.d/*.conf;                   # 子配置文件存放路径
        error_page 404 /404.html;                              # 404错误返回给客户端的页面
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;              # 500、502、503和504错误是返回客户端的页面
            location = /50x.html {
        }
    }
~~~



### Nginx 信号传递

#### nginx 的工作原理

1. master 主进程是不处理请求的，而是分配请求发给 worker 进程，主进程负责重启，热加载，热部署等信号。
2. master 是根据 nginx.conf 中 worker_process 定义启动时创建的工作进程数。
3. 当 worker 运行后，master 就处于一个等待的状态，等待用户的请求来临，或者系统信号。
4. 系统管理员可以发送 kill 指令，或者 nginx -s 信号，这样的形式操控 nginx。

#### nginx 信号集

nginx -s 常用的信号集功能如下：

| 参数   | 信号 | 含义                                        |
| ------ | ---- | ------------------------------------------- |
| stop   | TERM | 强制关闭 nginx 服务                         |
| null   | INT  | 强制关闭整个 nginx 服务                     |
| quit   | QUIT | 优雅关闭整个 nginx 服务                     |
| reopen | USR1 | 重新打开日志记录                            |
| reload | HUB  | 重新读取配置文件，并且优雅的退出旧的 worker |

## 功能实践

### Nginx 命令行

~~~shell
1.nginx #启停指令，-s 参数，值得是给 nginx 进程发送某种信号
nginx # 启动
nginx -s stop # 停止
nginx -s reload # 平滑重启，不重启进程，重载配置文件

2.查看nginx命令的帮助信息
[root@nginx-server conf]# nginx -h
nginx version: nginx/1.25.4
Usage: nginx [-?hvVtTq] [-s signal] [-p prefix]
             [-e filename] [-c filename] [-g directives]

Options:
  -?,-h         : this help		# 显示 nginx 的帮助信息
  -v            : show version and exit	# 显示版本
  -V            : show version and configure options then exit	# 显示版本和编译参数信息
  -t            : test configuration and exit	# 检测 nginx 语法是否正确
  -T            : test configuration, dump it and exit	# 检查配置，并输出配置信息
  -q            : suppress non-error messages during configuration testing	# 在检测配置文件期间屏蔽非错误信息
  -s signal     : send signal to a master process: stop, quit, reopen, reload	# 给 nginx 主进程发生一个信号，stop，停止运行，quit，优雅停止，reopen，重启记录nginx日志，reload，重读配置文件
  -p prefix     : set prefix path (default: /opt/nginx/)	# 设置 nginx 目录前缀
  -e filename   : set error log file (default: logs/error.log)  # 设置错误文件的路径
  -c filename   : set configuration file (default: conf/nginx.conf)  # 设置主文件的路径  nginx -c /my/nginx.conf
  -g directives : set global directives out of configuration file	# 从配置文件中设置全局指令
~~~



### nginx 热部署（未成功）

热部署大致流程

1. 备份旧的程序（二进制文件，nginx 命令，/opt/nginx/sbin/nginx）
2. 编译安装新的二进制文件，覆盖旧的二进制文件（再装一个版本的ngin，且替换旧的nginx命令）
3. 发送 USR2 信号给旧的 master 进程
4. 发送 WINCH 信号给旧的 master 进程
5. 发送 QUIT 信号给旧的 master 进程、

示例：

~~~shell
# 1.备份
[root@nginx-server ~]# cp /opt/nginx/sbin/nginx /opt/nginx/sbin/nginx-1.25.4

# 2.安装不同版本的 nginx 
[root@nginx-server ~]# wget https://nginx.org/download/nginx-1.24.0.tar.gz
# 压缩
[root@nginx-server ~]# tar -zxvf nginx-1.24.0.tar.gz -C /opt/
# 编译安装三部曲
[root@nginx-server ~]# cd /opt/nginx-1.24.0/
./configure --prefix=/opt/nginx --with-http_ssl_module --with-http_flv_module --with-http_gzip_static_module --with-http_stub_status_module --with-threads --with-file-aio
[root@nginx-server nginx-1.24.0]# make && make install

# 3.姿势发送一个 USR2 信号给旧的 master process，作用是使得 nginx 旧的版本停止接受用户请求，并且切换为新的 nginx 版本。
[root@nginx-server sbin]# kill -USR2 `cat /opt/nginx/logs/nginx.pid `

# 4.发送WINCH信号
kill -WINCH ` cat /var/nginx_1.17/logs/nginx.pid.oldbin `
~~~



### Ngingx 日志切割

日志切割是线上很常见的操作，能够控制单个日志文件的大小，便于对日志进行管理。

~~~shell
# 1.针对 nginx 的访客日志进行切割
[root@nginx-server logs]# du -h *
4.0K	access.log
4.0K	error.log
4.0K	nginx.pid

# 2.以当前时间为文件名进行备份
[root@nginx-server logs]# mv access.log "access.log_$(date +"%Y-%m-%d")"

# 3.发信号给 nginx 主进程，给他发送一个重新打开的信号，让 nginx 重新生成新的日志
nginx -s reopen 	# 等同于  kill -USR1 process id
~~~

> 注意在以上的 nginx 重命名日志切割链，不要着急立即对文件修改，而是需要等待几秒钟，可能当业务量访问过大时，这个修改操作可能会有点急，不会立即生效。

在生产环境下，日志切割主要时以定时任务的形式来操作

编写一个定时日志切割的脚步

vim cut_nginx_log.sh

~~~shell
#!/bin/bash

# nginx 日志存放点
logs_path="/opt/nginx/logs/"
mkdir -p ${logs_path}$(date -d "yesterday" +"%Y")/$(date -d "yesterday" + "%m")
mv ${logs_path}access.log ${logs_path}$(date -d "yesterday" + "%Y")/$(date -d "yesterday" + "%m")/access_$(date -d "yesterday" + "%Y-%m-%d").log
kill -USR1 `cat /opt/nginx/logs/nginx.pid`
~~~



指定定日任务如下：

每天的 0 点执行日志切割脚本。

~~~shell
crontab -e 
0 0 * * * /bin/bash /root/cut_nginx_log.sh
~~~



### Nginx 虚拟主机

Nginx虚拟主机是一种特殊的互联网服务，它利用特殊的软硬件技术将一台运行在因特网上的服务器主机分成多个“虚拟”的主机。每台虚拟主机都可以作为一个独立的网站，拥有独立的域名，并具备完整的Internet服务器功能，如WWW、FTP、Email等。对于网站访问者来说，每一台虚拟主机就像是一台独立的主机。



> 小技巧：在编辑 nginx.conf 中的配置文件时，将光标移动到{/}（任意一个花括号的位置，比如，在 { 位置，按下 `%` 就可以跳到与之对应的另一半挂括号的位置了。
>
> 注意：nginx.conf 中 server{} 虚拟主机标签的定义，默认加载顺序时自伤而下的匹配规则。

#### 静态站点

~~~shell
vim /opt/nginx/conf/nginx.conf
# 站点配置块 server{}
   server {
   	    # 定义监听端口
        listen       80;
        # 定义虚拟主机的域名配置，没有域名就可以写 localhost 或者 _ 也行
        server_name  localhost;
        # 给 nginx 定义网站的编码
        charset utf-8;
        
        # 日志，现在不用，可以不用管
        #access_log  logs/host.access.log  main;
        
        # nginx 的路径匹配规则
        # 如下的规则时最低级匹配，任何的 nginx 请求，都会进入如下 location 的匹配，去它所定义的目录中寻找资料
        location / {
            # root 关键词，时定义网页根目录的，这个 html 时以 nginx 安装的路径为相对
            root   /www/root/;
            # index 关键词，定义 nginx 的有也文件名字，默认找哪个文件
            index  index.html index.htm;
        }
        
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }

# 创建网页站点根目录
mkdir -p /www/root
echo "hello world!" > /www/root/index.html
~~~

#### 基于 IP 的多虚拟主机

1.添加 ip

~~~shell
[root@nginx-server conf.d]# ip ad add 192.168.113.101/24 dev ens33
[root@nginx-server conf.d]# ip ad add 192.168.113.102/24 dev ens33
[root@nginx-server conf.d]# ip ad add 192.168.113.103/24 dev ens33
~~~



2.添加配置到 nginx.conf

~~~shell
server {
        listen       192.168.113.101:80;
        server_name  localhost;
        location / {
            root   /www/server1/;
            index  index.html index.htm;
        }
}
server {
        listen       192.168.113.102:80;
        server_name  localhost;
        location / {
            root   /www/server2/;
            index  index.html index.htm;
        }
}
server {
        listen       192.168.113.103:80;
        server_name  localhost;
        location / {
            root   /www/server3/;
            index  index.html index.htm;
        }
}
~~~



3.设置站点文件

~~~shell
mkdir /www/server{1..3}
for i in `ls /www/ | grep server` ;do echo this is the $i > /www/$i/index.html ;done
~~~



4.重载配置文件

~~~shell
nginx -s reload
~~~



5.测试网站

~~~shell
[root@nginx-server nginx]# curl 192.168.113.101
this is the server1
[root@nginx-server nginx]# curl 192.168.113.102
this is the server2
[root@nginx-server nginx]# curl 192.168.113.103
this is the server3
~~~



#### 基于多域名的虚拟主机

基于多域名的虚拟主机，用的还是不多的，还可能造成 ip 不足等问题，一般如果没有特殊需求，用的更多，且更方便的是基于多域名的虚拟主机。

1. /etc/hosts 设置域名，进行本地测试使用

   ~~~shell
   [root@nginx-server ~]# vim /etc/hosts
   # 追加
   192.168.113.31	server1.com
   192.168.113.31	server2.com
   192.168.113.31	server3.com
   ~~~

   

2. 修改配置

   ~~~shell
   # 还是拿之前那个基于 ip 的配置改
   server {
           listen       80;
           server_name  server1.com;
           location / {
               root   /www/server1/;
               index  index.html index.htm;
           }
   }
   server {
           listen       80;
           server_name  server2.com;
           location / {
               root   /www/server2/;
               index  index.html index.htm;
           }
   }
   server {
           listen       80;
           server_name  server3.com;
           location / {
               root   /www/server3/;
               index  index.html index.htm;
           }
   }
   ~~~

   

3. 设置站点文件

   ~~~shell
   # 还是拿之前那个基于 ip 的站点文件 index.html
   ~~~

   

4. 重载服务

   ~~~shell
   nginx -s reload
   ~~~

   

5. 测试网站

   ~~~shell
   [root@nginx-server conf.d]# curl server1.com
   this is the server1
   [root@nginx-server conf.d]# curl server2.com
   this is the server2
   [root@nginx-server conf.d]# curl server3.com
   this is the server3
   ~~~

   



#### 基于多端口的虚拟主机

只需要修改 nginx.conf 中 server{}块里定义的 listen 端口参数即可，实验不同的端口，进行虚拟主机匹配

1. 修改配置

   ~~~shell
   # 还是拿之前那个基于 ip 的配置改
   server {
           listen       81;
           location / {
               root   /www/server1/;
               index  index.html index.htm;
           }
   }
   server {
           listen       82;
           location / {
               root   /www/server2/;
               index  index.html index.htm;
           }
   }
   server {
           listen       83;
           location / {
               root   /www/server3/;
               index  index.html index.htm;
           }
   }
   ~~~

   

2. 设置站点文件

   ~~~shell
   # 还是拿之前那个基于 ip 的站点文件 index.html
   ~~~

   

3. 重载服务

   ~~~shell
   nginx -s reload
   ~~~

   

4. 测试网站

   ~~~shell
   [root@nginx-server conf.d]# curl localhost:81
   this is the server1
   [root@nginx-server conf.d]# curl localhost:82
   this is the server2
   [root@nginx-server conf.d]# curl localhost:83
   this is the server3
   ~~~

   

### Nginx 静态资源压缩

nginx 支持 gzip 压缩功能，经过 gzip 压缩之后的页面，图片，动态图这类的静态文件，能够压缩为原本的 30% 甚至更小，用户访问网络的体验会更好。

启动 gzip 压缩功能，只需要在 http{} 配置中，打开如下参数即可：

```nginx
http {  
    ...  
    gzip on;  
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;  
    gzip_proxied any;  
    gzip_min_lenth 1;                                          # 压缩生效的最小值，如配置中设为1，表示小于1字节的文件不会进行压缩
    gzip_comp_level 6;		# 压缩比，最大为9，最小为1  
    gzip_vary on;  
    gzip_min_length 1000;  
    gzip_buffers 16 8k;  
    ...  
}
```

这里的配置项解释如下：

- `gzip on;`：启用 Gzip 压缩。
- `gzip_types`：定义哪些 MIME 类型的响应体将被 Gzip 压缩。你可以根据需要添加或删除类型。
- `gzip_proxied any;`：允许或者禁止压缩基于请求和响应的代理请求。
- `gzip_comp_level 6;`：设置压缩级别，范围从 1（最低压缩，最快处理速度）到 9（最高压缩，最慢处理速度）。
- `gzip_vary on;`：在响应头中添加 "Vary: Accept-Encoding" 以告知客户端内容经过了压缩。
- `gzip_min_length 1000;`：设置启用压缩的响应体最小长度（单位：字节）。小于这个长度的响应体将不会被压缩。
- `gzip_buffers 16 8k;`：设置压缩缓冲区大小和数量。





### 日志功能

日志对于程序员是很重要的，可以用于问题排错，记录程序运行时的状态。

#### 访客（access）日志

Nginx 开启日志功能只需要再 nginx.conf 中找到 log_format 参数，定义日志的格式，以及定义日志的存储位置。

~~~shell
http {

....
	# 定义日志的内容格式（记录内容的详细程度）
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
                      
    access_log  logs/access.log  main;
    
....

}
~~~



nginx 的访客日志内容如下

~~~shell
192.168.113.31 - - [08/Mar/2024:10:41:11 +0800] "GET / HTTP/1.1" 200 20 "-" "curl/7.29.0"

# 定义格式如下
'$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'"$http_user_agent" "$http_x_forwarded_for"'

$remote_addr	# 记录访问者的 ip
$remote_user	# 记录访问者的用户名
$time_local		# 记录访问的时间和地区信息
$request		# 记录用户的 http 请求的首行信息
$status		# 记录用户的 http 请求状态，也就是请求发出之后，响应的状态，200,301,404，502
$body_bytes_sent		# 记录服务器发给客户端的响应数据字节大小
$http_referer		# 记录本次请求时从哪个链接过来的，可以根据 refer 信息来进行防盗链
$http_user_agent		# 记录客户端的访问信息，如浏览器信息，手机信息
$http_x_forwarded_for 		# 防止恶意爬虫，跳板机等，并捉到代理服务器后面的客户端ip
~~~

#### 错误（error）日志

对于错误信息的调试，也是运维人员维护 nginx 的一个重要的手段。

nginx 想要使用 error_log 就得打开 nginx.conf，找到关键字参数 `error_log`，它是放在 http{}称之为全局的变量参数，针对所有定义的 server{}的虚拟主机都会生效。

~~~shell
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
~~~



也可以单独的在写在虚拟主机中，单独记录某一个虚拟主机的错误日志。

~~~shell
    server {
        listen       80;
        server_name  localhost;
        
        charset utf-8;
        # 单独定义一个虚拟主机的错误日志
        error_log logs/s1_error.log
        
        #access_log  logs/host.access.log  main;
        
        location / {
            root   /www/root/;
            index  index.html index.htm;
        }   
        
	....
	
	}

~~~



错误日志的语法

~~~
error_log file level;

日志级别有如下几种
debug
info
notice
warn
error
crit
alert
emerg

这个级别是越来越严重，级别写的越低，记录的日志越详细，圣餐环境下常用的是 warn,error,crit 模式，日志的级别，可能会给服务器增加额外的大量 IO 消耗，因此，根据你实际的工作环境需求。
~~~



#### Nginx 多虚拟主机日志

针对每一个虚拟主机都配置好 access.log 更方便更清晰的对每一个虚拟主机进行访客信息管理。

~~~shell
# 针对虚拟主机，添加日志的路径
server {
        listen       81;
        # 在每一个虚拟主机都添加 access_log 参数，就可以对每一个虚拟主机有不同的日志文件
        access_log logs/server1_81.log;
        location / {
            root   /www/server1/;
            index  index.html index.htm;
        }
}

~~~



### 目录浏览

能够将你的机器上目录资料，提供一个展示，无论是谁都可以快速访问，类似于 FTP 服务。

配置文件如下，在server{}块中添加：

~~~shell
server {
        listen       81;
        access_log logs/server1_81.log;
        location / {
            root   /www/server1/;		# root 目录里，不能存在 index.html 文件，因为浏览器默认访问的是 baidu.com/index.html
            #index  index.html index.htm;		# 关闭虚拟主机的默认首页功能
            autoindex on;		# 启动目录浏览功能
        }
}
~~~



### 状态页功能

nginx 提供了 status 模块，用于检测 nginx 的请求链接信息。

这个功能需要再编译安装 nginx 的时候，添加 `--with-http-status_module` 参数，才能使用。

~~~shell
nginx -V 	# 检查当前 nginx 是否支持 status 功能
~~~



修改 nginx.conf，支持此功能，内容如下：

~~~shell
# 1.确保你的 nginx.conf 主配置文件，支持 include 语法

include ../conf.d/*.conf;	# 当当前的上一级目录 conf.d 目录下的所有 conf 文件，都识别成 nginx 的配置文件。

# 2.创建 status.conf
vim /opt/nginx/conf.d/status.conf
# 写入如下内容
server{
	listen 88;
	location / {
		stub_status on;		# 开启状态页功能
		access_log off;		# 关闭访客日志功能
	}
}
~~~



状态页内容分析

~~~shell
# 表示激活中的连接数
Active connections: 1 		
# server: nginx 启动后一共处理的请求数
# accepts handled: nginx 启动后处理的握手数
# requests: 表示 nginx 一共处理了多少次的请求
server accepts handled requests
 96 96 912 
# Reading: nginx 读取到客户端的 headers 数量
# Writing：nginx 响应给客户端的 headers 数量
# Waiting：nginx 处理完毕请求之后，等待下一次的请求驻留的连接数  waiting=actve-()
Reading: 0 Writing: 1 Waiting: 0 
~~~



使用 ab 命令，对 nginx 进行压力测试

~~~shell
# 1.安装 ab 命令
yum install -y httpd-tools

# 2.使用 ab 命令对 nginx 发送大量的连接
-n		# 请求数量，一共发出多少个请求
-c		# 请求并发数 
-k		# 表示启动 keepalived 保持连接功能
[root@nginx-server ~]# ab -kc 1000 -n 100000 http://127.0.0.1/
~~~



### location 路径匹配

nginx 的 location 作用，是根据用户访问的 URL 路径，进行不同的处理方式，

针对用户请求的网站 URL 进行匹配处理

~~~nginx
# location 在 nginx.conf 的写法
location / {
    # root 关键词，是定义网页根目录的，这个 html 是以 nginx 安装的路径作为匹配路径
    root /www/root/;
    # index 关键词，定义 nginx 的首页文件名字
    index index.html index.htm;
}
~~~



location 相关语法

~~~nginx
location [ = | ~ | ~* | / | ^~] url {
	# 做出的响应的处理动作
}
~~~



匹配符号：

| 匹配符 | 匹配规则                                                     | 优先级 |
| ------ | ------------------------------------------------------------ | ------ |
| =      | 精确匹配                                                     | 1      |
| ^~     | 以某个字符开头，不做正则匹配                                 | 2      |
| ~*     | 支持正则的匹配模式                                           | 3      |
| /blog/ | 当你访问 192.168.100.100/blog/xxx，nignx 就会把请求返回给这个 location | 4      |
| /      | 通用匹配，不符合其他的 location 的匹配规则，返回这个 location | 5      |



示例：

~~~nginx
server {
    listen 888;
    server_name _;
    
    # 最低级匹配，不符合其他 location 就来这
    location / {
        return 500;
    }
    
    # 优先级最高，精确匹配，当用户访问 192.168.100.100:888/，请求就会到这来
    location = / {
        return 401;
    }
    
    # 匹配任何以 /img/ 开头的请求，不支持正则
    location ^~ /img/ {
        return 402;
    }
    
    # 匹配任何以 .gif 结尾的请求，支持正则
    location ~*  \.(gif|jpg|jpeg)$ {
        return 403;
    }
    
    # 以 /blog/ 开头的 URL，来这里，如符合其他 location,则以其他优先
    location /blog/ {
        return 404;
    }
}
~~~



### URL 地址重写

Nginx 的 URL 地址重写功能，主要是使用 nginx 提供的 `rewrite` 功能，且支持正则表达式。

rewrite 能够使用 URL 的跳转，实现 URL 规范化，根据请求的变量实现 URL 跳转等，基于 URl 重写功能常见的效果如下：

- 对于爬虫程序的封禁，让其跳转都一个错误的页面。
- 动态的 URL，伪装成静态的 html 页面，便于搜索引擎的抓取。
- 新旧域名的更新。

rewrite 语法

~~~nginx
rewrite ^/(.*) http://192.168.113.31/$1 parmanent;

# 解释
rewrite # 是 nginx 地址重写的关键词指令，开启跳转功能。
正则 ^/(.*)  # 表示匹配所有的请求，匹配成功后，跳转到后面指定的 rul 地址。
$1 	# 是取出前面用户访问的 url 路径
parmanment	# 表示 301 重定向的标记
~~~



rewrite 结尾的参数标记如下：

- last：规则匹配完成后，继续向下匹配新的 location。
- break：本条规则匹配完成后，立即停止匹配动作。
- redirct：返回 302 临时重定向状态码，浏览器地址栏显示跳转后的 rul，爬虫不会更新该 url。
- permanent：返回 301 永久重定向，浏览器地址栏也显示跳转后的 rul ，爬虫会更新该网站 url。

> last 和 break 用于实现 url 重写，浏览器的地址栏不会发生变化。
>
> redirect 和 permanent 是用于 url 跳转，浏览器 url 地址栏会发生变化，跳转到新的 url 地址栏。

示例，实现 301 url 跳转

准备一个虚拟主机的配置文件，实现，当用于访问该虚拟主机，直接跳转到百度的首页面。

~~~nginx
server {
    listen 90;
    server_name _;
    location / {
        rewrite ^/(.*) http://www.baidu.com/$1 permanent;
    }
}
~~~





### 反向代理

其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器 IP 地址。

Nginx反向代理的工作流程：

1. 客户端向Nginx发送请求

2. Nginx在收到客户端发送的请求后，将请求转发到后端服务器

3. 后端服务器将客户端请求的资源回复给Nginx

4. Nginx将资源返回给客户端

反向代理在 nginx 中可以在 location{}块中来定义 `proxy_pass` 来实现反向代理。

语法演示

~~~nginx
# 当客户端访问到 location，而 location 的动作就是将请求转发到 http://127.0.0.1:888 上。
location / {
	proxy_pass http://127.0.0.1:888;		# 这种方式为四层代理
}
~~~



示例实例1：

实现效果：使用 Nginx 反向代理，访问 `http://www.123.com` 直接跳转到 127.0.0.1:82。

> 注意：此处如果要想从 `http://www.123.com` 跳转到本机指定的ip，需要修改本机的hosts文件。此处略过

配置代码

~~~nginx
server {
	listen 80;
    server_name www.123.com;
    location / {
		proxy_pass http://localhost:82;
    }
}
~~~

测试

~~~shell
[root@nginx-server nginx]# curl www.123.com
this is the server2
~~~



示例实例2：

实现效果：使用 Nginx 反向代理，根据访问的路径跳转到不同端口的服务中，Nginx 监听端口为 8080。

配置代码

~~~nginx
server {
    listen 8080;
    server_name www.123.com;
    location / {
        root /www/server1/;
        autoindex on;
    }
    
    location  /php/ {
        proxy_pass https://www.csdn.net/;
    }
    location  /ws/ {
        proxy_pass  http://www.baidu.com/;
    }
}
~~~

- 在配置反向代理时，location中路径和proxy_pass中url包不包含“/”会对后面转发有很大影响，假设要访问的地址为http://192.168.1.123/test/index.html，不同写法会造成不同的转发路径，具体如下：

  - location /test/ { proxy_pass http://192.168.1.123/;}，最终转发路径为：http://192.168.1.123/index.html

  - location /test/ { proxy_pass http://192.168.1.123;}，最终转发路径为：http://192.168.1.123/test/index.html

  - location /test { proxy_pass http://192.168.1.123/;}，最终转发路径为：http://192.168.1.123//index.html

  - location /test { proxy_pass http://192.168.1.123;}，最终转发路径为：http://192.168.1.123/test/index.html



### 负载均衡

Nginx要实现七层负载均衡需要用到proxy_pass代理模块配置。Nginx默认安装支持这个模块，我们不需要再做任何处理。Nginx的负载均衡是在Nginx的反向代理基础上把用户的请求根据指定的算法分发到一组【[upstream](https://so.csdn.net/so/search?q=upstream&spm=1001.2101.3001.7020)虚拟服务池】。

负载均衡的指令

- upstream 指令

  upstream：该指令是用来定义一组服务器，它们可以使监听不同端口的服务器，并且也可以是同时监听TCP和Unix socket的服务器。服务器可以指定不同的权重，默认为1.

  ~~~shell
  # 语法 	
  upstream name{...}
  ~~~

  

- server 指令

  server：该指令用来指定后端服务器的名称和一些参数，可以使用域名、IP、端口或者unix socket。

  ~~~shell
  # 语法
  server name [parameters];
  ~~~

  

常用负载均衡的三种模式：

1. RR简单轮询（默认）

   RR（round-robin）轮询，每个请求按时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，能自动剔除。

   示例

   ~~~nginx
   upstream test {
     server localhost:8080;
     server localhost:8081;
   }
   ~~~

   

2. balance权重

   又叫平滑的加权轮询（smooth weighted round-robin balancing），指定轮询几率，weight和访问比率成正比，用于后端服务器性能不均的情况。

   示例

   ~~~nginx
   upstream test {
     server localhost:8080 weight=2;
     server localhost:8081 weight=1;
   }
   ~~~

   

3. ip_hash（可确保session一致）

   接下来是，基于IP哈希分配，上面的2种方式都有一个问题，那就是下一个请求来的时候请求可能分发到另外一个服务器，当我们的程序不是无状态的时候（采用了session保存数据），这时候就有一个很大的很问题了，比如把登录信息保存到了session中，那么跳转到另外一台服务器的时候就需要重新登录了，除非做session共享或者同步，所以很多时候我们需要一个客户只访问一个服务器，那么就需要用ip_hash了，ip_hash的每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。

   示例

   ~~~nginx
   upstream test {
     ip_hash;
     server localhost:8080;
     server localhost:8081;
   }
   ~~~



示例

实验目的：三个基于端口的虚拟主机，使用端口号 8080 负载均衡这三台主机。

nginx 配置文件如下所示

~~~nginx
upstream backend{
	server localhost:81;
	server localhost:83;
	server localhost:82;
}
server {
        listen       81;
	    server_name	 _;
        location / {
            root   /www/server1/;
            index  index.html index.htm;
        }
}
server {
        listen       82;
        location / {
            root   /www/server2/;
            index  index.html index.htm;
        }
}
server {
        listen       83;
        location / {
            root   /www/server3/;
            index  index.html index.htm;
        }
}
server {
	listen 8080;
	server_name localhost;
	location / {
            proxy_pass http://backend/;
	}
}
~~~



### 认证模块

为了防止有点小网站不想让别人看到，nginx 就提供了可以在小网站上开启认证模块，别想看到哦我们的小网站，就需要账号密码，不然访问不了，让我们操作一下吧。

nginx 提供了认证模块，语法是

~~~nginx
location {
    auth_basic 'string';
    auth_basic_user_file conf/htpasswd;
}
~~~

语法知道了，现在需要生成一个密码文件，linux 提供了密码生成命令 htpasswd 是 apache 提供的密码生成工具，nginx 也支持 auth_basic 模块，因此我们可以利用 htpasswd 命令生成账户密码文件，提供给 nginx 去使用。

~~~shell
# 安装命令
yum install -y httpd-tools

# 语法
htpasswd -bc <filename> [username] [password]

# 解释
-b		# 在命令行中输入账号密码
-c		# 创建密码文件
~~~

> 默认 .access 文件采用加密方式 md5 加密。

示例：

~~~shell
# 生成密码文件
htpasswd -bc /opt/nginx/conf.d/.access ws 000000

# 配置文件编写如下
vim /opt/nginx/conf.d/learn_auth.conf
server{
	listen 85;
	server_name _;
	location / {
		root /opt/nginx/html/;
		index index.html index.htm;
	    auth_basic 'learn nginx auth_module';
    	auth_basic_user_file /opt/nginx/conf.d/.access;
	}
}
~~~



动态请求分离



缓存服务



























