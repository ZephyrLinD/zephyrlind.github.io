# CentOS7安装LNMP记录


## 前言

曾经沉迷Ubuntu的我终于真香了，转移全线阵地到了CentOS7，关闭主网站了一阵子，然后开始熟悉CentOS下的环境配置、服务器配置。难度比Ubuntu真的难了好多，也复杂了不少，但是有了上次的经验，参考了很多书和博客，总结出了一套适合自己的路线。

之后还会学习Nginx的结构，配置负载均衡，然后读读Nginx的源代码。当然了，顺便还会复习一下计算机网络早就还给老师的知识。毕竟掌握的计算机科学硬核知识越多，职业危机越不那么严重......

在本文，源码文件保存在`/usr/local/src`中

安装默认在`/usr/local/`下

## 安装配置MySQL服务

`MySQL`在CentOS中被弄成收费的了的原因（貌似），默认`yum`安装`MySQL`会安装成`MariaDB`，虽然`MariaDB`是`MySQL`的子集，但是为了实际操作和以后的就业，还是选择了`MySQL`。

### 安装CMake

MySQL需要用CMake进行安装，大家可以到[CMake下载页面](https://cmake.org/download/)寻找自己需要的下载版本，在下面的操作实例中我将以自己为例子：

```shell
yum install gcc gcc-c++ ncurses ncurses-devel bison
mkdir /usr/local/src
cd /usr/local/src
wget https://github.com/Kitware/CMake/releases/download/v3.15.3/cmake-3.15.3.tar.gz
tar zxvf cmake-3.15.3
cd cmake-3.15.3
./configure
make && make install
```

不出意外，`CMake`就安装完了

### 源代码编译安装MySQL

MySQL除了源代码安装还有Yum源安装，具体方法和源代码的地址都可以在[下载页面](https://dev.mysql.com/downloads/mysql/)找到，我选择的是MySQL5.7，不为啥，因为MySQL8安装需要高版本GCC我懒得升级了（别问我怎么知道的，呵呵，感谢CentOS让我一遍遍熟练各种软件的安装和配置），而且我的网站规模不大，MySQL5.7作为稳定的版本也是个不错的选择。

```bash
cd ..
wget https://dev.mysql.com/get/Downloads/MySQL-5.7/mysql-boost-5.7.27.tar.gz
tar zxvf mysql-boost-5.7.27.tar.gz
cd mysql-boost-5.7.27
```

CMake需要单独创建一个文件夹来存放编译出来的文件。

```bash
mkdir build 
cd build
```

建议先把cmake的配置代码复制到文本编辑器里面改成自己需要的，删掉注释和`\`后的空格，在粘贴到终端里面执行即可

```bash
cmake .. \
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql \                         //指定mysql数据库安装目录
-DMYSQL_UNIX_ADDR=/usr/local/mysql/mysql.sock \                   //连接文件位置
-DSYSCONFDIR=/etc \                                               //指定配置文件目录
-DSYSTEMD_PID_DIR=/usr/local/mysql \                              //进程文件目录
-DDEFAULT_CHARSET=utf8  \                                         //指定默认使用的字符集编码
-DDEFAULT_COLLATION=utf8_general_ci \                             //指定默认使用的字符集校对规则
-DWITH_INNOBASE_STORAGE_ENGINE=1 \                                //存储引擎
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \                                 //存储引擎
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \                               //存储引擎
-DWITH_PERFSCHEMA_STORAGE_ENGINE=1 \                              //存储引擎
-DMYSQL_DATADIR=/usr/local/mysql/data \                           //数据库文件
-DWITH_BOOST=../boost \                                           //指定Boost库的位置，mysql5.7必须添加该参数
-DWITH_SYSTEMD=1
```

最后：

```shell
make && make install
```

**P.S. 如果在CMAKE的过程中有报错（报错多是环境包安装错误），当报错解决后，需要把源码目录中的CMakeCache.txt文件删除，然后再重新CMAKE，否则错误依旧**

**P.P.S.. make过程非常费时间，建议在tmux下或者开个screen执行**

### 各种配置

- 首先，我们需要在系统中创建一个名为mysql的用户，专门用于负责运行MySQL数据库。请记得要把这类账户的Bash终端设置成nologin解释器，避免被黑，从而提高系统安全性。

  ```bash
  useradd mysql -s /sbin/nologin
  ```

- 该目录的所有者和所属组身份修改为mysql。并且将MySQL的配置文件`my.ini`的所有者和所有组身份修改为mysql。

  ```bash
  chown -Rf mysql:mysql /usr/local/mysql
  chown -Rf mysql:mysql /etc/my.cnf
  ```

- 之后按照如下修改my.cnf

  ```bash
  $ vim /etc/my.cnf
  [client]
  port = 3306
  default-character-set=utf8
  socket = /usr/local/mysql/mysql.sock
  [mysql]
  port = 3306
  default-character-set=utf8
  socket = /usr/local/mysql/mysql.sock
  [mysqld]
  user = mysql
  basedir = /usr/local/mysql
  datadir = /usr/local/mysql/data
  port = 3306
  character_set_server=utf8
  pid-file = /usr/local/mysql/mysqld.pid
  socket = /usr/local/mysql/mysql.sock
  server-id = 1
  sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_AUTO_VALUE_ON_ZERO,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,PIPES_AS_CONCAT,ANSI_QUOTES
  ```

- 设置环境变量
  
  ```bash
  echo 'PATH=/usr/local/mysql/bin:/usr/local/mysql/lib:$PATH' >> /etc/profile
  echo 'export PATH' >> /etc/profile
  source /etc/profile   
  ```

- 初始化数据库

  ```bash
  cd /usr/local/mysql/
  bin/mysqld \
  --initialize-insecure \
  --user=mysql \
  --basedir=/usr/local/mysql \
  --datadir=/usr/local/mysql/data
  ```

- 添加到系统服务

  ```bash
  cp usr/lib/systemd/system/mysqld.service /usr/lib/systemd/system/
  systemctl daemon-reload
  systemctl enable mysqld       # 自启动
  systemctl start mysqld
  ```

  此时如果没有报错的话说明已经启动了，如果报错就按错误的代码查。

### 访问MySQL数据库和NaviCat远程连接

首先用root用户直接免密码进入MySQL数据库修改一下密码：

```bash
$ mysql
mysql>SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的密码');
```
然后就可以执行`show databases;`等等指令看一下是否出错了，一般是不会出错的。

在mysql终端内开启远程权限

```bash
mysql>grant all privileges on *.* to 'root'@'%' identified by '你的密码' with grant option;
```
第一个\*代表所有数据库，第二\*代表所有表，赋予root权限 “%”代表所有服务器终端，也可设为IP地址

之后，去服务器管理页面打开3306的端口就可以远程连接啦。

## 安装Nginx高性能静态服务器

### 首先安装所需的组件：

```shell
yum install pcre-devel zlib-devel openssl-devel
```

### 在安装部署好具有依赖关系的软件包之后，创建一个用于执行Nginx服务程序的账户。账户名称可以自定义，但一定别忘记，因为在后续需要调用：

```bash
cd ../src 
useradd www -s /sbin/nologin
```

### 之后来到[Nginx官网](http://nginx.org)的[下载页面](http://nginx.org/en/download.html)，找到需要的版本，存储链接，在服务器中下载并解压：

```bash
wget http://nginx.org/download/nginx-1.16.1.tar.gz
tar zxvf nginx-1.16.1.tar.gz
cd nginx-1.16.1
```

### 接下来，编译、安装

```bash
./configure \
--prefix=/usr/local/nginx \
--user=www \
--group=www \
--with-file-aio \
--with-http_stub_status_module \
--with-http_gzip_static_module \
--with-http_flv_module \
--with-http_ssl_module
```

如果出现`configure error: Please reinstall the libzip dirtribution`或者`configure error: system libzip must be upgraded to version >= 0.11`。使用Yum最新版只到0.10，不能达到要求。

- 先删除libzip
  
  ```bash
  yum remove libzip libzip-devel
  ```

- 下载安装并且手动编译（1.5之前configure，1.5之后cmake，以1.5为例）
  
  首先进入[libzip官网](https://libzip.org)，选择Download进入[下载页面](https://libzip.org/download/)，复制链接粘贴至终端，执行以下操作：

  ```bash
  cd ..
  wget https://libzip.org/download/libzip-1.5.2.tar.gz
  tar zxvf libzip-1.5.2.tar.gz
  cd libzip-1.5.2
  mkdir build
  cd build
  cmake ..
  make && make install
  ```

- 之后继续执行Nginx的configure命令即可。

在之后，安装

```bash
make && make install
```

### 添加到系统service中

要想启动Nginx服务程序以及将其加入到开机启动项中，也需要有脚本文件。在安装完Nginx软件包之后默认并没有为用户提供脚本文件，因此我从《Linux就该这样学》这本书找到了一份可用的启动脚本文件，大家只需在/etc/rc.d/init.d目录中创建脚本文件并直接复制下面的脚本内容即可：

- 首先，打开文件：

  ```bash
  vim /etc/rc.d/init.d/nginx
  ```

- 之后，粘贴如下文本进入文件：

  ```shell
  #!/bin/bash
  # nginx - this script starts and stops the nginx daemon
  # chkconfig: - 85 15
  # description: Nginx is an HTTP(S) server, HTTP(S) reverse \
  # proxy and IMAP/POP3 proxy server
  # processname: nginx
  # config: /etc/nginx/nginx.conf
  # config: /usr/local/nginx/conf/nginx.conf
  # pidfile: /usr/local/nginx/logs/nginx.pid
  # Source function library.
  . /etc/rc.d/init.d/functions
  # Source networking configuration.
  . /etc/sysconfig/network
  # Check that networking is up.
  [ "$NETWORKING" = "no" ] && exit 0 
  nginx="/usr/local/nginx/sbin/nginx"
  prog=$(basename $nginx)
  NGINX_CONF_FILE="/usr/local/nginx/conf/nginx.conf"
  [ -f /etc/sysconfig/nginx ] && . /etc/sysconfig/nginx
  lockfile=/var/lock/subsys/nginx
  make_dirs() {
  # make required directories  
  user=`$nginx -V 2>&1 | grep "configure arguments:" | sed 's/[^*]*--user=\([^ ]*\).*/\1/g' -`
          if [ -z "`grep $user /etc/passwd`" ]; then
                  useradd -M -s /bin/nologin $user
          fi
  options=`$nginx -V 2>&1 | grep 'configure arguments:'`
  for opt in $options; do
          if [ `echo $opt | grep '.*-temp-path'` ]; then
                  value=`echo $opt | cut -d "=" -f 2`
                  if [ ! -d "$value" ]; then
                          # echo "creating" $value
                          mkdir -p $value && chown -R $user $value
                 fi
          fi
  done
  }
  start() {
      [ -x $nginx ] || exit 5
      [ -f $NGINX_CONF_FILE ] || exit 6
      make_dirs
      echo -n $"Starting $prog: "
      daemon $nginx -c $NGINX_CONF_FILE
      retval=$?
      echo
      [ $retval -eq 0 ] && touch $lockfile
      return $retval
  }
  stop() {
      echo -n $"Stopping $prog: "
      killproc $prog -QUIT
      retval=$?
      echo
      [ $retval -eq 0 ] && rm -f $lockfile
      return $retval
  }
  restart() {
      #configtest || return $?
      stop
      sleep 1
      start
  }
  reload() {
      #configtest || return $?
      echo -n $"Reloading $prog: "
      killproc $nginx -HUP
      RETVAL=$?
      echo
  }
  force_reload() {
      restart
  }
  configtest() {
      $nginx -t -c $NGINX_CONF_FILE
  }
  rh_status() {
      status $prog
  }
  rh_status_q() {
      rh_status >/dev/null 2>&1
  }
  case "$1" in
  start)
          rh_status_q && exit 0
          $1
          ;;
  stop)
          rh_status_q || exit 0
          $1
          ;;
  restart|configtest)
  $1
  ;;
  reload)
          rh_status_q || exit 7
          $1
          ;;
  force-reload)
          force_reload
          ;;
  status)
          rh_status
          ;;
  condrestart|try-restart)
          rh_status_q || exit 0
          ;;
  *)
  echo $"Usage: $0 {start|stop|status|restart|condrestart|try-restart|reload|force-reload|configtest}"
  exit 2
  esac
  ```

- 保存脚本文件后记得为其赋予755权限，以便能够执行这个脚本。然后以绝对路径的方式执行这个脚本，通过restart参数重启Nginx服务程序，最后再使用chkconfig命令将Nginx服务程序添加至开机启动项中。大功告成！

  ```bash
  $ chmod 775 /etc/rc.d/init.d/nginx
  $ /etc/rc.d/init.d/nginx.conf restart
  Restarting nginx (via systemctl):                          [  OK  ]
  chkconfig nginx on
  ```

- 之后来curl一下localhost，看一下网站是否配置成功。

  ```bash
  $ curl localhost
  <!DOCTYPE html>
  <html>
  <head>
  <title>Welcome to nginx!</title>
  <style>
      body {
          width: 35em;
          margin: 0 auto;
          font-family: Tahoma, Verdana, Arial, sans-serif;
      }
  </style>
  </head>
  <body>
  <h1>Welcome to nginx!</h1>
  <p>If you see this page, the nginx web server is successfully installed and
  working. Further configuration is required.</p>

  <p>For online documentation and support please refer to
  <a href="http://nginx.org/">nginx.org</a>.<br/>
  Commercial support is available at
  <a href="http://nginx.com/">nginx.com</a>.</p>

  <p><em>Thank you for using nginx.</em></p>
  </body>
  </html>
  ```

  如果请求返回了网站的HTML文件，说明配置成功，大功告成！

## 安装PHP服务器

首先安装依赖包：

```bash
yum install libxml2 libxml2-devel openssl openssl-devel bzip2 bzip2-devel libcurl libcurl-devel libjpeg libjpeg-devel libpng libpng-devel freetype freetype-devel gmp gmp-devel libmcrypt libmcrypt-devel readline readline-devel libxslt libxslt-devel zlib zlib-devel glibc glibc-devel glib2 glib2-devel ncurses curl gdbm-devel db4-devel libXpm-devel libX11-devel gd-devel gmp-devel expat-devel xmlrpc-c xmlrpc-c-devel libicu-devel libmcrypt-devel libmemcached-devel libzip
```

下载、解压安装包

```bash
cd ..
wget https://www.php.net/distributions/php-7.1.32.tar.gz
tar zxvf php-7.1.32.tar.gz
cd php-7.1.32
```

接下来编译安装(./configure --help可查看编译参数)

```bash
./configure \
 --prefix=/usr/local/php\
 --enable-fpm\
 --with-fpm-user=www\
 --with-fpm-group=www\
 --with-config-file-path=/usr/local/php/conf\
 --disable-rpath\
 --enable-soap\
 --with-libxml-dir\
 --with-xmlrpc\
 --with-openssl\
 --with-mhash\
 --with-pcre-regex\
 --with-zlib\
 --enable-bcmath\
 --with-bz2\
 --enable-calendar\
 --with-curl\
 --enable-exif\
 --with-pcre-dir\
 --enable-ftp\
 --with-gd\
 --with-openssl-dir\
 --with-jpeg-dir\
 --with-png-dir\
 --with-zlib-dir\
 --with-freetype-dir\
 --enable-gd-jis-conv\
 --with-gettext\
 --with-gmp\
 --with-mhash\
 --enable-mbstring\
 --with-onig\
 --with-mysqli=mysqlnd\
 --with-pdo-mysql=mysqlnd\
 --with-zlib-dir\
 --with-readline\
 --enable-shmop\
 --enable-sockets\
 --enable-sysvmsg\
 --enable-sysvsem \
 --enable-sysvshm \
 --enable-wddx\
 --with-libxml-dir\
 --with-xsl\
 --enable-zip\
 --with-pear
 ```
 
 ## 配置

 执行完安装命令后php7就已经安装在到了`/usr/local/php`目录下了。

```bash
/usr/local/php/bin/php -v
```

查看安装是否成功。

为了以后方便，可以编辑 `/etc/profile` 添加环境变量 ，添加到最后面

```shell
PATH=$PATH:/usr/local/php/bin
export PATH
```

然后更新环境变量。

```
source /etc/profile
```

查看环境变量

```bash
echo $PATH
```

查看php版本

```bash
php -v
```

删除当前默认的配置文件，然后将php服务程序目录中相应的配置文件复制过来：

```bash
rm -rf /etc/php.ini
ln -s /usr/local/php/etc/php.ini /etc/php.ini
cp php.ini-production /usr/local/php/etc/php.ini
cp /usr/local/php/etc/php-fpm.conf.default /usr/local/php/etc/php-fpm.conf
ln -s /usr/local/php/etc/php-fpm.conf /etc/php-fpm.conf
```

配置妥当后便可把用于管理php服务的脚本文件复制到/etc/rc.d/init.d中了。为了能够执行脚本，请记得为脚本赋予755权限。最后把php-fpm服务程序加入到开机启动项中：

```bash
cp sapi/fpm/init.d.php-fpm /etc/rc.d/init.d/php-fpm
chmod 755 /etc/rc.d/init.d/php-fpm
cp /usr/local/php/etc/php-fpm.d/www.conf.default /usr/local/php/etc/php-fpm.d/www.conf
chkconfig php-fpm on
```

由于php服务程序的配置参数直接会影响到Web服务服务的运行环境，因此，如果默认开启了一些不必要且高危的功能（如允许用户在网页中执行Linux命令），则会降低网站被黑难度，入侵人员甚至可以拿到整台Web服务器的管理权限。因此我们需要编辑php.ini配置文件，在305行的disable_functions参数后面追加上要禁止的功能。下面的禁用功能名单是刘遄老师依据网站运行的经验而定制的，不见得适合每个生产环境，建议大家在此基础上根据自身工作需求酌情删减：

```bash
 307 ; This directive allows you to disable certain functions for security reasons.
 308 ; It receives a comma-delimited list of function names.
 309 ; http://php.net/disable-functions
 310 disable_functions = passthru,exec,system,chroot,scandir,chgrp,chown,shell_exec,proc_open,proc_get_status,ini_alter,ini_alter,ini_restore,dl,openlog,syslog,readlink,symlink,popepassthru,stream_socket_server,escapeshellcmd,dll,popen,disk_free_space,checkdnsrr,checkdnsrr,getservbyname,getservbyport,disk_total_space,posix_ctermid,posix_get_last_error,posix_getcwd,posix_getegid,posix_geteuid,posix_getgid,posix_getgrgid,posix_getgrnam,posix_getgroups,posix_getlogin,posix_getpgid,posix_getpgrp,posix_getpid,posix_getppid,posix_getpwnam,posix_getpwuid,posix_getrlimit,posix_getsid,posix_getuid,posix_isatty,posix_kill,posix_mkfifo,posix_setegid,posix_seteuid,posix_setgid,posix_setpgid,posix_setsid,posix_setuid,posix_strerror,posix_times,posix_ttyname,posix_uname
