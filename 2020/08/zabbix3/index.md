# Zabbix配置文档（三）—— 自动发现、自动注册和 SNMP


## 使用自动发现添加主机

### 配置选项

我们点击配置 - 自动发现，再点击名称进入选项

修改网段为我们需要的网段，为了快捷我设置的网段非常短，更新间隔也设置的较短，注意控制时间为每秒一个 IP 以上即可。

之后点击下面的`已启用`

![1_autoFind1.png](https://img.zephyrl.co/images/2020/08/18/1_autoFind1.png)

之后，我们选择配置 - 动作，选择 `Discovery actions` 即自动发现动作，点击名称进入设置界面

![2_autoFind2.png](https://img.zephyrl.co/images/2020/08/18/2_autoFind2.png)

动作我们没什么可修改的，点击操作 - 添加按如下操作即可

![3_autoFindAndAdd.png](https://img.zephyrl.co/images/2020/08/18/3_autoFindAndAdd.png)

### 后续操作

之后我们就可以配置好目标主机的 `zabbix-agent` 然后重启 `zabbix-server` 就可以啦

### 自动发现原理

Zabbix 通过 `Zabbix-agent` 来对主机发送 `uname` 

![5_uname.png](https://img.zephyrl.co/images/2020/08/18/5_uname.png)

来查看里面有没有 Linux 字样

![4_autoFindThe.png](https://img.zephyrl.co/images/2020/08/18/4_autoFindThe.png)


## 使用自动注册发现主机

我们点击动作，添加主机元数据包含 vm，这里的主机元数据就是 `HostMetadata`

![6_automaticAddServer.png](https://img.zephyrl.co/images/2020/08/18/6_automaticAddServer.png)

本质上，自动注册是通过修改 `zabbix_agentd.conf` 来达到快速寻找主机的目的的，我们需要修改以下内容：

```conf
Server=192.168.50.101
ServerActive=192.168.50.101
Hostname=vm04
HostMetadata=vms
```

之后我们查看一下我们是否成功修改

```bash
[root@vm04 ~]# grep -Ev '^$|#' /etc/zabbix/zabbix_agentd.conf
PidFile=/var/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
DenyKey=system.run[*]
Server=192.168.50.101
ServerActive=192.168.50.101
Hostname=vm04
HostMetadata=vms
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

修改后，刷新 Zabbix web 面板即可发现添加成功，这样速度是最快的

## 使用 SNMP 注册主机

不是所有设备都支持包管理或者编译安装 zabbix-agent，但是大部分设备都支持 SNMP 协议，所以当设备不支持的时候（比如设备是网络打印机或者交换机路由器），我们可以使用 SNMP 协议传输设备相关信息。

SNMP 监控的指标全部来自 `oid`，即 Object ID，是唯一的编号。每个我们需要的指标都有唯一的 `oid`。

汇总所有的 `oid` 信息的数据库叫做 `MIB` 数据库。它是由标准的，我们不会因为协议不同获取到错误的信息

常见的 `oid` 信息可以在这里查到：https://www.cnblogs.com/aspx-net/p/3554044.html

自动发现 SNMP 主机是通过以 `zabbix-server` 为客户端，被监控主机为 SNMP 服务端监控的，通过 `present` 认证，通过 `udp:161` 传输 `oid` 来接收被监控机器的数据

我还是以 CentOS 为例，其他主机只需要配置 SNMP 的密码即可，仅为试用例，生产环境需要具体情况具体分析

### 安装配置服务端（被监控端）

首先使用 Ansible 为所有主机安装 `snmp` 服务端

```bash
ansible all -m yum -a 'name=net-snmp state=present'
```

或者在所在主机上安装

```bash
yum install net-snmp
```

之后修改 `/etc/snmp/snmpd.conf` 修改 `community` 密码

```conf
com2sec notConfigUser  default       123456
view    systemview    included   .1
```

图示：

![8_snmpdconf.png](https://img.zephyrl.co/images/2020/08/21/8_snmpdconf.png)

之后启动并开机启动服务

```bash
systemctl start snmpd
systemctl enable snmpd
```

### 安装配置客户端（Zabbix server）

之后在 `Zabbix-Server` 上安装客户端

```bash
yum install net-snmp-utils.x86_64 -y
```

之后测试，密码（-c）为上面设置的 `community` 密码

```bash
snmpwalk -v 2c -c 123456 10.0.0.100 .1.3.6.1.4.1.2021.11.11.0
snmpwalk -v 2c -c 123456 10.0.0.100
```

测试成功后我们进入web管理页面

首先选择添加主机，按图示进行添加

![9_snmpSetting.png](https://img.zephyrl.co/images/2020/08/21/9_snmpSetting.png)

链接模板要注意，不是普通的模板，要连接 SNMP 的模板

![10_snmpSettingTemplate.png](https://img.zephyrl.co/images/2020/08/21/10_snmpSettingTemplate.png)

之后点击主机，进入宏，修改密码为上面的 `community` 密码

![11_snmpSettingPassword.png](https://img.zephyrl.co/images/2020/08/21/11_snmpSettingPassword.png)

之后重启 `zabbix-server` 即可快速完成
