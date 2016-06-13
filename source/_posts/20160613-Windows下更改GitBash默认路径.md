---
title: Windows下更改GitBash默认路径
date: 2016-06-13 09:22:54
tags:
        - Windows
        - Git
categories:
        - Program
lang:
        - zh-CN
---
运行Git Bash后，控制台会进入默认路径（目录）。Git安装后，默认路径为用户目录。

<!-- more -->

## **Git Bash默认路径** ##
- 方法一
> Git Bash快捷方式的属性中，有一项配置“起始位置”，改为所需目录即可
> 通常情况下，该方法无效，故采用方法二

- 方法二
> Git安装目录下，如“D:\Program Files\Git\etc”，找到profile文件，增加以下内容
> ```
HOME="所要设置的起始目录"
HOME="$(cd "$HOME" ; pwd)"
cd
export PATH="$HOME/bin:$PATH"
```

本文使用的Git版本是2.8.2。 