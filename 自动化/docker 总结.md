# docker



## docker 基本构成

### 镜像（image）

- 镜像是一个静态的文件，它包含了运行容器所需的所有文件和配置，类似于虚拟机的镜像。镜像中包括了文件系统、环境变量、运行命令等。
- Docker 镜像是使用 Dockerfile 或者从已有的镜像中构建而来的，它们可以用来创建和运行容器。Docker 镜像是不可修改的，当需要修改时，可以通过构建新的镜像来实现。

### 容器（container）

- 容器是基于镜像创建的运行实例，它可以独立运行并拥有自己的文件系统、网络配置、进程空间等。容器是镜像的运行时实体。
- Docker 容器是轻量级的，可以快速启动和停止，容器之间相互隔离，具有一定的安全性和独立性。

### 仓库（repository）

- 仓库是用来存储和管理 Docker 镜像的地方。它可以分为本地仓库（Local Repository）和远程仓库（Remote Repository）。

## docker 为什么比 vm 虚拟机快？

1. docker 有着比虚拟机更少的抽象层

   由于 docker 不需要 hypervisor（虚拟机）实现硬件资源虚拟化，运行在 docker 容器上的程序直接使用的都是宿主机的硬件资源。

2. docker 利用的是宿主机的内核，而不需要加载操作系统 OS 内核

   当新建一个容器时，docker 不需要和虚拟机一样重新加载一个操作系统内核，进而避免引导，加载操作系统内核返回等比较费时费资源的过程，当新建一个虚拟机时，虚拟机需要加载 OS，返回新建过程是分钟级别的。而 docker 由于直接利用宿主机的操作系统，则省略了返回过程，因此新建一个 docker 容器只需要几秒钟。



## docker 常用命令

### 容器

#### 进入容器进行交互

- docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
  在正在运行的容器中运行命令
- docker attach [OPTIONS] CONTAINER
  附加到一个正在运行的容器

#### 从容器内拷贝文件到主机上

- docker cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|-
  在容器和本地文件系统之间复制文件/文件夹

#### 导入和导出容器

- 导出

  docker export 79cfc59a7a19 -o 123.tar

- 导入

  cat 123.tar |docker import - nginx:latest

### 镜像

#### 提交容器副本使之成为一个新的镜像

- docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]



## 镜像

UnionFS（联合文件系统）：是一种文件系统技术，它允许将多个不同位置的文件系统合并为一个单一的虚拟文件系统。这种技术在操作系统和存储系统中被广泛应用，包括 Linux、Docker 等。

在容器技术中，UnionFS 被广泛应用于实现容器镜像的存储和管理。比如 Docker 使用 UnionFS 技术来实现镜像的分层存储和容器的文件系统。通过 UnionFS，Docker 可以有效地管理容器的镜像和文件系统，提供高效的资源共享和管理能力。

~~~shell
[root@jenkins-server ~]# docker pull tomcat
Using default tag: latest
Trying to pull repository docker.io/library/tomcat ... 
latest: Pulling from docker.io/library/tomcat
0e29546d541c: Pull complete 
9b829c73b52b: Pull complete 
cb5b7ae36172: Pull complete 
6494e4811622: Pull complete 
668f6fcc5fa5: Pull complete 
dc120c3e0290: Pull complete 
8f7c0eebb7b1: Pull complete 
77b694f83996: Pull complete 
0f611256ec3a: Pull complete 
4f25def12f23: Pull complete 
Digest: sha256:9dee185c3b161cdfede1f5e35e8b56ebc9de88ed3a79526939701f3537a52324
Status: Downloaded newer image for docker.io/tomcat:latest
~~~



docker 镜像层都是只读的，容器层是可写的，当容器启动时，一个信的可写层被加载到镜像顶部。这一层通常被称为“容器层”，“容器层”之下的都叫“镜像层”。



## Dockerfile	

Dockerfile 是用来构建 Docker 镜像的文本文件，是由一条条构建镜像所需的指令和参数构成的脚本。

### 基础知识

- 每条保留字指令都要`大写字母`且后面要跟随只要一个参数。
- 指令按照从上到下，顺序执行。
- 每条指令都会创建一个新的镜像层并对镜像进行提交。



### 执行 dockerfile 的大致流程

#### 1.编写 dockerfile

示例：

~~~dockerfile
FROM python:3.7.2

COPY config/requirements.txt ./

RUN pip config set global.cache-dir "/data/pip-cache"

RUN pip install --upgrade pip -i https://pypi.tuna.tsinghua.edu.cn/simple

# 下载所需要的 python 依赖包
RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple

# 设置时区
RUN cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

EXPOSE 1234

COPY . /home

WORKDIR /home

CMD ["python", "app.py", "--config", "config/boot/boot.min.properties"]
~~~



#### 2.构建 dockerfile

~~~ 
# usage: docker build [OPTIONS] PATH | URL | -
docker build -t test:v1.0 .
~~~



#### 3.运行容器

~~~shell
docker run -d test:v1.0 
~~~



## docker 网络

### 网络模式

1. **桥接网络（bridge）**：Docker 默认的网络模式，容器通过桥接方式连接到宿主机的网络。每个容器都有一个独立的 IP 地址，并且它们可以相互通信。桥接网络适用于在单个主机上运行多个容器的场景。
2. **主机网络（host）**：使用主机网络模式时，容器与宿主机共享网络命名空间，容器可以直接访问宿主机的网络接口，性能更高，但容器的网络配置受限于宿主机。
3. **无网络（none）**：在这种模式下，容器没有网络接口，只能通过其他方式与外部通信，例如与其他容器连接的桥接网络。
4. **container**：在这种模式下，新创建的Docker容器会与一个已经存在的容器共享Network Namespace。这意味着它们共享相同的IP地址和端口空间，而不是与宿主机共享。
5. **自定义网络**：自定义网络和 bridge 网络最大的区别就是主机名和 ip 都有对应关系，而 bridge 只有 ip 有对应关系，早期版本有命令 link 代替自定义网络。

## compose 容器编排

参考文档： https://www.runoob.com/docker/docker-compose.html



































