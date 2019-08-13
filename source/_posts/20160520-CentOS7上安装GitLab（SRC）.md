---
title: CentOS7上安装GitLab（SRC）
date: 2016-05-20 13:31:11
tags:
        - CentOS
        - GitLab
        - Git
        - Redis
        - Ruby on Rails
        - Gem
        - Nginx
        - Go
        - PostgreSQL
        - SELinux
        - 2016
categories: Program
comments: false
lang:
        - zh-CN

---

[GitLab](https://about.gitlab.com/)是基于[Ruby on Rails](http://www.ruby-lang.org/en/)的一个开源的版本管理系统，其内嵌了Redis。本文在[CentOS7](https://www.centos.org/)上采用[源码方式](http://docs.gitlab.com/ce/install/installation.html)安装[GitLab汉化版](https://gitlab.com/larryli/gitlab/tags).
<!-- more -->

## **1. CentOS** ##

[CentOS7](https://www.centos.org/)（Community Enterprise Operation System, 社区企业操作系统）,截至发文时，最新版为CentOS7，本文安装的是64位。

- 安装环境：

    > VMware® Workstation 12 Pro

- 安装版本

    > [CentOS-7-x86_64-Everything-1511](https://www.centos.org/download/)

- 安装方式

    > Minimal Install



----------
## **2. 基本工具** ##
正式安GitLab及其依赖库之前，系统必须具备一些条件，需执行以下命令安装一些必要的基本工具


```
yum -y update
yum -y group install "development tools"
yum -y install vim cmake wget
yum -y install readline-devel gdbm-devel openssl-devel expat-devel sqlite-devel libyaml-devel libffi-devel libxml2-devel libxslt-devel libicu-devel system-config-firewall-tui python-devel xmlto logwatch perl-ExtUtils-CBuilder
```


> 以下如非特殊说明，均采用root登录安装



----------
## **3. Git** ##
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
## **4. Ruby** ##
- Ruby
```
cd /root
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
## **5. Go** ##
Go源码安装，需要系统本身已有旧版本。
```
yum -y install go
cd /root
wget https://storage.googleapis.com/golang/go1.5.3.linux-amd64.tar.gz
tar xvf go1.5.3.linux-amd64.tar.gz -C /usr/local/
ln -sf /usr/local/go/bin/{go,godoc,gofmt} /usr/bin/
```

----------
## **6. Git用户** ##
```
adduser --system --shell /bin/bash --comment 'GitLab' --create-home --home-dir /home/git/ git
```

## **7. PostgreSQL** ##


- PostgreSQL安装
```
yum -y install postgresql-server postgresql-contrib postgresql-devel
postgresql-setup initdb
systemctl enable postgresql
systemctl start postgresql
```

- 默认用户postgres的密码修改
```
su postgres
psql -U postgres
```
> 执行SQL语句
> ```
 ALTER USER postgres WITH PASSWORD 'postgres'
```
> 退出命令
> ```
 \q
```

- GitLab库
```
sudo -u postgres psql -d template1 -c "CREATE USER git CREATEDB;"
sudo -u postgres psql -d template1 -c "CREATE DATABASE gitlabhq_production OWNER git;"
sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"
```

- 查看创建情况
```
sudo -u git -H psql -d gitlabhq_production
```
> 执行SQL语句
> ```
SELECT true AS enabled FROM pg_available_extensions WHERE name = 'pg_trgm';
```
> 结果
> ```
  enabled
  -------
  t
  (1 row)
```

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
systemctl start redis
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
> 本文将8080改为9000

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
sudo -u git cp config/database.yml.postgresql config/database.yml
sudo -u git -H chmod o-rwx config/database.yml
```
- gitlab-shell
> 编辑Gemfile
> 改为ruby.taobao.org
> ```
sudo -u git -H bundle install --deployment --without development test mysql aws
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
sudo -u git -H bundle exec rake gitlab:setup RAILS_ENV=production
```
> 选 yes

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
sudo chmod -R ug+rwX,o-rwx /home/git/repositories/
sudo chmod -R ug-s /home/git/repositories/
sudo find /home/git/repositories/ -type d -print0 | sudo xargs -0 chmod g+s
```

- 自启动
```
sudo cp lib/support/init.d/gitlab /etc/init.d/gitlab
chkconfig gitlab on
systemctl start gitlab
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
> 增加server 127.0.0.1:9000

- 修改nginx.conf
> 增加include /etc/nginx/sites-enabled/*
> 去掉默认的80 server

- 自启动
```
systemctl enable nginx
systemctl start nginx
firewall-cmd --permanent --add-service=http
systemctl reload firewalld
```

----------

## **17. SELinux设置** ##
此时nginx还无法访问gitlab等资源，根据SELinux规则做些权限修改
```
setenforce Permissive
```
运行gitlab一段时间，并访问gitlab
```
cat /var/log/audit/audit.log
```
可以查看到一些日志
```
grep ssh-keygen /var/log/audit/audit.log | audit2allow -M postgreylocal
```
可以制定关于ssh-keygen的一些允许规则，其他“comm”同理
```
semodule -i postgreylocal.pp
```
可以添加自定义的SELinux策略


----------
## **18. 确认安装情况** ##
```
cd /home/git/gitlab/
sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```

## **19. 注意事项** ##
> 整个安装过程，在最后配置nginx代理时碰到很多权限问题，这和[SELinux](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/SELinux_Users_and_Administrators_Guide/index.html)有关，还需要研究，细化权限配置。
> 前文中，端口9000默认在SELinux里开放，故直接使用，后续还应保持8080端口并添加相应权限，避免与其他应用的9000端口冲突。
> 查看端口开放情况
> ```
semanage port -l | grep http
```
> 安装semanage
> ```
yum -y install policycoreutils-python
```



- [CentOS SELinux](https://wiki.centos.org/zh/HowTos/SELinux?highlight=%28selinux%29)
- [SELinux Booleans](https://wiki.centos.org/TipsAndTricks/SelinuxBooleans)
- [EPEL information](http://www.thegeekstuff.com/2012/06/enable-epel-repository/)
