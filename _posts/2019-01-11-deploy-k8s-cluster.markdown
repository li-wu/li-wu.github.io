---
title: Deploy a Kubernetes Cluster using kubeadm on CentOS
date: 2019-01-11
categories: Kubernetes
---

Kubernetes集群的搭建方法和介绍很多，本文主要介绍利用kubeadm来搭建一个三个节点的Kubernetes集群。

## 软件版本

| Name         | Version |
| ------------ | ------- |
| CentOS       | 7       |
| Linux Kernel | 4.20    |
| Docker-CE    | 18.06   |
| Kubernetes   | 1.13.1  |

## 节点规划

一主两从的三个节点的集群。

| Host Name   | IP            | Role   |
| ----------- | ------------- | ------ |
| k8s-master  | 10.66.128.205 | Master |
| k8s-node-01 | 10.66.131.225 | Node   |
| k8s-node-02 | 10.66.129.162 | Node   |

其中所有节点都需要安装的组件：

* Docker

  > 可以参考上一篇文章

* kubelet

  * 用来处理Master节点下发到本节点的任务,管理Pod和其中的容器

* kubeadm

  * 用来初始化集群

* kubectl

### 每台机器的准备工作

* 确保Docker运行状态正常并且没有错误；

  * `systemctl status docker`

* 关闭防火墙

  ```shell
  systemctl disable firewalld.service 
  systemctl stop firewalld.service
  ```

* 禁用SELINUX

  ```shell
  setenforce 0
  
  vi /etc/selinux/config
  SELINUX=disabled
  ```

* 关闭swap

  ```shell
  swapoff -a
  ```

* 对每台机器分别设置节点主机名

  ```shell
  hostnamectl --static set-hostname  k8s-master
  hostnamectl --static set-hostname  k8s-node-01
  hostnamectl --static set-hostname  k8s-node-02
  ```

* 将主机名/IP加入`hosts`解析

  将下述内容加入`/etc/hosts`:

  ```
  10.66.128.205 k8s_master
  10.66.131.225 k8s_node_01
  10.66.129.162 k8s_node_02
  ```

* `kubelet`, `kubeadm`, `kubectl` 组件安装

  * 准备repo

    ```shell
    cat>>/etc/yum.repos.d/kubrenetes.repo<<EOF
    [kubernetes]
    name=Kubernetes Repo
    baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
    gpgcheck=0
    gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
    EOF
    ```

  * 安装

    ```shell
    yum install -y kubelet kubeadm kubectl
    systemctl enable kubelet && systemctl start kubelet
    ```

### Master节点配置

#### 准备Docker Image

```shell
docker pull k8s.gcr.io/kube-apiserver:v1.13.1
docker pull k8s.gcr.io/kube-controller-manager:v1.13.1
docker pull k8s.gcr.io/kube-scheduler:v1.13.1
docker pull k8s.gcr.io/kube-proxy:v1.13.1
docker pull k8s.gcr.io/pause:3.1
docker pull k8s.gcr.io/etcd:3.2.24
docker pull k8s.gcr.io/coredns:1.2.6
docker pull quay.io/coreos/flannel:v0.10.0-amd64
```

> 注意如果不能pull，请从国内的镜像下载然后重命名

在Master节点上执行如下命令初始化集群：

```
kubeadm init --kubernetes-version=v1.13.1 --apiserver-advertise-address 10.66.128.205 --pod-network-cidr=10.244.0.0/16
```

- `--kubernetes-version`: 指定 kubernetes版本，这里用1.13.1。
- `--apiserver-advertise-address`：用于指定使用 Master的哪个network interface进行通信，若不指定，则 kubeadm会自动选择具有默认网关的 interface。
- `--pod-network-cidr`：用于指定Pod的网络范围。该参数使用依赖于使用的网络方案，本文将使用经典的flannel网络方案。具体文档请参考[https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#instructions](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/#instructions)

如果上述命令执行成功，会有如下提示：

```shell
Your Kubernetes master has initialized successfully!
```

> 如果配置失败，需要用`kubeadm reset` 来重置。排除错误之后重新执行上述初始化集群的命令。

#### 配置kubectl

```shell
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
source /etc/profile 
echo $KUBECONFIG
```

#### 安装Pod网络

Kubernetes支持很多网络解决方案，这里选择flannel。

设置系统参数：

```shell
sysctl net.bridge.bridge-nf-call-iptables=1
```

执行如下命令创建flannel网络：

```shell
kubectl apply -f kube-flannel.yaml
```

> `kube-flannel.yaml`文件[在此](https://gist.github.com/li-wu/e06cb332b47c4e07fa403becbc8d7556)

检查CoreDNS Pod是否正常运行：

```shell
kubectl get pods --all-namespaces -o wide
```

 同时可以查看Master节点是否已经就绪：

```shell
kubectl get nodes
```

#### 加入Node节点

在Master主机上执行如下命令获取`join`的命令：

```
kubeadm token create --print-join-command
```

会打印出从Node节点上加入集群的命令，类似下面这样：

```shell
kubeadm join --token 5d2dc8.3e93f8449167639b 10.66.128.205:6443 --discovery-token-ca-cert-hash sha256:44a68d4a2c2a86e05cc0d4ee8c9c6b64352c54e450021331c483561e45b34388
```

#### 验证集群状态

* 查看节点状态

  ```shell
  kubectl get nodes
  ```

* 查看所有Pod的状态

  ```shell
  kubectl get pods --all-namespaces -o wide
  ```

#### Restart after OS reboot

如果系统重启了而Kubernetes没有启动重启(kubelet会负责Kubernetes的重启)，需要重新启动`kubelet`。

先看看`kubelet`没有启动的原因：

```shell
journalctl -xeu kubelet
```

启动`kublet`，如果`systemctl enable kubelet`失败，先要`system disable kubelet`：

```shell
setenforce 0
swapoff -a
systemctl enable kubelet
systemctl start kubelet
```

### 参考链接

* [https://www.codesheep.cn/2018/12/27/kubeadm-k8s1-13-1/](https://www.codesheep.cn/2018/12/27/kubeadm-k8s1-13-1/)
* [https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)
* [https://github.com/opsnull/follow-me-install-kubernetes-cluster](https://github.com/opsnull/follow-me-install-kubernetes-cluster)
* [https://zhuanlan.zhihu.com/p/40931670](https://zhuanlan.zhihu.com/p/40931670)
* [https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/](https://kubernetes.io/blog/2016/12/container-runtime-interface-cri-in-kubernetes/)
* [https://github.com/kubernetes/kubeadm/issues/110](https://github.com/kubernetes/kubeadm/issues/110)



-End-
