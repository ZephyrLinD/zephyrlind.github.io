# 大数据企业课 —— 完全分布式开发环境配置（一）


## 前言

一直没更博客感觉很愧疚，主要是图床的备案崩了，弄了快两个月，备个案真麻烦。

正好就这企业课需要做大数据的分布式搭配文档，写一篇针对企业的大数据环境完全分布式的搭建的文档

## NAT 设置

不多说话，直接上图：

![vmnetSettings1](https://img.zephyrl.co/images/2020/05/27/1_vmnetSettings1.png)

然后打开 NAT 选项 记住网关：

![vmnetSettings2](https://img.zephyrl.co/images/2020/05/27/2_vmnetSettings2.png)

## 虚拟机操作系统的安装

这一步我将我配置虚拟机的过程截图下来，可能与老师讲的内容不太一样，但是最终结果是一样的。这里面有我个人习惯的问题。

我习惯先弄一个不插光盘镜像的虚拟机，然后再开开，避免VMWare自动配置一些东西

然后插入光盘进入虚拟机

### 网卡名称设置

由于 CentOS7 之后默认网卡名是网卡的插槽名，不知道为啥 VMWare 的网卡插槽是 33，所以默认网卡名为 `ens33` 。按理说无所谓，可是我是个强迫症，不从 1 开始就很难受。

所以在的在开机启动页面按下 `tab` 开始自己配置网卡名，按下 `tab` 之后不用管别的二话不说直接输入一下内容：

```bash
net.ifnames=0 biosdevname=0
```

效果如下图：

![网卡名称](https://img.zephyrl.co/images/2020/05/27/3_netcardname.png)

### 网络设置

敲一下回车继续安装，稍等片刻，选择英语，再点击下一步，下拉，我们设置网络。

![网络设置页面](https://img.zephyrl.co/images/2020/05/27/4_configs.png)

点击 NetWork Settings

![NetWork Settings](https://img.zephyrl.co/images/2020/05/27/5_hostName.png)

左下角的 hostname 可以更改，比如我改成了 vm00。之后点击 Configure，按照下图所示填写相关信息

![netSetting](https://img.zephyrl.co/images/2020/05/27/6_netSettings.png)

填写完毕后点击 Done，然后点击开上图的  OFF 变绿即可。

### 硬盘设置

之后点击硬盘设置（INSTALLATION DESTINATION）

首先点击第一个箭头所指的 `I will configure...`，然后点击 Done

![diskSettings1](https://img.zephyrl.co/images/2020/05/27/7_diskSettings1.png)

然后点击下拉菜单，选择 `Standerd Partition`，点击加号

![partition](https://img.zephyrl.co/images/2020/05/27/8_diskSettings2.png)

之后按照图示给硬盘分区，我是这么分的，仅供参考，但是如果不懂那就这么分就行，绝对没问题的

![mnt](https://img.zephyrl.co/images/2020/05/27/9_diskSettings3.png)

之后点击 Accept Change

![Accept Change](https://img.zephyrl.co/images/2020/05/27/10_diskSettings4.png)

### 其他内容 & 系统软件安装

之后点击 Security Policy 和 Kdump，都关掉

![closesth](https://img.zephyrl.co/images/2020/05/27/11_diskSettings5.png)

之后安装系统软件，选择 Minimal install 的 1、2、3、6 项安装即可

![software](https://img.zephyrl.co/images/2020/05/27/12_software.png)

之后就一切结束啦，我们点击安装系统，等待系统安装后重启即可

### XShell连接SSH

在这之前我们需要修改一下本机的网络设置，我们在桌面右键网络，打开网络设置

!localnetwork](https://img.zephyrl.co/images/2020/06/02/13_localnetwork.png)

之后修改一下 `vmnet8` 的 ip 地址

![localnetwork2](https://img.zephyrl.co/images/2020/06/02/14_localnetwork2.png)

首先，互相 `ping` 一下看看通不通

电脑 ping 虚拟机：

![电脑ping虚拟机](https://img.zephyrl.co/images/2020/06/02/15_pc2vmping.png)

虚拟机在 ping 下电脑

![虚拟机ping电脑](https://img.zephyrl.co/images/2020/06/02/16.vm2pcping.png)

在之后就可以链接 xshell 啦

![xshell](https://img.zephyrl.co/images/2020/06/02/17_xshell.png)

## 软件的安装

### JDK 安装

在[这个链接](https://www.oracle.com/java/technologies/javase/javase-jdk8-downloads.html)下载 `Linux x64 Compressed Archive`

粘贴到 `/usr/local` 解压压缩包

```shell
tar zxvf jdk-8u241-linux-x64.tar.gz
rm -rf jdk-8u241-linux-x64.tar.gz
cd jdk1.8.0_241
pwd
```

获取到当前路径后，复制路径链接，配置环境变量

```shell
vim /etc/profile
```

在末尾添加如下：

```bash
# JDK 设置
export JAVA_HOME=/usr/local/jdk1.8.0_241
export JRE_HOME=$JAVA_HOME/jre
export CLASSPATH=$CLASSPATH:$JAVA_HOME/lib:$JRE_HOME/lib
export PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
```

之后将环境变量生效

```shell
source /etc/profile
```

然后运行一下 java 查看是否安装成功：

```bash
[root@vm00 ~]# java -version
java version "1.8.0_241"
Java(TM) SE Runtime Environment (build 1.8.0_241-b07)
Java HotSpot(TM) 64-Bit Server VM (build 25.241-b07, mixed mode)
```

### MySQL 安装

因为按照默认yum源安装MySQL非常慢，所以换一下yum源

首先备份原始yum源

```shell
sudo cp /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
```

之后，直接使用如下内容覆盖掉 /etc/yum.repos.d/CentOS-Base.repo 文件

```bash
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#


[base]
name=CentOS-$releasever - Base
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/os/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-7

#released updates
[updates]
name=CentOS-$releasever - Updates
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/updates/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-7



#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/extras/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-7



#additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos/$releasever/centosplus/$basearch/
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=centosplus
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-7
```

最后，更新软件包缓存

```shell
sudo yum makecache
```

在[这个链接](https://dev.mysql.com/downloads/repo/yum/)找到 `Red Hat Enterprise Linux 7 / Oracle Linux 7 (Architecture Independent), RPM Package` 并下载，个人习惯直接在虚拟机内wget

按照以下步骤安装：

```shell
wget https://dev.mysql.com/get/mysql80-community-release-el7-3.noarch.rpm
yum install mysql80-community-release-el7-3.noarch.rpm
yum install mysql-community-server
```

经过漫长的等待，安装完了 MySQL8，我们首先启动 `mysqld` 服务来获取他的默认密码：

```shell
systemctl start mysqld
```

之后，从 log 中找到临时密码

```shell
grep 'temporary password' /var/log/mysqld.log
```

然后就可以用这个密码登录 MySQL 之后修改密码，首先降低密码复杂等级和最低密码长度：

```shell
mysql> set global validate_password.policy=0;
mysql> set global validate_password.length=6;
```

然后，修改密码

```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY '123456';
```

### 安装 Hadoop

首先找到 [Hadoop 下载页面](https://hadoop.apache.org/releases.html)，复制合适的链接，然后下载到虚拟机中的 `/usr/local/` 下

我选择的是 [tuna 的镜像下载地点](https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz)

之后，解压文件

```shell
tar zxvf hadoop-2.9.2.tar.gz
```

之后记住现在的安装路径，设置一下环境变量，在 `/etc/profile` 里面添加如下内容：

```bash
# Hadoop
export HADOOP_HOME=/usr/local/hadoop-2.9.2
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
```

最后，保存设置生效并验证

```shell
[root@vm00 hadoop-2.9.2]$ source /etc/profile
[root@vm00 local]$ hadoop version
Hadoop 2.9.2
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r 826afbeae31ca687bc2f8471dc841b66ed2c6704
Compiled by ajisaka on 2018-11-13T12:42Z
Compiled with protoc 2.5.0
From source with checksum 3a9939967262218aa556c684d107985
This command was run using /usr/local/hadoop-2.9.2/share/hadoop/common/hadoop-common-2.9.2.jar
[root@vm00 local]$ hdfs dfs -ls /home
Found 1 items
drwx------   - zephyr zephyr         62 2018-04-11 12:59 /home/zephyr
```

### 安装 Hive

首先找到 [Hadoop 下载页面](http://www.apache.org/dyn/closer.cgi/hive/)，复制合适的链接，然后下载到虚拟机中的 `/usr/local/` 下

我选择的是 [tuna 的镜像下载地点](https://mirrors.tuna.tsinghua.edu.cn/apache/hive/hive-2.3.7/apache-hive-2.3.7-bin.tar.gz)

之后，解压文件

```shell
tar zxvf apache-hive-2.3.7-bin.tar.gz
```

然后记住现在的安装路径，设置一下环境变量，在 `/etc/profile` 里面添加如下内容：

```bash
# Hive
export HIVE_HOME=/usr/local/apache-hive-2.3.7-bin
export PATH=$PATH:$HIVE_HOME/bin
```

最后，保存设置生效并验证

```shell
[root@vm00 hadoop-2.9.2]$ source /etc/profile
[root@vm00 hadoop-2.9.2]$ hive --version
Hive 2.3.7
Git git://Alans-MacBook-Air.local/Users/gates/git/hive -r cb213d88304034393d68cc31a95be24f5aac62b6
Compiled by gates on Tue Apr 7 12:42:45 PDT 2020
From source with checksum 9da14e8ac4737126b00a1a47f662657e
```

### 配置 Hive

在 MySQL 中创建数据库 hive。由于 hive 有默认的数据库，但是我们需要自己的数据库，所以需要修改一下配置

1、配置文件的修改

所有配置文件均在 Hive 安装目录下的 `conf` 文件夹中

```shell
[root@vm00 conf]$ cp hive-default.xml.template hive-site.xml
[root@vm00 conf]$ vim hive-set.xml
<!--用户名-->
<property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>
<!--密码-->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>123456</value>
    </property>
<!--mysql-->
   <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://192.168.1.100:3306/hive</value>
    </property>
<!--mysql驱动程序-->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>
```

2、替换临时目录

所以将 `hive-set.xml` 中的 `${system:java.io.tmpdir}` 换成 Hive 目录下的 `tmp` 文件夹即可

3、配置 hive-env.sh 和 hive-config.sh

配置的内容是一样的，从 `hive-env.sh` 开始

```shell
mv hive-env.sh.template hive-env.sh
```

之后是 `hive-config.sh`，他在 Hive 安装目录的 `bin` 文件夹下，加入以下内容

```shell
export JAVA_HOME=/usr/local/jdk1.8.0_241
export HIVE_HOME=/usr/local/apache-hive-2.3.7-bin
export HADOOP_HOME=/usr/local/hadoop-2.9.2
```

3、复制驱动

把从[这里](https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.20.tar.gz)下载的驱动包 `mysql-connector-java-8.0.20.jar` 解压粘贴到 Hive 安装目录下的 `lib` 文件夹即可

4、在 MySQL 中 Hive 的 Schema -- 作用是在 MySQL 的 Hive 数据库下建立相关的表格

```shell
[root@vm00 local]$ schematool -dbType mysql -initSchema
```

5、执行hive

## Sqoop 安装

首先找到 [Hadoop 下载页面](http://www.apache.org/dyn/closer.lua/sqoop/)，复制合适的链接，然后下载到虚拟机中的 `/usr/local/` 下

之后，解压文件

```shell
tar zxvf sqoop-1.4.7.bin__hadoop-2.6.0.tar.gz
mv sqoop-1.4.7.bin__hadoop-2.6.0 sqoop-1.4.7
```

之后记住现在的安装路径，设置一下环境变量，在 `/etc/profile` 里面添加如下内容：

```bash
# Hadoop
export SQOOP_HOME=/usr/local/sqoop-1.4.7
export PATH=$PATH:$SQOOP_HOME/bin
```

之后对 Sqoop 进行简单配置

首先修改配置文件：

```bash
source /etc/profile
cd $SQOOP_HOME/conf
mv sqoop-env-template.sh sqoop-env.sh
```

打开sqoop-env.sh并编辑下面几行：

```shell
export HADOOP_COMMON_HOME=/usr/local/hadoop-2.9.2
export  HADOOP_MAPRED_HOME=/usr/local/hadoop-2.9.2
export HIVE_HOME=/usr/local/apache-hive-2.3.7-bin
```

加入 MySql 的 JDBC 驱动包。首先冲[这里](https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.20.tar.gz)下资 MySQL 驱动包 然后提取里面的jar文件放在 $SQOOP_HOME/lib/ 中，即可大功告成。

```bash
$ sqoop-version
20/06/16 16:55:23 INFO sqoop.Sqoop: Running Sqoop version: 1.4.7
Sqoop 1.4.7
git commit id 2328971411f57f0cb683dfb79d19d4d19d185dd8
Compiled by maugli on Thu Dec 21 15:59:58 STD 2017
```

到这里，整个Sqoop安装工作完成。
