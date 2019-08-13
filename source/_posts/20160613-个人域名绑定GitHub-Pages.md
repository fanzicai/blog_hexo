---
title: 个人域名绑定GitHub Pages
date: 2016-06-13 16:52:37
tags:
        - 2016
        - GitHub
categories:
        - Program
comments: false
lang:
        - zh-CN

---
[GitHub Pages](https://pages.github.com/)提供了最多1GB的免费空间，适用于静态页面，如博客等。
[凡尘 博客](https://www.fanzicai.com)亦即采用该方式，并且使用了hexo发布博客。

<!-- more -->

## **1. 解析设置** ##
> 在域名提供商处，对域名进行解析
> 比如[www.fanzicai.com](www.fanzicai.com)
> 解析地址可为下列地址之一
> ```
192.30.252.153
192.30.252.154
```

----------
## **2. GitHub设置** ##
> [fanzicai.github.io](fanzicai.github.io)的repo中根目录增加文件CNAME
> CNAME的内容为
> ```
www.fanzicai.com
```
> 使用Hexo deploy时，会把服务端的CNAME冲掉

----------

## **3. Hexo** ##
> 在source目录下，创建CNAME即可
