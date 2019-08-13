---
title: TortoiseGit连接Gitlab
date: 2016-06-08 16:03:36
tags:
        - Git
        - GitLab
        - TortoiseGit
        - 2016
categories:
        - Program
comments: false
lang:
        - zh-CN

---
docker gitlab部署完成后，准备采用git bash和TortoiseGit连接gitlab。
<!-- more -->

## **Git Bash连接** ##
- 密钥生成
> git bash
> ```
ssh-keygen -t rsa -C "fanzicai@yahoo.com" path_to_key_~/.ssh
```
> 自动生成id_rsa和id_rsa.pub

- 密钥上传
> 将id_rsa.pub的数据添加到gitlab的ssh key

## **TortoiseGit连接** ##
- 格式转换
> 采用TortoiseGit的工具PuTTYgen
> 1、先"Load"上文的id_rsa
> 2、再"Save private key"存成ppk格式

- PuTTY设置
> 设置gitlab服务器的连接，并存储
> 在Connection->SSH->Auth中添加上文ppk文件

- Pageant设置
> 启动Pageant，添加ppk文件的key
