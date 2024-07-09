# KubeVirt

## 一、虚拟化介绍

虚拟化分类：

- paravirtualization（半虚拟化）
- Partial Virtualization（部分虚拟化）
- Full Virtualization（全虚拟化）

kubevirt使用到的虚拟化技术：

- libvirtd：libvirtd是一个守护进程，本质上是对不同的 hypervisor（如qemu，Xen，LXC等）系统操作做了封装，并且对外提供了统一的 api 接口，可以把 libvirt 看做是一个统一抽象层。
- qemu：QEMU 是一套由法布里斯·贝拉 (Fabrice Bellard ）所编写的以 GPL 许可证分发源码的模拟处理器软件，在 GNU/Linux 平台上使用广泛。
- kvm：全程是 Kernel-Based Virtual Machine，对应的是 2007 年加进 linux 2.26.20 的内核的 kvm.ko 模块。

> qemu 虚拟机是可以独立运行的，但是由于 qemu 完全是软件模拟硬件，在速度上比真正的硬件要差不少。为了解决速度问题， qemu 允许使用 kvm 作为加速器，以便可以使用物理 CPU 虚拟化扩展。当 qemu 独立运行时，qemu 模拟 cpu 、内存和硬件资源；当 kvm 和 qemu 协同工作时， kvm 负责 cpu 和内存的访问，而 qemu 则模拟其它硬件资源。

## 二、KubeVirt 是什么？

kubevirt 是一个容器方式运行虚拟机的项目，用于实现在 k8s 集群中像对 Pod 进行编排部署一样管理虚拟机。类似的产品：OpenShift CNV。



## 三、下载

下载地址：https://github.com/kubevirt/kubevirt/releases

~~~shell
# 安装
kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/v1.2.2/kubevirt-operator.yaml

kubectl create -f https://github.com/kubevirt/kubevirt/releases/download/v1.2.2/kubevirt-cr.yaml

wget https://github.com/kubevirt/kubevirt/releases/download/v1.3.0-beta.0/virtctl-v1.3.0-beta.0-linux-amd64
chmod +x virtctl-v1.3.0-beta.0-linux-amd64
mv virtctl-v1.3.0-beta.0-linux-amd64 /usr/local/bin/virtctl
virtctl version
~~~



## 四、创建 VirtualMachine

vm（VirtualMachine）是在 vmi 基础之上更完善的逻辑控制，可以认为是控制器；vmi（VirtualMachineInstance）是虚拟机实例。vm 与 vmi 之间的关系类似于 deployment 与 pod 之间的关系。

### 4.1、准备 VM 部署描述文件

~~~shell
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  annotations:
    kubevirt.io/latest-observed-api-version: v1
    kubevirt.io/storage-observed-api-version: v1
  creationTimestamp: "2023-09-01T00:25:41Z"
  finalizers:
  - kubevirt.io/virtualMachineControllerFinalize
  generation: 1
  name: testvm
  namespace: default
  resourceVersion: "12691"
  uid: d5509381-5f21-438b-9440-cb7c9ee82b37
spec:
  running: false
  template:
    metadata:
      creationTimestamp: null
      labels:
        kubevirt.io/domain: testvm
        kubevirt.io/size: small
    spec:
      architecture: amd64
      domain:
        devices:
          disks:
          - disk:
              bus: virtio
            name: containerdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
          interfaces:
          - masquerade: {}
            name: default
        machine:
          type: q35
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
      - containerDisk:
          image: quay.io/kubevirt/cirros-container-disk-demo
        name: containerdisk
      - cloudInitNoCloud:
          userDataBase64: SGkuXG4=
        name: cloudinitdisk
~~~



