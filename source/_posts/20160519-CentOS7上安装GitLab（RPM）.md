---
title: CentOS7上安装GitLab（RPM）
date: 2016-05-19 14:18:11
tags: 
        - CentOS
        - GitLab
        - Git
        - Redis
        - Ruby on Rails
        - 2016
categories: Program
comments: false
lang:
        - zh-CN

---

[GitLab](https://about.gitlab.com/)是基于[Ruby on Rails](http://www.ruby-lang.org/en/)的一个开源的版本管理系统，其内嵌了Redis。本文在[CentOS7](https://www.centos.org/)上安装GitLab.
<!-- more -->

## **1. CentOS安装** ##

[CentOS7](https://www.centos.org/)（Community Enterprise Operation System, 社区企业操作系统）,截至发文时，最新版为CentOS7，本文安装的是64位。

- 安装环境：

    > VMware® Workstation 12 Pro

- 安装版本

    > [CentOS-7-x86_64-Everything-1511](https://www.centos.org/download/) 

- 安装方式

    > Minimal Install

## **2. GitLab安装** ##


- 准备工作

    ```
yum -y install curl openssh-server
systemctl enable sshd
systemctl start sshd
yum -y install postfix
systemctl enable postfix
systemctl start postfix
firewall-cmd --permanent --add-service=http
systemctl reload firewalld
    ```
- 安装GitLab
无法按照[官网](https://about.gitlab.com/downloads/#centos7)的说明进行安装，原因你懂的！不过，官网提供的镜像安装方式<https://mirror.tuna.tsinghua.edu.cn/help/gitlab-ce/>。

    - 创建文件*/etc/yum.repos.d/gitlab-ce.repo*

        ```
[gitlab-ce]
name=gitlab-ce
baseurl=http://mirrors.tuna.tsinghua.edu.cn/gitlab-ce/yum/el7
repo_gpgcheck=0
gpgcheck=0
enabled=1
gpgkey=https://packages.gitlab.com/gpg.key
        ```
    - 执行命令

    ```
yum makecache
yum -y install gitlab-ce
    ```


    - 初始化

    ```
gitlab-ctl reconfigure
    ```

- 访问GitLab

    > http://*.*.*.*:80
    > 默认管理用户为
    > ```
        root
        admin@example.com
        ```
    > 第一次登录需要更改密码

- 常见问题
    - 建议使用Chrome等浏览器
    - 亲测360等浏览器不好用