# 利用 ansible 批量编译安装 zabbix-agent 实现完全自动注册


## 环境准备

准备五台按照[这篇文章](https://tech.zephyrl.co/2020/05/bigdataenvforneuedu/)的 CentOS 7 虚拟机。

![1_prepareVM.png](https://img.zephyrl.co/images/2020/08/25/1_prepareVM.png)

其中第一台按照前面那个文章装好 MySQL。之后按之前写过的 [Zabbix](https://tech.zephyrl.co/2020/08/zabbix0/) 和 [Absible](https://tech.zephyrl.co/2020/08/ansible1/) 配置文档安装配置好，将几台主机配置好加入 `ansible` 的 `Hosts` 里，ping 一下

![2_pingAll.png](https://img.zephyrl.co/images/2020/08/25/2_pingAll.png)

好的全绿，可以开始进一步操作了

## 需求分析

我们的假设是，如果机器不是常规主流操作系统，也不是常规的芯片组（比如 ARM、MIPS64）等等，我们就需要通过源代码编译。

但是源代码编译相对来说比较麻烦，我们就需要编写 Playbook 脚本让他按照我们的安排一步步走下去

也在龙芯和飞腾处理器上成功编译了，区别下面说

## 编写 Playbook

首先，需要创建 zabbix 系统用户和用户组，而且为了安全要设定 shell 为 `/sbin/nologin`

之后，从本机复制代码，到 `/usr/local/src` 中

由于本机的实验 发现没有 `pcre` 库，所以要安装 `pcre-devel`，实际生产环境各种环境齐全，应该是不用配的（如果需要配，脚本类似）

之后就是喜闻乐见的 `make && make install`

最后，用 `sed` 修改 conf 文件，配置好以后添加到系统服务中

根据以上需求，编写 `yaml`

```yaml
---
- hosts: vms
  remote_user: root

  tasks:
    - name: Create zabbix group
      group: name=zabbix system=yes
    - name: Create zabbix user
      user: name=zabbix system=yes shell=/sbin/nologin
    - name: unarchive zabbix source
      unarchive: src=/usr/local/src/zabbix-5.0.2.tar.gz dest=/usr/local/src
    - name: install pcre-devel
      yum: name=pcre-devel state=present
    - name: make and make install
      shell: cd /usr/local/src/zabbix-5.0.2 && ./configure --prefix=/usr/local/zabbix_agent --enable-agent && make && make install
    - name: copy the systemctl file to target
      copy: src=zabbix-agent.service.bak dest=/usr/lib/systemd/system/zabbix-agent.service
    - name: mkdir for PidFile
      shell: mkdir /var/run/zabbix/ && chown -R zabbix:zabbix /var/run/zabbix/
    - name: change server and serverActice
      shell: sed -i "s#127.0.0.1#192.168.1.169#g" /usr/local/zabbix_agent/etc/zabbix_agentd.conf
    - name: change host name
      shell: sed -i "s#Zabbix server#`hostname -I`#g" /usr/local/zabbix_agent/etc/zabbix_agentd.conf
    - name: change hostmetadata
      shell: echo 'HostMetadata=x86' >> /usr/local/zabbix_agent/etc/zabbix_agentd.conf
    - name: add pidFile
      shell: echo "PidFile=/var/run/zabbix/zabbix_agentd.pid" >> /usr/local/zabbix_agent/etc/zabbix_agentd.conf
    - name: add other
      shell: echo "LogFileSize=0" >> /usr/local/zabbix_agent/etc/zabbix_agentd.conf
    - name: start service
      service: name=zabbix-agent state=started enabled=yes
```

在编写里面的 `zabbix-agent.service`

```service
[Unit]
Description=Zabbix Agent
After=syslog.target
After=network.target

[Service]
Environment="CONFFILE=/usr/local/zabbix_agent/etc/zabbix_agentd.conf"
#EnvironmentFile=/usr/local/zabbix/etc/zabbix_agentd.conf.d/
Type=forking
Restart=on-failure
PIDFile=/run/zabbix/zabbix_agentd.pid
KillMode=control-group
ExecStart=/usr/local/zabbix_agent/sbin/zabbix_agentd -c $CONFFILE
ExecStop=/bin/kill -SIGTERM $MAINPID
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

## 问题

由于最早我没做相关 PID 的相关代码，会造成偶尔无法启动的bug，排错的时候顺便加上了，没想到就成了......

我也不知道咋回事，反正加上就好了

## 运行

执行 playbook 命令：

```bash
ansible-playbook compileAgent_x86_rhel.yaml -k
```

输入密码后，成功运行

![3_doSuccess.png](https://img.zephyrl.co/images/2020/08/25/3_doSuccess.png)

此时切回 `Zabbix web` 页面也可发现添加托管成功

![4_regist.png](https://img.zephyrl.co/images/2020/08/25/4_regist.png)

我们查看动作日志，也可以发现成功的日志，只是不知道为啥没发通过任何方法发送到钉钉，倒是可以发送邮件，而且自动发现也不是什么重要的预警。我就没再研究。

## 总结

利用 Ansible-playbook 可以大大减少安装 Zabbix-agent 的步骤，而且可以大批量的运行，尤其是需要编译的时候，这个脚本省了很多事。

目前唯一的问题就是自动发现的日志没法实时推送到钉钉群。以后有机会继续研究一下
