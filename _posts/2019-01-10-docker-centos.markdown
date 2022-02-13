---
title: Install Docker on CentOS 7
date: 2019-01-10
categories: Docker
---

之前在公司lab的机器上准备搭建一个开发环境的Kubernetes集群环境，发现公司机器上的CentOS 7上Docker都跑不起来，错误信息如下：

```
docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: inactive (dead) since Thu 2018-05-17 15:47:26 CEST; 17h ago
     Docs: https://docs.docker.com
 Main PID: 11843 (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/docker.service

May 18 08:48:38 temp systemd[1]: docker.service: Job docker.service/start failed with result 'dependency'.
May 18 08:49:09 temp systemd[1]: Stopped Docker Application Container Engine.
May 18 08:49:09 temp systemd[1]: Dependency failed for Docker Application Container Engine.
May 18 08:49:09 temp systemd[1]: docker.service: Job docker.service/start failed with result 'dependency'.
May 18 08:49:15 temp systemd[1]: Dependency failed for Docker Application Container Engine.
May 18 08:49:15 temmp systemd[1]: docker.service: Job docker.service/start failed with result 'dependency'.
May 18 09:00:03 temp systemd[1]: Dependency failed for Docker Application Container Engine.
May 18 09:00:03 temp systemd[1]: docker.service: Job docker.service/start failed with result 'dependency'.
May 18 09:03:51 temp systemd[1]: Dependency failed for Docker Application Container Engine.
May 18 09:03:51 temp systemd[1]: docker.service: Job docker.service/start failed with result 'dependency'.
```

查了一下原来是因为Linux内核版本是3.8，而Docker要求的最低内核版本是3.10. https://docs.docker.com/install/linux/docker-ce/binaries/ 。 所以解决办法就是升级内核。如果在CentOS上遇到类似的问题，你可以按照如下步骤来升级内核。

###升级Linux内核

首先检查当前Linux内核版本：

```shell
uname -sr
```

导入ELRepo repository：

```shell
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
```

列出可用的内核版本：

```shell
yum --disablerepo="*" --enablerepo="elrepo-kernel" list available
```

安装最新的稳定版内核:

```shell
yum --enablerepo=elrepo-kernel install kernel-ml
```

检查已经安装可用的内核版本：

```shell
awk -F\' '$1=="menuentry " {print i++ " : " $2}' /etc/grub2.cfg
```

选择一个作为默认的：

```shell
sudo grub2-set-default 0
```

用`gurb2-mkconfig`生成grub2配置，并且重启机器：

```shell
grub2-mkconfig -o /boot/grub2/grub.cfg
reboot
```

### 安装Docker

```shell
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install docker-ce-18.06.0.ce
# Start Docker
systemctl start docker
```

>  这里安装的是Docker 18.06，因为这是Kubernetes已验证的最高版本。如果要安装最新版，直接`yum install docker-ce`就行。

### 参考链接

* [<https://docs.docker.com/install/linux/docker-ce/centos/>](<https://docs.docker.com/install/linux/docker-ce/centos/>)
* [<https://www.tecmint.com/install-upgrade-kernel-version-in-centos-7/>](<https://www.tecmint.com/install-upgrade-kernel-version-in-centos-7/>)
* [<https://stackoverflow.com/questions/50405666/docker-service-failed-see-journalctl-xe-for-details>](<https://stackoverflow.com/questions/50405666/docker-service-failed-see-journalctl-xe-for-details>)
* [<https://www.howtoforge.com/tutorial/how-to-upgrade-kernel-in-centos-7-server/>](<https://www.howtoforge.com/tutorial/how-to-upgrade-kernel-in-centos-7-server/>)



-End-
