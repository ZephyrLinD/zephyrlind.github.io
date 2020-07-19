# 使用树莓派在 Docker 中运行 OpenWrt 旁路网关


## 前言

在学校为了打比赛买的树莓派 4B 4G 已经完全吃灰了，把它废物利用一下。看同学 Pi 4B 做旁路由挺稳定的，心痒痒，想折腾......

## 旁路网关

普通的路由器往往集无线信号转发、网关、DNS 服务器等角色为一身，其中的“网关”角色负责路由器内部数据的处理。但因为一般家用的路由器硬件性能很有限，在运行一些比较吃资源的应用（如酸酸乳、去广告等）时，几乎会占满所有硬件资源，导致路由器网络/系统不稳定等诸多问题。既然路由器的硬件性能有限，那我们就把网关的重任交给硬件性能更好的设备去做，也就是树莓派 Docker 中的 OpenWrt 去做。把路由器肩负的网关重任交给其他更适合的设备来做，同时，旁路网关处理完的结果会返回给路由器，由路由器继续进行无线转发。

这样，每个角色各司其职，路由器肩上的任务轻了，即使是油管 4K 也能轻松跑满网速了，而旁路由（树莓派）也结束了它吃灰的命运（好像只是换了个地方吃灰）。由此资源的充分利用，一举两得。

我将在 Docker 容器中运行 OpenWrt，并通过设置，让 Docker 容器中的 OpenWrt 网关接管路由器自身的网关，减轻路由器的负担，同时，由于网关被 OpenWrt 接管，所以 OpenWrt 中的大部分应用都是可用的，比如酸酸乳，V2ray，Trojan，去广告等。

OpenWrt 镜像所用系统为基于 Lean 大源码编译的 32位/ IPV4 Only 固件。至于为啥不自编译...因为懒。最近天气很热但是爸妈身体不好，没开空调，有点烦躁，以后会补上（我自己挖了好几个坑了还没填上呢呜呜呜）。

## 烧录系统

这次我使用“树莓派爱好者基地”编译的 64 位 Debian，此版本 Debian 可以充分发挥 64 位 CPU 的性能，同时默认开启 Docker，KVM 等功能，其中，Docker 功能开箱即用，非常方便。

“树莓派爱好者基地”专版64位 Debian 功能介绍在如下链接中，下载时推荐选择“无桌面基础系统加强版”：

https://github.com/openfans-community-offical/Debian-Pi-Aarch64/blob/master/README_zh.md

下载镜像烧写到树莓派的 SD 卡并上电开机后，系统会自动完成扩展 SD 卡空间的操作，这个过程大约需要3~5分钟（期间会重启几次）。估摸着时间差不多以后，用网线将树莓派的板载网口与路由器的 Lan 口连接，此时我们需要进入路由器管理页面对树莓派进行 Mac 地址绑定，最好绑定在 DHCP 服务器配置的范围之外，比如我的 DHCP 服务器分配 IP 是从 192.168.2.100 分配到 192.168.2.199，子网掩码 24 位，我们就可以把树莓派的 IP 绑定到 200 - 254 之间。

之后连接树莓派的 SSH：

用户名：pi

密码：raspberry

准备工作到此结束。

## 安装配置 OpenWrt 镜像并配置

成功登陆到树莓派的 SSH 后，在拉取镜像之前，我们还需要进行一些额外的工作：

### 打开网卡混杂模式

```shell
sudo ip link set eth0 promisc on
```

### 结合自己网络情况创建网络

```shell
docker network create -d macvlan --subnet=192.168.2.0/24 --gateway=192.168.2.1 -o parent=eth0 macnet
```

此时，我们使用 docker network ls命令可以看到网络macnet已建立成功：

```bash
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
10e676133746        bridge              bridge              local
f5308b94e8fa        host                host                local
16745ea66852        macnet              macvlan             local
5e72e41ea02a        none                null                local
```

### 拉取镜像

为提高拉取速度，我拉取阿里云仓库中的镜像：

```bash
docker pull registry.cn-shanghai.aliyuncs.com/suling/openwrt:latest
```

