---
title: CentOS中MySQL远程访问
comments: false
meta:
  - 'name="robots";content="noindex, follow"'
  - name="";value="";enabled=false
date: 2016-11-01 10:50:52
tags:
        - 2016
        - MySQL
        - Firewall
        - Iptables
categories:
        - Program
lang:
        - zh-CN
---
本文描述如何在CentOS7中允许MySQL的端口。

<!-- more -->
## **1、方法一** ##
```
firewall-cmd --add-service=mysql
```
> 永久改变
> ```
firewall-cmd --permanent --add-service=mysql
```

----------

## **2、方法二** ##
采用iptables，前提是要先安装iptables相关
> 相关命令有
> 添加端口3306
> ```
iptables -I INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```
> 查看状态
> ```
iptables -L -n  //或者 service iptables status
```
> 删除端口
> ```
iptables -D INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT  
```
> 永久保存
> ```
service iptables save
```
>  或者
>  ```
/etc/init.d/iptables save
```

或者编辑配置文件
> ```
vi /etc/sysconfig/iptables //在该文件中加入下面这条规则也是可以生效的  
-A INPUT -p tcp -m state --state NEW -m tcp --dport 3306 -j ACCEPT
```


----------
   