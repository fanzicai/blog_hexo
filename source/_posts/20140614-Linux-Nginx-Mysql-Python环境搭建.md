---
title: Linux-Nginx-Mysql-Python环境搭建
date: 2014-06-14 13:35:53
tags:
        - 2014
        - Linux
        - MySQL
        - Nginx
        - Python
        - Pylons
        - SuSE
categories:
        - Program
lang:
        - zh-CN

---
本文在Linux环境下自动部署Nginx、MySQL、Pylons，并自动创建myweb测试用工程，可直接对外提供Web服务。

<!-- more -->

## **1. 软件包** ##
- 核心包
> ```
nginx-1.4.6.tar.gz
mysql-5.1.53-linux-i686-glibc23.tar.gz
Python-2.7.5.tar.bz2
MySQL-python-1.2.5.tar
```

- 辅助包
> ```
openssl-1.0.1f.tar.gz
pcre-8.34.tar.bz2
PyXML-0.8.4.tar.gz
readline-6.2.4.1.tar.gz
setuptools-3.3.tar.gz
sqlite-autoconf-3080002.tar.gz
```

- 其他包
> ```
python2.tar
```
> 该包为提前在某台Linux机器上安装好的Python目录的压缩包，安装路径为
> ```
/usr/local/python2
```
> 制作过程：
> 1、源码安装Python2.7.5
> 2、在该python环境下，easy_install安装pylons


----------
## **2. 辅助文件** ##
- mysql.cfg
> ```
# Example MySQL config file for very large systems.
#
# This is for a large system with memory of 1G-2G where the system runs mainly
# MySQL.
#
# You can copy this file to
# /etc/my.cnf to set global options,
# mysql-data-dir/my.cnf to set server-specific options (in this
# installation this directory is /usr/local/mysql/data) or
# ~/.my.cnf to set user-specific options.
#
# In this file, you can use all long options that a program supports.
# If you want to know which options a program supports, run the program
# with the "--help" option.

# The following options will be passed to all MySQL clients
[client]
#password	= your_password
port		= 3306
socket		= /tmp/mysql.sock

# Here follows entries for some specific programs

# The MySQL server
[mysqld]
port		= 3306
socket		= /tmp/mysql.sock
skip-locking
key_buffer_size = 384M
max_allowed_packet = 1M
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
# Try number of CPU's*2 for thread_concurrency
thread_concurrency = 8
default-character-set=utf8

# Don't listen on a TCP/IP port at all. This can be a security enhancement,
# if all processes that need to connect to mysqld run on the same host.
# All interaction with mysqld must be made via Unix sockets or named pipes.
# Note that using this option without enabling named pipes on Windows
# (via the "enable-named-pipe" option) will render mysqld useless!
# 
#skip-networking

# Replication Master Server (default)
# binary logging is required for replication
log-bin=mysql-bin

# required unique id between 1 and 2^32 - 1
# defaults to 1 if master-host is not set
# but will not function as a master if omitted
server-id	= 1

# Replication Slave (comment out master section to use this)
#
# To configure this host as a replication slave, you can choose between
# two methods :
#
# 1) Use the CHANGE MASTER TO command (fully described in our manual) -
#    the syntax is:
#
#    CHANGE MASTER TO MASTER_HOST=<host>, MASTER_PORT=<port>,
#    MASTER_USER=<user>, MASTER_PASSWORD=<password> ;
#
#    where you replace <host>, <user>, <password> by quoted strings and
#    <port> by the master's port number (3306 by default).
#
#    Example:
#
#    CHANGE MASTER TO MASTER_HOST='125.564.12.1', MASTER_PORT=3306,
#    MASTER_USER='joe', MASTER_PASSWORD='secret';
#
# OR
#
# 2) Set the variables below. However, in case you choose this method, then
#    start replication for the first time (even unsuccessfully, for example
#    if you mistyped the password in master-password and the slave fails to
#    connect), the slave will create a master.info file, and any later
#    change in this file to the variables' values below will be ignored and
#    overridden by the content of the master.info file, unless you shutdown
#    the slave server, delete master.info and restart the slaver server.
#    For that reason, you may want to leave the lines below untouched
#    (commented) and instead use CHANGE MASTER TO (see above)
#
# required unique id between 2 and 2^32 - 1
# (and different from the master)
# defaults to 2 if master-host is set
# but will not function as a slave if omitted
#server-id       = 2
#
# The replication master for this slave - required
#master-host     =   <hostname>
#
# The username the slave will use for authentication when connecting
# to the master - required
#master-user     =   <username>
#
# The password the slave will authenticate with when connecting to
# the master - required
#master-password =   <password>
#
# The port the master is listening on.
# optional - defaults to 3306
#master-port     =  <port>
#
# binary logging - not required for slaves, but recommended
#log-bin=mysql-bin
#
# binary logging format - mixed recommended 
#binlog_format=mixed

# Point the following paths to different dedicated disks
#tmpdir		= /tmp/		
#log-update 	= /path-to-dedicated-directory/hostname

# Uncomment the following if you are using InnoDB tables
#innodb_data_home_dir = /usr/local/mysql/data/
#innodb_data_file_path = ibdata1:2000M;ibdata2:10M:autoextend
#innodb_log_group_home_dir = /usr/local/mysql/data/
# You can set .._buffer_pool_size up to 50 - 80 %
# of RAM but beware of setting memory usage too high
#innodb_buffer_pool_size = 384M
#innodb_additional_mem_pool_size = 20M
# Set .._log_file_size to 25 % of buffer pool size
#innodb_log_file_size = 100M
#innodb_log_buffer_size = 8M
#innodb_flush_log_at_trx_commit = 1
#innodb_lock_wait_timeout = 50

[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash
# Remove the next comment character if you are not familiar with SQL
#safe-updates

[myisamchk]
key_buffer_size = 256M
sort_buffer_size = 256M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout

```

