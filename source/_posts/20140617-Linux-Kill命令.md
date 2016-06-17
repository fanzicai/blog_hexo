---
title: Linux Kill命令
date: 2014-06-17 09:18:08
tags:
        - 2014
        - Linux
        - Shell
categories: Program
lang: zh-CN

---
使用Linux kill最安全的方法是不加修饰符，不带标志。

<!-- more -->
## **1. 绝杀** ##
```
kill -9 PID
```

----------
## **2. 优雅** ##
```
kill -l PID
```

----------

## **3. 其他** ##
```
ps -ef | grep httpd
ps -ef  PID
```

----------
