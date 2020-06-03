# 大数据企业课 —— 完全分布式开发环境配置（二）


## 设置 Hosts

打开我们上次配置好的 vm00 虚拟机的 `/etc/hosts` 文件，加入以下内容：

```shell
192.168.1.100 vm00
192.168.1.101 vm01
192.168.1.102 vm02
192.168.1.103 vm03
192.168.1.104 vm04
192.168.1.105 vm05
```

之后进入本机（Windows）的 `C:\Windows\system32\Drivers\etc\` 下编辑 `hosts` 文件，将以上内容加入

## 克隆虚拟机

把 vm00 关机，右键管理，选择克隆

![cloneVM](https://img.zephyrl.co/images/2020/06/02/1_cloneVm.png)

这里选择完整克隆即可，后面按照配置虚拟机的时候配置位置就可以啦！

![2_cloneVm2.png](https://img.zephyrl.co/images/2020/06/02/2_cloneVm2.png)

克隆三个（001-003）即可

修改网络时可以通过相对直观的 `nmtui` 来修改 IP 地址和主机名称。

也可以通过以下命令更改

```shell
nmcli connection modify eth0 ipv4.address 192.168.1.101/24
nmcli connection up eth0
hostnamecli set-hostname vm01
```

不论如何修改，不要忘记

```bash
systemctl restart network
```

## SSH 通信

在第一个终端输入以下命令获得当前的 SSH Keygen

```bash
[zephyr@vm01 ~]$ ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/home/zephyr/.ssh/id_rsa): 
Created directory '/home/zephyr/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/zephyr/.ssh/id_rsa.
Your public key has been saved in /home/zephyr/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:KJe7A4mQqYqEY+V9pUUg84Np6Jq87BzZcZfk39iNG/Y zephyr@vm01
The key's randomart image is:
+---[RSA 2048]----+
|     o ..        |
|    . *  .       |
| o . + +.        |
|+ ... oooo       |
|o.o+oo++S        |
|=o*.=+.+. + o    |
|=B . .o  o * .   |
|= o   ..  . +    |
|.=    ..   . E   |
+----[SHA256]-----+
[zephyr@vm01 ~]$ ls .ssh
id_rsa  id_rsa.pub
```

其中，id_rsa 为 私钥，id_rsa.pub 为公钥

之后，把私钥放入认证文件中

```bash
[zephyr@vm01 ~]$ cd .ssh
[zephyr@vm01 .ssh]$ cat id_rsa.pub >> authorized_keys
```

之后修改权限如下：

```bash
[zephyr@vm01 .ssh]$ cd ..
[zephyr@vm01 ~]$ chmod 700 ./.ssh
[zephyr@vm01 ~]$ chmod 644 ./.ssh/authorized_keys
[zephyr@vm01 ~]$ chmod 600 ./.ssh/id_rsa
```

最后在其他虚拟机里执行以下步骤：

```shell
mkdir .ssh
cd .ssh
touch authorized_keys
cd ..
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

然后再第一个虚拟机下，复制 authorized_keys 到其他虚拟机

```bash
scp .ssh/authorized_keys root@vm02:~/.ssh/authorized_keys
```

## Hadoop 分布式配置

之后，在 `vm01` 虚拟机，进入 `/usr/local/hadoop-2.9.2/etc/hadoop` 文件夹，

打开 `core-site.xml`，输入以下内容：

```xml
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://vm01/</value>
</property>
```

打开 `hdfs-site.xml`，输入以下内容：

```xml
<property>
    <name>dfs.replication</name>
    <value>3</value>
</property>
```

然后复制文件 `cp mapred-site.xml.template mapred-site.xml`，并打开 `mapred-site.xml`，输入以下内容

```xml
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
```

打开 `yarn-site.xml`，输入以下内容：

```xml
<property>
    <name>yarn.resourcemanager.hostname</name>
    <value>vm01</value>
</property>
<property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
</property>
```

打开 `slaves`，输入以下内容：

```slaves
vm02
vm03
vm04
```

至此，所有配置写入完毕，我们来分发配置

```shell
scp /usr/local/hadoop-2.9.2/etc/hadoop/* root@vm02:/usr/local/hadoop-2.9.2/etc/hadoop/
scp /usr/local/hadoop-2.9.2/etc/hadoop/* root@vm03:/usr/local/hadoop-2.9.2/etc/hadoop/
scp /usr/local/hadoop-2.9.2/etc/hadoop/* root@vm04:/usr/local/hadoop-2.9.2/etc/hadoop/
```

之后我们就可以格式化 NameNode 并且启动集群了

```shell
hadoop namenode -format
start-all.sh
```

一切启动好了可以去 vm01:50070 和 vm01:8088 看一看即可

![3_overview.png](https://img.zephyrl.co/images/2020/06/03/3_overview.png)
