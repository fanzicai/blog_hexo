---
title: lighttpd中pylons的SCGI发布
date: 2016-06-13 09:40:57
tags:
        - LightTPD
        - SCGI
        - Pylons
        - Pyramid
categories: Program
lang: zh-CN

---
[Lighttpd](https://www.lighttpd.net/) 是一个德国人领导的开源Web服务器软件，其根本的目的是提供一个专门针对高性能网站，安全、快速、兼容性好并且灵活的web server环境。具有非常低的内存开销、cpu占用率低、效能好以及丰富的模块等特点。

<!-- more -->

## **1. 安装环境** ##
- 操作系统
SuSE Linux 11, 2.6.32.12-0.7-pae
- LightTPD
lighttpd-1.4.32



----------
## **2. LightTPD** ##
- 安装LightTPD
> ```
tar xvf lighttpd-1.4.32.tar.gz
cd lighttpd-1.4.32/
./configure --prefix=/usr/local/lighttpd --without-pcre --without-bzip2
make
make install
```
- 添加服务
> ```
cp doc/initscripts/rc.lighttpd /etc/init.d/lighttpd
chkconfig lighttpd on
```
> 修改该文件，行35
> ```
LIGHTTPD_BIN=/usr/local/lighttpd/sbin/lighttpd
```
- 配置文件
> ```
cp doc/initscripts/sysconfig.lighttpd /etc/sysconfig/lighttpd
mkdir /etc/lighttpd
cp doc/config/lighttpd.conf /etc/lighttpd/
cp doc/config/modules.conf /etc/lighttpd/
cp –r doc/config/conf.d/ /etc/lighttpd/
```
- 配置修改
> 修改modules.conf，行127增加
> ```
include “conf.d/scgi.conf”
```
> 修改lighttpd.conf，行127增加
> ```
server.bind = “0.0.0.0”
```
> 注释掉lighttpd.conf的315~317行，内容如下
> ```
#$HTTP["url"] =~ "\.pdf$" {
#  server.range-requests = "disable"
#}
```
> 修改scgi.conf，增加内容
> ```
scgi.server = ( "/" =>
                   ((
                        "host" => "127.0.0.1",
                        "port" => 5000,
                        "check-local" => "disable",
                        "docroot" => "/"
                   ))
                )
```
- 运行服务
> ```
server lighttpd start
```

----------
## **3. Pylons** ##
本文pylons可参照官网方式安装。
- flup
> flup-1.0.3.dev-20110405.tar.gz
> ```
source pylons/bin/activate
tar xvf flup-1.0.3.dev-20110405.tar.gz
cd flup-1.0.3.dev-20110405
python setup.py install
```
- development.ini
> ```
[server:main]
        use = egg:PasteScript#flup_scgi_thread
        host = 127.0.0.1
        port = 5000
```



----------
注：目前pylons已经被[Pyramid](http://www.pylonsproject.org/)替代，建议新的项目应采用[Pyramid](http://www.pylonsproject.org/).