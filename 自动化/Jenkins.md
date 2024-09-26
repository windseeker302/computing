# Jenkins

## Jenkins 是什么？

Jenkins 是一个开源的持续集成（Continuous Integration）工具，用于自动化构建、测试和部署软件项目。它可以帮助开发团队实现代码的持续集成和交付，提高开发效率和软件质量。Jenkins 支持多种编程语言和开发环境，拥有丰富的插件生态系统，可以灵活扩展其功能。通过 Jenkins，开发团队可以定期自动构建项目、运行测试、生成文档、发布应用程序等，从而加快软件开发周期并降低错误率。



## 部署 CI/CD

"CI/CD" 是 "持续集成/持续交付"（Continuous Integration/Continuous Delivery）的缩写。它是一种软件开发实践，旨在通过自动化和持续性的方式来改进软件开发过程。

### gitlab + jenkins + docker

GitLab、Jenkins 和 Docker 是三种常用的软件工具，它们通常结合使用以实现持续集成和持续交付（CI/CD）流程。



#### 安装 gitlab

官方文档：[The most-comprehensive AI-powered DevSecOps platform | GitLab](https://gitlab.com/)

中文官方文档：[GitLab-10万企业使用的一站式DevOps平台_GitLab中文官网](https://gitlab.cn/)

##### 方法一：ssh 安装 gitlab

参考官方文档：[GitLab下载安装_GitLab最新中文官网免费版下载-极狐GitLab](https://gitlab.cn/install/?version=ce)

1. 安装和配置所需的依赖

   在 CentOS 7 上，下面的命令会在系统防火墙中打开 HTTP、HTTPS 和 SSH 访问。这是一个可选步骤，如果您打算仅从本地网络访问极狐GitLab，则可以跳过它。

   ```shell
   sudo yum install -y curl policycoreutils-python openssh-server perl
   sudo systemctl enable sshd
   sudo systemctl start sshd
   sudo firewall-cmd --permanent --add-service=http
   sudo firewall-cmd --permanent --add-service=https
   sudo systemctl reload firewalld
   ```

   

   （可选）如果要使用 Postfix 来发送电子邮件通知，执行以下安装命令。

   ```shell
   sudo yum install postfix
   sudo systemctl enable postfix
   sudo systemctl start postfix
   ```

   

   在安装 Postfix 的过程中可能会出现一个配置界面，在该界面中选择“Internet Site”并按下回车。把“mail name”设置为您服务器的外部 DNS 域名并按下回车。如果还有其他配置界面出现，继续按下回车以接受默认配置。

   如果您想使用其他解决方案发送电子邮件，请跳过上面 Postfix 安装步骤并在安装极狐GitLab 后[配置外部 SMTP 服务器](https://docs.gitlab.cn/omnibus/settings/smtp.html)。

2. 下载并安装极狐GitLab

   执行以下命令配置极狐GitLab 软件源镜像。

   ```shell
   curl -fsSL https://get.gitlab.cn | /bin/bash
   ```

   接下来，安装极狐GitLab。安装之前，需要确保[您的DNS设置正确](https://docs.gitlab.cn/omnibus/settings/dns.html)。此外，还需要通过设置 `EXTERNAL_URL` 环境变量来指定极狐GitLab 实例的 URL。

   如果您想通过 `HTTPS` 来访问实例，那么您可以根据[官方文档](https://docs.gitlab.cn/omnibus/settings/ssl/index.html)进行配置，让实例使用 Let's Encrypt 自动请求 SSL 证书，这需要有效的主机名和入站 HTTP 访问。您也可以使用自己的证书或仅使用 `http://`（不带 `s`）。

   如果您想为初始管理员用户（ `root` ）指定自定义的初始密码，可以根据[文档指导](https://docs.gitlab.cn/omnibus/installation/index.html#设置初始密码)进行配置。否则将默认生成随机密码。

   接下来执行如下命令开始安装：

   ```shell
   sudo EXTERNAL_URL="https://xxx" yum install -y gitlab-jh		# xxx 可以是 ip 也可以是域名
   ```

   其他配置详情可以查看 [Omnibus 安装配置文档](https://docs.gitlab.cn/omnibus/settings)。

3. 登录极狐GitLab 实例

   使用第二步 `EXTERNAL_URL` 中配置的地址来访问安装成功的极狐GitLab 实例。用户名默认为 `root` 。如果在安装过程中指定了初始密码，则用初始密码登录，如果未指定密码，则系统会随机生成一个密码并存储在 `/etc/gitlab/initial_root_password` 文件中， 查看随机密码并使用 `root` 用户名登录。

   注意：出于安全原因，24 小时后，`/etc/gitlab/initial_root_password` 会被第一次 `gitlab-ctl reconfigure` 自动删除，因此若使用随机密码登录，建议安装成功初始登录成功之后，立即修改初始密码。

常用 gitlab-ctl 命令

- gitlab-ctl start：启动所有 gitlab 组件。
- gitlab-ctl stop：停止所有 gitlab 组件。
- gitlab ctl stop porstgresql：停止相关数据连接服务。
- gitlab-ctl restart 重启所有gitlab 组件。
- gitlab-ctl restart xxx：重启 xxx 组件。
- gitlab-ctl status：查看服务状态。

##### 方法二：docker engine 安装 gitlab

参考官方文档：[极狐GitLab Docker 镜像 | 极狐GitLab](https://docs.gitlab.cn/jh/install/docker.html)

对于 Linux 用户，将路径设置为 `/srv/gitlab`：

```shell
export GITLAB_HOME=/srv/gitlab
```



您可以微调这些目录以满足您的要求。 一旦设置了 `GITLAB_HOME` 变量，您就可以运行镜像：

```shell
sudo docker run --detach \
  --hostname gitlab.example.com \
  --publish 443:443 --publish 80:80 --publish 22:22 \
  --name gitlab \
  --restart always \
  --volume $GITLAB_HOME/config:/etc/gitlab \
  --volume $GITLAB_HOME/logs:/var/log/gitlab \
  --volume $GITLAB_HOME/data:/var/opt/gitlab \
  --shm-size 256m \
  registry.gitlab.cn/omnibus/gitlab-jh:latest
```

这将下载并启动极狐GitLab 容器，并发布访问 SSH、HTTP 和 HTTPS 所需的端口。所有极狐GitLab 数据将存储在 `$GITLAB_HOME` 的子目录中。系统重启后，容器将自动 `restart`。



> 不仅可以使用这两种方式使用，还可以使用 kubernetes 中的 helm 包管理软件来安装。
>
> 注：官网文档使用最新 Helm Chart 安装最新版本，如果想要使用其他版本的镜像来部署，可以根据[极狐GitLab Helm Chart 版本查找指南](https://mp.weixin.qq.com/s/ZIHnB8OHbR7ayJkUIg3lXw)来找到对应的 Chart 进行部署。

#### 安装 jenkins

官方文档：[Jenkins](https://www.jenkins.io/)

##### 方法一：ssh 安装 jenkins

参考文档：[开始使用 Jenkins](https://www.jenkins.io/zh/doc/pipeline/tour/getting-started/)

1. [下载 Jenkins](http://mirrors.jenkins.io/war-stable/latest/jenkins.war).
2. 打开终端进入到下载目录.
3. 运行命令 `java -jar jenkins.war --httpPort=8080`.
4. 打开浏览器进入链接 `http://localhost:8080`.
5. 按照说明完成安装.

##### 方法二：docker engine 安装 jenkins

下载 `jenkinsci/blueocean` 镜像并使用以下docker run 命令将其作为Docker中的容器运行 ：

```shell
docker run \ 
  -u root \
  -d \ 
  -p 8080:8080 \ 
  -p 50000:50000 \ 
  -v jenkins-data:/var/jenkins_home \ 
  -v /var/run/docker.sock:/var/run/docker.sock \ 
  jenkinsci/blueocean 
  
  
docker run   -u root   -d   -p 8080:8080   -p 50000:50000   -v jenkins-data:/var/jenkins_home   -v /var/run/docker.sock:/var/run/docker.sock   jenkinsci/blueocean 
```



> 注：如果在容器内无法下载镜像可以选择更换插件下载地址：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json 
>
> 参考文档：https://blog.csdn.net/G_whang/article/details/131982215

> 坑：更换国内源，也是没法安装插件，原因是：上方的源（链接）是最新版的jenkins源，jenkins不会识别当前版本，下载插件的时候默认使用的是最新版的，更换与 jenkins 相同版本的源就可以了。
>
> 清华源：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/



### 构建触发自动化

#### 方法一：定时触发

![image-20240320202212210](picture/image-20240320202212210.png)

#### 方法二：webhook 

1.打开项目仓库设置，添加 webhooks。

![image-20240320200045296](picture/image-20240320200045296.png)



2.创建 webhook，URL填 jenkins 服务器的地址，加上后缀(/github-webhook/)，最后的斜杠必须带。

![image-20240320211359680](picture/image-20240320211359680.png)



3.在 gitlab 上创建 access token，获取令牌，拥有获取仓库和 webhooks 的权限。

![image-20240320195943306](picture/image-20240320195943306.png)

4.在 jenkins 开启 webhooks，不同的仓库有不同的插件，下图是 gitee的。

![image-20240924201046171](Jenkins.assets/image-20240924201046171.png)

## 插件配置

### 更换国内源，下载插件

Dashboard --> 系统管理 --> 插件管理 --> Advanced settings，在 https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/ 找到自己jenkins 版本，填上这个 URL 即可。

![image-20240925105227569](Jenkins.assets/image-20240925105227569.png)

### 离线下载

- .hpi格式

  1.hpi格式需在官方网站[传送门](https://plugins.jenkins.io/)上下载，输入需要下载的插件。

  ![image-20240925103901984](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240925103901984.png)

  2.在插件里会有 Document 文档，Release 发布的版本，lssues 问题，Dependencies 依赖，health score 健康得分，我们可以选择Release。

  ![image-20240925103939023](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240925103939023.png)

  3.网站中会有两种下载方式，一种CLI，一种本地下载，我们找到适合的版本，把 direct link 粘贴，用 wget 下载，就可以了。

  ~~~shell
  root@mi-easytime:/opt# wget https://updates.jenkins.io/download/plugins/gitee/1.2.7/gitee.hpi
  ~~~

  4.导入，Dashboard --> 系统管理 --> 插件管理 --> Advanced settings，选择下载的 .hpi文件，如果你的网络没问题的情况下，可以直接选择URL。

  ![image-20240925104803449](https://gitee.com/ws203/pic-go-images/raw/master/imgs/image-20240925104803449.png)

  

- .jpi格式

  直接将jpi文件放入Jenkins的plugins文件夹下（rpm安装的jenkins，路径是：/var/lib/jenkins/plugins/），然后重启Jenkins即可。

  用这种方式既可以批量安装插件，安装时又可以忽视插件之间的关联性。若依赖的插件不存在或者存在版本问题，则重启之后会在Manage Jenkins中进行提示，根据提示逐一解决问题即可。

  ~~~shell
  root@mi-easytime:/opt# cd /var/lib/docker/volumes/jenkins-data/_data
  root@mi-easytime:/var/lib/docker/volumes/jenkins-data/_data/plugins# ls
  ace-editor                                                 display-url-api.jpi                               pipeline-build-step.jpi.pinned
  ace-editor.jpi                                             display-url-api.jpi.pinned                        pipeline-build-step.jpi.version_from_image
  ace-editor.jpi.pinned                                      display-url-api.jpi.version_from_image            pipeline-github-lib
  ace-editor.jpi.version_from_image                          durable-task                                      pipeline-github-lib.jpi
  ant                                                        durable-task.bak                                  pipeline-graph-analysis
  ~~~

  

## 实战 Gitee 配置 jenkins Webhook

### 0.环境

- 系统版本

  ~~~shell
  root@mi-easytime:/opt# uname -a
  Linux mi-easytime 6.1.0-25-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.106-3 (2024-08-26) x86_64 GNU/Linux 
  ~~~

  

- jenkins版本
  - JENKINS_VERSION=2.346.3

​	

### 1.插件安装

1.在可用插件中查找 `gitee`，这里已经下载好了，在 installed plugins 中显示。

![image-20240923195926542](Jenkins.assets/image-20240923195926542.png)

![image-20240923195847041](Jenkins.assets/image-20240923195847041.png)

### 2.插件配置

1. 前往 Jenkins -> Manage Jenkins -> Configure System -> Gitee Configuration -> Gitee connections
2. 在 `Connection name` 中输入 `Gitee` 或者你想要的名字
3. `Gitee host URL` 中输入Gitee完整 URL地址： `https://gitee.com` （Gitee私有化客户输入部署的域名），如“https://gitee.com/ws203/share.git”
4. Credentials` 中如还未配置Gitee APIV5 私人令牌，点击 `Add` - > `Jenkins
   1. `Domain` 选择 `Global credentials`
   2. `Kind` 选择 `Gitee API Token`
   3. `Scope` 选择你需要的范围
   4. `Gitee API Token` 输入你的Gitee私人令牌，获取地址：https://gitee.com/profile/personal_access_tokens
   5. `ID`, `Descripiton` 中输入你想要的 ID 和描述即可。
5. `Credentials` 选择配置好的 Gitee APIV5 Token
6. 点击 `Advanced` ，可配置是否忽略 SSL 错误（视您的Jenkins环境是否支持），并可设置链接测超时时间（视您的网络环境而定）
7. 点击 `Test Connection` 测试链接是否成功，如失败请检查以上 3，5，6 步骤。

![image-20240925110914002](Jenkins.assets/image-20240925110914002.png)

>  坑：测试失败也没关系，后续操作不影响。
>

### 3.创建流水线（Pipeline）

新建任务。

![image-20240925111135841](Jenkins.assets/image-20240925111135841.png)

### 4.任务全局配置

需要选择前一步中的Gitee链接。前往某个任务（如'Gitee Test'）的 Configure -> General，Gitee connection 中选择前面所配置的Gitee链接，如图：

![image-20240925154035722](Jenkins.assets/image-20240925154035722.png)

可在 Manage Jenkins -> Configure System -> 全局属性 ，设置环境变量。如运行 shell 可以借助 jenkins 中的环境变量进行使用。

### 5.源代码管理配置

前往某个任务（如'Gitee Test'）的 Configure -> Source Code Management 选项卡

1. 点击 *Git*
2. 输入你的仓库地址，例如 `git@your.gitee.server:gitee_group/gitee_project.git`
3. 凭据Credentials 中请输入 git 仓库 https 地址对应的用户名密码凭据，或者 ssh 对应的 ssh key 凭据，注意 Gitee API Token 凭据不可用于源码管理的凭据，只用于 gitee 插件的 API 调用凭据。
4. 选择分支，选择仓库的分支即可。

配置如图所示：

![image-20240925154144721](Jenkins.assets/image-20240925154144721.png)

### 6.触发器配置

选择 Gitee webhook 触发构建。默认设置即可，将gitee webhook 密码生成一下。

### 7.构建后步骤配置

构建失败，仅为构建失败回评到Gitee。

![image-20240925160126674](Jenkins.assets/image-20240925160126674.png)

### 8.新建Gitee项目WebHook

返回 gitee，选择仓库设置，选择 WebHooks 填写下图标红的内容后，点击添加按钮。

![image-20240925155118805](Jenkins.assets/image-20240925155118805.png)

文档：[Jenkins 插件 - Gitee.com](https://gitee.com/help/articles/4193#article-header0)



## 流水线（Pipeline）

![image-20240925160552990](Jenkins.assets/image-20240925160552990.png)

点击下方流水线语法可打开片段生成器。

