# Ansible 学习笔记（二）：Ad-Hoc 使用


## Ansible 命令

Ansible系列命令如下：

```bash
ansible ansible-doc ansible-playbook ansible-vault ansible-console ansible-galaxy ansible-pull
```

如果要查看相关模块帮助，可以使用 `ansible-doc` 命令：

```bash
ansible-doc [options] [module...]
    -a            显示所有模块的文档
    -l, --list    列出可用模块
    -s, --snippet 显示指定模块的playbook片段(简化版,便于查找语法)
```

示例：

|        命令         |             作用             |
| :-----------------: | :--------------------------: |
|   ansible-doc -l    |         列出所有模块         |
|  ansible-doc ping   |     查看ping模块帮助用法     |
| ansible-doc -s ping | 查看ping模块帮助用法（简化） |

## Ansible 使用细则

ansible通过ssh实现配置管理、应用部署、任务执行等功能

建议配置ansible端能基于密钥认证的方式联系各被管理节点

```bash
ansible <host-pattern> [-m module_name] [-a args]
    --version              显示版本
    -m module              指定模块，默认为command
    -v                     详细过程 –vv -vvv更详细
    --list-hosts           显示主机列表，可简写 --list
    -k, --ask-pass         提示输入ssh连接密码,默认Key验证
    -C, --check            检查，并不执行
    -T, --timeout=TIMEOUT  执行命令的超时时间,默认10s
    -u, --user=REMOTE_USER 执行远程执行的用户
    -b, --become           代替旧版的sudo切换
        --become-user=USERNAME 指定sudo的runas用户,默认为root
    -K, --ask-become-pass  提示输入sudo时的口令
```

举例：

```shell 
ansible all --list      # 列出所有主机
ansible-doc -s ping     # 查看ping模块的语法
```

## ansible 的 Host-pattern

### All 表示所有Inventory中的所有主机

```bash
ansible all –m ping
```

### * :通配符

```bash
ansible "*" -m ping  (*表示所有主机)
ansible 192.168.1.* -m ping
ansible "*srvs" -m ping
```

### 或关系 ":"

```bash
ansible "websrvs:appsrvs" -m ping
ansible “192.168.1.10:192.168.1.20” -m ping
```

### 逻辑与 ":&"

在websrvs组并且在dbsrvs组中的主机

```bash
ansible "websrvs:&dbsrvs" –m ping
```

### 逻辑非 ":!"

在websrvs组，但不在dbsrvs组中的主机

```bash
ansible 'websrvs:!dbsrvs' –m ping
```

注意：此处为单引号

### 综合逻辑

```bash
ansible 'websrvs:dbsrvs:&appsrvs:!ftpsrvs' –m ping
```

### 正则表达式

```bash
ansible "websrvs:&dbsrvs" –m ping
ansible "~(web|db).*\.magedu\.com" –m ping
```

## ansible 常用模块

模块文档：https://docs.ansible.com/ansible/latest/modules/modules_by_category.html

### Command

在远程主机执行命令，默认模块，可忽略-m选项

```bash
ansible srvs -m command -a 'service vsftpd start'
ansible srvs -m command -a 'echo adong |passwd --stdin 123456'
```

此命令不支持 `$VARNAME` `<` `>` `|` `;` `&` 等高端用法，但是他们可以用 `shell` 模块实现

### Shell

和command相似，用shell执行命令

```bash
ansible all -m shell  -a 'getenforce'  查看SELINUX状态
ansible all -m shell  -a "sed -i 's/SELINUX=.*/SELINUX=disabled' /etc/selinux/config"
ansible srv -m shell -a 'echo zephyr | passwd –stdin zephyr'
```

调用bash执行命令 类似 `cat /tmp/stanley.md | awk -F'|' '{print$1,$2}' &> /tmp/example.txt` 这些复杂命令，即使使用shell也可能会失败

解决办法：写到脚本时，copy到远程执行，再把需要的结果拉回执行命令的机器

如果可以的话，我们可以修改配置文件,使shell作为默认模块

打开 `/etc/ansible/ansible.cfg` 修改

```conf
module_name = shell
```

### Script

在远程主机上运行ansible服务器上的脚本

```shell
ansible websrvs -m script -a /data/test.sh
```

### Copy

从主控端复制文件到远程主机

|  参数   |                               意义                                |
| :-----: | :---------------------------------------------------------------: |
|   src   | 源文件，指定拷贝文件的本地路径(如果有/则拷贝目录内比拷贝目录本身) |
|  dest   |                           指定目标路径                            |
|  mode   |                             设置权限                              |
| backup  |                            备份源文件                             |
| content |            代替src  指定本机文件内容,生成目标主机文件             |

如果目标存在，默认覆盖，此处指定先备份

```shell
ansible websrvs -m copy -a "src=/root/test1.sh dest=/tmp/test2.showner=wang mode=600 backup=yes"
```

指定内容，直接生成目标文件

```shell
ansible websrvs -m copy -a "content='test content\nxxx' dest=/tmp/test.txt"
```

### Fetch

从远程主机提取文件至主控端，copy相反，目前不支持目录,可以先打包,再提取文件

会生成每个被管理主机不同编号的目录,不会发生文件名冲突

```bash
ansible websrvs -m fetch -a 'src=/root/test.sh dest=/datscripts'
```

