---
title: GitHub和Jekyll搭建静态博客
date: 2016-08-03 17:07:27
tags:
        - 2016
        - GitHub
        - Jekyll
        - Ruby
categories:
        - Program
comments: false
lang:
        - zh-CN

---
根据官方教程进行搭建，<https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/>。

<!-- more -->
## **1. 官方教程** ##
> 根据官方教程
> 安装ruby、bundle
> 创建blog_jekyll目录
> 初始化jekyll工程


----------

## **2. Jekyll配置** ##
> _config.yml
> ```
title: fanzi.home@web
email: fanzicai@yahoo.com
description: > # this means to ignore newlines until "baseurl:"
  nothing to say.
baseurl: "/blog_jekyll" # the subpath of your site, e.g. /blog
url: "https://www.fanzicai.com" # the base hostname & protocol for your site
#twitter_username: jekyllrb
github_username:  fanzicai
# Build settings
markdown: kramdown
```


----------

## **3. GitHub配置** ##
> repo
> blog_jekyll
> ```
git push --set-upstream origin gh-pages
```
> 注意，git推送时，需把.gitconfig中的_site删除

> 访问<https://fanzicai.github.io/blog_jekyll>


----------