- nginx.conf
> ```

#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            #root   html;
            #index  index.html index.htm;
            fastcgi_pass 127.0.0.1:5000;
            fastcgi_param PATH_INFO $fastcgi_script_name;
            fastcgi_param REQUEST_METHOD $request_method;
            fastcgi_param QUERY_STRING $query_string;
            fastcgi_param CONTENT_TYPE $content_type;
            fastcgi_param CONTENT_LENGTH $content_length;
            fastcgi_param  SERVER_ADDR        $server_addr;
            fastcgi_param  SERVER_PORT        $server_port;
            fastcgi_param  SERVER_NAME        $server_name;
            fastcgi_param  SERVER_PROTOCOL    $server_protocol;
            fastcgi_pass_header Authorization;
            fastcgi_intercept_errors off;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443;
    #    server_name  localhost;

    #    ssl                  on;
    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_timeout  5m;

    #    ssl_protocols  SSLv2 SSLv3 TLSv1;
    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers   on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

- development.ini
> myweb工程的配置文件
> ```
#
# myweb - Pylons development environment configuration
#
# The %(here)s variable will be replaced with the parent directory of this file
#
[DEFAULT]
debug = true
# Uncomment and replace with the address which should receive any error reports
#email_to = you@yourdomain.com
smtp_server = localhost
error_email_from = paste@localhost

[server:main]
use = egg:Flup#fcgi_thread
host = 127.0.0.1
port = 5000

[app:main]
use = egg:myweb
full_stack = true
static_files = true

cache_dir = %(here)s/data
beaker.session.key = myweb
beaker.session.secret = somesecret

# If you'd like to fine-tune the individual locations of the cache data dirs
# for the Cache data, or the Session saves, un-comment the desired settings
# here:
#beaker.cache.data_dir = %(here)s/data/cache
#beaker.session.data_dir = %(here)s/data/sessions

# WARNING: *THE LINE BELOW MUST BE UNCOMMENTED ON A PRODUCTION ENVIRONMENT*
# Debug mode will enable the interactive debugging tool, allowing ANYONE to
# execute malicious code after an exception is raised.
#set debug = false


# Logging configuration
[loggers]
keys = root, routes, myweb

[handlers]
keys = console

[formatters]
keys = generic

[logger_root]
level = INFO
handlers = console

[logger_routes]
level = INFO
handlers =
qualname = routes.middleware
# "level = DEBUG" logs the route matched and routing variables.

[logger_myweb]
level = DEBUG
handlers =
qualname = myweb