```bash
ansible all -m shell -a 'tar jxvf test.tar.gz /root/test.sh'
ansible all -m fetch -a 'src=/root/test.tar.gz dest=/data/'
```

### File

设置文件属性：

|  参数   |                                     意义                                      |
| :-----: | :---------------------------------------------------------------------------: |
|  path   |                          要管理的文件路径 (强制添加)                          |
| recurse |                              递归,文件夹要用递归                              |
|   src   | 创建硬链接,软链接时,指定源目标,配合'state=link''state=hard' 设置软链接,硬链接 |
|  state  |                           状态（absent 缺席，删除）                           |

```shell
ansible websrvs -m file -a 'path=/app/test.txtstate=touch'                              # 创建文件
ansible websrvs -m file -a "path=/data/testdirstate=directory"                          # 创建目录    
ansible websrvs -m file -a "path=/root/test.sh owner=wangmode=755"                      # 设置权限755
ansible websrvs -m file -a 'src=/data/testfile dest=/datatestfile-link state=link'      # 创建软链接
```

### unarchive

解包解压缩，有两种用法:

1. 将ansible主机上的压缩包传到远程主机后解压缩至特定目录，设置copy=yes.

2. 将远程主机上的某个压缩包解压缩到指定路径下，设置copy=no

常见参数：

- copy：默认为yes，当copy=yes，拷贝的文件是从ansible主机复制到远程主机上，如果设置为copy=no，会在远程主机上寻找src源文件
- src： 源路径，可以是ansible主机上的路径，也可以是远程主机上的路径，如果是远程主机上的路径，则需要设置copy=no
- dest：远程主机上的目标路径
- mode：设置解压缩后的文件权限

示例：

```bash
ansible websrvs -m unarchive -a 'src=foo.tgz dest=/var/lib/foo'   #默认copy为yes ,将本机目录文件解压到目标主机对应目录下
ansible websrvs -m unarchive -a 'src=/tmp/foo.zip dest=/data copy=no mode=0777' # 解压被管理主机的foo.zip到data目录下, 并设置权限777
ansible websrvs -m unarchive -a 'src=https://example.com/example.zip dest=/data copy=no'
```

### Archive

打包压缩

将远程主机目录打包

```bash
ansible all -m archive -a 'path=/etc/sysconfig dest=/data/sysconfig.tar.bz2 format=bz2 owner=wang mode=0777'
```

|  参数  |     意义     |
| :----: | :----------: |
|  path  |   指定路径   |
|  dest  | 指定目标文件 |
| format | 指定打包格式 |
| owner  |  指定所属者  |
|  mode  |   设置权限   |

### Hostname

管理主机名示例：

```bash
ansible appsrvs -m hostname -a "name=app.adong.com"             # 更改一组的主机名
ansible 192.168.38.103 -m hostname -a "name=app2.adong.com"     # 更改单个主机名
```

### Cron

有点类似于 `Crontab`，计划任务，其实也是添加 `Crontab`

支持时间：minute,hour,day,month,weekday

```bash
ansible websrvs -m cron -a "minute=*/5 job='/usr/sbin/ntpdate 172.16.0.1 &>/dev/null' name=Synctime"
```

创建任务:

```bash
ansible websrvs -m cron -a 'state=absent name=Synctime'
```

删除任务：

```bash
ansible websrvs -m cron -a 'minute=*/10 job='/usr/sbin/ntpdate 172.30.0.100" name=synctime disabled=yes'
```

删除任务的本质就是注释 `Crontab` 任务，不在生效

### Yum

管理包：

```shell
ansible websrvs -m yum -a 'list=httpd'  查看程序列表
ansible websrvs -m yum -a 'name=httpd state=present' 安装
ansible websrvs -m yum -a 'name=httpd state=absent'  删除
```

可以同时安装多个程序包

### Service

管理服务（systemctl）：

```bash
ansible srv -m service -a 'name=httpd state=stopped'  停止服务
ansible srv -m service -a 'name=httpd state=started enabled=yes' 启动服务,并设为开机自启
ansible srv -m service -a 'name=httpd state=reloaded'  重新加载
ansible srv -m service -a 'name=httpd state=restarted' 重启服务
```

### User

管理用户：

参数|意义
:-:|:-:
home|指定家目录路径
system|指定系统账号
group|指定组
remove|清除账户
shell|指定shell类型

用例：

```bash
ansible websrvs -m user -a 'name=user1 comment="test user" uid=2048 home=/app/user1 group=root'
ansible websrvs -m user -a 'name=sysuser1 system=yes home=/app/sysuser1'
ansible websrvs -m user -a 'name=user1 state=absent remove=yes'  清空用户所有数据
ansible websrvs -m user -a 'name=app uid=88 system=yes home=/app groups=root shell=/sbin/nologin password="$1$zfVojmPy$ZILcvxnXljvTI2PhP2Iqv1"'  创建用户
ansible websrvs -m user -a 'name=app state=absent'  不会删除家目录
```

删除用户及家目录等数据

Group：管理组

```bash
ansible srv -m group -a "name=testgroup system=yes"   创建组
ansible srv -m group -a "name=testgroup state=absent" 删除组
```

其他模块也可通过上面的文档链接来仔细查阅
