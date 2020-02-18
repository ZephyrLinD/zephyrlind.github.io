# 大数据学习笔记（一） —— Hadoop运行环境的搭建


## 网络环境配置

这里以我用的 VMWare 做例子。首先修改的是虚拟网络编辑器

我将连接模式改为了 NAT 模式，子网 IP 改为 192.168.0.1，子网掩码改成 255.255.255.0（24位）

![1_virtualnet.png](https://img.zephyrl.co/images/2020/02/16/1_virtualnet.png)

由于我没有其他的虚拟机，我就只弄了一个虚拟网卡。

然后安装 CentOS，我安装的是 CentOS7 版本。等安装完毕后，我们首先打开终端，输入 `ifconfig`，查看与我们之前设置相同的网络

![2_ifconfig.png](https://img.zephyrl.co/images/2020/02/16/2_ifconfig.png)

我的图时候来截的，已经配置好，请无视qaq

然后打开 虚拟机 —— 选项 —— 网络适配器 —— 高级 复制一下 mac 地址

![3_mac.png](https://img.zephyrl.co/images/2020/02/16/3_mac.png)

然后，打开 `/etc/sysconfig/network-scripts/ifcfg-` 这个文件，文件名的后面就是 ifconfig 查到的网络名称。比如我就是 ens33

文件内容如下：

```bash
$ cat /etc/sysconfig/network-scripts/ifcfg-ens33
HWADDR=00:0C:29:99:54:8F
TYPE="Ethernet"
PROXY_METHOD="none"
BROWSER_ONLY="no"
BOOTPROTO="static"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
UUID="5234c912-54b6-4a64-8076-5dfbc1272737"
DEVICE="ens33"
ONBOOT="yes"

IPADDR=192.168.1.100
GATEWAY=192.168.1.2
DNS1=192.168.1.2
```

之后打开 windows 网络设置，修改一些配置

![4_netwin.png](https://img.zephyrl.co/images/2020/02/16/4_netwin.png)

可以从控制面板中找到网络设置，也可以通过 win10 的搜索功能，win+s 搜索`网络`找到。

如图，我给自己分配的 ip 是 192.168.1.5，此时主机和虚拟机互相 ping 一下，可以 ping 通即成功。

## 安装 JDK

首先卸载自带的 openjdk

```bash
[root@hadoop000 ~]# java -version
openjdk version "1.8.0_242"
OpenJDK Runtime Environment (build 1.8.0_242-b08)
OpenJDK 64-Bit Server VM (build 25.242-b08, mixed mode)
[root@hadoop000 ~]# rpm -qa | grep java
python-javapackages-3.4.1-11.el7.noarch
javapackages-tools-3.4.1-11.el7.noarch
tzdata-java-2019c-1.el7.noarch
java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64
```

需要删掉 `java-1.8.0` 开头的。执行以下命令

```bash
rpm -e --nodeps java-1.8.0-openjdk-1.8.0.242.b08-0.el7_7.x86_64
rpm -e --nodeps java-1.8.0-openjdk-headless-1.8.0.242.b08-0.el7_7.x86_64
```

此时再执行 `java -version` 操作的时候就不会看见OpenJDK的版本了，说明卸载成功

此时下载 [Oracle JDK 1.8](https://www.oracle.com/java/technologies/javase-jdk8-downloads.html) 官网的 **Linux x64 Compressed Archive** 包

通过 *SFTP* 等工具解压到 `/usr/local` 文件夹内，进入文件夹，记录位置并且复制

（因为下载 Oracle 的 JDK1.8需要浏览器 cookie 认证，所以我们需要这种办法）

接着，打开 `/etc/profile` 配置环境变量，在最下面加入如下

```shell
## JAVA Home
## /usr/local/jdk1.8.0_241 这个路径是我记录下来的
export JAVA_HOME=/usr/local/jdk1.8.0_241
export PATH=$PATH:$JAVA_HOME/bin
```

此时配置好了不要忘了 `source /etc/profile`，在执行 `java -version` 即可发现正确的 JAVA 版本

## 安装 Hadoop

首先从 [Apache Hadoop 的 release 页面](https://hadoop.apache.org/releases.html)下载相关版本的 hadoop

wget到 `/usr/local`，解压缩，进入解压的文件夹里，`pwd` 记录路径

继续打开 `/etc/profile`

```shell
## Hadoop Home
export HADOOP_HOME=/usr/local/hadoop-2.10.0
export PATH=$PATH:$HADOOP_HOME/bin
export PATH=$PATH:$HADOOP_HOME/sbin
```

此时配置好了不要忘了 `source /etc/profile`

## 虚拟机的复制

复制之前，关闭主机防火墙。在实验中我们为了省事关闭了防火墙，在实际生产环境中，需要从云服务器设置开放的端口或者从 firewalld 开放相关端口

```bash
sudo systemctl disable firewalld
```

之后，打开 /etc/hosts 文件，增加如下内容

```bash
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.1.100 hadoop100
192.168.1.101 hadoop101
192.168.1.102 hadoop102
192.168.1.103 hadoop103
192.168.1.104 hadoop104
192.168.1.105 hadoop105
192.168.1.106 hadoop106
192.168.1.107 hadoop107

192.168.1.5 laptop
```

这样就可以方便的通过简单的名称找到集群其他的虚拟机了

之后就可以嗨皮的跟着 VMWare 复制虚拟机啦
