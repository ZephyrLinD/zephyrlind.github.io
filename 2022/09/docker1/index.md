# Docker学习笔记（一）：一大坨基础命令


## 前言

进入工作很久了，难免压力上头失眠的时候，回忆起上学的时代，有什么比学东西更催眠的呢...根据经验，学编程很提神，于是打开了收藏已久老狂神的 Docker 课程。本笔记根据 [BiliBili UP主“遇见狂神说”](https://space.bilibili.com/95256449) 的[《Docker 入门课程》](https://www.bilibili.com/video/BV1og4y1q7M4)做笔记。同时他是我很喜欢的博主，希望他长久的做下去。

一直想入坑 `Docker`，没系统学习过，一知半解，索性从他开始！学习后重构个人文档及服务器。

## 提前准备

本人实验环境为虚拟机中尝鲜搭建的 `Ubuntu`，具体搭建步骤请参照 `Docker 官方文档`，内含有非常详细的步骤

官方文档地址：https://docs.docker.com/desktop/install/linux-install/

个人建议无论什么版本的 Linux，尽量使用包管理器管理安装包。如果是 CentOS，建议升级内核版本

## Hello World

Docker 的 Hello World 只需通过一行命令即可完成，由此也可知道 Docker 的基本结构，我会在下面的运行结果中注释出：

```bash
$ docker run hello-world
[sudo] zephyrlin 的密码：
Unable to find image 'hello-world:latest' locally                                   # 尝试从本地找 Images，未找到      
latest: Pulling from library/hello-world                                            # 从 Docker Hub 下载镜像
2db29710123e: Pull complete
Digest: sha256:7d246653d0511db2a6b2e0436cfd0e52ac8c066000264b3ce63331ac66dca625     # 成功通过镜像，运行容器实例
Status: Downloaded newer image for hello-world:latest

Hello from Docker!                                                                  # 运行结果
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

由此可以看出，`Docker` 中有非常重要的三个基本概念，理解了这三个概念，就理解了 `Docker` 的整个生命周期。

- 镜像（Image）
- 容器（Container）
- 仓库（Repository）

理解了这三个概念，就理解了 `Docker` 的整个生命周期

## 常用命令

> 命令的帮助文档地址：https://docs.docker.com/engine/reference/commandline/docker/

### 基础命令

#### `docker version`：查看docker的版本信息

```bash
$ docker version
Client: Docker Engine - Community
 Version:           20.10.18
 API version:       1.41
 Go version:        go1.18.6
 Git commit:        b40c2f6
 Built:             Thu Sep  8 23:11:43 2022
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.18
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.18.6
  Git commit:       e42327a
  Built:            Thu Sep  8 23:09:30 2022
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.8
  GitCommit:        9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

#### `docker info`：查看docker的系统信息,包括镜像和容器的数量

```bash
$ docker info
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.9.1-docker)
  compose: Docker Compose (Docker Inc., v2.10.2)
  scan: Docker Scan (Docker Inc., v0.17.0)

Server:
 Containers: 3
  Running: 1
  Paused: 0
  Stopped: 2
 Images: 7
 Server Version: 20.10.18
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: io.containerd.runc.v2 io.containerd.runtime.v1.linux runc
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 9cd3357b7fd7218e4aec3eae239db1f68a5a6ec6
 runc version: v1.1.4-0-g5fd4c4d
 init version: de40ad0
 Security Options:
  apparmor
  seccomp
   Profile: default
  cgroupns
 Kernel Version: 5.15.0-47-generic
 Operating System: Ubuntu 22.04.1 LTS
 OSType: linux
 Architecture: x86_64
 CPUs: 8
 Total Memory: 7.73GiB
 Name: zephyrlin-vm
 ID: BPEP:INYQ:V44F:M2YR:IMSB:EVFP:LGXA:EMCW:6E7X:4AYF:UYFW:AQ7O
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false
```

#### `docker 命令 --help`：帮助命令(可查看可选的参数)，以下命令为查看 `docker rmi` 的帮助和相关参数

```bash
$ docker rmi --help

Usage:  docker rmi [OPTIONS] IMAGE [IMAGE...]

Remove one or more images

Options:
  -f, --force      Force removal of the image
      --no-prune   Do not delete untagged parents
```

### 镜像命令

#### `docker images`：查看本地主机的所有镜像

```bash
$ docker images
REPOSITORY               TAG       IMAGE ID       CREATED             SIZE
hello-world              latest    feb5d9fea6a5   11 months ago       13.3kB
```

其中，各项字段意义为：

- `REPOSITORY`：镜像的仓库源
- `TAG`：镜像的标签
- `IMAGE ID`：镜像的 ID
- `CREATED`：镜像的创建时间
- `SIZE`：镜像的大小

此外，还有其它参数：

- `-a`：查看所有镜像
- `-q`：之查看镜像的 ID

可以配合使用，查看所有镜像 ID：`docker images -aq`

#### `docker search`：搜索镜像

```bash
$ docker search portainer
NAME                                   DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
portainer/portainer                    This Repo is now deprecated, use portainer/p…   2262
portainer/portainer-ce                 Portainer CE - a lightweight service deliver…   1343
portainer/agent                        An agent used to manage all the resources in…   163
portainer/templates                    App Templates for Portainer http://portainer…   26
portainer/portainer-ee                 Portainer BE - a fully featured service deli…   25
portainer/golang-builder               Utility to build Golang binaries.               6                    [OK]
portainer/portainer-k8s-beta           Portainer for Kubernetes BETA                   5
portainer/volume-browser               Experimental app used to browser the content…   4
portainer/base                         Multi-stage build image to create the Portai…   3                    [OK]
portainer/dev-toolkit                  The entire Portainer development stack insid…   3
portainer/helper-reset-password                                                        2
portainer/portainer-docker-extension                                                   1
portainer/agent-k8s-beta               Portainer for Kubernetes BETA (agent)           1
portainer/authenticator                Helps you use the Docker CLI with the Portai…   1
rancher/portainer-agent                                                                1
portainer/docbuilder                   Portainer.io documentation builder              1
portainer/kube-tools                   Image including Docker, kubectl and kind        1
portainer/gosec                                                                        1
portainer/pri-fidoiot                  Docker images for the FIDO Device Onboard (F…   0
portainer/helper-templates             A container helper for template file operati…   0
portainer/portainer-extension                                                          0
portainer/angular-builder              Builder image for Portainer frontend.           0                    [OK]
portainer/portable-env                                                                 0
portainer/kubectl-shell                                                                0
portainer/integration-starter                                                          0
```

还有其它参数：

- `-f`：filter，过滤器

使用示例如下，查看 `STAR` 大于1000的镜像

```bash
$ docker search portainer --filter=STARS=1000
NAME                     DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
portainer/portainer      This Repo is now deprecated, use portainer/p…   2262
portainer/portainer-ce   Portainer CE - a lightweight service deliver…   1343
```

除此之外，建议到官方 Docker Hub 中查看，版本号更为清晰，更能筛选自己需要的容器镜像

#### `docker pull 镜像名[:tag]`：下载镜像

使用示例如下，下载 MySQL 5.7 版本的镜像：

```bash
$ docker pull mysql:5.7
5.7: Pulling from library/mysql
9815334b7810: Pull complete
f85cb6fccbfd: Pull complete
b63612353671: Pull complete
447901201612: Pull complete
9b6bc806cc29: Pull complete
24ec1f4b3b0d: Pull complete
207ed1eb2fd4: Pull complete
27cbde3edd97: Pull complete
0a5aa35cc154: Pull complete
e6c92bf6471b: Pull complete
07b80de0d1af: Pull complete
Digest: sha256:c1bda6ecdbc63d3b0d3a3a3ce195de3dd755c4a0658ed782a16a0682216b9a48
Status: Downloaded newer image for mysql:5.7
docker.io/library/mysql:5.7
```

#### `docker rmi`：删除镜像

使用例如下，`-f` 为强制删除，可省略：

```bash
# 删除指定的镜像id
$ docker rmi -f  镜像id
# 删除多个镜像id
$ docker rmi -f  镜像id 镜像id 镜像id
# 删除全部的镜像id
$ docker rmi -f  $(docker images -aq)
```

### 容器命令

#### `docker run [可选参数] image`：运行容器

可选参数说明：
- `--name="名字"`：指定容器名字
- `-d`：后台方式运行
- `-it`：使用交互方式运行,进入容器查看内容
- `-p`：指定容器的端口
  - `-p ip`：主机端口:容器端口  配置主机端口映射到容器端口
  - `-p`：主机端口:容器端口
  - `-p`：容器端口)
