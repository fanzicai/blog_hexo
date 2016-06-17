---
title: CentOS7上安装Docker及Jenkins
date: 2016-05-25 15:50:55
tags:
        - CentOS
        - Docker
        - Jenkins
        - 2016
categories: Program
lang:
        - zh-CN

---
[Jenkins](https://jenkins.io/)是基于Java开发的一种持续集成工具，用于监控持续重复的工作，功能包括：

- 持续的软件版本发布/测试项目。
- 监控外部调用执行的工作。
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
参考[CentOS Wiki](https://wiki.centos.org/zh/Cloud/Docker?highlight=%28docker%29)。
- 准备工作
```
sudo yum -y update
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
## **3. [Jenkins](https://jenkins.io/)** ##
采用[docker jenkins](https://hub.docker.com/_/jenkins/)
- 安装
```
docker run --detach --name jenkins --restart always -p 8080:8080 -p 50000:50000 jenkins
```
> 下载image及创建名为jenkis的容器并运行
> 启动、停止容器
> ```
docker start jenkins
docker stop jenkins
```
- 访问
> 建议采用IE内核的浏览器
> 在chrome浏览器下，配置任务时容易出错
> ```
http://*.*.*.*:8080
```

----------
## **3. Git插件** ##
> 管理插件
> 因为采用gitlab管理源码，需要添加
> - git plugin
> - git client plugin
> - gitlab hook plugin
> - gitlab plugin
> - gitlab merge request builder
> - gitlab logo plugin
> - Gitlab Authentication plugin


----------
## **4. 访问Gitlab** ##
> 采用git clone git@host_gitlab:usr_name/repo_name.git
> 需先在jenkins容器中生成私钥
> ```
cd /var/jenkins_home
ssh-keygen -t rsa -C "fanzicai@yahoo.com" -f /var/jenkins_home/.ssh/id_rsa
```
> 把id_rsa.pub的公钥内容，配置到gitlab服务器上
> 同时，在jenkins构建工程设置中私钥采用
> From the Jenkins master ~/.ssh

 

----------
 