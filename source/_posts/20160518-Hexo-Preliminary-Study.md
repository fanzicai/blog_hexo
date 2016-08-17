---
title: Hexo Preliminary Study
date: 2016-05-18 10:51:44
tags: 
        - GitHub
        - Hexo
        - Node.js
        - 2016
categories: Program
comments: false
lang: 

---

**[Hexo](https://hexo.io)** is a fast, simple & powerful blog framework. In hexo, we can write posts in **Markdown** (including **GFM**, Github Flavored Markdown).


<!-- more -->

## **1. Hexo Install** ##
Refered in *<https://hexo.io/docs/index.html>*

Before Hexo installing, need install:
- [Node.js](https://nodejs.org/en/), recommend the LTS Version
- [Git](https://git-scm.com/download)

Install Hexo in git bash:
```
mkdir %Blog_Root_Dir%
cd %Blog_Root_Dir%
npm install -g hexo-cli
```

Init Hexo:
```
hexo init [dir]
```

## **2. Hexo and GitHub Pages** ##
- Create Repo on GitHub

    > the repo name is ***[fanzicai.github.io](https://github.com/fanzicai/fanzicai.github.io)***

- Edit *_config.yml*
```
deploy:
     type: git
     repo: git@github.com:fanzicai/fanzicai.github.io.git
     branch: master
```

- Install deploy-tools
```
npm install hexo-deployer-git --save
```

- Deploy
```
hexo clean 
hexo generate         ## or "hexo g"
hexo deploy           ## or "hexo d"
```
一键发布
```
hexo deploy g
```
- View
> <https://fanzicai.github.io>


## **3. Hexo Test** ##

Test on <http://127.0.0.1:4000>

```
hexo server          ## or "hexo s"
```

## **4. Hexo Theme** ##
For example, the theme "[hexo-theme-athene](https://github.com/fanzicai/hexo-theme-athene)"


- About athene

    -  for [hexo](https://github.com/hexojs/hexo) 3.* and latest
    -  refer to [hexo-theme-yilia](https://github.com/litten/hexo-theme-yilia)
    -  for learning


- Use athene
    
    - install
    ```
    cd %Blog_Root_Dir%
    git clone git@github.com:fanzicai/hexo-theme-athene  themes/athene
    ```

    - configure
    > edit the "_config.yml"
    > ```
    theme: athene
     ```

- Fork Me on GitHub
    > Edit "themes/athene/layout.ejs", insert
    > ```
    <a href="https://github.com/fanzicai"><img style="position: absolute; top: 0; left: 0; border: 0;" src="https://camo.githubusercontent.com/82b228a3648bf44fc1163ef44c62fcc60081495e/68747470733a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f6c6566745f7265645f6161303030302e706e67" alt="Fork me on GitHub" data-canonical-src="https://s3.amazonaws.com/github/ribbons/forkme_left_red_aa0000.png"></a>
    ```
    > before < / body >
