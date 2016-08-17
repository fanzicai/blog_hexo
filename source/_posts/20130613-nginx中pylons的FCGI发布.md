---
title: nginx中pylons的FCGI发布
date: 2013-06-13 10:05:26
tags:
        - Nginx
        - FCGI
        - Pylons
        - Pyramid
        - 2013
categories:
        - Program
comments: false
lang:
        - zh-CN

---
[Nginx](http://nginx.org/) ("engine x") 是一个高性能的HTTP和反向代理服务器，也是一个IMAP/POP3/SMTP服务器。Nginx是由Igor Sysoev为俄罗斯访问量第二的Rambler.ru站点开发的，第一个公开版本0.1.0发布于2004年10月4日。其将源代码以类BSD许可证的形式发布，因它的稳定性、丰富的功能集、示例配置文件和低系统资源的消耗而闻名。

<!-- more -->

## **1. 安装环境** ##
- 操作系统
SuSE Linux 11, 2.6.32.12-0.7-pae
- Nginx
nginx-1.5.1.tar.gz
- pcre
> ```
tar xvf pcre-8.02.tar.gz
cd pcre-8.02/
./configure
make
make install
```

----------
## **2. Nginx** ##
- 安装Nginx
> ```
tar xvf nginx-1.5.1.tar.gz
cd nginx-1.5.1/
./configure
make
make install
```
- 修改usr/local/ngnix/conf/nginx.conf
> 注释掉行44：root html;
> 注释掉行45：index index.html index.htm;
> 增加
> ```
fastcgi_pass   127.0.0.1:5000;
fastcgi_param  PATH_INFO $fastcgi_script_name;
include        fastcgi_params;
fastcgi_intercept_errors off;
```
- 添加服务
> /etc/init.d目录下创建nginx文件（启动脚本），用于执行/usr/local/nginx/sbin/nginx
> ```
chkconfig nginx on
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
        use = egg:Flup#fcgi_thread
        host = 127.0.0.1
        port = 5000
```

----------
注：目前pylons已经被[Pyramid](http://www.pylonsproject.org/)替代，建议新的项目应采用[Pyramid](http://www.pylonsproject.org/).