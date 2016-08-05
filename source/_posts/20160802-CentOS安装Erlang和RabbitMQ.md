---
title: CentOS安装Erlang和RabbitMQ
date: 2016-08-02 16:27:15
tags:
        - 2016
        - CentOS
        - Erlang
        - RabbitMQ
        - AMQP
categories:
        - Program
lang:
        - zh-CN

---
本文描述esl-erlang、esl-erlang-compat、rabbitmq-server的安装。

<!-- more -->
AMQP, Advanced Message Queuing Protocol，高级消息队列协议。

## **1. 安装Erlang** ##

- 安装epel-release
> ```
rpm -ivh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

- 安装Erlang
> 编辑/etc/yum.repos.d/erlang.repo，增加内容
> ```
[erlang-solutions]
name=Centos 7 - x86_64 - Erlang Solutions
baseurl=https://packages.erlang-solutions.com/rpm/centos/7/x86_64
gpgcheck=1
gpgkey=https://packages.erlang-solutions.com/rpm/erlang_solutions.asc
enabled=1
```
> ```
yum update -y
yum install -y esl-erlang
rpm -ivh http://www.rabbitmq.com/releases/erlang/esl-erlang-compat-18.1-1.noarch.rpm
```

----------

## **2.安装RabbitMQ** ##

- 安装socat
> ```
yum install -y socat
```

- 安装rabbitmq
> ```
rpm -ivh http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.4/rabbitmq-server-3.6.4-1.noarch.rpm
```

- 启动
> ```
systemctl enable rabbitmq-server
systemctl start rabbitmq-server
```
----------

## **3.启用Web-Plugin** ##
- RabbitMQ management
> ```
rabbitmq-plugins enable rabbitmq_management
systemctl restart rabbitmq-server
```

- 防火墙端口
> ```
firewall-cmd --permanent --zone=public --add-port=15672/tcp
firewall-cmd --reload
```
- 本地用户
> 用户名guest，密码guest
> 无法远程登录

- 远程用户
> ```
rabbitmqctl add_user fanzi 123
rabbitmqctl set_permissions -p "/" fanzi ".*" ".*" ".*"
rabbitmqctl set_user_tags fanzi administrator
```

----------