```

这样就把php服务程序配置妥当了。最后，还需要编辑Nginx服务程序的主配置文件，把第2行的井号（#）删除，然后在后面写上负责运行Nginx服务程序的账户名称和用户组名称；在第45行的index参数后面写上网站的首页名称。最后是将第65～71行参数前的井号（#）删除来启用参数，主要是修改第69行的脚本名称路径参数，其中$document_root变量即为网站信息存储的根目录路径，若没有设置该变量，则Nginx服务程序无法找到网站信息，因此会提示“404页面未找到”的报错信息。还有就是fastcgi_param配置的时候不要忘记取消注释后不要忘记改成自己的位置。在确认参数信息填写正确后便可重启Nginx服务与php-fpm服务。

下面做一个例子

```nginx
server {
        listen       80;
        server_name  site.com www.site.co;
        location / {
            root   /var/www/mysite;
            index  index.html index.htm index.php;
        }
        error_page   500 502 503 504  /50x.html;
        location ~ \.php$ {
            root           html;
            fastcgi_pass   127.0.0.1:9000;
            fastcgi_index  index.php;
            fastcgi_param  SCRIPT_FILENAME  /var/www/mysite$fastcgi_script_name;
            include        fastcgi_params;
        }
}
```

之后，重启php服务

```bash
servide php-fpm restart
```

然后，进入网页的文件夹，新建一个文件叫`info.php`，然后打开，输入以下内容：

```PHP
<?php
    phpinfo();
?>
```

之后保存，打开浏览器，输入`域名/info.php`即可看见自己的php详细内容啦！

## 后记

至此，LNMP框架配置完成。

之后我还会用这个搭建WordPress、重新搭建图床还有邮件服务器。

除此之外，也激发了我对于Nginx的兴趣，我会进一步学习Nginx的源代码，部署负载均衡等等东西。

学校也在学习大数据，我也会在最近部署HDFS、Hadoop等等服务，来进一步加强自己的运维能力。

未来服务器除了WordPress等，还要搭配GitLab、Jupyter Notebook等等。

也会把笔记记录在这里！