# 我的Ubuntu Server LNMP折腾笔记


## 前言

鄙人不知为何骨骼精奇，有一癖好，就是对着黑框框敲敲敲怎么也不累，能直接敲一天。其实主要也是小时候受家里的老386机器熏陶，养成了对命令的酷爱。随着时代的发展，386扔掉了（现在觉得好可惜），图形化界面渐渐普及了，但是我对黑框框的热爱从未减少。遂后来学习Linux，转向服务器方向，励志成为一个合格的DevOPs。所以到了大学就开始折腾服务器、玩Linux甚至日用Linux。说得多了又扯远了。

所谓LNMP就是Linux+Nginx+MySQL+PHP，与LA(apache)MP相比，不同的是，动态请求会转发给php-fpm来处理，Nginx会直接返回静态页面的处理。

LNMP也有不同的搭配：

![dfoflnmp.png](https://img.zephyrl.co/images/2020/02/02/dfoflnmp.png)

本文旨在记录Ubuntu Server上面稍微好配的LNMP环境的一些坑还有一些特殊的地方。因为像CentOS、RHEL都比较适合源代码安装，但是Ubuntu Server源代码安装反而过于折腾而且没必要，所以跟RedHat家的安装方法比，安装虽然简单，但是包名不同，也算有点坑吧。把它们记录下来，以后再弄的时候看这里就可以了。

本文环境为 Ubuntu Server 18.04 LTS，阿里云轻量应用服务器。

## N —— Nginx

Nginx的安装属于最简单的，没什么坑，直接执行以下代码即可：

```shell
sudo apt-get install nginx
```

安装的过程中选择 `Y` ，之后环境就搭起来了，再执行下面的代码来重启服务。

```shell
sudo service nginx restart 
```

此时可以直接访问IP地址，会显示Nginx的欢迎页面

![Nginx Welcome](https://img.zephyrl.co/images/2020/02/02/nginxwelcome.png)

或者curl一下localhost

```shell
curl localhost
```

出现如下代码即可证明Nginx开启成功 

```html
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

## M —— MySQL

由于使用的是Ubuntu18.04，如果直接不选MySQL版本的话，则自动安装最新的MySQL，指令如下

```shell
sudo apt-get install mysql-server mysql-client 
```

之后用root用户直接执行`mysql`，即可打开MySQL终端，然后修改一下root密码：

```shell
mysql> SET PASSWORD FOR 'root'@'localhost' = PASSWORD('你的密码');
```

*注:最前面的`mysql>`代表在MySQL下输入，并不用输入。*

之后查一下表：

```shell
mysql> select host,user,authentication_string from mysql.user;
+-----------+------------------+-------------------------------------------+
| host      | user             | authentication_string                     |
+-----------+------------------+-------------------------------------------+
| localhost | root             |                                           |
| localhost | mysql.session    | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| localhost | mysql.sys        | *THISISNOTAVALIDPASSWORDTHATCANBEUSEDHERE |
| localhost | debian-sys-maint | *01803E55701BD25F26000A75E6D3EC91F71358B0 |
+-----------+------------------+-------------------------------------------+
4 rows in set (0.00 sec)
```

大功告成！

## P —— ~~Postfix~~ PHP

只需把这三个软件安装即可完成PHP的安装：

```shell
sudo apt-get install php7.2 php7.2-fpm php7.2-mysql
```

## Nginx解析PHP

首先修改Nginx配置文件：

```shell
sudo vim /etc/nginx/sites-available/default
```

修改第56-63行如下：

![修改行](https://img.zephyrl.co/images/2020/02/02/phpchanges.png)

之后还有的修改就是第41行的`root`选项，可以修改网站的根目录，就不一一赘述了。

然后执行命令，重启Nginx：

```shell
sudo service nginx restart 
```

之后修改PHP的设置：

```shell
sudo vim /etc/php/7.2/fpm/pool.d/www.conf
```

大约第37行加入语句：

![修改行](https://img.zephyrl.co/images/2020/02/02/phpchanges2.png)

修改之后，我们重启php7.2-fpm

```shell
sudo service php7.2-fpm restart
```

之后我们在网站的root目录下创建info.php文件，内容如下：

```html
<?php
    phpinfo();
?>
```

然后在浏览器打开：IP地址/info.php 效果如下

![phpinfo](https://img.zephyrl.co/images/2020/02/02/phpinfo.png)

## 确认可以通过PHP连接到MySQL

仍然是在网站根目录下，创建一个mysql.php，输入如下代码：

```html
<?php
    $servername = "localhost";
    $username = "mysql账号";
    $password = "mysql密码";

    // 创建连接
    $conn = new mysqli($servername, $username, $password);

    // 检测连接
    if ($conn->connect_error) {
        die("连接失败: " . $conn->connect_error);
    }
    echo "<h1>连接成功</h1>";
?>
```

保存后，打开IP地址/mysql.php，显示“连接成功”即可。也可以用curl方法，和上面一样，在这里就不赘述。

## 后记

让网站“跑起来”只是我们的第一步，千里之行始于足下。后续将更新创建图床（因为正在等备案审核），创建Worldpress博客的内容。