- `-P`：随机指定端口(大写的P)

示例，运行一个 Tomcat 容器，名称为`tomcat01`，并将容器内的 8080 端口映射到宿主机的 3344 端口：

```bash
$ docker run -d -p 3355:8080 --name tomcat01 tomcat
6601419c9d9eeb869a0c80fc3a5b4a7f8c3659a09c2df4a28223c2b19b48e050
```

#### 进入容器

这里直接贴出示例，运行一个 CentOS 容器并使用容器内的 `/bin/bash` 进入该容器操作

```bash
$ docker run -it centos /bin/bash
[root@104acdc5ee47 /]#
```

#### `exit` 退出容器

此处注意两种退出的区别：

- `exit` 停止并退出容器（后台方式运行则仅退出）
- `Ctrl+P+Q` 不停止容器退出

继续上面的示例，停止并退出容器

```bash
$ docker run -it centos /bin/bash
[root@104acdc5ee47 /]# exit
exit

$
```

#### `docker ps`：列出容器

参数如下：

- 直接输入 `docker ps`：列出当前正在运行的容器
- `-a`：列出所有容器的运行记录
- `-n=?`：显示最近创建的n个容器
- `-q`：只显示容器的编号

示例输出如下：

```bash
$ docker ps
CONTAINER ID   IMAGE                           COMMAND             CREATED          STATUS          PORTS                                                                                            NAMES
6601419c9d9e   tomcat                          "catalina.sh run"   36 minutes ago   Up 36 minutes   0.0.0.0:3355->8080/tcp, :::3355->8080/tcp                                                        tomcat01
7e3445a24b27   portainer/portainer-ce:latest   "/portainer"        6 hours ago      Up 6 hours      0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp, 9000/tcp   portainer

$ docker ps -a
CONTAINER ID   IMAGE                           COMMAND             CREATED             STATUS                           PORTS                                                                                            NAMES
104acdc5ee47   centos                          "/bin/bash"         34 minutes ago      Exited (0) 31 minutes ago                                                                                                         sharp_kowalevski
6601419c9d9e   tomcat                          "catalina.sh run"   36 minutes ago      Up 36 minutes                    0.0.0.0:3355->8080/tcp, :::3355->8080/tcp                                                        tomcat01
e821c9f65e4b   hello-world                     "/hello"            About an hour ago   Exited (0) About an hour ago                                                                                                      xenodochial_ride
17065d6147a4   tomcat                          "catalina.sh run"   6 hours ago         Exited (130) About an hour ago                                                                                                    charming_mendel
7e3445a24b27   portainer/portainer-ce:latest   "/portainer"        6 hours ago         Up 6 hours                       0.0.0.0:8000->8000/tcp, :::8000->8000/tcp, 0.0.0.0:9443->9443/tcp, :::9443->9443/tcp, 9000/tcp   portainer
```

