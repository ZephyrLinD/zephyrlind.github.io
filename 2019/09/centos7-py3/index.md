# Centos7安装Pyhon3


## 前言

CentOS 的第三方软件源中（如：IUS，SCL，EPEL）还没有 Python 3.7 的 RPM 软件包，本文介绍使用源码方式安装 Python 3.7。

## 安装依赖包

因为在CentOS7中源码安装Python 3.7之前需要安装GCC和make编译工具，该工具保安在“Development toos”软件组中，可以直接安装

```bash
yum groupinstall "Development tools"
```

再安装一些其他的依赖

```bash
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install libffi-devel -y
```

## 下载源代码包解压

```bash
cd /usr/local/src/
wget https://www.python.org/ftp/python/3.7.4/Python-3.7.4.tgz
cd Python-3.7.4
./configure --prefix=/usr/local/python3
make && make install
```

## 建立软连接

```bash
ln -s /usr/local/python3/bin/python3 /usr/local/bin/python3
ln -s /usr/local/python3/bin/pip3 /usr/local/bin/pip3
```

之后就可以在终端输入python3或者pip来玩耍啦！

