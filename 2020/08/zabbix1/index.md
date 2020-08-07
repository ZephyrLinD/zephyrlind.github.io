# Zabbix配置文档（一）—— 监控和触发器


## 添加自定义监控案例

我们想添加一条对于 sda 硬盘的 TPS 监控，我们不禁想到了命令 `iostat` 、

### 编写命令获取 TPS 值

首先我们通过命令来查看我们要取的值在哪

```bash
[root@zep2 ~]# iostat
Linux 3.10.0-862.el7.x86_64 (zep2.test)         08/05/2020      _x86_64_        (2 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           2.33    0.00    2.71    0.06    0.00   94.90

Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
sda               0.42         3.36        66.43     540882   10699175
dm-0              0.31         2.99        32.73     482054    5271735
dm-1              8.50         0.32        33.68      50792    5425372
```

所以我们可以使用以下命令

```bash
[root@zep2 ~]# iostat | awk '/sda/{print $2}'
0.42
```

### 添加到配置文件中

我们打开 `/etc/zabbix/zabbix_agentd.conf`，找到 `UserParameter` 字段，按照 <key, value> 写入

key 代表这个动作的名字，value 就是这个动作的命令

```shell
UserParameter=sda_tps, iostat | awk '/sda/{print $2}'
```

之后我们重启服务即可生效

```shell
systemctl restart zabbix-agent
```

### 测试是否成功取值

### 添加到 Web 管理页面中

按照如下图片顺序选择监控项

![6-findHowToAddMonitor1.png](https://img.zephyrl.co/images/2020/08/05/6-findHowToAddMonitor1.png)

![7-findHowToAddMonitor2.png](https://img.zephyrl.co/images/2020/08/05/7-findHowToAddMonitor2.png)

然后就可以添加监控项啦

![8-sda_tps.png](https://img.zephyrl.co/images/2020/08/05/8-sda_tps.png)

我们进入最新数据看一下，可以发现已经成功取值

![9-getItem.png](https://img.zephyrl.co/images/2020/08/05/9-getItem.png)

## 自定义触发器

### 更改触发器的阈值

我们修改 Swap 的阈值为 10s，这样方便发现错误

![1_swapTriggerTest.png](https://img.zephyrl.co/images/2020/08/05/1_swapTriggerTest.png)

之后我们再目标主机执行以下命令，关掉 Swap

```shell
swapoff -a
```

不久之后就可以发现，服务器成功报警

![2_swapTriggerTestSuccess.png](https://img.zephyrl.co/images/2020/08/05/2_swapTriggerTestSuccess.png)

然后我们重新开启 swap 即可

```shell
swapon -a
```

### 自定义触发器监控登陆人数（Zabbix 5 未发现相关函数）、

首先，按照配置-主机-触发器-创建触发器来创建触发器

![3_findMakeTrigger.png](https://img.zephyrl.co/images/2020/08/05/3_findMakeTrigger.png)

![4_findMakeTrigger2.png](https://img.zephyrl.co/images/2020/08/05/4_findMakeTrigger2.png)

之后，在新的页面上修改如下

![5_tirggerInfoSetting.png](https://img.zephyrl.co/images/2020/08/05/5_tirggerInfoSetting.png)

此时，多开几个 SSH 窗口即可报警

但是在 Zabbix 5 上并没发现类似的模板，在查阅资料后会补上（找不到就不补了 意思明白了）
