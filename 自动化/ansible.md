# Ansible

### 1、Ansible简介

#### 1.1.Ansible的历史和架构

Ansible 是一种基于 SSH 协议管理、部署和自动化运维的工具，它的历史可以追溯到 2012 年。下面分别介绍 Ansible 的历史和架构。

1. Ansible 的历史
   Ansible 由 Michael DeHaan 创建，旨在提供一种简单、易学易用的自动化运维工具。它最初由 YAML 语言编写，后来发展为一种基于 Python 的工具，能够利用 SSH 协议等方式进行分布式管理。目前，Ansible 已经成为自动化运维工具领域的佼佼者之一，得到了广泛应用和认可。
2. Ansible 的架构
   Ansible 采用了基于 agent-less 架构的设计，即在被管理的主机上不需要安装任何客户端软件，只需要在中控主机上安装 Ansible 工具即可实现远程管理。其主要组件包括：

- `Inventory`：清单，用于定义需要管理的主机列表。
- `Ad-hoc commands`：临时命令，用于执行简单的、短暂的命令，例如文件拷贝、命令执行等。
- `Playbooks`：剧本，用于定义任务和执行步骤的脚本。
- `Modules`：模块，用于实现不同的操作，例如文件操作、软件安装、用户管理等。
- `Plugins`：插件，用于扩展和定制 Ansible 功能，例如自定义模块、连接器等。

在使用 Ansible 进行自动化运维时，管理员需要首先在 Inventory 中配置需要管理的主机列表，然后通过 Playbook 文件定义管理任务和执行步骤，最后使用 Ansible 命令行工具执行管理任务。整个过程通过 SSH 协议等方式进行通信，安全可靠，具有简单、易学易用等优点。

#### 1.2.Ansible的优点和特点

Ansible 是一种基于 SSH 协议的自动化运维工具，它有以下优点和特点：

1. ansible基于Python开发，集合了众多老牌运维工具的优点，实现了批量系统配置、批量程序部署、批量运行命令等功能。
2. ansible是基于模块工作的，本身没有批量部署的能力。真正具有批量部署的是ansible所运行的模块，ansible只是提供一种框架。
3. Ansible特性包括：不需要在被管控主机上安装任何客户端；无服务器端，使用时直接运行命令即可。
4. Ansible基于模块工作，可使用任意语言开发模块；使用yaml语言定制剧本playbook；基于SSH工作；可实现多级指挥。
5. 批量任务执行可以写成脚本，而且不用分发到远程就可以执行；使用python编写，维护更简单，支持sudo。
6. 连接插件connection plugins：负责和被管控端实现通信；host inventory：指定操作的主机，是一个配置文件里面定义管控的主机。
7. 各种模块核心模块、command模块、自定义模块；借助于插件完成记录日志邮件等功能；剧本执行多个任务时，可以让被管控端一次性运行多个任务。

### 2、安装和配置Ansible

