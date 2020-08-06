# Zabbix 配置文档（〇）—— 安装


## 前言

这是 Zabbix5.0 LTS 的安装，基于 CentOS7.5

## 选择清华源安装 Zabbix 包

从[清华大学镜像源](https://mirrors.tuna.tsinghua.edu.cn/)中找到zabbix，选择需要的版本

```bash
wget https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
```

之后安装 RPM 包 并且查看包内

```shell
[root@zep0 ~]# rpm -ivh zabbix-release-5.0-1.el7.noarch.rpm
warning: zabbix-release-5.0-1.el7.noarch.rpm: Header V4 RSA/SHA512 Signature, key ID a14fe591: NOKEY
Preparing...                          ################################# [100%]
Updating / installing...
   1:zabbix-release-5.0-1.el7         ################################# [100%]
[root@zep0 ~]# rpm -ql zabbix-release
/etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
/etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
/etc/yum.repos.d/zabbix.repo
/usr/share/doc/zabbix-release-5.0
/usr/share/doc/zabbix-release-5.0/GPL
```

查看下载地址

```shell
[root@zep0 ~]vim /etc/yum.repos.d/zabbix.repo
[zabbix]
name=Zabbix Official Repository - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-frontend]
name=Zabbix Official Repository frontend - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/frontend
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591

[zabbix-debuginfo]
name=Zabbix Official Repository debuginfo - $basearch
baseurl=http://repo.zabbix.com/zabbix/5.0/rhel/7/$basearch/debuginfo/
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX-A14FE591
gpgcheck=1

[zabbix-non-supported]
name=Zabbix Official Repository non-supported - $basearch
baseurl=http://repo.zabbix.com/non-supported/rhel/7/$basearch/
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-ZABBIX
gpgcheck=1
```

可以发现虽然从清华源下的 里面的下载地址仍让是官网 太慢 我们换成清华源的

[![1-sg.png](https://img.zephyrl.co/images/2020/08/05/1-sg.png)](https://img.zephyrl.co/image/yVU3)

通过 `:%s###g` 可以快速修改

接着 关闭 `gpgcheck`

[![2-sg.png](https://img.zephyrl.co/images/2020/08/05/2-sg.png)](https://img.zephyrl.co/image/ydE4)

同样的方法

这样源就搞定了，接下来我们安装这几个包

## 安装 Zabbix 客户端和服务端

首先安装 Zabbix 服务器和代理

```bash
yum install zabbix-server-mysql zabbix-agent
```

然后安装前端

回到 `/etc/yum.repos.d/zabbix.repo` 中，编辑

```vim
[zabbix-frontend]
...
enabled=1
...
```

之后将 zabbix web 安装

```sh
yum install zabbix-web-mysql-scl zabbix-apache-conf-scl
```

## 设置 MySQL 数据库

首先 创建 zabbix 数据库并创建 zabbix 用户并且赋予权限

```shell
[root@zep0 ~]# mysql -uroot -p
password
mysql> create database zabbix character set utf8 collate utf8_bin;
mysql> create user zabbix@localhost identified by 'password';
mysql> grant all privileges on zabbix.* to zabbix@localhost;
mysql> quit;
```

然后用自带的 SQL 语句写入数据库 创建表

```shell
zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix
```

如果忘了在哪 可以这样查询：

```shell
[root@zep0 ~]# rpm -ql zabbix-server-mysql | grep sql
/usr/sbin/zabbix_server_mysql
/usr/share/doc/zabbix-server-mysql-5.0.2
/usr/share/doc/zabbix-server-mysql-5.0.2/AUTHORS
/usr/share/doc/zabbix-server-mysql-5.0.2/COPYING
/usr/share/doc/zabbix-server-mysql-5.0.2/ChangeLog
/usr/share/doc/zabbix-server-mysql-5.0.2/NEWS
/usr/share/doc/zabbix-server-mysql-5.0.2/README
/usr/share/doc/zabbix-server-mysql-5.0.2/create.sql.gz
/usr/share/doc/zabbix-server-mysql-5.0.2/double.sql
```

也可以找到 `create.sql.gz`

稍等片刻，完成后可以用如下命令来显示已有的表

```shell
mysql -uroot -p zabbix -e 'show tables;'
```

如果出现一大堆表就证明成功

之后 配置启动zabbix-server

```shell
vim /etc/zabbix/zabbix_server.conf
DBHost=localhost
DBName=zabbix
DBUser=zabbix
DBPassword=password
```

然后通过编辑 `/etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf` 来修改时区

```shell
php_value[date.timezone] = Asia/Shanghai
```

之后 启动服务器

```shell
systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

## 安装 Zabbix 网页面板

启动服务器后，进入 ip/zabbix 即可访问面板安装程序，按照步骤输入即可

但是再连接数据库的时候 我出现了验证问题

```shell
The server requested authentication method unknown to the client [duplicate]
```

后发现是 MySQL8 和 PHP7 的兼容问题

MySQL8 的默认插件是auth_socket，于 PHP7不兼容

解决办法是登录 MySQL 数据库 输入以下 SQL 语句即可解决

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

解决问题是从 StackOverflow 找到的，链接[在这里](https://stackoverflow.com/questions/52364415/php-with-mysql-8-0-error-the-server-requested-authentication-method-unknown-to)

## 添加主机

### 被监控主机操作

添加主机只需要在目标机器上安装 `zabbix-agent` 即可，而 `zabbix-agent` 没什么依赖包，安装非常简单快捷

```bash
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-agent-5.0.1-1.el7.x86_64.rpm
```

之后进入 `/etc/zabbix/zabbix_agentd.conf` 中修改主机地址为监控机地址

```shell
server = xxxxx
```

最后，启动服务

```bash
systemctl start zabbix-agent
systemctl enable zabbix-agent
```

### Web 面板操作

进入面板，选择配置-主机-创建主机

![3_addPC.png](https://img.zephyrl.co/images/2020/08/05/3_addPC.png)

然后通过图中提示添加主机

![3_addPC2.png](https://img.zephyrl.co/images/2020/08/05/3_addPC2.png)

点击模板，搜索 Linux，选择如下模板之后，点击添加即完成

![3_addPC3.png](https://img.zephyrl.co/images/2020/08/05/3_addPC3.png)

之后回到主机页面，如果主机页面的 `ZBX` 变绿，及证明添加成功

![4_addPcSuccess.png](https://img.zephyrl.co/images/2020/08/05/4_addPcSuccess.png)

## 压力测试

使用 `Stress` 软件进行一些压力测试

[![5_stressTest.png](https://img.zephyrl.co/images/2020/08/05/5_stressTest.png)](https://img.zephyrl.co/image/yCqO)

可以发现，正常报警

[![5_stressTestResult.png](https://img.zephyrl.co/images/2020/08/05/5_stressTestResult.png)](https://img.zephyrl.co/image/ysWI)