镜像拉取完成后，我们可以执行`docker images`命令查看现存镜像：

```bash
$ docker images
REPOSITORY                                              TAG                 IMAGE ID            CREATED             SIZE
registry.cn-shanghai.aliyuncs.com/suling/openwrt        latest              4f4bc5dca2d9        1 hours ago         112MB
```

此时证明镜像已成功拉取到本地。

### 创建并启动容器

```bash
docker run --restart always --name openwrt -d --network macnet --privileged registry.cn-shanghai.aliyuncs.com/suling/openwrt:latest /sbin/init
```

其中：

`--restart always` 参数表示容器退出时始终重启，使服务尽量保持始终可用；

`--name openwrt` 参数定义了容器的名称；

`-d` 参数定义使容器运行在 Daemon 模式；

`--network macnet` 参数定义将容器加入 macnet 网络；

`--privileged` 参数定义容器运行在特权模式下；

`registry.cn-shanghai.aliyuncs.com/suling/openwrt:latest`为 Docker 镜像名，因容器托管在阿里云 Docker 镜像仓库内，所以在镜像名中含有阿里云仓库信息；

`/sbin/init` 定义容器启动后执行的命令。

启动容器后，我们可以使用 `docker ps -a` 命令查看当前运行的容器：

```shell
$ docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
a26cee7cade6        openwrt:latest      "/sbin/init"        3 hours ago         Up 3 hours                              openwrt
```

若容器运行信息 `STATUS` 列为 `UP` 状态，则说明容器运行正常。

### 进入容器并按照需求修改相关参数

首先进去容器

```shell
docker exec -it openwrt bash
```

其中：

`openwrt`为容器名称；

`bash`为进入容器后执行的命令。

执行此命令后我们便进入 OpenWrt 的命令行界面，首先，我们需要编辑 OpenWrt 的网络配置文件：

```shell
vim /etc/config/network
```

我们需要更改 Lan 口设置：

```bash
config interface 'lan'
    option type 'bridge'
    option ifname 'eth0'
    option proto 'static'
    option ipaddr '192.168.123.100'
    option netmask '255.255.255.0'
    option ip6assign '60'
    option gateway '192.168.123.1'
    option broadcast '192.168.123.255'
    option dns '192.168.123.1'
```

其中：

所有的 192.168.123.x 需要根据树莓派所处网段修改，`option gateway` 和 `option dns` 填写路由器的 IP，若树莓派获得的 IP 为 `192.168.2.200`，路由器 IP 为 `192.168.2.1`，则需要这样修改：

```bash
config interface 'lan'
    option type 'bridge'
    option ifname 'eth0'
    option proto 'static'
    option ipaddr '192.168.2.254'
    option netmask '255.255.255.0'
    option ip6assign '60'
    option gateway '192.168.2.1'
    option broadcast '192.168.2.255'
    option dns '192.168.2.1'
```

option ipaddr 项目定义了 OpenWrt 的 IP 地址，在完成网段设置后，IP最后一段可根据自己的爱好修改（前提是符合规则且不和现有已分配 IP 冲突）。

### 重启网络

```bash
/etc/init.d/network restart
```

### 进入控制面板

在浏览器中输入第 5 步 `option ipaddr` 项目中的 IP 进入 Luci 控制面板，若 `option ipaddr` 的参数为 `192.168.123.100`，则可以在浏览器输入 http://192.168.123.100 进入控制面板。

用户名：root

密码：password

之后就可以干想干的事了......

## 链接相关

需要手动配置 IP 地址，我们把 DNS 服务器和网关都改成我们刚刚设置的 OpenWrt 地址就可以啦

## 其他杂七杂八

我使用了 ShadowSocksR+ 和一个机场进行了科学上网，结果发现上外网速度贼 jier 快，但是国内 DNS 解析速度慢的一批

后来查询发现是因为国内网站走了一遍防火墙，我们设置防火墙即可解决问题

```shell
iptables -t nat -I POSTROUTING -o eth0 -j MASQUERADE
```

## 最后

暂时没遇到其他问题，如果遇到了会更新博客！
