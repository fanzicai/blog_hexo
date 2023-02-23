---
title: Dockerfile之fanzi/gitlab-cn
date: 2016-06-02 09:02:37
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

Dockerfile是创建docker image的一种常用方式。本文采用dockerfile创建[gitlab-cn image](https://hub.docker.com/r/fanzi/gitlab-cn/)。该image基于Centos Image 7创建，并与mysql image联合使用。
<!-- more -->

## **1. 准备工作** ##

可参考[CentOS7上安装Docker及GitLab](http://fanzicai.github.io/Program/2016/05/25/CentOS7%E4%B8%8A%E5%AE%89%E8%A3%85Docker%E5%8F%8AGitLab.html)一文，先行安装CentOS、Docker。

- Docker MySQL
Oracle官方已提供MySQL的[Docker Image](https://hub.docker.com/r/mysql/mysql-server/)。
> docker -v 权限
```
chcon -Rt svirt_sandbox_file_t /root/docker-mysql-data
```
> 创建
```
docker run --restart always --name mysql -e MYSQL_ROOT_PASSWORD=123 -v /root/docker-mysql-data:/var/lib/mysql -p 3306:3306 -p 33060:33060 -d mysql/mysql-server:5.7 --character-set-server=utf8 --collation-server=utf8_general_ci
```
> 进入mysql控制台
```
docker exec -it mysql /bin/bash
mysql -uroot -p123
```
> 创建git用户及database，本文git用户密码设置为git
```
CREATE USER 'git'@'%' IDENTIFIED BY 'git';
CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, CREATE TEMPORARY TABLES, DROP, INDEX, ALTER, LOCK TABLES ON `gitlabhq_production`.* TO 'git'@'%';
flush privileges;
\q
```
----------
## **2. gitlab-cn Dockerfile** ##
以下为Dockfile的内容
```
FROM centos:centos7
MAINTAINER fanzi "fanzicai@yahoo.com"

RUN yum -y install wget \
 && wget http://repo.mysql.com//mysql57-community-release-el7-8.noarch.rpm \
 && rpm -Uvh mysql57-community-release-el7-8.noarch.rpm \
 && yum -y update \
 && yum -y group install "development tools" \
 && yum -y install epel-release \
 && yum -y install sudo vim cmake mysql mysql-devel openssh-server go ruby redis nodejs nginx readline-devel gdbm-devel openssl-devel expat-devel sqlite-devel libyaml-devel libffi-devel libxml2-devel libxslt-devel libicu-devel python-devel xmlto logwatch perl-ExtUtils-CBuilder

RUN adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git \
 && chmod -R go+rx /home/git

WORKDIR /home/git
RUN wget https://www.kernel.org/pub/software/scm/git/git-2.8.3.tar.gz \
 && wget https://cache.ruby-lang.org/pub/ruby/2.3/ruby-2.3.1.tar.gz \
 && wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz \
 && wget  https://gitlab.com/larryli/gitlab/repository/archive.tar.gz?ref=v8.8.0.zh1 \
 && tar xvf git-2.8.3.tar.gz \
 && tar xvf ruby-2.3.1.tar.gz \
 && tar xvf go1.5.3.linux-amd64.tar.gz -C /usr/local/ \
 && tar xvf archive.tar.gz?ref=v8.8.0.zh1 \
 && mv gitlab-v8.8.0.zh1* gitlab \
 && ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/bin/ \
# Git install
 && cd git-2.8.3 \
 && ./configure --prefix=/usr \
 && make \
 && make install \
 sudo -u git -H "/usr/bin/git" config --global core.autocrlf "input" \
# Ruby update
 && cd ../ruby-2.3.1/ \
 && ./configure --prefix=/usr --disable-install-rdoc \
 && make \
 && make install \
# Bundler
 && gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/ \
 && gem install bundler \
 && bundle config mirror.https://rubygems.org https://ruby.taobao.org \
# Redis config & start
 && echo 'unixsocket /var/run/redis/redis.sock' >> /etc/redis.conf \
 && echo 'unixsocketperm 770' >> /etc/redis.conf \
 && chown redis:redis /var/run/redis \
 && chmod 755 /var/run/redis \
 && usermod -aG redis git \
 && (redis-server /etc/redis.conf &)

# GitLab
WORKDIR /home/git/gitlab/
RUN cp config/gitlab.yml.example config/gitlab.yml \
 && cp config/secrets.yml.example config/secrets.yml \
 && chmod 0600 config/secrets.yml \
 && chown -R git log/ \
 && chown -R git tmp/ \
 && chmod -R u+rwX,go-w log/ \
 && chmod -R u+rwX tmp/ \
 && chmod -R u+rwX tmp/pids/ \
 && chmod -R u+rwX tmp/sockets/ \
 && mkdir public/uploads/ \
 && chmod 0700 public/uploads \
 && chmod -R u+rwX builds/ \
  && chmod -R u+rwX shared/artifacts/ \
 && cp config/unicorn.rb.example config/unicorn.rb \
 && cp config/initializers/rack_attack.rb.example config/initializers/rack_attack.rb \
 && git config --global core.autocrlf input \
 && git config --global gc.auto 0 \
 && cp config/resque.yml.example config/resque.yml \
 && cp config/database.yml.mysql config/database.yml \
 && sed -i 's/"secure password"/git/' config/database.yml \
 && sed -i 's/# host: localhost/host: mysql/' config/database.yml \
 && sed -i 's/rubygems.org/ruby.taobao.org/' Gemfile \
 && bundle install --deployment --without development test postgres aws \
 && bundle exec rake gitlab:shell:install REDIS_URL=unix:/var/run/redis/redis.sock RAILS_ENV=production \
 && chown -R git:git /home/git/repositories/ \
 && chmod -R ug+rwX,o-rwx /home/git/repositories/ \
 && chmod -R ug-s /home/git/repositories/ \
 && find /home/git/repositories/ -type d -print0 | xargs -0 chmod g+s \
 && chmod 700 /home/git/gitlab/public/uploads \
 && chown -R git:git config/ log/ \
# GitLab rc.file
 && cp lib/support/init.d/gitlab /etc/init.d/gitlab

# Nginx config
WORKDIR /home/git/gitlab/
RUN mkdir /etc/nginx/sites-available \
 && mkdir /etc/nginx/sites-enabled \
 && cp lib/support/nginx/gitlab /etc/nginx/sites-available/gitlab \
 && ln -s /etc/nginx/sites-available/gitlab /etc/nginx/sites-enabled/gitlab \
 && sed -i '20a\  server 127.0.0.1:8080;' /etc/nginx/sites-available/gitlab \
 && sed -i '35,54d' /etc/nginx/nginx.conf \
 && sed -i '33a\    include /etc/nginx/sites-enabled/*;' /etc/nginx/nginx.conf

WORKDIR /home/git
# Gitlab-Workhorse
RUN git clone https://gitlab.com/gitlab-org/gitlab-workhorse.git \
 && cd gitlab-workhorse/ \
 && git checkout v0.7.2 \
 && make \
 && sshd-keygen

EXPOSE 80 22

VOLUME ["/tmp/gitlab-cn"]

ADD ./gitlab-init.sh /home/git/gitlab-init.sh
ADD ./gitlab-run.sh /home/git/gitlab-run.sh

RUN chmod +x /home/git/gitlab-init.sh
RUN chmod +x /home/git/gitlab-run.sh

ENTRYPOINT ["/home/git/gitlab-run.sh"]

```

其中，gitlab-init.sh脚本内容如下：
```
#!/bin/bash

cd /home/git/gitlab
bundle exec rake gitlab:setup RAILS_ENV=production force=yes
bundle exec rake assets:precompile RAILS_ENV=production
chown -R git:git /home/git/
sudo -u git -H "/usr/bin/git" config --global core.autocrlf "input"

```
gitlab-run.sh脚本内容如下：
```
#!/bin/bash

#gitlab start
/etc/init.d/gitlab start &

#nginx start
nginx &

#sshd start
/usr/sbin/sshd

#redis start
sudo -u redis -H redis-server /etc/redis.conf

```
----------
## **3. Image Build** ##
本Image已上传<https://hub.docker.com/r/fanzi/gitlab-cn/>
```
docker build --rm=true -t fanzi/gitlab-cn .
```
----------
## **4. Image USE** ##
- docker -v 权限
> ```
chcon -Rt svirt_sandbox_file_t /root/docker-gitlab-bak/backups
```
- 创建容器
```
docker run -it --detach --restart always  -v /root/docker-gitlab-bak/backups:/home/git/gitlab/tmp/backups --link mysql:mysql -p 80:80 -p 22:22 --name gitlab fanzi/gitlab-cn
```
- 初始化数据库
> 确保mysql容器运行，只在创建容器后运行一次
> ```
docker exec -it gitlab /bin/bash -c /home/git/gitlab-init.sh
```

- 修改域名
> /home/git/gitlab/config/gitlab.yml
> ```
url: 192.168.1.*
```