#### 删除容器

此处直接输出示例：

```bash
$ docker rm 容器id                 #删除指定的容器,不能删除正在运行的容器,$ 强制删除使用 rm -f
$ docker rm -f $(docker ps -aq)   #删除所有的容器
$ docker ps -a -q|xargs docker rm #删除所有的容器
```

#### 启动和重启容器命令

此处直接输出示例：

```bash
$ docker start 容器id          #启动容器
$ docker restart 容器id        #重启容器
$ docker stop 容器id           #停止当前运行的容器
$ docker kill 容器id           #强制停止当前容器
```

### 其他命令

#### `docker logs` 查看日志

常用参数如下：

- `-t`：显示时间戳
- `-f`：持续跟踪日志输出
- `--tail`：仅列出最新 N 条容器日志

使用示例如下：

```bash
docker logs -tf 容器id
docker logs --tail 数量 容器id
```

此时，`docker` 容器后台运行，必须要有一个前台的进程，否则会自动停止

编写 `shell` 脚本循环执行，使得 `centos` 容器保持运行状态

```bash
$ docker run -d centos /bin/sh -c "while true;do echo hi;sleep 5;done"
ea997750d91c7f28dbed1cba06ff71a96d89e51f84d777cfe985e56693fc3dec

$ docker ps
(此处省略)

$ docker logs -tf --tail 10 ea997750d91c
2022-09-11T15:18:57.224063266Z hi
2022-09-11T15:19:02.230408571Z hi
2022-09-11T15:19:07.233461526Z hi
2022-09-11T15:19:12.237500674Z hi
2022-09-11T15:19:17.240665738Z hi
2022-09-11T15:19:22.244025273Z hi
```

#### `docker top 容器ID` 查看日志

```bash
$ docker top 7e3445a24b27
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                106160              106135              0                   16:52               ?                   00:00:03            /portainer
```

#### 查看容器的元数据

```bash
$ docker inspect 7e3445a24b27
```

#### 进入当前正在运行的容器

方式一：进入容器后开启一个新的终端，可以在里面操作

```bash
$ docker exec -it 容器ID /bin/bash
```

方式二：进入容器正在执行的终端，不会启动新的进程

```bash
docker attach 容器ID
```

#### 拷贝容器文件到主机

```bash
docker cp 容器id:容器内路径 目的主机路径
```

## 写在最后

本篇笔记仅作为学习之后记录常用命令。还有大量命令未列出。具体请看 Docker 官方文档，前面也有列出
