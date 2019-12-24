# 通过Let's Encrypt来获取SSL证书


## 前言

`Let's Encrypt`是一个免费，自动化，并且开放的证书颁发机构。由非营利性组织[互联网安全研究小组](https://www.abetterinternet.org/about/)（ISRG）运营。

简单的说，我们可以借助 Let's Encrypt 颁发的证书可以为我们的网站免费启用 HTTPS(SSL/TLS) 。

Let's Encrypt免费证书的签发/续签都是脚本自动化的，官方提供了几种证书的申请方式方法，[点击此处](https://letsencrypt.org/docs/client-options/)快速浏览。

其中官方推荐使用Certbot客户端来签发证书，但是这种方式配置起来过于麻烦，我就选择了官网说的第三方客户端[acme.sh](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)来申请，这是是目前 Let's Encrypt 免费证书客户端最简单、最智能的 shell 脚本，可以自动发布和续订 Let's Encrypt 中的免费证书。

## 安装`acme.sh`

安装只需一条命令即可完成

```shell
curl https://get.acme.sh | sh
```

其中，首先会把acme.sh安装到`~/.acme.sh`，此目录也会存储生成的证书文件。然后脚本会指定别名`alias acme.sh=~/.acme.sh/acme.sh`。然后自动创建`cronjob`定时任务，每天0:00:00会自动检测所有证书，及时更新证书。

为了保险起见我还是进行了如下操作

```shell
echo "alias acme.sh=~/.acme.sh/acme.sh" >> ~/.zshrc
```

安装命令执行完毕后，执行`acme.sh --version`确认是否能正常使用`acme.sh`命令。

```shell
$ acme.sh --version
https://github.com/Neilpang/acme.sh
v2.8.3
```

## 生成证书

acme支持DNS和HTTP认证，我选择了HTTP验证的方式

其中，我以example.com的域名举例，实际操作换成自己的就可以了：

```shell
acme.sh --issue -d example.com -d www.example.com -w /home/wwwroot/example.com
```

简单解释下这条命令涉及到的一些参数：

- `--issue`是 acme.sh 脚本用来颁发证书的指令；
- `-d`是`--domain`的简称，其后面必须填写已备案的域名；
- `-w`是`--webroot`的简称，其后面需填写网站储存的根目录（我以`/home/wwwroot/example.com`为例）

证书签发成功会有如下输出：

[![success.md.png](http://graph.zephyrl.co/images/2019/08/25/success.md.png)](http://graph.zephyrl.co/image/TTsA)

从截图看出，生成的证书放在了`/root/.acme.sh/esofar.cn`目录。

另外，可以通过下面两个常用`acme.sh`命令查看和删除证书：

```shell
# 查看证书列表
acme.sh --list 

# 删除证书
acme.sh remove <SAN_Domains>
```

## 安装证书

本文将以Nginx为例，记录如何将证书安装，如果其他webserver大家可以参考acme.sh的文档来说。

由于证书生成的目录并不是Nginx存储SSL证书的正确目录，所以我们要用`--installcert`命令来指定复制的目标位置，来让Nginx的配置文件直接读取该目录下的证书。

```shell
acme.sh --installcert -d zephyrl.co \
    --key-file /etc/nginx/ssl/zephyrl.co.key \
    --fullchain-file /etc/nginx/ssl/fullchain.cer \
    --reloadcmd "service nginx force-reload"
```

如果发现出现`/root/.acme.sh/acme.sh: line 5121: /etc/nginx/ssl/example.com.key: No such file or directory`的错误，请先执行以下操作：

```shell
mkdir /etc/nginx/ssl
```

成功后：

![successinstall.png](http://graph.zephyrl.co/images/2019/08/25/successinstall.png)

最后一步就是修改Nginx配置文件，启动SSL，之后重启Nginx，其中Nginx配置可以参考以下，转载自别人的博客

这个：

```nginx
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;
    server_name esofar.cn;
    ssl on;
    ssl_certificate      /etc/nginx/ssl/fullchain.cer;
    ssl_certificate_key  /etc/nginx/ssl/esofar.cn.key;
    root /home/wwwroot/esofar.cn;
    index index.html;
    location / {
        try_files $uri $uri/ @router;
        index index.html;
    }

    location @router {
        rewrite ^.*$ /index.html last;
    }
}

server {
    listen 80;
    server_name esofar.cn;
    return 301 https://$server_name$request_uri;
}
```

也是从此，我的域名下的一个网站升级到了HTTPS，23333333333333

