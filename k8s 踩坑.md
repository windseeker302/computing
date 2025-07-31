## 1、kubelet 未启动

get pod 的时候发现，kube-proxy 两台节点的 pod 删都删不了，去两台节点上使用运行时环境，发现一个容器都未启动，然后找了找原因发现是 kubelet 未启动。

~~~shell
[root@k8s-master ~]# kubectl get pod -A                                                                                                                                                                         
NAMESPACE     NAME                                       READY   STATUS        RESTARTS        AGE                                                                                                              
default       busbox                                     1/1     Terminating   0               147m                                                                                                             
kube-system   calico-kube-controllers-7ddc4f45bc-g27mc   1/1     Terminating   0               4h58m                                                                                                            
kube-system   calico-kube-controllers-7ddc4f45bc-rpxvs   1/1     Running       1 (3m14s ago)   132m                                                                                                             
kube-system   calico-node-tg8pp                          0/1     Running       1 (3m14s ago)   133m                                                                                                             
kube-system   calico-node-tgx6p                          1/1     Running       0               4h58m                                                                                                            
kube-system   calico-node-vsvbx                          1/1     Running       0               3h24m                                                                                                            
kube-system   coredns-857d9ff4c9-7b4b4                   1/1     Running       1 (3m9s ago)    132m                                                                                                             
kube-system   coredns-857d9ff4c9-k62xg                   1/1     Terminating   0               7h29m                                                                                                            
kube-system   coredns-857d9ff4c9-kbgtd                   1/1     Terminating   0               7h29m                                                                                                            
kube-system   coredns-857d9ff4c9-z6224                   1/1     Running       1 (3m9s ago)    132m                                                                                                             
kube-system   etcd-k8s-master                            1/1     Running       2 (3m14s ago)   7h30m                                                                                                            
kube-system   kube-apiserver-k8s-master                  1/1     Running       2 (3m13s ago)   7h30m                                                                                                            
kube-system   kube-controller-manager-k8s-master         1/1     Running       2 (3m14s ago)   7h30m                                                                                                            
kube-system   kube-proxy-6n4kw                           1/1     Running       2 (3m14s ago)   7h29m                                                                                                            
kube-system   kube-proxy-9lmmq                           1/1     Terminating   0               5h17m                                                                                                            
kube-system   kube-proxy-h2t8r                           1/1     Terminating   0               5h17m                                                                                                            
kube-system   kube-scheduler-k8s-master                  1/1     Running       2 (3m14s ago)   7h30m 
~~~

```shell
# 清理不再使用的 Docker 对象，包括镜像、容器、网络以及卷
docker system prune
```