# Ansible 学习笔记（三）：Ansible PlayBook 入门


## Playboot 介绍

playbook 是由一个或多个"play"组成的列表

play 的主要功能在于将预定义的一组主机，装扮成事先通过 ansible 中的 task 定义好的角色。

Task实际是调用 ansible 的一个 module，将多个 play 组织在一个 playbook 中，

即可以让它们联合起来，按事先编排的机制执行预定义的动作

Playbook 采用 YAML 语言编写

![1.png](https://img.zephyrl.co/images/2020/08/14/1.png)

用户通过 ansible 命令直接调用 yml 语言写好的 playbook，playbook 由多条 play 组成

每条 play 都有一个任务(task)相对应的操作，然后调用模块 modules，应用在主机清单上，通过 ssh 远程连接

从而控制远程主机或者网络设备

## PlayBook 核心元素

|           元素           |                         意义                         |
| :----------------------: | :--------------------------------------------------: |
|          Hosts           |         执行的远程主机列表(应用在哪些主机上)         |
|          Tasks           |                        任务集                        |
|        Variables         |         内置变量或自定义变量在playbook中调用         |
|      Templates模板       |    可替换模板文件中的变量并实现一些简单逻辑的文件    |
| Handlers和notify结合使用 |  由特定条件触发的操作，满足条件方才执行，否则不执行  |
|           tags           | 指定某条任务执行，用于选择运行playbook中的部分代码。 |

> ansible具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过tags跳过此些代码片断

```bash
ansible-playbook -t tagsname useradd.yml
```

## Playbook 基础组件

### Hosts

playbook中的每一个play的目的都是为了让特定主机以某个指定的用户身份执行任务。

hosts用于指定要执行指定任务的主机，须事先定义在主机清单中

可以是如下形式：

```conf
one.example.com
one.example.com:two.example.com
192.168.1.50
192.168.1.*
Websrvs:dbsrvs       或者，两个组的并集
Websrvs:&dbsrvs      与，两个组的交集
webservers:!phoenix  在websrvs组，但不在dbsrvs组
```

示例: - hosts: websrvs：dbsrvs

### remote_user

可用于Host和task中。

也可以通过指定其通过sudo的方式在远程主机上执行任务，其可用于play全局或某任务；

此外，甚至可以在sudo时使用sudo_user指定sudo时切换的用户

- hosts: websrvs

    ```conf
    remote_user: root   (可省略,默认为root)  以root身份连接
    tasks:    指定任务
    ```

- name: test connection
  
    ```conf
    ping:
    remote_user: zephyr
    sudo: yes               # 默认sudo为root
    sudo_user:zephyr        # sudo为zephyr
    ```

### task 列表和 action

任务列表task:由多个动作,多个任务组合起来的,每个任务都调用的模块,一个模块一个模块执行

1. play的主体部分是 task list，task list 中的各任务按次序逐个 hosts 中指定的所有主机上执行，即在所有主机上完成第一个任务后，再开始第二个任务

2. task 的目的是使用指定的参数执行模块，而在模块参数中可以使用量。
   模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致

3. 每个 task 都应该有其 name，用于 playbook 的执行结果输出，建议其内能清晰地描述任务执行步骤。

如果未提供 name，则 action 的结果将用于输出

## Ansible-Playbook 初级用例

### Playbook 创建用户

```yaml
- hosts: all
  remote_user: root

  tasks:
    - name: create mysql user
      user: name=mysql system=yes uid=36
    - name: create a group
      group: name=httpd system=yes
```

### 安装 Apache httpd 服务

```yaml
- hosts: vms
  remote_user: root

  tasks:
    - name: Install httpd
      yum: name=httpd state=present
    - name: Install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
    - name: start service
      service: name=httpd state=started enabled=yes
```

这里的 `files` 文件夹存在于 `~/.ansible` 中

### 安装 Nginx 服务

```yaml
- hosts: all
  remote_user: root

  tasks:
    - name: add group nginx
      user: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: Start Nginx
      service: name=nginx state=started enabled=yes
```

## Handlers 和 notify

Handlers 实际上就是一个触发器

是 task 列表，这些 task 与前述的 task 并没有本质上的不同,用于当关注的资源发生变化时，才会采取一定的操作

Notify 此 action 可用于在每个 play 的最后被触发，这样可避免多次有改变发生时每次都执行指定的操作，仅在所有的变化发生完成后一次性地执行指定操作。

在 notify 中列出的操作称为 handler，也即 notify 中调用 handler 中定义的操作

### 当 `httpd.conf` 改变的时候重启 httpd 服务

```yaml
- hosts: websrvs
  remote_user: root

  tasks:
    - name: Install httpd
      yum: name=httpd state=present
    - name: Install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
      notify: restart httpd
    - name: ensure apache is running
      service: name=httpd state=started enabled=yes
  
  handlers:
    - name: restart httpd
      service: name=httpd state=restarted
```

### 更新 Nginx 新的配置并重启服务

```yaml
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: add group nginx
      tags: user
      user: name=nginx state=present
    - name: add user nginx
      user: name=nginx state=present group=nginx
    - name: Install Nginx
      yum: name=nginx state=present
    - name: config
      copy: src=/root/config.txt dest=/etc/nginx/nginx.conf
      notify:
        - Restart Nginx
        - Check Nginx Process
  
  handlers:
    - name: Restart Nginx
      service: name=nginx state=restarted enabled=yes
    - name: Check Nginx process
      shell: killall -0 nginx > /tmp/nginx.log
```

> 注：可以通过向进程发送信号0，然后根据返回值来判断一个进程是否存在。

## Tags 标签

可以指定某一个任务添加一个标签,添加标签以后,想执行某个动作可以做出挑选来执行

多个动作可以使用同一个标签

```yaml
# httpd.yaml
- hosts: websrvs
  remote_user: root
  
  tasks:
    - name: Install httpd
      yum: name=httpd state=present
      tage: install 
    - name: Install configure file
      copy: src=files/httpd.conf dest=/etc/httpd/conf/
      tags: conf
    - name: start httpd service
      tags: service
```

此为 `httpd.yaml`，我们执行以下代码

```bash
ansible-playbook –t install,conf httpd.yml
```

就是执行了里面的 `install` 和 `conf` 模块
