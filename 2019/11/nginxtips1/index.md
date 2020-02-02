# NginxTips（一） —— Nginx的组成、Vim语法高亮、编译、命令行、热部署、日志切割脚本


## Nginx的四个组成部分

- Nginx二进制可执行文件 —— 由各模块源码编译出的一个文件，一般在`/usr/local/nginx/sbin/`中

- Nginx.conf配置文件 —— 控制Nginx的行为

- Access.log访问日志 —— 记录每一条HTTP请求信息

- Error.log错误日志 —— 定义问题

## 让你的VIM对nginx.conf高亮

在nginx源代码的`contrib`目录中，有文件夹`vim`，它可以提供语法高亮的vim配置文件，我们可以把它复制到`~/.vim`中以开启对nginx配置文件的语法高亮。

假设我们在nginx源代码文件中：

```bash
cp -r contrib/vim/* ~/.vim
```

效果如下：

![vim高亮](https://img.zephyrl.co/images/2020/02/02/113c975dbbd1057df.png)

## 编译相关

### ./confugre相关：

```bash
./configure --help || more
```

一般带`--with`的都是默认不编译的，带`--without`的是默认编译的。

我一般的代码：

```shell
./configure \
--prefix=/usr/local/nginx \
--user=www \
--group=www \
--with-http_gzip_static_module \
--with-http_flv_module \
--with-http_ssl_module
```

`configure`后编译出的中间文件在`objs`文件夹。其中最重要的文件是`ngx_model.c`，它决定了接下来哪些模块会被编译进我们的Nginx。

他们会形成一个叫`ngx-models的数组`。

### 执行`make`后的东西

执行`make`后，nginx的二进制文件存在了`objs`文件夹里。

因为我们如果是为了将Nginx版本升级，我们需要知道这个二进制文件的位置并且将它拷贝到安装目录中。

除此之外，所有的中间文件都会在`objs/src`目录。

## Nginx命令行

格式举例：`nginx -s reload`

帮助：`nginx -?`或者`nginx -h`

使用指定的配置文件：`nginx -c 配置文件路径`

使用配置指令：`nginx -g 覆盖指令`

指定运行目录：`nginx -p 指定网页目录`

发送信号 `-s`

- 立即停止服务：`nginx -s stop`
- 优雅的停止服务：`nginx -s quit`
- 重载配置文件（在不停止对用户的服务下重载）：`nginx -s reload`
- 重新开始记录配置文件：`nginx -s reopen`

测试配置文件是否有语法错误：`nginx -t`

打印Nginx的版本信息：`nginx -v`

## Nginx热部署

首先要备份旧的Nginx二进制文件，然后按照上面的方法编译出新的Nginx二进制文件并且拷贝到该文件夹内。

然后我们需要给旧Nginx的master进程发送信号USR2。

```bash
cp /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
# 然后拷贝新的二进制文件进入目录下
kill -USR2 [nginx.master.PID]
```

此时会新起一个master进程，他将执行最新的二进制文件。

老的Worker进程也会保留，只是不在监听80或者443端口，他们会平滑的过度。

此时我们要对老的进程发送一个WINCH指令，让他（老master进程）优雅的停止，交付给新的进程

```bash
kill -WINCH [nginxOLD.master.PID]
```

此时执行`ps -ef | grep nginx`会发现，只有新的Nignx Worker进程保留着。

然而老的Master进程也会得以保留，可以给他发送RELOAD命令把他拉起，以便解决新的Nginx版本出现一些兼容性问题。

## 日志切割

其实很简单，只是把老的日志文件剪切重命名一份，然后执行reload命令即可生成新的脚本。

但是我们可以把它写成一个bash脚本，放在`crontab`中，脚本如下。

```shell
#!/bin/bash
# Rotate Nginx logs to prevent a single logfile from consuming too much disk space.
LOGS_PATH=/usr/local/nginx/logs/history
CUR_LOGS_PATH=/usr/local/nginx/logs
YESTERDAY=$(date -d "yesterday" + %Y-%m-%d)
mv ${CUR_LOGS_PATH}/access.log ${LOGS_PATH}/access_${YESTERDAY}.log
mv ${CUR_LOGS_PATH}/error.log ${LOGS_PATH}/error_${YESTERDAY}.log
# 向Nginx主进程发送USR1信号。USR1信号时重新打开日志文件
kill -USR1 $(cat /usr/local/nginx/logs/nginx.pid)
```

*注：crontab是一个用来设置、删除或显示供守护进程cron执行的定时任务的命令。每一个用户都可以拥有属于自己的定时任务，定时任务文件默认以用户名命名，并放在/var/spool/cron目录，该目录普通用户无访问权限。可以用`crontab -l`来查看目前的定时任务。*