[handler_console]
class = StreamHandler
args = (sys.stderr,)
level = NOTSET
formatter = generic

[formatter_generic]
format = %(asctime)s,%(msecs)03d %(levelname)-5.5s [%(name)s] [%(threadName)s] %(message)s
datefmt = %H:%M:%S

```

- rc.nginx
> nginx启动脚本
> ```
#!/bin/sh

### BEGIN INIT INFO
# Provides: nginx
# Required-Start: 
# Should-Start: 
# Required-Stop: 
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop nginx
# Description: 
### END INIT INFO


# SamERP start/stop script.

service_startup_timeout=20

# Set some defaults
nginxdir=/usr/local/nginx

pid_file=$nginxdir/nginx-`/bin/hostname`.pid
out_file=$nginxdir/runtime.out

#
# Use LSB init script functions for printing messages, if possible
#
lsb_functions="/lib/lsb/init-functions"
if test -f $lsb_functions ; then
  . $lsb_functions
else
  log_success_msg()
  {
    echo " SUCCESS! $@"
  }
  log_failure_msg()
  {
    echo " ERROR! $@"
  }
fi

mode=$1    # start or stop
shift
other_args="$*"   # uncommon, but needed when called from an RPM upgrade action
           # Expected: "--skip-networking --skip-grant-tables"
           # They are not checked here, intentionally, as it is the resposibility
           # of the "spec" file author to give correct arguments only.

case `echo "testing\c"`,`echo -n testing` in
    *c*,-n*) echo_n=   echo_c=     ;;
    *c*,*)   echo_n=-n echo_c=     ;;
    *)       echo_n=   echo_c='\c' ;;
esac

wait_for_pid () {
  verb="$1"
  manager_pid="$2"  # process ID of the program operating on the pid-file
  i=0
  avoid_race_condition="by checking again"
  while test $i -ne $service_startup_timeout ; do
    case "$verb" in
      'created')
        # wait for a PID-file to pop into existence.
        test -s $pid_file && i='' && break
        ;;
      'removed')
        # wait for this PID-file to disappear
        test ! -s $pid_file && i='' && break
        ;;
      *)
        echo "wait_for_pid () usage: wait_for_pid created|removed manager_pid"
        exit 1
        ;;
    esac
    echo $echo_n ".$echo_c"
    i=`expr $i + 1`
    sleep 1
  done

  if test -z "$i" ; then
    log_success_msg
    return 0
  else
    log_failure_msg
    return 1
  fi
}

case "$mode" in
  'start')
    # Start daemon
    echo $echo_n "Starting Nginx..."
    echo $echo_n $echo_c

    if test -n "$other_args"
    then
      log_failure_msg "Nginx does not support options '$other_args'"
      exit 1
    fi

    if test -s "$pid_file"
    then
      log_failure_msg "Nginx process is already running, stop it first"
      #$0 stop
      exit 1
    fi

    $nginxdir/sbin/nginx &
    nginxpid=`expr $! + 2`
    echo $nginxpid > $pid_file

    wait_for_pid created $!; return_value=$?

    exit $return_value
    ;;

  'stop')
    # Stop daemon.

    if test -s "$pid_file"
    then
      nginx_pid=`cat $pid_file`
      
      if (kill -0 $nginx_pid 2>/dev/null)
      then
        echo $echo_n "Shutting down Nginx..."
        kill $nginx_pid
        rm -f $pid_file $out_file
        wait_for_pid removed "$nginx_pid"; return_value=$?
      else
        log_failure_msg "Nginx process #$nginx_pid is not running!"
        rm -f $pid_file $out_file
      fi
      exit $return_value
    else
      log_failure_msg "Nginx PID file could not be found!"
    fi
    ;;

  'restart')
    # Stop the service and regardless of whether it was
    # running or not, start it again.
    if $0 stop  $other_args; then
      $0 start $other_args
    else
      log_failure_msg "Failed to stop running nginx, so refusing to try to start."
      exit 1
    fi
    ;;

    *)
      # usage
      echo "Usage: $0  {start|stop|restart}  [ Nginx server options ]"
      exit 1
    ;;
esac

exit 0

```

- rc.myweb
> myweb启动脚本
> ```
#!/bin/sh

