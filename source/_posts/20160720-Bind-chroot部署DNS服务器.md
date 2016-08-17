---
title: Bind-chroot部署DNS服务器
date: 2016-07-20 13:49:06
tags:
        - 2016
        - CentOS
        - Bind
        - Chroot
        - DNS
categories: Program
comments: false
lang:
        - zh-CN
---
CentOS 7部署DNS内网服务器，采用bind-chroot工具。

<!-- more -->

## **1. 安装工具** ##
- bind-utils
- ```
yum -y install bind-utils
```
----------

## **2. 修改配置** ##
> 涉及目录有
> ```
/var/named/chroot/etc/
    named.conf
    named.rfc1912.zones
/var/named/chroot/var/named/
    nid.com.zone
    1.168.192.local
```

- named.conf
```
cp -f /etc/named.conf /var/named/chroot/etc/
```
> ```
options {
        listen-on port 53 { 192.168.1.183; };

        ...

        allow-query     { any; };

        recursion yes;

        dnssec-enable no;
        dnssec-validation no;

        ...

```

- named.rfc1912.zones
> 添加内容
> ```
zone "nid.com" IN {
        type master;
        file "nid.com.zone";
        allow-update {none;};
};

zone "1.168.192.in-addr.arpa" IN {
        type master;
        file "1.168.192.local";
        allow-update {none;};
};

```

- nid.com.zone
> ```
$TTL 86400
@       IN SOA  nid.com. mail.nid.com. (
                                        46      ; serial
                                        43200   ; refresh
                                        3600    ; retry
                                        3600000 ; expire
                                        2592000 )       ; minimum
        IN      NS      www.nid.com.
        IN      A       192.168.1.200
www     IN      A       192.168.1.200
gitlab  IN      A       192.168.1.10
jenkins IN      A       192.168.1.80
nginx   IN      A       192.168.1.200
tomcat  IN      A       192.168.1.80

```

- 1.168.192.local
> ```
$TTL 86400
@       IN SOA  nid.com. mail.nid.com. (
                                        45      ; serial
                                        28800   ; refresh
                                        14400   ; retry
                                        36000000        ; expire
                                        86400 ) ; minimum
        IN      NS      www.nid.com.
200     IN      PTR     www.nid.com.
10      IN      PTR     gitlab.nid.com
80      IN      PTR     jenkins.nid.com

www     IN      A       192.168.1.200
```

- /etc/resolv.conf
> ```
nameserver 192.168.1.183
nameserver 192.168.1.1
```
> 其中，192.168.1.183为DNS服务器的IP

----------
## **3. 启动DNS** ##
> ```
/usr/libexec/setup-named-chroot.sh /var/named/chroot on
systemctl enable named-chroot
systemctl start named-chroot
```

----------
## **4. 测试** ##
> 采用nslookup
> server 查看DNS服务器列表
> www.nid.com 查看该域名的ip


----------

