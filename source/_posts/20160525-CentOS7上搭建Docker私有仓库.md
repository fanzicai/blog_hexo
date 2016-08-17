---
title: CentOS7上搭建Docker私有仓库
date: 2016-05-25 16:19:37
tags:
        - CentOS
        - Docker
        - 2016
categories: Program
comments: false
lang:
        - zh-CN

---
[Docker](https://www.docker.com/)是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到任何流行的 Linux 机器上，也可以实现虚拟化。容器是完全使用沙箱机制，相互之间不会有任何接口。
<!-- more -->

## **1. CentOS** ##

[CentOS7](https://www.centos.org/)（Community Enterprise Operation System, 社区企业操作系统）,截至发文时，最新版为CentOS7，本文安装的是64位。

- 安装环境：

    > VMware® Workstation 12 Pro

- 安装版本

    > [CentOS-7-x86_64-Everything-1511](https://www.centos.org/download/) 

- 安装方式

    > Minimal Install



----------
## **2. [Docker](https://www.docker.com/)** ##
参考[官网安装流程](https://docs.docker.com/engine/installation/linux/centos/)和[CentOS Wiki](https://wiki.centos.org/zh/Cloud/Docker?highlight=%28docker%29)。
- 准备工作
```
sudo yum -y update
```
- CentOS Wiki推荐方式
```
yum -y install docker
```

- 自启动
```
systemctl enable docker
systemctl start docker
```

- 本地仓库
```
docker pull registry
docker run -d -p 5000:5000 --restart=always -v /opt/data/registry:/tmp/registry --name registry docker.io/registry
```

- 访问仓库
```
curl *.*.*.*:5000/v1/search
```

----------
## **3. 上传镜像** ##
在已安装docker gitlab的虚拟机上，向上文仓库提交镜像，采用gitlab image。

- 列出镜像
```
docker images
```
- 创建镜像链接
```
docker tag docker.io/gitlab/gitlab-ce *.*.*.*:5000/gitlab-ce
```
- 修改客户端配置文件
> ```
vim /etc/sysconfig/docker
```
> 增加"--insecure-registry \*.\*.\*.\*:5000"

- 提交镜像
```
docker push *.*.*.*:5000/gitlab-ce
```