### BEGIN INIT INFO
# Provides: myweb
# Required-Start: 
# Should-Start: 
# Required-Stop: 
# Default-Start:  2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: start and stop myweb
# Description: 
### END INIT INFO


# myweb start/stop script.

service_startup_timeout=20

# Set some defaults
mywebdir=/root/myweb
pylonsdir=/usr/local/python2

pid_file=$mywebdir/myweb-`/bin/hostname`.pid
out_file=$mywebdir/runtime.out

#
# Use LSB init script functions for printing messages, if possible
#
lsb_functions="/lib/lsb/init-functions"
if test -f $lsb_functions ; then
  . $lsb_functions
else
  log_success_msg()
  {
    echo " SUCCESS! $@"
  }
  log_failure_msg()
  {
    echo " ERROR! $@"
  }
fi

mode=$1    # start or stop
shift
other_args="$*"   # uncommon, but needed when called from an RPM upgrade action
           # Expected: "--skip-networking --skip-grant-tables"
           # They are not checked here, intentionally, as it is the resposibility
           # of the "spec" file author to give correct arguments only.

case `echo "testing\c"`,`echo -n testing` in
    *c*,-n*) echo_n=   echo_c=     ;;
    *c*,*)   echo_n=-n echo_c=     ;;
    *)       echo_n=   echo_c='\c' ;;
esac

wait_for_pid () {
  verb="$1"
  manager_pid="$2"  # process ID of the program operating on the pid-file
  i=0
  avoid_race_condition="by checking again"
  while test $i -ne $service_startup_timeout ; do
    case "$verb" in
      'created')
        # wait for a PID-file to pop into existence.
        test -s $pid_file && i='' && break
        ;;
      'removed')
        # wait for this PID-file to disappear
        test ! -s $pid_file && i='' && break
        ;;
      *)
        echo "wait_for_pid () usage: wait_for_pid created|removed manager_pid"
        exit 1
        ;;
    esac
    echo $echo_n ".$echo_c"
    i=`expr $i + 1`
    sleep 1
  done

  if test -z "$i" ; then
    log_success_msg
    return 0
  else
    log_failure_msg
    return 1
  fi
}

case "$mode" in
  'start')
    # Start daemon
    echo $echo_n "Starting myweb..."
    echo $echo_n $echo_c

    if test -n "$other_args"
    then
      log_failure_msg "myweb does not support options '$other_args'"
      exit 1
    fi

    if test -s "$pid_file"
    then
      log_failure_msg "myweb process is already running, stop it first"
      #$0 stop
      exit 1
    fi

    export PATH=$pylonsdir/bin:$PATH
    cd $mywebdir
    paster serve --reload development.ini &
    echo $! > $pid_file

    wait_for_pid created $!; return_value=$?

    exit $return_value
    ;;

  'stop')
    # Stop daemon.

    if test -s "$pid_file"
    then
      myweb_pid=`cat $pid_file`
      
      if (kill -0 $myweb_pid 2>/dev/null)
      then
        echo $echo_n "Shutting down myweb..."
        kill $myweb_pid
        rm -f $pid_file $out_file
        wait_for_pid removed "$myweb_pid"; return_value=$?
      else
        log_failure_msg "myweb process #$myweb_pid is not running!"
        rm -f $pid_file $out_file
      fi
      exit $return_value
    else
      log_failure_msg "myweb PID file could not be found!"
    fi
    ;;

  'restart')
    # Stop the service and regardless of whether it was
    # running or not, start it again.
    if $0 stop  $other_args; then
      $0 start $other_args
    else
      log_failure_msg "Failed to stop running myweb, so refusing to try to start."
      exit 1
    fi
    ;;

    *)
      # usage
      echo "Usage: $0  {start|stop|restart}  [ myweb server options ]"
      exit 1
    ;;
esac

exit 0

```


----------
## **3. 安装脚本** ##
- setup
> ```
#!/bin/sh

FLAG_CP=
FLAG_CPR=-r
FLAG_TAR=-xf
FLAG_CFG=\>/dev/null
FLAG_MAKE=\>/dev/null
FLAG_INST=\>/dev/null

toolsdir=/root/tools
builddir=/root/tools/build

