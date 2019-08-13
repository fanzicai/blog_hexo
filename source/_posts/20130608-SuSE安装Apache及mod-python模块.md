---
title: SuSE安装Apache及mod_python模块
date: 2013-06-08 16:23:25
tags:
        - SuSE
        - Apache
        - Python
        - 2013
categories:
        - Program
comments: false
lang:
        - zh-CN
---
本文在SuSE 11发行版上源码安装apache以及mod_python模块。

<!-- more -->

## **Apache** ##
本文采用httpd-2.2.19.tar.gz为例。
- 安装
```
tar xvf httpd-2.2.19.tar.gz
cd httpd*/
./configure --prefix=/usr/local/apache --with-python=/usr/local/bin/python
make
make install
```
其中，指定了python的执行路径
- 运行
```
/usr/local/apache/bin/apachectl -k start
```

----------

## **mod_python** ##
本文以mod_python-3.3.1.tgz为例
- 安装
```
tar xvf mod_python-3.3.1.tgz
cd mod_python*
./configure --with-apxs=/usr/local/apache/bin/apxs --with-python=/usr/local/bin/python2.6
make
make install
```
> 编译中出现错误
> Connobject.c:142:error:request for member ‘next’ in something not a structure or union
> 需修改mod_python-3.3.1/src/connobject.c文件
> "!(b==APR_BRIGADE_SENTINEL(b))”
> 修改为：“!(b==APR_BRIGADE_SENTINEL(bb))”

- 配置
> httpd.conf文件中增加
> ```
LoadModule python_module /usr/local/apache/modules/mod_python.so
```

## **测试** ##


- apache/htdocs/目录下，添加文件test/mptest.py
```
   from mod_python import apache
    
   def handler(req):
        req.write("Hello World!")
        return apache.OK
```
- httpd.conf配置
```
    <Directory /usr/local/apache/htdocs/test>
        AllowOverride FileInfo
        AddHandler mod_python .py  #注意：mod_python与“.py”中间有空格
        PythonHandler mptest
        PythonDebug On
        Order allow,deny
         Allow from all
    </Directory>
    <IfModule alias_module>
        ScriptAlias /htdocs/test/ "/usr/local/apache/htdocs/test/"
    </IfModule>
```

- 访问
http://192.168.100.125:8080/test/mptest.py