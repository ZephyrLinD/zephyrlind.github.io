# NginxTips（三） —— 用Nginx搭建一个具备缓存功能的反向代理服务器


## 前言

由于上游服务业务逻辑复杂，需要较高的开发效率所以性能不怎么样。我们可以用Nginx进行反向代理，一台Nginx服务器采用负载均衡算法代理给多台上游服务器使用。这样我们就实现了业务的水平扩展，在用户没感觉的情况下添加多台服务器。也可以在上游服务器出现问题的时候将请求转发给正常的服务器。

## 将从前的页面变更为上游服务器

由于一般上游服务器是不对外网访问的。所以我们要先控制nginx让这个网页不对外访问，代码如下：

```nginx
server {
    listen 127.0.0.1:8080;
}
```

表示只有本机可以访问8080端口。在重启Nginx即可达成效果。

## 搭建Nginx反向代理

我们用OpenResty搭建一个反向代理

首先我们添加一个`upstream`，来指定上游服务器的地址：

```nginx
upstream local {
    server 127.0.0.1:8080;
}
```

我们也可以用多种算法来选择上游服务。在这里就不多赘述。而`upstream`定义的这一批服务器的命名就是代码中的`local`。

反向代理服务器用`proxy_pass`把他转发到上游服务里。此时访问域名，在抓包，我们会发现响应头都是反向代理服务器发给我们客户端的（server是OpenResty）。

```nginx
server {
    server_name 2nd.zephyrl.co;
    listen 80;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxt_add_x_forwarded_for;

        proxy_pass http://local;
    }
}
```

其中`proxy_set_header`等等所有的配置特性都可以在官网的http_proxy_module中找到。

## 配置缓存服务器

```nginx
proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
```

上述代码的意思是：10M的共享内存（keys_zone=my_cache:10m）

缓存的使用方法就是在缓存的URL路径下加入如下：

```nginx
locathon / {
    proxy_cache my_cache
    proxy_cache_key $host$uri$is_args$args;
    proxy_cache_valid 200 304 302 1d;
}
```

此时即可采用缓存。
