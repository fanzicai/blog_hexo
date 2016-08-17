---
title: CentOS7上安装Docker及GitLab
date: 2016-05-25 15:04:14
tags:
    - CentOS
    - Docker
    - GitLab
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
- 官网方式
> 添加repo
> ```
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
> 正式安装
> ```
sudo yum -y install docker-engine
```
- CentOS Wiki推荐方式
> ```
yum -y install docker
```

- 自启动
```
systemctl enable docker
systemctl start docker
```

----------
## **3. GitLab** ##
安装教程详见<http://docs.gitlab.com/omnibus/docker/>。
- 安装
```
sudo docker run --detach \
    --hostname gitlab.example.com \
    --env GITLAB_OMNIBUS_CONFIG="external_url 'http://my.domain.com/'; gitlab_rails['lfs_enabled'] = true;"\
    --publish 443:443 --publish 80:80 --publish 22:22\
    --name gitlab \
    --restart always \
    gitlab/gitlab-ce:latest
```
- 进入gitlab控制台
```
sudo docker exec -it gitlab /bin/bash
```
- 配置
```
sudo docker exec -it gitlab vi /etc/gitlab/gitlab.rb
```
- 运行
```
sudo docker start gitlab
```
- 其他命令
> 停止
```
sudo docker stop gitlab
```
> 删除
```
sudo docker rm gitlab
```
> 更新
```
sudo docker pull gitlab/gitlab-ce:latest
```
> 查看日志
```
sudo docker logs gitlab
```
> 查看Container
```
docker ps -a
```
> 查看Image
```
docker images
```
