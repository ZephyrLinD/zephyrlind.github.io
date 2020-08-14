# Ansible 学习笔记（一）：安装配置和初步使用


## 安装 Ansible

### Yum 通过 Epel 源安装

Ansible 位于 `epel` 源，我们在安装 Ansible 之前需要安装 `epel-release`

```shell
yum install epel-release
```

之后就可以通过 yum 安装 Ansible 了

```shell
yum install ansible
```

### 编译安装

1、安装运行库

```bash
yum -y install python-jinja2 PyYAML python-paramiko python-babel
python-crypto
```

2、下载解压源码包

```bash
wget https://github.com/ansible/ansible/archive/v2.9.12.tar.gz
mv v2.9.12.tar.gz /usr/local
tar zxvf v2.9.12.tar.gz
cd 
```

3、编译并安装

```bash
python setup.py build
python setup.py install
mkdir /etc/ansible
cp -r examples/* /etc/ansible
```

## Ansible 相关文件

配置文件：

|            目录            |                     功能                     |
| :------------------------: | :------------------------------------------: |
| `/etc/ansible/ansible.cfg` | 主配置文件,配置ansible工作特性(一般无需修改) |
|    `/etc/ansible/hosts`    |      主机清单(将被管理的主机放到此文件)      |
|   `/etc/ansible/roles/`    |                存放角色的目录                |


程序：

|            目录             |                  功能                  |
| :-------------------------: | :------------------------------------: |
|     `/usr/bin/ansible`      |        主程序，临时命令执行工具        |
|   `/usr/bin/ansible-doc`    |     查看配置文档，模块功能查看工具     |
|  `/usr/bin/ansible-galaxy`  | 下载/上传优秀代码或Roles模块的官网平台 |
| `/usr/bin/ansible-playbook` |      定制自动化任务，编排剧本工具      |
|   `/usr/bin/ansible-pull`   |           远程执行命令的工具           |
|  `/usr/bin/ansible-vault`   |              文件加密工具              |
| `/usr/bin/ansible-console`  |  基于Console界面与用户交互的执行工具   |

## 第一个命令

我们用一个 `ping` 命令来开始 `ansible` 的学习

```bash
[root@zep0 ~]# ansible 192.168.1.136 -m ping -k
SSH password:
192.168.1.136 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

输入 SSH 密码之后，收到以上回应即为成功

## 一些配置

### 添加 Inventory 主机清单

ansible 的主要功用在于批量主机操作，为了便捷地使用其中的部分主机，可以在 inventory file 中将其分组命名 

默认的 inventory file 为 `/etc/ansible/hosts`

inventory file 可以有多个，且也可以通过 Dynamic Inventory 来动态生成

打开 `/etc/ansible/hosts`，可以发现上面注释了好多的示例，我们可以按照他的注释编写自己的 `hosts` 主机文件

```ini
[vms]
192.168.50.101
192.168.50.102
192.168.50.103

[servers]
192.168.1.169
192.168.1.136
```

### 修改推荐的设置

Ansible 配置文件/etc/ansible/ansible.cfg （一般保持默认）

其中 Default 项目和意义如下

```conf
[defaults]
#inventory     = /etc/ansible/hosts      # 主机列表配置文件
#library       = /usr/share/my_modules/  # 库文件存放目录
#remote_tmp    = $HOME/.ansible/tmp      # 临时py命令文件存放在远程主机目录
#local_tmp     = $HOME/.ansible/tmp      # 本机的临时命令执行目录  
#forks         = 5                       # 默认并发数,同时可以执行5次
#sudo_user     = root                    # 默认sudo 用户
#ask_sudo_pass = True                    # 每次执行ansible命令是否询问ssh密码
#ask_pass      = True                    # 每次执行ansible命令是否询问ssh口令
#remote_port   = 22                      # 远程主机的端口号(默认22)
```

我们打开 `/etc/ansible/ansible.cfg` 取消注释修改一下配置

```conf
host_key_checking = False
log_path = /var/log/ansible.log
module_name = command
```

其中，第一个是禁用 SSH 的连接检查，如果不禁用我们需要通过 SSH 链接过目标主机，如未连接过则会报错

第二个是用来开启 Ansible 的日志记录

第三个指的是默认的命令，默认是执行 `bash` 命令
