---
title: GitLab自动备份by crond
date: 2016-08-09 13:54:08
tags:
        - 2016
        - GitLab
        - Crond
categories:
        - Program
comments: false
lang:
        - zh-CN

---
本文涉及的GitLab为中文版，采用[fanzi/gitlab-cn](https://hub.docker.com/r/fanzi/gitlab-cn/)创建。
可参见[Dockerfile之fanzi/gitlab-cn](http://www.fanzicai.com/blog_hexo/Program/20160602-Dockerfile%E4%B9%8Bfanzi-gitlab-cn.html)

<!-- more -->

## **1. 备份** ##
- 在旧GitLab服务器上执行
> ```
cd /home/git/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```

- 备份文件所在目录
> ```
cd /home/git/gitlab/tmp/backups/
```

----------

## **2. 还原** ##
> 将原GitLab服务器的备份文件（*.tar）拷贝到新GitLab服务器的备份目录（同旧服务器）
> 重要！修改脚本文件
> ```
vim /home/git/gitlab/lib/tasks/gitlab/db.rask
```
> 将
> ```
tables.each { |t| connection.execute("DROP TABLE IF EXISTS #{t} CASCADE") }
```
> 改为
> ```
tables.each { |t| connection.execute("DROP TABLE IF EXISTS `#{t}` CASCADE") }
```
> 还原命令
> ```
cd /home/git/gitlab
sudo -u git -H bundle exec rake gitlab:backup:restore RAILS_ENV=production 
```
> 执行还原命令之前，须执行初始化并关闭gitlab服务
> ```
docker exec -it gitlab-new /bin/bash -c /home/git/gitlab-init.sh
/etc/init.d/gitlab stop
```

----------

## **3. 附录** ##
- docker mysql
> ```
docker run --restart always --name mysql-new -e MYSQL_ROOT_PASSWORD=123 -d mysql/mysql-server:latest --character-set-server=utf8 --collation-server=utf8_general_ci 
```
> 进入mysql控制台
```
docker exec -it mysql-new /bin/bash
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

- docker gitlab
> ```
docker run -it --detach --restart always --link mysql-new:mysql -p 80:80 -p 22:22 --name gitlab-new fanzi/gitlab-cn
```

----------

## **4. 自动化** ##
- docker gitlab backup脚本
> /home/git/gitlab-backup.sh
> ```
#!/bin/bash
cd /home/git/gitlab
bundle exec rake gitlab:backup:create RAILS_ENV=production
```
- crond配置文件
> /root/docker-gitlab-bak/gitlab.cron
> ```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
0 0 * * * docker exec -it gitlab /bin/bash /home/git/gitlab-backup.sh
10 0 * * * docker cp gitlab:/home/git/gitlab/tmp/backups /root/docker-gitlab-bak/
```
- crond服务
> ```
crontab -s /root/docker-gitlab-bak/gitlab.cron
systemctl enable crond
systemctl start crond
```

----------

## **5. 其他方式** ##
> 主要指采用安装包安装的gitlab
> ```
/opt/gitlab/bin/gitlab-rake gitlab:backup:create
gitlab-rake gitlab:backup:restore BACKUP=*******  #编号
```

----------
