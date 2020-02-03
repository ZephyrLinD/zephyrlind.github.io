# NginxTips（二） —— Nginx作为静态Web服务器的二三十


## GZip压缩文本网页

在 `Nginx.conf` 中可以找到 `gzip` 的选项。

```nginx
# 省略
gzip                on;
gzip_min_length     1;      # 小于一字节的文件就不进行压缩。小文件在一个TCP报文中就发出去了。在压缩消耗CPU意义不大
gzip_comp_level     2;      # gzip压缩级别
gzip_types          text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpge image/gif image/png;    # 压缩文件类型
```

加好这些配置后，我们进行 `nginx -s reload`。此时再次进行抓包，可以发现传输的大小小了很多。响应头中的 `Context-Encoding` 也变成了 `gzip`

以后我们的静态资源传输过程效率会高很多。

## 分享文件目录 —— AutoIndex

我们可以通过 `AutoIndex` 模块将文件夹和目录信息分享给用户，让用户决定使用哪部分。

当我们访问一个反斜杠（'/'）的时候可以显示文件目录信息

```nginx
location / {
    autoindex on;
}
```

效果如下：

![autoindex](https://img.zephyrl.co/images/2020/02/03/autoindex.jpg)

## 大文件限制访问速度

当大量用户并发的时候，我们需要确保一些基础的文件文件如css的正常访问，我们可以使用 `set` 配合一些内置的变量实现这个功能，比如：

```nginx
location / {
    alias dlib/;
    autoindex on;
    set $limit_rate 1k;
}
```

其中 `set $limit_rate 1k;` 就是在克制客户端到服务器发送请求响应的速度。

在nginx官网的`http_core`中提供的`Embedded Variables`中可以找到 [变量$limit_rate](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables), 他的用法就是上述代码，单位是Byte。

## Access 日志

我们可以在`nginx.conf`中使用`log_format`定义日志格式

```nginx
http {
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "http_x_forwarded_for" ';
    access_log logs/test.access.log main;
}
```

其中，日志被命名成main，日志格式就是后面的。所有的变量的意义都可以在[nginx官方文档](http://nginx.org/en/docs/http/)中找到。也可以加入各种日志变量。

比如 `$remote_addr` 的意思就是远端（客户端）地址。 `$time_local` 就是当时时间。`$status` 就是返回的详情，如403、301等。

有一款适合监控Nginx Access日志的文件GoAccess，未来会写如何搭建GoAccess并且在Web中展示。就不放在NginxTips里了。
