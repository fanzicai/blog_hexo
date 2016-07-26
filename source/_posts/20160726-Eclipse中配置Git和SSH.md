---
title: Eclipse中配置Git和SSH
date: 2016-07-26 15:09:16
tags:
        - 2016
        - Eclipse
        - Git
        - SSH
categories:
        - Program
lang:
        - zh-CN

---
Eclipse中使用Git进行Team协同。

<!-- more -->
## **1. Git Plug-in** ##
> 默认情况下，Eclipse已经安装了Git插件，一般为EGit或JGit。
> 直接使用即可。


----------

## **2. Git配置** ##
> 系统环境变量
> ```
HOME = /.gitconfig目录
```
> Eclipse配置
> Team - Git - Configuration
> ```
key     - user.name
value  - fanzi
key     - user.email
value  - fanzicai@yahoo.com
```
> Eclipse配置
> General - Network Connections - SSH2
> ```
SSH2 home  = /git目录/.ssh    # 一般为gitlab或github等的私钥存放地
Key Management    #如有现成私钥，则导入，如无则创建，同时把公钥上传到git服务器
``` 
> 使用时，采用ssh访问 git@gitlab.com/***.git方式


----------

