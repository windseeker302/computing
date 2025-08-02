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

1. docker 有着比虚拟机更少的抽象层。

   由于 docker 不需要 hypervisor（虚拟机）实现硬件资源虚拟化，运行在 docker 容器上的程序直接使用的都是宿主机的硬件资源。

2. docker 利用的是宿主机的内核，而不需要加载操作系统 OS 内核。

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

- docker commit -m="提交的描述信息" -a="作者" 容器ID 要创建的目标镜像名:[标签名]。

- docker history [OPTIONS] IMAGE ：查看指定镜像的历史层信息。
  - -H, --human: 以人类可读的格式显示镜像大小（默认启用）。
  - --no-trunc: 显示完整的输出，不截断信息。
  - -q, --quiet: 仅显示镜像 ID。

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



### dockerfile 第一次

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

~~~ shell
# usage: docker build [OPTIONS] PATH | URL | -
docker build -t test:v1.0 .
~~~



#### 3.运行容器

~~~shell
docker run -d test:v1.0 
~~~



### 参数概述

官方文档：[传送门](https://docs.docker.com/reference/dockerfile/)

#### ADD

语法：ADD [OPTIONS] `<src> ... <dest>`

- 添加本地文件或远程文件和目录。

  ~~~shell
  ADD file1.txt file2.txt /usr/src/things/
  ADD git@github.com:user/repo.git /usr/src/things/
  ~~~

> 注意：
>
> - 如果 `<src>` 是本地文件或目录，则该目录的内容将复制到指定的目标。
>
> - 如果 `<src>` 是本地 tar 存档，则会解压缩并提取到指定的目标。
>
> - 如果 `<src>` 是 URL，则将下载该 URL 的内容并将其放置在指定的目标位置。
> - 如果 `<src>` 是 Git 仓库，则该仓库将克隆到指定的目标。

------

#### ARG

语法：ARG `<name>[=<default value>]`

- 使用构建时变量

- 示例：

  ~~~shell
  ARG  CODE_VERSION=latest
  FROM base:${CODE_VERSION}
  ~~~

  

------

#### CMD

- 设置从映像运行容器时要执行的命令。

- Dockerfile中只能有一个`CMD`指令。如果列出多个 CMD，则只有最后一个 `CMD` 生效。

- 示例：

  ~~~shell
  CMD ["sh","-c","sleep 100"]
  ~~~

> 注意：不要将 `RUN` 与 `CMD` 混淆。`RUN`实际上运行一个命令并提交结果;`CMD` 在构建时不执行任何操作，但指定镜像的预期命令。

------

#### COPY

语法：COPY [OPTIONS] `<src> ... <dest>`

- `COPY` 指令从 `<src>` 复制新文件或目录，并将它们添加到映像的文件系统中的路径 `<dest>`。
- 可以从生成上下文、生成阶段、命名上下文或图像中复制文件和目录。

[Dockerfile 参考 |Docker 文档 --- Dockerfile reference | Docker Docs](https://docs.docker.com/reference/dockerfile/#copy)

> ADD和COPY指令在功能上相似，但用途略有不同。
>
> `ADD` 和 `COPY` 在功能上相似。`COPY` 支持从[构建上下文](https://docs.docker.com/build/building/context/)或[多阶段构建](https://docs.docker.com/build/building/multi-stage/)中的某个阶段将文件基本复制到容器中。`ADD` 支持从远程 HTTPS 和 Git URL 获取文件的功能，以及在从构建上下文添加文件时自动提取 tar 文件的功能。

------

#### ENTRYPOINT

- `docker run <image>` 的命令行参数将附加在 exec 表单 `ENTRYPOINT` 中的所有元素之后，并将覆盖使用 `CMD` 指定的所有元素。

- 示例：

  ~~~shell
  ENTRYPOINT ["executable", "param1", "param2"]
  ~~~

------

#### ENV

语法：ENV `<key>=<value>` ...

- 设置环境变量。

- 示例：

  ~~~shell
  ENV MY_NAME="John Doe"
  ENV MY_DOG=Rex\ The\ Dog
  ENV MY_CAT=fluffy
  ~~~

------

#### EXPOSE

语法：EXPOSE `<port> [<port>/<protocol>`...]

- 描述应用程序正在侦听的端口。

- 通知 Docker 容器在运行时侦听指定的网络端口。您可以指定端口是侦听 TCP 还是 UDP，如果您不指定协议，则默认为 TCP。

- `EXPOSE` 指令实际上并未发布端口。它充当构建映像的人员和运行容器的人员之间的一种文档，用于说明要发布的端口。要在运行容器时发布端口，请在 `docker run` 上使用 `-p` 标志来发布和映射一个或多个端口，或使用 `-P` 标志来发布所有公开的端口并将它们映射到高阶端口。

- 示例：

  ~~~shell
  EXPOSE 80/tcp
  EXPOSE 80/udp
  ~~~

------

#### FROM

语法：FROM `[--platform=<platform>] <image> [AS <name>]`

- 从基础映像创建新的生成阶段。
- 始化新的构建阶段，并为后续指令设置[基础映像](https://docs.docker.com/glossary/#base-image)。因此，有效的 Dockerfile 必须以 `FROM` 指令开头。图像可以是任何有效的图像。

------

#### HEALTHCHECK

作用：在启动时检查容器的运行状况。

两种形式：

- `HEALTHCHECK [OPTIONS] CMD 命令`（通过在容器内运行命令来检查容器运行状况）

- `HEALTHCHECK NONE`（禁用从基础映像继承的任何运行状况检查）

官方文档：[Dockerfile 参考 |Docker 文档 --- Dockerfile reference | Docker Docs](https://docs.docker.com/reference/dockerfile/#healthcheck)

------

#### SHELL

- `SHELL` 指令允许覆盖用于 shell 形式的命令的默认 shell。Linux 上的默认 shell 是 `[“/bin/sh”， “-c”`]，Windows 上的默认 shell 是 `[“cmd”， “/S”， “/C”]。``SHELL` 指令必须在 Dockerfile 中以 JSON 形式编写。

- 示例：

  ~~~shell
  SHELL ["executable", "parameters"]
  ~~~

------

#### LABEL

语法：LABEL `<key>=<value> <key>=<value> <key>=<value> ...`

- 指令将元数据添加到图像中。`LABEL`是一个键值对。若要在 `LABEL` 值中包含空格，请像在命令行分析中一样使用引号和反斜杠。

- 示例：

  ~~~shell
  LABEL "com.example.vendor"="ACME Incorporated"
  LABEL com.example.label-with-value="foo"
  LABEL version="1.0"
  LABEL description="This text illustrates \
  that label-values can span multiple lines."
  ~~~

  

------

#### MAINTAINER（已弃用）

- 指定图像的作者。

- 示例：

  ~~~shell
  MAINTAINER <name>
  ~~~

> 注意：`MAINTAINER` 指令设置生成的图像的 *Author* 字段。`LABEL`指令是一个更灵活的版本，你应该使用它，因为它允许设置你需要的任何元数据，并且可以很容易地查看，例如使用`docker inspect`。要设置与 `MAINTAINER` 字段对应的标签，您可以使用：

```dockerfile
LABEL org.opencontainers.image.authors="SvenDowideit@home.org.au"
```

------

#### RUN

语法：

```dockerfile
# Shell form:
RUN [OPTIONS] <command> ...
# Exec form:
RUN [OPTIONS] [ "<command>", ... ]
```

- 执行构建命令。

------

#### USER

语法：

  USER `<user>[:<group>]`

  USER UID[:GID]

- 设置用户和组 ID。

- 示例：

  ~~~shell
  USER patrick
  ~~~

------

#### VOLUME

- 创建卷挂载。

- 示例：

  ~~~dockerfile
  VOLUME ["/data"]
  ~~~

  

------

#### WORKDIR

- 更改工作目录。

- 示例：

  ~~~dockerfile
  WORKDIR /path/to/workdir
  ~~~

  



## docker 网络

### 网络模式

1. **桥接网络（bridge）**：Docker 默认的网络模式，容器通过桥接方式连接到宿主机的网络。每个容器都有一个独立的 IP 地址，并且它们可以相互通信。桥接网络适用于在单个主机上运行多个容器的场景。
2. **主机网络（host）**：使用主机网络模式时，容器与宿主机共享网络命名空间，容器可以直接访问宿主机的网络接口，性能更高，但容器的网络配置受限于宿主机。
3. **无网络（none）**：在这种模式下，容器没有网络接口，只能通过其他方式与外部通信，例如与其他容器连接的桥接网络。
4. **container**：在这种模式下，新创建的Docker容器会与一个已经存在的容器共享Network Namespace。这意味着它们共享相同的IP地址和端口空间，而不是与宿主机共享。
5. **自定义网络**：自定义网络和 bridge 网络最大的区别就是主机名和 ip 都有对应关系，而 bridge 只有 ip 有对应关系，早期版本有命令 link 代替自定义网络。



## docker 代理

### docker 命令走代理

[dockerd |Docker 文档 --- dockerd | Docker Docs](https://docs.docker.com/reference/cli/dockerd/#proxy-configuration)

然而实际测试下来，就算我们修改成功了国内的镜像源，有时候由于国内镜像更新不及时，或者需要拉取的镜像比较冷门，只有域外镜像站才有，那么我们不得不让docker pull命令，走我们的代理。
我们在docker的进程服务文件夹配置我们的代理设置，如果没有我们就新建这个文件夹：

~~~shell
sudo mkdir /etc/systemd/system/docker.service.d
~~~




然后在docker.service.d文件夹里新建我们的代理文件proxy.conf

~~~shell
sudo vim /etc/systemd/system/docker.service.d/proxy.conf
~~~



并把文件写如下面这个格式：

~~~shell
[Service] 
Environment="HTTP_PROXY=代理服务器ip:port" 
Environment="HTTPS_PROXY=代理服务器ip:port"
~~~

假如我们本机已经设置好代理了，那么代理服务器就可以写为localhost，端口就是我们设置的http和https代理端口即可，形如：

~~~shell
[Service] 
Environment="HTTP_PROXY=localhost:port" 
Environment="HTTPS_PROXY=localhost:port"	
~~~


保存并退出proxy.conf文件，和更改镜像源一样，重启docker，并重启daemon进程。

~~~shell
sudo systemctl daemon-reload		#重启daemon进程
sudo systemctl restart docker		#重启docker
~~~


最后我们仍然是验证一下是否修改成功，运行

~~~shell
docker info
# 结果
HTTP Proxy: 10.64.150.195:7890
HTTPS Proxy: 10.64.150.195:7890
~~~



那就说明我们已经成功设置docker pull命令走代理了，一般情况下也就不会出现拉取镜像卡死的情况了。



### docker 加速器

~~~shell
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://docker.cloudyshore.top/"]
}
EOF
systemctl daemon-reload
systemctl restart docker
~~~





## docker 仓库

docker 仓库有很多，有些

- 本地私有镜像仓库 registry
- 企业级仓库 Harbor

注意：

- 镜像名称常用命名规则：${registry_name}/${repository_name}/$image_name}:$tag_name}

- 远端仓库地址urI/分类仓库名字/镜像名字:标签名字
- 示例： harbor.test.com/test/nginx:v1

### 搭建本地私有镜像仓库 registry

registry是一个非常简单的轻量级本地私有仓库，通过push命令，存储本地(自定义)镜像到私有仓库registry。

#### 搭建

拉取并启动容器。

~~~shell
docker run -d -p 5000:5000 --name registry registry:latest
~~~



容器启动后，访问 ip:5000/v2/_catalog 就能看见私有仓库中存储的镜像。

刚刚搭建完成的，肯定是没有的，怎么上传呢？

#### 上传镜像

编辑 dokcer 配置文件，要注意文件的格式，json 格式严禁。

~~~shell
vim /etc/docker/daemon.json
{
"insecure-registries": ["192.168.100.10:5000"]		# ip 是本机的，也可以写 0.0.0.0/0
}
~~~



重启服务

~~~shell
systemctl daemon-reload
systemctl restart docker
~~~



尝试把本地的 nginx 镜像上传到私有镜像仓库 registry。

~~~shell
docker tag nginx:latest 192.168.100.10:5000/mynginx:v1	 # 给已有镜像打标签
docker push 192.168.100.10:5000/mynginx:v1		# 上传镜像到私有仓库
~~~



### 共有镜像仓库 

共有仓库有很多，比如 dockerhub，华为云镜像仓库，阿里云镜像仓库，官方的是 dockerhub，我们这里就以 dockerhub 举例，

共有仓库和私有仓库的操作步骤，有些区别，共有仓库不需要搭建，只需要注册一个 hub.docker.com 的账号即可。

#### 登录

注册完成后，即可在 docker 机器上输入：

~~~shell
[root@docker-node ~]# docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: windseeker302		# 注册时的用户名
Password: 
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded

# 注销登录
[root@kubeedge-node ~]# docker logout
Removing login credentials for https://index.docker.io/v1/
~~~



#### 上传镜像

~~~shell
# 上传本地镜像 nginx 到 dockerhub 上。
# 修改标签，选择 test 仓库。
docker tag nginx windseeker302/test:mynginx

# dockerhub 会自动创建一个 test 仓库，并存放镜像。
docker push windseeker302/test:mynginx
~~~



## compose 容器编排

### 简介

compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟 openstack 中的 Heat 十分类似。

其代码目前在 https://github.com/docker/compose 上开源。

Docker Compose 是一个用于定义和运行多容器 Docker 应用程序的工具。通过 Docker Compose，你可以使用 YAML 文件来配置应用程序的服务。然后，使用一个命令来启动和运行所有配置的服务。



### 第一个案例

官方文档：[Compose Build 规范 |Docker 文档 --- Compose Build Specification | Docker Docs](https://docs.docker.com/compose/compose-file/build/)

1. 创建一个测试目录

   ~~~shell
   mkdir ems
   ~~~

   

2. 编写 docker-compose.yml 文件

   ~~~yml
   version: "3.1"
   
   services:
     tomcat:		# 服务名称，可以自定义，但是要确定唯一性
   #    container_name: tomcat01 	# 容器名称，推荐默认名称
       image: tomcat:8.0
       ports:
        - 8080:8080        # 最好加引号
     mysql:
       image: mysql:5.6
       ports:
        - "3306:3306"
       restart: always
       environment:
        - "MYSQL_ROOT_PASSWORD=root"       # 第一种写法
   #     MYSQL_ROOT_PASSWORD: root        # 第二种写法
       volumes:
   #    - "/root/mysqldata:/var/lib/mysql"   # 绝对路径
        - "mysqlData:/var/lib/mysql"              # 别名，必须要声明
   
   volumes:
     mysqlData:          # 声明数据卷别名
   ~~~

   1. version：docker-compose 版本和 docker 对应版本关系。
   2. servers：定义和配置应用程序所需的所有服务。
   3. volumes: 定义命名卷，这里是 `mysqlData`，用于持久化数据库数据。

3. 启动 docker-compose

   ~~~shell
   docker-compose up -d 
   ~~~

   

### docker-compose 模板指令

很多很多~，官方文档奉上：[概览 |Docker 文档 --- Overview | Docker Docs](https://docs.docker.com/compose/compose-file/)

以下是常见的模板指令：

#### build 

通过 docker-compose 在启动容器之前根据 dockerfile 文件构建镜像，然后根据构建好的镜像启动容器。

~~~yaml
version: "3.1"
service:
  apps:
    # build: ./	# 指定 Dockerfile 上下文目录 context，一切都是默认值
    build:
      context: ./	# 指定 dockerfile 上下文目录
      dockerfile: Dockerfile 	# 指定 dockerfile 文件名称
    ports:
     - "8081:8081"
~~~



#### command 

覆盖容器启动后默认指定的命令

~~~yaml
version: "3.1"
service:
  apps:
    image: mysql
    ports:
     - "3306:3306"
    command: ["sh","-c","sleep 10"]
~~~



#### depends_on 

解决容器的依赖，***启动先后***的问题，以下的例子中，先启动 ***apps1*** 再启动 ***apps2***

~~~yml
version: "3.1"
service:
  apps1:
    image: redis
 
  apps2:
    image: mysql
    ports:
     - "3306:3306"
    environment:
      MYSQL_PASSWORD_ROOT: root
    command: ["sh","-c","sleep 10"]
    depends_on:
     - apps1
~~~



#### env_file

用于指定一个或多个文件，这些文件包含要传递给容器的环境变量。

~~~yml
version: "3.1"
service:
  apps1:
    image: redis
 
  apps2:
    image: mysql
    ports:
     - "3306:3306"
    env_file:
     - ./.env
    command: ["sh","-c","sleep 10"]
    depends_on:
     - apps1
~~~



#### environment

指定环境变量，有两种方法：

Map syntax:



```yml
environment:
  RACK_ENV: development
  SHOW: "true"
  USER_INPUT:
```

Array syntax:



```yml
environment:
  - RACK_ENV=development
  - SHOW=true
  - USER_INPUT
```



#### networks

In the following example, at runtime, networks front-tier and back-tier are created and the frontend service is connected to front-tier and back-tier networks.
在下面的示例中，在运行时，创建了***前端***和***后端***网络，并将***前端***服务连接到***前端***和***后端***网络。

~~~yaml
services:
  frontend:
    image: example/webapp
    networks:
      - front-tier
      - back-tier

networks:
  front-tier:
  back-tier:
~~~



#### restart

定义了平台在容器终止时应用的策略。

- `no`：默认重启策略。在任何情况下，它都不会重新启动容器。
- `always`：策略始终重新启动容器，直到将其删除。
- `on-failure[：max-retries]`：如果退出代码指示错误，策略将重新启动容器。（可选）限制 Docker 守护程序尝试的重新启动重试次数。
- `unless-stopped`：无论退出代码如何，策略都会重新启动容器，但在服务停止或删除时会停止重新启动。

~~~yaml
    restart: "no"
    restart: always
    restart: on-failure
    restart: on-failure:3
    restart: unless-stopped
~~~



### docker-compose 常用命令

```shell
Usage:
  docker-compose [-f ...] [options] [COMMAND] [ARGS...]  
```

> 注意：如果 docker-compose 没有跟特殊说明（service）时，默认就是对整个项目操作。

#### up[重点]

语法： `docker-compose up [options] [--scale SERVICE=NUM...] [SERVICE]`

- 它可以将尝试自动完成包括构建镜像，（重新）创建服务，启动服务，并关联相关容器的一系列操作。
- 默认情况下，docker-compose up 启动的容器都在前台，控制台会同时打印所有容器的输出信息，可以很方便进行调试。

- 当通过 Ctrl-c 停止命令时，所有容器将会停止。
- 如果使用 docker-compose up -d 将会在后台启动并运行所有的容器，一般推荐生产环境下使用该选项。

------

#### down[重点]

语法：`docker-compose down [options]` 

- docker-compose down 关闭所有容器。
- 此命令会停止 up 命令启动的容器，并移除网络。

------

#### exec

语法：`docker-compose exec [options] [-e KEY=VAL...] SERVICE COMMAND [ARGS...]`

- 进入指定的容器。

------

#### ps

语法：`docker-compose ps [options] [SERVICE...]`

- 列出项目中目前的所有容器
- 选项:
  - -q：只打印容器的ID信息。

------

#### rm

语法：`docker-compose rm [options] [SERVICE...]`

- 删除所有（停止状态的）服务容器。

------

#### restart

语法：`dokcer-compose restart [options] [SERVICE...]`

- 重启整个项目或指定 id 服务。

------

#### top

语法：`docker-compose top [SERVICE...]`

- 显示正在运行的进程。

------

#### logs

语法：`docker-compose logs [options] [SERVICE...]`

- 查看容器的输出信息。

------

#### unpause 和 pause

语法：`docker-compose unpause/pause [SERVICE...] `

- 恢复和停止服务容器。





参考文档： https://www.runoob.com/docker/docker-compose.html



































