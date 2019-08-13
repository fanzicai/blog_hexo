---
title: Tomcat PermGen Space Error
date: 2015-06-17 09:12:44
tags:
        - 2015
        - Tomcat
categories:
        - Program
comments: false
lang: 
        - zh-CN

---
PermGen space的全称是Permanent Generation space,是指内存的永久保存区域，这块内存主要是被JVM存放Class和Meta信息的。

<!-- more -->

## **1. PermGen Space** ##
PermGen space的全称是Permanent Generation space,是指内存的永久保存区域，这块内存主要是被JVM存放Class和Meta信息的,Class在被Loader时就会被放到PermGen space中，它和存放类实例(Instance)的Heap区域不同,GC(Garbage Collection)不会在主程序运行期对PermGen space进行清理，所以如果你的应用中有很CLASS的话,就很可能出现PermGen space错误，这种错误常见在web服务器对JSP进行pre compile的时候。如果你的WEB APP下都用了大量的第三方jar, 其大小超过了jvm默认的大小(4M)那么就会产生此错误信息了。

----------
## **2. MaxPermSize** ##
解决PermGen Space Error：
> catalina.sh下为：
> ```
JAVA_OPTS="$JAVA_OPTS -server -XX:PermSize=128M -XX:MaxPermSize=512m"
```

----------
## **3. Heap Size** ##
解决Heap Size Error：
> catalina.sh下为：
> ```
JAVA_OPTS="$JAVA_OPTS  -server -Xms800m -Xmx800m   -XX:MaxNewSize=256m"
```

----------