ansible官网：[传送门](https://www.ansible.com/)

- 安装Ansible软件

  ~~~shell
  # 添加epel存储库
  sudo dnf install epel-release
  # 安装ansible
  sudo dnf install -y ansible
  # 验证是否安装成功
  ansible --version
  ~~~

  

- 配置Ansible基本环境

  | 主机名            | ip地址       | 系统            |
  | ----------------- | ------------ | --------------- |
  | linux1.skills.lan | 10.10.70.101 | linux Rocky 9.1 |
  | linux2.skills.lan | 10.10.70.102 | linux Rocky 9.1 |
  | linux3.skills.lan | 10.10.70.103 | linux Rocky 9.1 |

  1. 安装python，大多数linux都会默认安装python

  2. 配置ansible主机清单

     默认情况下，Ansible 在 /etc/ansible/hosts 目录下查找主机清单文件。

     ~~~shell
     vim /etc/ansible/hosts
     [server]
     linux1.skills.lan
     [web]
     linux2.skills.lan
     [db]
     linux3.skills.lan
     
     [client:children]
     web
     db
     ~~~
  
     
  
  3. 配置SSH认证
  
     ~~~shell
     # 生成ssh公钥和私钥
     ssh-keygen 
     Generating public/private rsa key pair.
     Enter file in which to save the key (/root/.ssh/id_rsa): 
     Created directory '/root/.ssh'.
     Enter passphrase (empty for no passphrase): 
     Enter same passphrase again: 
     Your identification has been saved in /root/.ssh/id_rsa
     Your public key has been saved in /root/.ssh/id_rsa.pub
     The key fingerprint is:
     SHA256:RV0hBr9gApBszmVIQg4TBB43yArlbhyx6ABL1xWEBdk root@linux3.skills.lan
     The key's randomart image is:
     +---[RSA 3072]----+
     |XB**+=X=. ooo.o. |
     |=X++*+oE . o..   |
     |*.=+ o  . + .    |
     |+o .o    + . .   |
     | .+     S   .    |
     | .               |
     |                 |
     |                 |
     |                 |
     +----[SHA256]-----+
     # 授予公钥
     # linux1.skills.lan
     ssh-copy-id linux1.skills.lan
     /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
     The authenticity of host 'linux1.skills.lan (10.10.70.101)' can't be established.
     ED25519 key fingerprint is SHA256:SIyQhLdEgbBjdPBxp3urM2NfgzOshkRkgFidDFTqfRk.
     This key is not known by any other names
     Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
     /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
     /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
     root@linux1.skills.lan's password: 
     
     Number of key(s) added: 1
     
     Now try logging into the machine, with:   "ssh 'linux1.skills.lan'"
     and check to make sure that only the key(s) you wanted were added.
     # linux2.skills.lan
     ssh-copy-id linux2.skills.lan
     /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
     The authenticity of host 'linux2.skills.lan (10.10.70.102)' can't be established.
     ED25519 key fingerprint is SHA256:SIyQhLdEgbBjdPBxp3urM2NfgzOshkRkgFidDFTqfRk.
     This host key is known by the following other names/addresses:
         ~/.ssh/known_hosts:1: linux1.skills.lan
     Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
     /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
     /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
     root@linux2.skills.lan's password: 
     
     Number of key(s) added: 1
     
     Now try logging into the machine, with:   "ssh 'linux2.skills.lan'"
     and check to make sure that only the key(s) you wanted were added.
     # linux3.skills.lan
     ssh-copy-id linux3.skills.lan
     /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
     The authenticity of host 'linux3.skills.lan (10.10.70.103)' can't be established.
     ED25519 key fingerprint is SHA256:SIyQhLdEgbBjdPBxp3urM2NfgzOshkRkgFidDFTqfRk.
     This host key is known by the following other names/addresses:
         ~/.ssh/known_hosts:1: linux1.skills.lan
         ~/.ssh/known_hosts:4: linux2.skills.lan
     Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
     /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
     /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
     root@linux3.skills.lan's password: 
     
     Number of key(s) added: 1
     
     Now try logging into the machine, with:   "ssh 'linux3.skills.lan'"
     and check to make sure that only the key(s) you wanted were added.
     ~~~
  
     
  
  4. 验证
  
     ~~~shell
     ansible all -m ping
     linux1.skills.lan | SUCCESS => {
         "ansible_facts": {
             "discovered_interpreter_python": "/usr/bin/python3"
         },
         "changed": false,
         "ping": "pong"
     }
     linux2.skills.lan | SUCCESS => {
         "ansible_facts": {
             "discovered_interpreter_python": "/usr/bin/python3"
         },
         "changed": false,
         "ping": "pong"
     }
     linux3.skills.lan | SUCCESS => {
         "ansible_facts": {
             "discovered_interpreter_python": "/usr/bin/python3"
         },
         "changed": false,
         "ping": "pong"
     }
     ~~~
  
     这应该在所有主机上执行 ping 测试并返回 pong，表示一切正常。
  
  

### 3、Ad-hoc命令行

#### 3.1.解释Ad-hoc命令行

Ansible Ad-hoc 命令是一种使用 Ansible 的快速方式，它允许用户在命令行上执行一个临时的命令，而不需要编写一个完整的 playbook。Ad-hoc 命令可以在单个或多个主机上执行，并且可以在不扰乱 Ansible 应用程序的情况下测试 Ansible 模块。

#### 3.2.如何在目标系统上运行Ad-hoc命令

使用Ansible的Ad-hoc命令可以在目标系统上迅速地运行一次性命令。以下是在目标系统上运行Ad-hoc命令的步骤：

1. 确认目标主机已经配置了SSH访问，并具有管理员权限；
2. 进入主控节点命令行终端，输入以下命令，并将 <TARGET_HOST> 替换为目标主机的IP或主机名：

```shell
ansible <TARGET_HOST> -u <REMOTE_USER> -a '<COMMAND>'
```

其中，<REMOTE_USER> 为目标主机上的远程用户，默认为root。

3. 运行 Ad-hoc 命令，例如输入以下命令来查看目标主机的系统信息：

```shell
ansible all -u root -a 'uname -a'
# 输出内容
linux2.skills.lan | CHANGED | rc=0 >>
Linux linux2.skills.lan 5.14.0-162.6.1.el9_1.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Nov 18 02:06:38 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
linux3.skills.lan | CHANGED | rc=0 >>
Linux linux3.skills.lan 5.14.0-162.6.1.el9_1.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Nov 18 02:06:38 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

运行上述命令后，Ansible会自动连接目标主机并执行指定的命令，输出命令执行结果。

注意：Ad-hoc命令仅适用于一次性的、简单的命令，如果需要运行复杂的、长期的任务，建议使用Ansible剧本（playbook）来完成。



### 4、Inventory概述

#### 4.1.Inventory文件的结构和内容

Ansible中的Inventory文件是一个文本文件，通常采用INI格式进行编写，包含了主机和组的定义以及相关的变量，`/etc/ansible/hosts` 就是ansible默认定义的一个Inventory清单文件。以下是一个标准的Inventory文件的结构和内容：

1. 主机定义

主机定义用来指定要管理的主机及其相关信息，包括主机名、IP地址、连接方式和端口等。其格式为：

```shell
hostname ansible_host=<IP address> ansible_connection=<connection> ansible_port=<port>
```

其中

- `hostname`：表示主机名

- `ansible_host`：用来指定主机的IP地址或主机名

- `ansible_connection`：指定连接方式

- `ansible_port`：指定端口号

2. 组定义

通过组定义，可以将主机归为某个组中，方便进行批量操作。其格式为：

```shell
[groupname]
hostname1
hostname2
```

其中，groupname表示组名，hostname1和hostname2表示该组包括的主机。

3. 组继承

可以通过继承机制来构建主机组之间的层次关系，其格式为：

```shell
[groupname:children]
child-group1
child-group2
```

其中，groupname表示父组名，child-group1和child-group2表示该父组包括的子组，即继承的组。

4. 主机变量

可以为每个主机或主机组定义一些额外的变量，以便在编写剧本时使用。其格式为：

```shell
[groupname:vars]
var1=value1
var2=value2
```

其中，groupname表示主机组名，var1和var2表示变量名，value1和value2表示变量的值。


除了上述常见的规则之外，还可以定义一些特殊的组或变量，例如all组、ungrouped组和_ansible_ssh_user等。需要注意的是，如果Inventory文件中存在语法错误或逻辑错误，则Ansible无法正常运行。因此，在编写Inventory文件时需要仔细检查每个组和主机的定义，确保其符合格式要求。以下是一个Inventory文件的示例：

~~~shell
[web_servers]
web1 ansible_host=192.168.1.1
web2 ansible_host=192.168.1.2

[db_servers]
db1 ansible_host=192.168.1.3

[linux_servers:children]
web_servers
db_servers

[linux_servers:vars]
ansible_ssh_user=root
ansible_ssh_port=22
~~~



#### 4.2.如何创建和管理Inventory

在 Ansible 中，可以通过以下几种方式创建和管理 Inventory：

1. 手动创建 Inventory 文件

可以手动编写 Inventory 文件，并保存在特定的位置。在运行 Ansible 命令时，可以使用 `-i` 参数来指定要使用的 Inventory 文件路径，例如：

```shell
ansible all -i my_inventory.ini -m ping
```

上述命令会使用名为 `my_inventory.ini` 的 Inventory 文件，向所有主机发送 ping 命令。

2. 使用动态 Inventory 脚本

可以编写脚本来动态生成 Inventory，例如使用脚本从 AWS 或 Azure 中获取主机信息，从而实现自动化管理。在运行 Ansible 命令时，可以使用 `-i` 参数来指定要使用的动态 Inventory 脚本，例如：

```shell
ansible all -i my_inventory_script.py -m ping
```

上述命令会使用名为 `my_inventory_script.py` 的动态 Inventory 脚本，向所有主机发送 ping 命令。

3. 使用第三方 Inventory 插件

可以使用 Ansible 支持的第三方 Inventory 插件，例如使用 GCP 或 OpenShift 插件来实现主机管理。在运行 Ansible 命令时，可以使用 `-i` 参数来指定要使用的 Inventory 插件，例如：

```shell
ansible all -i gcp_inventory -m ping
```

上述命令会使用名为 `gcp_inventory` 的 GCP Inventory 插件，向所有主机发送 ping 命令。

需要注意的是，在使用 Inventory 文件时，需要注意格式和内容的正确性，以避免出现错误。另外，还需要确保 Inventory 文件的权限设置正确，防止未授权用户访问敏感信息。



### 5、Playbook简介

#### 5.1.了解Playbook

Playbook 是 Ansible 的核心概念之一，用于定义一系列任务和操作的顺序，以实现特定的目标。每个 Playbook 都由一系列的任务组成，每个任务都描述了需要执行的操作和所需配置的目标主机或主机组。

以下是 Playbook 的一些基本组成部分：

1. `主机`：每个任务需要在哪些主机上执行，可以使用主机名、主机组名或通配符（例如 all）指定。
2. `变量`：每个任务可能需要使用的变量，包括任务参数和主机变量等。
3. `任务`：描述需要执行的操作，可以是标准模块或自定义模块、脚本等。
4. `处理器`：在执行完所有任务后，对主机进行统一处理的方法，例如发送通知、生成报告等。

一个基本的 Playbook 结构如下：

```yaml
- name: playbook_name		# Playbook 的名称
  hosts: target_hosts		 # 需要执行 Playbook 的目标主机或主机组
  vars:						# 需要使用的变量
    var1: value1
    var2: value2
  tasks:				
    - name: task_name		# 任务名称
      module_name:			# 需要使用的模块名称
        parameter1: value1		# 模块需要的参数和对应的值
        parameter2: value2
```

Playbook 使用 YAML 格式编写，格式要求严格，需要注意缩进、空格等细节。

使用 Playbook 可以将所需的操作和配置集中到一个文件中，便于维护和管理。可以使用 `ansible-playbook` 命令运行 Playbook：

```shell
ansible-playbook -i inventory.ini playbook.yml -C 
```

其中，`playbook.yml` 是 Playbook 文件的名称，`-i` 是ansible的inventory文件路径，`-C` 是模拟执行，没有实际执行任务，只是模拟执行，并且实际生产环境中的操作结果可能会受到许多因素的影响。

需要注意的是，使用 Playbook 执行任务时需要对任务顺序、操作结果、错误处理等细节进行仔细考虑，以避免因编写错误或执行错误导致的系统破坏或故障。

#### 5.2.Playbook演示	

下面是一个定义变量，使用变量，file 模块，copy 模块的简单使用：

~~~yaml
# 定义一个path变量
# 使用 file 模块在linux1上创建一个文件
# 使用 copy 模块将创建文件传送到所有的主机上。
- hosts: all
  vars:
    var1: "/root/test.txt" 
  tasks:			
    - name: task 1 create file		
      file:					
        path: "{{ var1 }}"
        state: touch
        mode: u=rw,g=r,o=r
      when: inventory_hostname == "linux1.skills.lan"
    - name: task 2 copy file
      copy:
        src: "{{ var1 }}"
        dest: "{{ var1 }}"
~~~



### 6、Ansible模块

#### 6.1.常用模块

Ansible有大量的模块可供使用，同时在执行Ansible Playbook时会有输出，其中包含了不同颜色的信息。这些不同颜色的信息代表不同的含义。

以下是常见的颜色及其含义：

绿色：表示任务成功完成。
红色：表示任务失败。
黄色：表示需要注意的信息或特别提醒。
蓝色：表示执行中的任务。
橙色：表示警告信息。

以下是一些常用模块的简介以及使用ad-hoc的演示：

1. command模块：执行一个命令并等待它完成。

   ~~~shell
   # command模块仅能执行可执行文件
   [root@linux1 ~]# ansible all -m command -a "hostname"
   linux2.skills.lan | CHANGED | rc=0 >>
   linux2.skills.lan
   linux3.skills.lan | CHANGED | rc=0 >>
   linux3.skills.lan
   # -m 如果不指定的话，默认就是command模块，演示：
   [root@linux1 ~]# ansible all -a "uname -a"
   linux2.skills.lan | CHANGED | rc=0 >>
   Linux linux2.skills.lan 5.14.0-162.6.1.el9_1.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Nov 18 02:06:38 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
   linux3.skills.lan | CHANGED | rc=0 >>
   Linux linux3.skills.lan 5.14.0-162.6.1.el9_1.x86_64 #1 SMP PREEMPT_DYNAMIC Fri Nov 18 02:06:38 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
   ~~~

   

2. shell模块：在远程主机上执行由字符串传递的命令，并返回其输出。

   ~~~shell
   # shell模块可以执行一切在目标主机上可能运行的命令，包括bash命令、管道、重定向、shell内置函数等
   [root@linux1 ~]# ansible all -m shell -a  "cat /etc/fstab | grep swap"
   linux3.skills.lan | CHANGED | rc=0 >>
   /dev/mapper/rl-swap     none                    swap    defaults        0 0
   linux2.skills.lan | CHANGED | rc=0 >>
   /dev/mapper/rl-swap     none                    swap    defaults        0 0
   ~~~

   

3. copy模块：将文件从控制机器复制到远程主机上。

   ~~~shell
   [root@linux1 ~]# touch test.txt
   [root@linux1 ~]# ansible all -m copy -a "src=/root/test.txt dest=/opt/ mode=0644"
   linux2.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
       "dest": "/opt/test.txt",
       "gid": 0,
       "group": "root",
       "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
       "mode": "0644",
       "owner": "root",
       "secontext": "system_u:object_r:usr_t:s0",
       "size": 0,
       "src": "/root/.ansible/tmp/ansible-tmp-1685141575.943086-13294-6525588277617/source",
       "state": "file",
       "uid": 0
   }
   linux3.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
       "dest": "/opt/test.txt",
       "gid": 0,
       "group": "root",
       "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
       "mode": "0644",
       "owner": "root",
       "secontext": "system_u:object_r:usr_t:s0",
       "size": 0,
       "src": "/root/.ansible/tmp/ansible-tmp-1685141575.9409778-13296-115905175599404/source",
       "state": "file",
       "uid": 0
   }
   ~~~

   

4. template模块：将模板文件的内容复制到远程主机上。

   ~~~shell
   [root@linux1 ~]# ansible all -m template -a "src=/etc/httpd/conf.d/ssl.conf dest=/root/http_ssl.conf"
   linux2.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "checksum": "c9e504bc3d6db87471086e7b8b375b9aeee22152",
       "dest": "/root/http_ssl.conf",
       "gid": 0,
       "group": "root",
       "md5sum": "28a15690fbec4d6c2d5ca4b0c63ab63c",
       "mode": "0644",
       "owner": "root",
       "secontext": "system_u:object_r:admin_home_t:s0",
       "size": 8720,
       "src": "/root/.ansible/tmp/ansible-tmp-1685141723.0213664-13733-29098652149974/source",
       "state": "file",
       "uid": 0
   }
   linux3.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "checksum": "c9e504bc3d6db87471086e7b8b375b9aeee22152",
       "dest": "/root/http_ssl.conf",
       "gid": 0,
       "group": "root",
       "md5sum": "28a15690fbec4d6c2d5ca4b0c63ab63c",
       "mode": "0644",
       "owner": "root",
       "secontext": "system_u:object_r:admin_home_t:s0",
       "size": 8720,
       "src": "/root/.ansible/tmp/ansible-tmp-1685141723.0731692-13735-128917087808771/source",
       "state": "file",
       "uid": 0
   }
   ~~~

   

5. apt/yum模块：在远程主机上安装和移除软件包。

   ~~~shell
   # 安装
   [root@linux1 ~]# ansible all -m yum -a "name=nginx state=present"
   linux2.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "msg": "",
       "rc": 0,
       "results": [
           "Installed: nginx-core-1:1.20.1-13.el9.x86_64",
           "Installed: nginx-1:1.20.1-13.el9.x86_64",
           "Installed: nginx-filesystem-1:1.20.1-13.el9.noarch"
       ]
   }
   linux3.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "msg": "",
       "rc": 0,
       "results": [
           "Installed: nginx-filesystem-1:1.20.1-13.el9.noarch",
           "Installed: nginx-core-1:1.20.1-13.el9.x86_64",
           "Installed: nginx-1:1.20.1-13.el9.x86_64"
       ]
   }
   # 移除
   [root@linux1 ~]# ansible all -m yum -a "name=nginx state=absent"
   linux3.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "msg": "",
       "rc": 0,
       "results": [
           "Removed: nginx-1:1.20.1-13.el9.x86_64"
       ]
   }
   linux2.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "msg": "",
       "rc": 0,
       "results": [
           "Removed: nginx-1:1.20.1-13.el9.x86_64"
       ]
   }
   ~~~

   

6. service模块：管理服务的状态（启动/停止/重启）。

   ~~~shell
   # 启动
   ansible all -m service -a "name=nginx state=started"
   # 停止
   ansible all -m service -a "name=nginx state=restarted"
   # 重启
   ansible all -m service -a "name=nginx state=stopped"
   ~~~

   

7. file模块：管理文件和目录的属性（权限，所有者，组，内容等）。

   ~~~shell
   [root@linux1 ~]# ansible all -m file -a "path=/opt/test.txt mode=u+rw,g-wx,o-rwx"
   linux2.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "gid": 0,
       "group": "root",
       "mode": "0640",
       "owner": "root",
       "path": "/opt/test.txt",
       "secontext": "system_u:object_r:usr_t:s0",
       "size": 0,
       "state": "file",
       "uid": 0
   }
   linux3.skills.lan | CHANGED => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": true,
       "gid": 0,
       "group": "root",
       "mode": "0640",
       "owner": "root",
       "path": "/opt/test.txt",
       "secontext": "system_u:object_r:usr_t:s0",
       "size": 0,
       "state": "file",
       "uid": 0
   }
   ~~~

   

8. ping模块：测试远程主机的可达性。

   ~~~shell
   [root@linux1 ~]# ansible all  -m ping
   linux2.skills.lan | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   linux3.skills.lan | SUCCESS => {
       "ansible_facts": {
           "discovered_interpreter_python": "/usr/bin/python3"
       },
       "changed": false,
       "ping": "pong"
   }
   ~~~

9. debug模块：通常用于程序开发和调试过程中，它可以通过输出变量的值或程序执行状态信息，帮助开发人员检测程序中的错误或证明其正确性。

   ~~~shell
   # 编写yaml文件
   vim q.yaml
   - hosts: all
     tasks:
     - name: print
       debug:
         msg: system {{ inventory_hostname }} has geteway {{ ansible_default_ipv4.gateway }}
   # 测试执行
   ansible-playbook q.yaml -C
   ~~~

   

10. Fetch模块：将文件从远程主机复制到控制机器上。

   ~~~shell
   ansible euler01 -m fetch -a "src=/root/123.txt dest=/root/"
   ~~~

   

11. Setup模块：收集被控制节点的信息。

    ~~~shell
    ansible euler01 -m setup
    ~~~

    

12. script脚本模块：在远程主机上执行 ansible 管理主机上的脚本。

    ~~~shell
    ansible euler01 -m script -a "/root/1.sh"
    ~~~

    

#### 6.2.自定义模块

您可以使用Python编写自己的Ansible模块，并使用它来扩展Ansible的功能，使其可以管理特定的应用程序和服务。

要创建自定义Ansible模块，请按照以下步骤：

1. 确定功能和参数

   首先，您需要确定您的模块将执行的功能以及需要的参数。可以通过编写“呈现的代码”来描述您的自定义模块的功能和参数。这些元数据应该以`ANSIBLE_METADATA`字典的形式存在，例如：

   ```python
   DOCUMENTATION = '''
   ---
   module: my_module
   short_description: This is a short description of my module
   description:
     - This is a longer description of the module
     - It can span multiple lines
   options:
     my_param:
       description:
         - This is a description of the parameter
       required: true
       type: str
   '''
   
   EXAMPLES = '''
   - name: Example usage
     my_module:
       my_param: foo
   '''
   
   RETURN = '''
   '''
   
   ANSIBLE_METADATA = {
       'metadata_version': '1.0',
       'status': ['preview'],
       'supported_by': 'community'
   }
   ```

2. 编写实现代码

   从上面的“rendered code”可以看出，Python自定义模块的开发与编写Python函数的开发非常相似。只需编写处理参数并执行所需功能的Python代码即可。

   ```python
   from ansible.module_utils.basic import AnsibleModule
   
   def main():
       module = AnsibleModule(
           argument_spec=dict(
               my_param=dict(required=True, type='str')
           ),
           supports_check_mode=True  # if check mode is supported
       )
   
       my_param = module.params['my_param']

       # Do something with my_param

       result = {"foo": "bar"}
   
       module.exit_json(**result)
   
   
   if __name__ == '__main__':
       main()
   ```

3. 使用模块

   要使用自定义Ansible模块，请将模块保存到Ansible控制器上的模块路径中，并在Playbook中使用`module_name`的值来引用您的模块。例如：

   ```python
   - name: Example usage
     my_module:
       my_param: foo
     register: result
   
   - debug:
       var: result
   ```

   在上面的示例中，我们在Playbook中使用`my_module`自定义模块，并将`my_param`参数设置为`foo`。

除了上面列出的步骤外，您还可以查看Ansible官方文档以获取更多详细信息和示例。

#### 6.3.扩展

`ansible-doc`是Ansible命令之一，用于查看Ansible模块的文档。

使用`ansible-doc`命令，您可以查看模块的用法、参数和示例。这将帮助您更好地理解模块的功能和如何使用它们来管理您的主机。

以下是一些使用`ansible-doc`的示例：

1. 查看模块的具体信息

   ```shell
   ansible-doc <module_name>
   ```

   `<module_name>`是要查看的模块名称。例如，要查看`yum`模块的信息，可以运行以下命令：

   ```shell
   ansible-doc yum
   ```

   运行此命令将显示`yum`模块的说明、用法、参数和示例。

2. 搜索模块

   ```shell
   ansible-doc -s <keyword>
   ```

   `<keyword>`是要搜索的关键字。例如，要寻找所有可以与`yum`模块一起使用的选项，可以运行以下命令：

   ```shell
   ansible-doc -s yum
   ```

   此命令将搜索所有与`yum`模块相关的文档，并列出包含`yum`关键字的所有模块和选项。

3. 查看特定版本的模块

   ```shell
   ansible-doc <module_name>@<version>
   ```

   `<version>`是要查看的模块版本。例如，要查看`yum`模块的特定版本，可以运行以下命令：

   ```shell
   ansible-doc yum@2.3.0
   ```

   运行此命令将显示`yum`模块2.3.0版本的说明、用法、参数和示例。

`ansible-doc`命令是一个非常有用的工具，可以帮助您更好地理解Ansible模块和如何使用它们来管理您的主机。



### 7、变量和条件语句

#### 7.1.变量和条件语句的概念

在Ansible中，变量是用于存储值的名称。您可以使用变量来存储各种值，如字符串、数字、布尔值、列表、字典等。变量可用于将值传递给Playbook中的任务，也可以在任务中进行计算和操作。

变量在Python中的使用非常相似。您可以在Ansible Playbook和Roles中定义变量，定义方式可以是临时变量，也可以是全局变量。变量可以在属于同一范围的其他变量和任务中共享。

条件语句是一种控制结构，用于根据条件的真假执行不同的代码块。在Ansible中，条件使用`when`关键字表示。`when`关键字后需要一个表达式，表达式的结果为布尔值。当表达式的结果为`true`时，与该条件关联的任务将被执行；当结果为`false`时，则跳过该任务。

#### 7.2.如何创建和使用变量和条件语句

以下是一些用于定义变量和条件语句的示例：

定义变量：

```
- name: Define variable
  set_fact:
    my_var: "my_value"
```

在上面的示例中，我们使用`set_fact`模块来定义一个名为`my_var`的变量，并将其值设置为`"my_value"`。

条件语句：

```yaml
- name: Run task when condition is true
  debug:
    msg: "This task will be executed only when my_var exists"
  when: my_var is defined

- name: Run task when condition is false
  debug:
    msg: "This task will be executed only when my_var does not exist"
  when: my_var is not defined
```

在上面的示例中，我们使用`debug`模块来演示如何应用条件语句。第一个任务仅在`my_var`存在时执行，第二个任务仅在`my_var`不存在时执行。

使用变量和条件语句可以帮助您在Ansible Playbook中更好地控制任务和操作。它们是Ansible中非常重要的概念，可以用于编写高效的、可重用的和具有条件性的Playbook。

### 8、循环和变量过滤器

#### 8.1.循环和变量过滤器的概述

循环 (Looping)：当需要重复执行某种操作或任务时，可以使用循环。Ansible 支持多种类型的循环，如 with_items 循环、with_list 循环、with_fileglob 循环等。循环中可以使用变量和模板。

变量过滤器（Variable Filters）：当需要在任务运行时对变量进行转换或过滤时，可以使用变量过滤器。变量过滤器提供一种简洁的方式来修改变量的输出。常见的变量过滤器有：upper/lower 模块、regex_replace 模块、map 模块、json_query 模块等。

#### 8.2.如何使用循环和变量过滤器

- item-with循环

~~~yaml
- hosts: all
  gather_facts: false
  tasks:
  - name: restart more service
  	systemd: 
  	  name: "{{ item }}"
  	  state: restarted
  	with_items:
  	  - nginx
  	  - crond
  	  - php-fpm
~~~

- 多变量循环

~~~yaml
- hosts: linux3.skills.lan
  tasks:
  - name: add user 
    user:
      name: "{{ item.user }}"
      uid: "{{ item.uid }}"
    with_items:
      - { user: 'user01', uid: '111' }
      - { user: 'user02', uid: '222' }
~~~

循坏：with_item(常用)  loop(功能多)

- 过滤器

在Ansible Playbook中使用变量过滤器，需要使用“{{ 变量名 | 过滤器名 }}”的形式。在过滤器名后面可以添加参数。

例如，假设有一个名为“my_variable”的变量，它的值是一个字符串，可以使用“lower”过滤器将它转换成小写形式：

```yaml
- name: 使用变量过滤器
  debug:
    var: my_variable | lower
```

如果要在“replace”过滤器中替换所有的“foo”字符串为“bar”

```yaml
- name: 使用变量过滤器
  debug:
    var: my_variable | replace('foo', 'bar')
```

如果要使用“join”过滤器将列表中的元素连接成一个字符串

```shell
- name: 使用变量过滤器
  debug:
    var: my_list | join(',')
```

如果要使用“map”过滤器对列表中的每个元素进行操作

```shell
- name: 使用变量过滤器
  debug:
    var: my_list | map('to_upper')
```

其中，“to_upper”是一个自定义的函数，它将列表中的字符串转换为大写形式。

Ansible变量过滤器是在Ansible Playbooks中使用的一种功能，用于对变量进行修改或处理。以下是一些常用的变量过滤器：

1. lower/upper：将变量字符串转换为小写/大写形式。

2. replace：用新的字符串替换变量字符串中匹配的旧字符串。

3. trim：去掉变量字符串中的空格。

4. regex_replace：用正则表达式替换变量字符串中匹配的部分。

5. join：将列表中的元素连接成一个字符串。

6. map：对列表中的每个元素进行变换，返回新的列表。

7. default：如果变量为未定义，则返回默认值。

8. to_yaml/to_json：将变量转换为YAML/JSON格式。

9. subset：从列表中选择一个子集。

10. dict2items：将字典变量转换为列表。

### 9、Role和任务调度

#### 9.1.Role的概念和结构

在 Ansible 中， roles 模块主要用于组织剧本文件以及对它们进行复用。使用 roles 可以将 playbook 中的任务按不同的组件、服务、配置等进行分开管理，方便模块化、可重复利用。

以下是一个简单的示例：

1. 首先你需要创建一个名为 `roles` 的目录，它应该在当前目录的同级目录中。

2. 在 `roles` 目录下创建你的角色(`role`)目录。例如 `webserver`。

3. 在你的角色目录中，通常有以下目录结构：

```
├── README.md
├── defaults
│   └── main.yml
├── files
│   ├── conf.d
│   └── nginx.conf
├── handlers
│   └── main.yml
├── meta
│   └── main.yml
├── tasks
│   ├── main.yml
│   └── extras.yml
├── templates
│   └── nginx.j2
└── vars
    └── main.yml
```

上面的目录结构中，各目录的主要作用为：

- `README.md` 文件用于描述这个角色的功能以及如何使用它。
- `defaults` 目录中的 `main.yml` 文件用于包含默认变量值，若在 playbook 中定义了同名变量，则 playbook 中变量会被覆盖。
- `files` 目录中放置一些不需要使用模板的文件，比如一些配置文件、证书，Ansible 会将这些文件复制到远程主机。
- `handlers` 目录中的 `main.yml` 文件，是一些由 Tasks 中的任务定义的通知（notify）操作。
- `meta` 目录中的 `main.yml` 文件是用于定义角色之间的依赖关系和一些其他元数据的，比如作者、证书、支持的平台等信息。
- `tasks` 目录中的 `main.yml` 文件是定义角色主要的步骤，该文件中会包含一些 tasks 模块使用的 Ansible 任务。
- `templates` 目录中的文件是与 `files` 目录类似的，但是需要使用 Jinja2 模板语言来进行计算和处理。对于 Web 服务器这样的服务，这是通常的配置方式。.
- `vars` 目录中主要用来定义这个角色的变量 。

roles 目录结构并不是固定不变的，你可以自定义该目录下的目录结构，只要按照自己的规范，然后在 playbook 中正确指定角色目录就可以了。除此之外，可以使用其他一些工具（如 Molecule，Ansible-Galaxy 等）来帮助你规范化的使用 Ansible roles。

#### 9.2.如何创建和管理Roles	

##### 部署lnmp

~~~shell
# roles流程
- 基本配置
- 配置nginx，php，db，nfs
- 启动
~~~

###### 1)lnmp环境目录

~~~shell
# 创建lnmp环境目录
mkdir /opt/roles/lnmp/ -p
cd /opt/roles/lnmp
mkdir -p {basic,nginx,db,php}/{files,tasks,handlers,templates} group_vars/all/
tree -F
.
├── basic/
│   ├── files/
│   ├── handlers/
│   ├── tasks/
│   └── templates/
├── db/
│   ├── files/
│   ├── handlers/
│   ├── tasks/
│   └── templates/
├── group_vars/
│   └── all/
└── nginx/
    ├── files/
    ├── handlers/
    ├── tasks/
    └── templates/
~~~



###### 2)配置基本环境

~~~shell
### 防火墙
### yum 源
### ssh
### 创建目录

# repo file
[root@linux1 lnmp]# vim basic/files/rocky.repo
[baseos]
name=Rocky Linux $releasever - BaseOS
baseurl=file:///media/BaseOS
gpgcheck=0

[appstream]
name=Rocky Linux $releasever - AppStream
baseurl=file:///media/AppStream
gpgcheck=0

# ssh
ssh-copy-id <client-ip>
# 编写 basic tasks
vim basic/tasks/main.yml
### 防火墙
- name: stop firewall
  systemd:
    name: firewalld
    state: stopped 
    enabled: no
### 配置yum源
- name: config yum repo
  copy:
    src: basic/files/rocky.repo
    dest: /etc/yum.repos.d/rocky.repo
- name: umount images
  shell: cat /etc/fstab | grep  media || echo "/dev/cdrom      /media  iso9660 defaults        0 0" >> /etc/fstab 
- name: umount images
  shell: mount -a
### 创建目录 /server/tools /server/scripts
- name: create directory
  file: 
    path: /server/tools
    state: directory
    mode: '0755'
- name: create directory
  file: 
    path: /server/scripts
    state: directory
    mode: '0755'
~~~



###### 3)配置nginx

~~~shell
### 安装nginx
### 配置文件
### 站点目录
### 启动

# 准备nginx配置文件
[root@linux1 lnmp]# vim nginx/templates/nginx.conf.j2
user {{ web_user }};
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 4096;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80;
        listen       [::]:80;
        server_name  _;
        root         /usr/share/nginx/html;
        include /etc/nginx/default.d/*.conf;
        error_page 404 /404.html;
        location = /404.html {
        }
        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
        }
    }
}
# 准备nginx站点目录
[root@linux1 lnmp]# echo "hellolnmp" >  nginx/files/index.html

# 定义变量
[root@linux1 lnmp]# vim group_vars/all/main.yml 
web_user: nginx

# 编写 nginx tasks
vim nginx/tasks/main.yml
### 安装nginx
- name: install nginx
  yum:
    name: nginx
    state: latest
### 配置文件
- name: config nginx
  template:
    src: nginx.conf.j2
    dest: /etc/nginx/nginx.conf
    backup: yes
### 站点目录
- name: index file
  copy:
    src: ./nginx/files/index.html
    dest: /usr/share/nginx/html/index.html
### 启动
- name: start nginx service
  systemd:
    name: nginx
    state: restarted
    enabled: yes
~~~

###### 4)配置php

~~~shell
### 安装php，可以使用本地包进行安装，这里就不怎么麻烦了，直接用yum安装了
### 发送配置文件
### 启动

# php配置文件模板
[root@linux1 lnmp]# vim php/templates/www.conf.j2
[www]
user = {{ web_user }}
group = {{ web_user }}
listen = /run/php-fpm/www.sock
listen.acl_users = apache,nginx
listen.allowed_clients = 127.0.0.1
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
slowlog = /var/log/php-fpm/www-slow.log
php_admin_value[error_log] = /var/log/php-fpm/www-error.log
php_admin_flag[log_errors] = on
php_value[session.save_handler] = files
php_value[session.save_path]    = /var/lib/php/session
php_value[soap.wsdl_cache_dir]  = /var/lib/php/wsdlcache


# 编写 php tasks
[root@linux1 lnmp]# vim php/tasks/main.yml 
### 安装php
- name: install php
  yum:
    name: php
    state: latest
### 发送配置文件
- name: copy config
  template:
    src: www.conf.j2
    dest: /etc/php-fpm.d/www.conf
    backup: yes
### 启动
- name: start php
  systemd:
    name: php-fpm
    state: restarted
    enabled: yes
~~~

###### 5)部署db

~~~shell
### 安装
### 启动数据库
### 创建数据库
### 创建用户

# 编写 db tasks
[root@linux1 lnmp]# vim db/tasks/main.yml
### 安装mysql-python
- name: install python
  yum:
    name: MySQL-python
    state: latest
### 安装mariadb
- name: install mariadb
  yum:
    name: mariadb-server
    state: latest
### 启动数据库
- name: start datbase
  systemd:
    name: mariadb
    state: restarted
    enabled: yes
### 创建数据库
- name: create datbase
  mysql_db:
    name: wordpress
    state: present
### 创建用户
- name: create user
  mysql_user:
    name: xiao
    login_user: root
    login_password: y
    password: 123456
    host_all: yes
    priv: '*.*:ALL'
    state: persent
~~~

###### 6)部署wordpress

~~~shell
#### 安装
#### 传送代码
[root@linux lnmp]# ll
total 19008
drwxr-xr-x. 6 root root       65 Jun 20 10:43 basic
drwxr-xr-x. 6 root root       65 Jun 20 10:43 db
drwxr-xr-x. 3 root root       17 Jun 20 10:43 group_vars
drwxr-xr-x. 6 root root       65 Jun 20 10:43 nginx
drwxr-xr-x. 6 root root       65 Jun 20 10:43 php
-rw-r--r--. 1 root root 19462197 Apr 19  2022 wordpress-5.9.2-zh_CN.tar.gz   		# 传输的文件
[root@linux lnmp]# tar -xf wordpress-5.9.2-zh_CN.tar.gz 
~~~



#### 9.3.如何使用任务调度

~~~shell
[root@linux lnmp]# vim role.yaml
---
- hosts: all
  remote_user: root

  roles:
    - basic
    - db
    - nginx
    - php

ansible-playbook role.yaml
~~~



### 10、文件和目录操作

##### 10.1.文件和目录操作的模块和选项

在 Ansible 中，文件和目录操作是经常用到的操作之一。一些常见的 Ansible 模块和选项用于文件和目录操作包括：

1. `file` 模块

用于文件和目录的通用操作，支持创建目录、创建删除、改变所有权、权限、更改有效用户/组以及获取文件和目录属性等操作。

常用选项：

- `path`：要操作的文件或目录的路径。
- `state`：文件或目录的状态，可以是 `absent`、`directory`、`file` 等。
- `owner`：设置文件或目录的所属用户。
- `group`：设置文件或目录的归属用户组。
- `mode`：设置文件或目录的权限，如“0644”。

2. `copy` 模块

用于复制文件和目录，可以复制本地和远程文件。

常用选项：

- `src`：源文件路径，可以是本地文件或远程文件。
- `dest`：目标文件路径，必须使用绝对路径，并且是文件，如果不存在则被创建。
- `backup`：是否备份目标文件，默认为“no”。
- `force`：当安全模式为打开时，强制复制文件。

3. `template` 模块

将源文件模板复制到目标文件中，并根据相应的变量进行渲染。

常用选项：

- `src`：模板相对路径。
- `dest`：目标文件路径，必须使用绝对路径，并且是文件，如果不存在则被创建。
- `mode`：设置文件或目录的权限。
- `backup`：是否备份目标文件，默认为“no”。
- `vars`：传递给模板的变量。

4. `fetch` 模块

从目标远程主机上提取文件或目录。

常用选项：

- `src`：远程目标文件路径
- `dest`：提取后文件本地保存的路径
- `flat`：使用指定目录将提取保存的文件和目录打包

以上是 Ansible 中一些常见的用于文件和目录操作的模块和选项，你可以根据具体的需求使用它们来执行文件和目录操作。

##### 10.2.如何用Ansible创建和修改文件和目录

以file模块为例：

1. 使用ad-hoc命令行

   ~~~shell
   # 创建目录：
   ansible all -m file -a 'dest=/path/to/directory state=directory'
   
   # 创建文件：
   ansible all -m file -a 'dest=/path/to/file state=touch'
   
   # 修改目录：
   ansible all -m file -a 'dest=/path/to/file_or_directory owner=user_name group=group_name'
   ansible all -m file -a 'dest=/path/to/file mode=0644'
   ~~~

   

2. 编写剧本

   ~~~yaml
   # 创建目录：
   - name: Create directory
     become: yes
     file:
       path: /path/to/directory
       state: directory
   # 创建文件：
   - name: Create file
     become: yes
     file:
       path: /path/to/file
       state: touch
   # 修改所有权和权限：
   - name: Change owner and group
     become: yes
     file:
       path: /path/to/file_or_directory
       owner: user_name
       group: group_name
   
   - name: Change permissions
     become: yes
     file:
       path: /path/to/file_or_directory
       mode: "0644"
   ~~~

   

### 11、命令和服务控制

##### 11.1.使用系统命令

- 使用系统命令可以使用 command 模块和 shell模块来实现。

  1. command模块：该模块适用于执行非交互式的 shell 命令，一般是简单的单条命令。

     ~~~yaml
     - name: 执行whoami命令获取当前用户
       command: whoami
       register: result
     - debug:
         var: result.stdout
     ~~~

     

  2. shell模块：如果需要在远程主机上执行复杂的命令，比如需要使用管道、重定向或者需要变量等，此时可以使用 `shell` 模块。

     ~~~shell
     - name: 执行 curl 命令获取指定URL的内容并保存到文件中
       shell: "curl https://example.com -o /path/to/file.txt"
     ~~~

注意:

- command 和 shell 模块都可以使用 register 参数将命令的执行结果保存到一个变量中，以供后续步骤使用。

- 使用 shell 模块时，需要注意命令本身是否需要转义，这样可以避免字符串插值问题。

- 在使用 shell 模块执行长时间运行的命令时，可以配合 async 和 poll 参数，实现异步运行和检查任务执行状态。

##### 11.2.启用、关闭和重启服务

在 Ansible 中可以使用 `systemd` 模块来启用、关闭和重启服务。下面是一些使用 `systemd` 模块来管理服务的实例：

1. 启用服务：
```yaml
- name: start service
  become: yes
  systemd:
    name: service_name
    enabled: yes
    state: started
```
- `name`: 要启用或停用的服务名称。
- `enabled`: 是否在启动时启用该服务。
- `state`: 服务的状态，即 `started` 运行或 `stopped` 停止。

2. 停用服务：
```yaml
- name: stop service
  become: yes
  systemd:
    name: service_name
    enabled: no
    state: stopped
```

3. 重启服务：
```yaml
- name: restart service
  become: yes
  systemd:
    name: service_name
    state: restarted
```

同时，还有一些 `systemd` 模块的其他常用选项：

- `enabled`: 是否在启动时启用该服务。
- `state`: 服务的状态，即 started 运行或 stopped 停止。
- `daemon_reload`: 是否重新加载 systemd 守护进程。
- `no_block`: 在启动或停止服务时使用该参数，减少等待时间。
- `timeout`: 启动或停止服务时的超时时间。

除了 `systemd` 模块，还可以使用其他模块来管理服务，如 `service` 模块和 `service_facts` 模块。但是推荐使用 `systemd` 模块，因为它是更现代的服务管理器，被越来越多的发行版所采用和支持。



### 12、克隆Git

##### 12.1.克隆Git目录

在 Ansible 中，可以使用 `git` 模块来克隆 Git 代码库。

示例：

```yaml
- name: 克隆 Git 代码库
  git:
    repo: https://github.com/username/repo.git
    dest: /path/to/local/directory
```

- `repo`: 要克隆的 Git 代码库的 URL。
- `dest`: 将代码库克隆到本地的目录路径。

可以通过添加以下选项来进一步配置 `git` 模块：

- `branch`: 要克隆的 Git 分支，默认为 `master`。
- `update`: 如果目录已经存在，是否更新 Git 代码库，默认为 `yes`。
- `version`: 要克隆的 Git 代码库的特定版本。

可以根据需要设置这些选项来满足特定的需求。

除了 `git` 模块，还可以使用其他模块来管理代码库，如 `subversion` 模块和 `mercurial` 模块。但是 `git` 是一种最常用的版本控制系统，已被广泛采用，而且 `git` 模块提供了许多选项和配置项，因此在绝大多数情况下都可以使用 `git` 模块来管理 Git 代码库。

##### 12.2.使用Git库更新多节点

在 Ansible 中，可以使用 `git` 模块来更新多个节点的 Git 代码库。可以通过 Ansible 的 `hosts` 参数指定多台主机，执行 playbook 时将同时在这些主机上应用任务。

示例：

```yaml
- name: update git database
  hosts: all
  become: yes

  tasks:
    - name: clone git codebase
      git:
        repo: https://github.com/username/repo.git
        dest: /path/to/local/directory
        version: master
    - name: pull git codebase
      git:
        repo: https://github.com/username/repo.git
        dest: /path/to/local/directory
        version: master
        update: yes
```

在上述 playbook 中，我们将两个任务分别用于克隆和拉取 Git 代码库。通过在主机群组 `webservers` 中指定多个主机，即可在这些主机上同时应用任务。

除了使用 `hosts` 参数，还可以使用 Ansible 的 `ansible-playbook` 命令的 `--limit` 和 `--extra-vars` 参数指定要更新的主机，以及执行 playbook 时提供视图特定的变量。

总之，使用 `git` 模块和 Ansible 的多节点特性，可以轻松地在多个节点上更新 Git 代码库，提高工作效率。



### 13、使用Vault

#### 13.1.Secret Management

使用 ansible-vault，可以非常方便地加密和管理敏感数据，提高安全性和可管理性。需要注意的是，ansible-vault 并不能确保绝对安全，因此应该慎重配置和使用，以最大限度地保护敏感信息的保密性和机密性。

ansible-vault 支持以下常用命令：

- `create`: 创建一个新的加密文件；

- `encrypt`: 加密一个已经存在的文件，或者一个明文；

- `decrypt`: 对一个加密的文件进行解密操作；

- `view`: 查看一个加密的文件；

- `edit`: 编辑一个加密的文件；

- `rekey`: 更改加密文件所使用的密钥；

  

使用ansible-vault的示例命令：

~~~shell
# 在配置文件中指定密码文件，如果指定，则创建新的加密文件不弹出密码，使用文件中的内容当做密码,当然，查看加密文件，也不需要指定密码。
vim /etc/ansible/ansible.cfg

[defaults]
vault_password_file=/path/password_file

# 以下示例没有配置这条配置参数
# 创建加密文件的帮助
[root@linux1 vault]# ansible-vault create --help
usage: ansible-vault create [-h] [--encrypt-vault-id ENCRYPT_VAULT_ID] [--vault-id VAULT_IDS]
                            [--ask-vault-password | --vault-password-file VAULT_PASSWORD_FILES] [-v]
                            [file_name ...]

positional arguments:
  file_name             Filename

optional arguments:
  -h, --help            show this help message and exit
  --encrypt-vault-id ENCRYPT_VAULT_ID
                        the vault id used to encrypt (required if more than one vault-id is provided)
  --vault-id VAULT_IDS  the vault identity to use
  --ask-vault-password, --ask-vault-pass
                        ask for vault password
  --vault-password-file VAULT_PASSWORD_FILES, --vault-pass-file VAULT_PASSWORD_FILES
                        vault password file
  -v, --verbose         Causes Ansible to print more debug messages. Adding multiple -v will increase the verbosity, the builtin
                        plugins currently evaluate up to -vvvvvv. A reasonable level to start is -vvv, connection debugging
                        might require -vvvv.
# 创建加密文件
[root@linux1 vault]# ansible-vault create test.file
New Vault password: 
Confirm New Vault password: 
# 输完密码后，会自动进入vim编辑模式，正常使用即可。

# 查看刚加密文件，会发现已经加密了。
[root@linux1 vault]# cat test.file
$ANSIBLE_VAULT;1.1;AES256
39616666316236333435366666656563636432656162666430666332323132343961356232626439
3865633866633264636438323130396164656434313338340a316534346363633930373432373766
38366338343465353834623734386166656564653234656336313335663735636130346261623934
3765626432333833380a616637643762336463323935333533313064666437363637333665393539
6436

# 修改已加密的文件的秘钥
[root@linux1 vault]# ansible-vault rekey test.file
Vault password: 			# 原密码
New Vault password: 			# 新密码
Confirm New Vault password: 	# 确认密码
Rekey successful

# 编辑加密文件
[root@linux1 vault]# ansible-vault edit test.file
Vault password: 	# 输入密码
# 查看加密文件，输入密码后，将会看见文件内的明文内容
[root@linux1 vault]# ansible-vault view test.file
Vault password: 
file_name : "hello world!"

## 解密
# 解密文件并保留原加密文件
[root@linux1 vault]# ansible-vault decrypt test.file --output=test1.file
[root@linux1 vault]# cat test1.file
file_name : "hello world!"
[root@linux1 vault]# ansible-vault decrypt test.file
Vault password: 
Decryption successful
[root@linux1 vault]# cat test.file
file_name : "hello world!"
~~~



##### 小技巧

--vault-id是一个可以让您在命令行上指定Ansible Vault密码的选项。您可以使用--vault-id选项来避免在播放期间手动输入密码。这个选项对于需要多个Vault密码或定期更改密码的情况非常有用。 

该选项需要一个或多个参数，例如：
```shell
ansible-playbook playbook.yml --vault-id mypassword@path/to/vault_password_file
```

在以上命令中，“mypassword”是Vault密码，“path/to/vault_password_file”是包含Vault密码的文件的路径。多个--vault-id选项可以用于多个Vault密码。

注意，如果您同时在--vault-id和--vault-password-file上指定密码，将使用--vault-id上指定的密码。建议使用其中一个选项来指定Vault密码。如果同时使用两个选项指定Vault密码，不保证使用哪一个。



#### 13.2.在仓库中加密选定内容

在Ansible中，可以使用vault来加密仓库中的选定内容以保护敏感数据。要在Ansible仓库中加密选定内容，可以按照以下步骤进行操作：

1. 创建一个加密文件：使用ansible-vault命令来创建一个加密文件，可以执行以下命令：`ansible-vault create <filename>`。例如，以下命令将创建一个名为“secrets.yml”的加密文件：

   ```shell
   ansible-vault create secrets.yml
   ```

2. 将要加密的仓库内容复制到文件中。

   ```shell
   ---
   # Content in plain text
   db_user: username
   db_password: secret_password
   ```
   例如，以上内容中的`db_password`是要加密的信息。

3. 保存文件并退出编辑器。当您保存并退出编辑器时，AnsibleVault会要求您输入加密文件的密码。

4. 引用加密文件中的内容。在playbook或task中引用加密文件中的内容时，需要在vars_files中指定加密文件路径并指定其密码，例如：

   ~~~yaml
   - hosts: client
     vars_files:
       - ./secrets.yml
     tasks:
       - name: 执行一些操作
         debug:
           msg: "用户名：{{ db_user }}, 密码：{{ db_password }}"
   ~~~

   在此示例中，我们将加密文件`secrets.yml`的路径放在了`vars_files`关键字中。Ansible随后会要求您输入该文件的密码，以便解密其中的内容。注意：在使用加密文件时，确保密码的安全存储，不要与其他人分享。
   
5. 执行剧本：

   ~~~shell
   [root@linux1 vault]# ansible-playbook test.yaml  --ask-vault-pass
   Vault password: 		# 输入密码
   
   PLAY [client] ********************************************************************************************************************
   
   TASK [Gathering Facts] ***********************************************************************************************************
   ok: [linux2.skills.lan]
   ok: [linux3.skills.lan]
   
   TASK [执行一些操作] **************************************************************************************************************
   ok: [linux2.skills.lan] => {
       "msg": "用户名：username, 密码：secret_password"
   }
   ok: [linux3.skills.lan] => {
       "msg": "用户名：username, 密码：secret_password"
   }
   
   PLAY RECAP ***********************************************************************************************************************
   linux2.skills.lan          : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
   linux3.skills.lan          : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0 
   ~~~

以上便是在Ansible仓库中使用vault加密选定内容的步骤。

### 14、主机/节点监控和告警

#### 系统监控、性能分析部署

#### 告警和警报的设置



### 15、最佳实践

Ansible最佳实践

如何保护Ansible安全



### 16、Ansible Tower介绍

Tower的简介和架构

如何使用Ansible Tower