#创建相关文件夹
echo "正在创建相关文件夹......"
mkdir $toolsdir
mkdir $builddir

#拷贝所有文件到指定目录
echo "正在拷贝文件到指定目录......"
cp $FLAG_CP ./*.* $toolsdir/

#安装openssl
echo "正在安装openssl......"
cd $builddir/
tar $FLAG_TAR $toolsdir/openssl-*.*
cd openssl*/
eval ./config $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE
ldconfig

#安装sqlite
echo "正在安装sqlite......"
cd $builddir/
tar $FLAG_TAR $toolsdir/sqlite-*.*
cd sqlite*/
eval ./configure $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE
#ldconfig

#安装zlib和python
echo "正在安装zlib......"
cd $builddir/
tar $FLAG_TAR $toolsdir/Python-*.*
cd Python*/Modules/zlib/
eval ./configure $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE
echo "正在安装python......"
cd $builddir/
cd Python*/
eval ./configure --prefix=/usr/local/python2  $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE
export PATH=/usr/local/python2/bin:$PATH

#安装setuptools
echo "正在安装setuptools......"
cd $builddir/
tar $FLAG_TAR $toolsdir/setuptools-*.*
cd setuptools*/
eval python setup.py install $FLAG_INST

#安装readline
echo "正在安装readline......"
cd $builddir/
tar $FLAG_TAR $toolsdir/readline-*.*
cd readline*/
eval python setup.py install $FLAG_INST

#安装PyXML
echo "正在安装PyXML......"
cd $builddir/
tar $FLAG_TAR $toolsdir/PyXML-*.*
cd PyXML*/
eval python setup.py install $FLAG_INST

#安装mysql
echo "正在安装mysql......"
cd $builddir/
tar $FLAG_TAR $toolsdir/mysql-*.*
cp $FLAG_CPR mysql*/ /usr/local/mysql
cd /usr/local/mysql
groupadd mysql
useradd -g mysql mysql
chown -R root:mysql .
./scripts/mysql_install_db --user=mysql
echo "正在启动mysql......"
#启动mysql、并修改root密码
cp $toolsdir/mysql.cfg /etc/my.cnf
./support-files/mysql.server start
./bin/mysqladmin -u root password 'hahaha'
chown -R mysql data
#指定mysql为开机启动
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
#update-rc.d mysql defaults
chkconfig -a mysql
#链接必要的lib*.so文件
ln -s /usr/local/mysql/lib/libmysqlclient_r.so.16 /usr/lib/

#安装pcre
echo "正在安装pcre......"
cd $builddir/
tar $FLAG_TAR $toolsdir/pcre-*.*
cd pcre*/
eval ./configure $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE

#安装nginx
echo "正在安装nginx......"
cd $builddir/
tar $FLAG_TAR $toolsdir/nginx-*.*
cd nginx*/
eval ./configure $FLAG_CFG
eval make $FLAG_MAKE
eval make install $FLAG_MAKE
cp -f $toolsdir/nginx.conf /usr/local/nginx/conf/
cp -f $toolsdir/rc.nginx   /etc/init.d/nginx
chkconfig -a nginx
echo "正在启动nginx......"
service nginx start

#拷贝pylons
echo "正在部署pylons......"
cd $builddir/
tar $FLAG_TAR $toolsdir/python2*.*
cp -fr python2/ /usr/local/

#echo "是否在/root目录下创建基本工程myweb,(Y/y or N/n)?"
#read tmp
#创建基本工程myweb
cd /root
paster create -t pylons myweb
cp -f $toolsdir/development.ini /root/myweb/
cp    $toolsdir/rc.myweb        /etc/init.d/myweb
chkconfig -a myweb
echo "正在启动myweb......"
service myweb start
echo "安装过程全部结束，可以通过nginx访问myweb......"

```


----------

## **4. 说明文件** ##
- readme
> ```
﻿lnmp
    l - linux
    n - nginx
    m - mysql
    p - python/pylons
    
target
    os: suse linux
    base on: gcc g++ bzip ncurses-devel
    
setup
    tar xvf lnmp_tools.tar.gz
    cd lnmp_tools/
    ./setup
    
test
    myweb: the sample project

```


----------
