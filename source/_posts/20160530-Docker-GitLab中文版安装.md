---
title: Docker GitLab中文版安装
date: 2016-05-30 15:11:22
tags: 
        - CentOS
        - Docker
        - GitLab
        - MySQL
        - 2016
categories: Program
comments: false
lang:
        - zh-CN

---

[GitLab](https://about.gitlab.com/)是基于[Ruby on Rails](http://www.ruby-lang.org/en/)的一个开源的版本管理系统，其内嵌了Redis。本文在[CentOS7](https://www.centos.org/)上采用[Docker](https://www.docker.com/)安装[GitLab汉化版](https://gitlab.com/larryli/gitlab/tags).
<!-- more -->

## **1. 准备工作** ##

可参考[CentOS7上安装Docker及GitLab](http://fanzicai.github.io/Program/2016/05/25/CentOS7%E4%B8%8A%E5%AE%89%E8%A3%85Docker%E5%8F%8AGitLab.html)一文，先行安装CentOS、Docker。

- Docker MySQL
Oracle官方已提供MySQL的[Docker Image](https://hub.docker.com/r/mysql/mysql-server/)。
```
docker run -it --detach -p 192.168.1.80:3306:3306 -p 192.168.1.80:33060:33060 --restart always  -e MYSQL_ROOT_PASSWORD=123 -v /root/mysql:/var/lib/mysql --name mysql mysql/mysql-server --character-set-server=utf8 --collation-server=utf8_general_ci
```
> docker -v 权限
> ```
chcon -Rt svirt_sandbox_file_t /root/mysql
```
> 进入mysql控制台
```
docker exec -it mysql /bin/bash
mysql -uroot -p123
```
> 创建git用户及database
```
CREATE USER 'git'@'localhost' IDENTIFIED BY '$password';
CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER, LOCK TABLES ON `gitlabhq_production`.* TO 'git'@'%';
flush privileges;
\q
```

- Docker [CentOS](https://wiki.centos.org/zh/Cloud/Docker?highlight=%28docker%29)
```
docker run --detach -ti \
    --publish 80:80 --publish 8080:8080 \
    --name gitlab \
    --restart always \
    --link mysql:mysql \
    centos:latest
```
> 其中--link参数用于表明连接之前mysql容器
- 进入gitlab-cn shell
```
docker exec -it gitlab /bin/bash
```

----------
## **2. 基本工具** ##
正式安GitLab及其依赖库之前，系统必须具备一些条件，需执行以下命令安装一些必要的基本工具


```
yum -y update
yum -y group install "development tools"
yum -y install vim cmake wget
yum -y install readline-devel gdbm-devel openssl-devel expat-devel sqlite-devel libyaml-devel libffi-devel libxml2-devel libxslt-devel libicu-devel system-config-firewall-tui python-devel xmlto logwatch perl-ExtUtils-CBuilder
yum -y install sudo
```


> 以下如非特殊说明，均采用root登录安装


----------
## **3. Git用户** ##
```
adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git
chmod -R go+rx /home/git
```


----------
## **4. Git** ##
因CentOS仓库的Git版本较低，本文采用源码安装的方式。
```
cd /root
wget https://www.kernel.org/pub/software/scm/git/git-2.8.3.tar.gz
tar xvf git-2.8.3.tar.gz
cd git-2.8.3
./configure --prefix=/usr
make
make install
```
----------
## **5. Ruby** ##
- Ruby
```
cd /home/git
yum -y install ruby
wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz
tar xvf ruby-2.3.1.tar.gz
cd ruby-2.3.1
./configure --prefix=/usr --disable-install-rdoc
make
make install
```
- Bundler
```
gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
gem install bundler
bundle config mirror.https://rubygems.org https://ruby.taobao.org
```

    > <https://ruby.taobao.org>是个完整的镜像。
----------
## **6. Go** ##
Go源码安装，但需要系统本身已有旧版本。
```
yum -y install go
cd /root
wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
tar xvf go1.5.3.linux-amd64.tar.gz -C /usr/local/ 
ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/bin/
```

----------

## **7. MySQL** ##


- MySQL安装
```
wget http://repo.mysql.com//mysql57-community-release-el7-8.noarch.rpm
rpm -Uvh mysql57-community-release-el7-8.noarch.rpm
yum -y update
yum -y install mysql mysql-devel
```

- MySQL连接
```
mysql -h mysql -ugit -p$password -D gitlabhq_production
```

> 其中$password为之前创建git用户时的密码


----------

## **8. Redis** ##
CentOS仓库默认没有Redis，先安装EPEL源。
- Redis安装
```
yum -y install epel-release
yum -y install redis
```

- 编辑/etc/redis.conf
> unixsocket设置为 /var/run/redis/redis.sock
> unixsocketperm 设置为 770

- 设置及启动
```
chown redis:redis /var/run/redis
chmod 755 /var/run/redis
usermod -aG redis git
systemctl enable redis
redis-server /etc/redis.conf &
```

----------
## **9. Node.js** ##
Node.js也需要EPEL源。
```
yum -y install nodejs
```

----------
## **10. GitLab** ##
本文是为了安装汉化版的GitLab才采用的源码安装方式。如若英文版，建议采用RPM方式安装，可参见另文或官网。
- GitLab下载
```
cd /home/git
sudo -u git -H wget  https://gitlab.com/larryli/gitlab/repository/archive.tar.gz?ref=v8.8.0.zh1
sudo -u git -H tar xvf archive.tar.gz?ref=v8.8.0.zh1
sudo -u git -H ln -s gitlab* gitlab
```
- gitlab.yml文件
```
cd /home/git/gitlab
sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml
```
> 编辑gitlab.yml文件
> ```
sudo -u git -H vim config/gitlab.yml
```
> 修改host，本文修改为本机IP

- 配置及权限
```
sudo -u git -H cp config/secrets.yml.example config/secrets.yml
sudo -u git -H chmod 0600 config/secrets.yml
sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX,go-w log/
sudo chmod -R u+rwX tmp/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/
sudo -u git -H mkdir public/uploads/
sudo chmod 0700 public/uploads
sudo chmod -R u+rwX builds/
sudo chmod -R u+rwX shared/artifacts/
```
- unicorn.rb文件
```
sudo -u git -H cp config/unicorn.rb.example config/unicorn.rb
```
> 必要时编辑unicorn.rb文件
> ```
sudo -u git -H vim config/unicorn.rb
```

- 其他配置
``` 
sudo -u git -H cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb
sudo -u git -H git config --global core.autocrlf input
sudo -u git -H git config --global gc.auto 0
```

- resque.yml文件
```
sudo -u git -H cp config/resque.yml.example config/resque.yml
```
> 编辑resque.yml文件
> ```
sudo -u git -H vim config/resque.yml
```
> 本文无修改

- database.yml文件
```
sudo -u git -H cp config/database.yml.mysql config/database.yml
sudo -u git -H chmod o-rwx config/database.yml
```
> 编辑database.yml文件
> 修改用户为git，host为mysql

- gitlab-shell
> 编辑Gemfile
> 改为ruby.taobao.org
> ```
sudo -u git -H bundle install --deployment --without development test postgres aws
sudo -u git -H bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production
```

----------
## **11. gitlab-workhorse** ##
```
cd /home/git
sudo -u git -H git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git
cd gitlab-workhorse
sudo -u git -H git checkout v0.7.2
sudo -u git -H make
```
----------
## **12. DB Init** ##
```
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production force=yes
```

----------
## **13. gitlab root** ##
更改gitlab root密码，默认无密码，也可初次访问时修改。
```
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production GITLAB_ROOT_PASSWORD=yourpassword GITLAB_ROOT_EMAIL=youremail
```

----------
## **14. gitlab服务** ##
- 一些配置
```
sudo chown -R git:git /home/git/repositories/
sudo chmod -R ug+rwX,o-rwx /home/git/repositories/
sudo chmod -R ug-s /home/git/repositories/
sudo find /home/git/repositories/ -type d -print0 | sudo xargs -0 chmod g+s
sudo chmod 700 /home/git/gitlab-v8.8.0.zh1-5c495e42d863613b5288d332c2c6f4189a684735/public/uploads
```

- 自启动
```
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
chkconfig gitlab on
/etc/init.d/gitlab start
```

----------
## **15. 常用查询** ##
- 确认应用状态
``` 
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
```

- 编译assets
```
sudo -u git -H bundle exec rake assets:precompile RAILS_ENV=production
```


----------
## **16. Nginx** ##
- Nginx安装
```
cd /home/git/gitlab/
yum -y install nginx
mkdir /etc/nginx/sites-available
mkdir /etc/nginx/sites-enabled
sudo cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab
sudo ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab
```
- 修改gitlab配置
> 增加server 127.0.0.1:8080

- 修改nginx.conf
> 增加include /etc/nginx/sites-enabled/*
> 去掉默认的80 server

- 自启动
```
systemctl enable nginx
nginx
```

----------
## **17. 确认安装情况** ##
> 编辑/home/git/gitlab-shell/config.yml
> 将url改为http://127.0.0.1:8080
> ```
cd /home/git/gitlab/
sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```
