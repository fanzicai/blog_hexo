---
title: Tomcat配置https
comments: false
meta:
  - 'name="robots";content="noindex, follow"'
  - name="";value="";enabled=false
date: 2016-10-31 14:48:44
tags:
        - Tomcat
        - SSL
        - 2016
categories:
        - Program
lang:
        - zh-CN
---
本文描述Tomcat配置https连接的方式之一。

<!-- more -->

## **1、Key生成** ##
- 生成ssh key
> ```
$JAVA_HOME/bin/keytool -genkey -alias tomcat -keyalg RSA -keystore /path/to/my/keystore
```

----------

## **2、配置** ##
- /conf/server.xml
> ```
 <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
               maxThreads="150" SSLEnabled="true" scheme="https" secure="true"
               keystoreFile="${user.home}/.keystore" keystorePass="changeit"
               clientAuth="false" sslProtocol="TLS" />
```


----------
