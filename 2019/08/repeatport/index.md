# 端口转发问题排查


## 在配置一台服务器多个网站的SSL的时候出现了一些问题...

当我端口转发第一个网站的时候，发现第二个配置好的会转发到第一个网站...

解决办法：在第二个网站的config里面吧return 403改掉

```Nginx
server {
    listen 80;
    server_name 2nd.example.com;
    rewrite ^ https://$server_name$request_uri? permanent;
}
```

即可解决问题

RTM的................................

问题解决的链接：http://blog.chinaunix.net/uid-231372-id-4584714.html

感谢这位大佬。侵删。

