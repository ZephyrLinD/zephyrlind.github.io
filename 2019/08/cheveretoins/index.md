# Chevereto的安装和一些坑


## 前言：

看着乱成一坨的腾讯COS图片地址，我受够了。这对我这个强迫症真的是极为不友好。

然后，突然想到知乎问题中，[自己拥有一台服务器可以做哪些很酷的事情](https://www.zhihu.com/question/40854395/answer/379997699)中，一个叫**小游**的大佬的[回答](
https://www.zhihu.com/question/40854395/answer/728324567)（[点击此处进入大佬的博客](https://xiaoyou66.com/)），我可以做一个图床来存储一些我想上传的图啊！

于是，一不做二不休，作为一个想做运维的菜鸡，这种事情一看见就手痒。

## 安装Chevereto

`Chevereto`分为免费版本和收费版本，因为我只作图片存储用，不需要评论等社交元素，所以免费版本足够使用。最好还是别用破解版了，律师~~含~~函警告。

源代码是从官方的[Github页面](https://github.com/Chevereto/Chevereto-Free)下载的，官方提供了四种方法，我选择的第一种，安装器安装。其他方式安装大同小异，不做赘述。

- 第一步：修改Nginx配置，创建一个新的网站

  因为我是打算未来弄一个`WordPress`主站的，所以我打算把主站先留出来，创建一个新的网站。

  首先，创建网站文件夹，将安装器下载到文件夹中备用，
  
  我用我的作为示例，在自己动手的时候需要修改目录：

  ```shell
  sudo mkdir /var/www/graph
  cd /var/www/graph
  wget https://chevereto.com/download/file/installer
  ```
  
- 然后，打开配置文件：

  ```shell
  sudo vim /etc/nginx/sites-available/default
  ```

  添加以下内容以配置新的网站的伪静态（部分配置参考自小游大佬的博客，[点击进入](https://xiaoyou66.com/archives/774)）

  ```Nginx
  server {
	listen 80;
	listen [::]:80;
	server_name graph.zephyrl.co;
	root /var/www/graph.zephyrl.co;
    index index.php index.html index.htm index.nginx-debian.html;

    # Image not found replacement
    location ~* (jpe?g|png|gif) {
        log_not_found off;
        error_page 404 /content/images/system/default/404.gif;
    }

    # CORS header (avoids font rendering issues)
    location ~ \.(ttf|ttc|otf|eot|woff|woff2|font.css|css|js)$ {
        add_header Access-Control-Allow-Origin "*";
    }

    # Pretty URLs
    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass 127.0.0.1:9000;
	}
  }
  ```

## 第二部，安装和安装过程中的坑

完成上面的步骤后，打开`网站名称/installer.php`，

发现下图即为成功：

[![installer1.md.png](http://graph.zephyrl.co/images/2019/08/24/installer1.md.png)](http://graph.zephyrl.co/image/PCD)

当然如果你和我一样衰，就会出现一个个问题。然后

首先是出现了`Chevereto can’t create the app/settings.php file. You must manually create this file`，之后应该打开终端，创建该文件，

```shell
cd /var/www/graph
touch app/settings.php
```

即可进入下一个问题：

由于当时手忙脚乱心态爆炸，以下问题均没有截图。我会努力描述的并且尽量找到找一些网上的图片，侵删。

[![installqus.md.png](http://graph.zephyrl.co/images/2019/08/24/installqus.md.png)](http://graph.zephyrl.co/image/gwF)

大概就是这个页面，我的还多了缺少curl、zip、libdb的提示，实在找不到图，只能自己描述一下。

先来解决权限问题：

```shell
chmod -R 775 ./*
```

然后来解决缺少组件的问题，由于是ubuntu系统，我们**不需要从官网下载源代码然后编译，会报错**，直接`apt`安装即可。

```shell
sudo apt install php-curl php-gd php-mbstring php-zip
```

省的跟我一样xjb make install然后浪费好多时间 =_=

之后出现了成功的页面，就很开心

[![installer1.md.png](http://graph.zephyrl.co/images/2019/08/24/installer1.md.png)](http://graph.zephyrl.co/image/PCD)

点击Continue，然后一路Skip，直到如下选择数据库。

[![installer2-db.md.png](http://graph.zephyrl.co/images/2019/08/24/installer2-db.md.png)](http://graph.zephyrl.co/image/RZd)

此时，打开终端，创建数据库.可以使用`phpmyadmin`，`navicat premium`等可视化工具创建数据库创建数据库，也可以在终端使用SQL命令；数据库名称自定。创建完成后输入你的数据库名称、账号和密码（依次是我打马的地方）。点击下一步即可。

[![install3.md.png](http://graph.zephyrl.co/images/2019/08/24/install3.md.png)](http://graph.zephyrl.co/image/qQG)

这里大家就跟着他说的写就可以了。坑已经没了，很快就可以嗨皮了（前提是你备案了）。

## 挂载阿里云OSSFS

- 首先，下载挂载工具，我是Ubuntu 18.04 LTS，其他版本安装方法请参照[官方说明页面](https://help.aliyun.com/document_detail/32196.html)

  ```shell
  wget http://gosspublic.alicdn.com/ossfs/ossfs_1.80.6_ubuntu18.04_amd64.deb?spm=a2c4g.11186623.2.13.5a037358XgQ4NQ&file=ossfs_1.80.6_ubuntu18.04_amd64.deb

  sudo dpkg -i ossfs_1.80.6_ubuntu18.04_amd64.deb
  ```

- 安装好后就可以设置bucket name 和 AccessKeyId/Secret信息，将其存放在/etc/passwd-ossfs 文件中。注意这个文件的权限必须正确设置，建议设为640。

  ```shell
  echo 你的bucket名字:你的keyid:你的keysecret > /etc/passwd-ossfs
  chmod 640 /etc/passwd-ossfs
  ```

- 挂载

  OSSFS的挂载方法如下：

  ```shell
  ossfs 你的bucket名字 挂载目录 -ourl=你的阿里云的访问网址
  ```
  
  我们将挂载目录挂载在Chevereto目录下的images文件夹就可以了

  以我为例，
  ```shell
  ossfs mybuckit /var/www/graph/images -ourl=阿里云地址 -o allow_other 
  ```

  完成后即可开始使用了！

