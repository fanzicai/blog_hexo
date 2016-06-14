---
title: Debian系统安装Python的MySQLdb库
date: 2014-06-14 09:10:00
tags:
        - 2014
        - Debian
        - Python
        - MySQL
categories:
        - Program
lang:
        - zh-CN

---
Python访问[MySQL](http://www.mysql.com/)数据库有很多可用的插件，本文选用的是[MySQL-Python](https://pypi.python.org/pypi/MySQL-python/1.2.5)。

<!-- more -->

## **1. 安装环境** ##

- MySQL 5.1
- Python 2.6.5

## **2. MySQLdb** ##

- 安装包
> MySQL-python-1.2.3c1.tar.gz
> setuptools-0.6c11.tar.gz

- Setuptools
> 安装
> ```
python setup.py build
sudo python setup.py install
```

- MySQLdb
> 编辑site.cfg
> ```
mysql_config = /usr/bin/mysql_config
```
> 找到本机中的mysql_config路径
> 安装
> ```
python setup.py build
python setup.py install
```
> 测试
> python命令行下
> ```
import MySQLdb
```

----------
