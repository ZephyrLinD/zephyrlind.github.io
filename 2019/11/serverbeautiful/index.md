# 服务器到手后要做的一些事（一）———— 终端初步美化和一些必要工具


## 前言

双十一剁手啦！！！我的阿里云已经快要到期了，而且上面作死想整一些骚操作然后欢声笑语打出GG，删库跑路吧。以下是我拿到一台新的、属于自己的、以后会长期使用的小鸡（服务器）后我会干的事情，做一个总结，以防万一自己忘掉，顺便用以回顾。

平台：腾讯云

操作系统：CentOS 7.6

注意：里面的`example`皆为用户名的指代。

## 第一步，添加sudo权限的普通用户

由于要长期使用，没准也会在服务器上敲代码，为了防止自己操作失误，添加一个带sudo权限普通用户。

首先，添加用户：

```bash
useradd -d /home/example example
```

然后，为他修改密码：

```bash
visudo
```

再之后找到代码段，添加如下代码

```shell
# Allow root to run any commands anywhere
root     ALL=(ALL)       ALL
example  ALL=(ALL)       ALL  # 添加的代码
```

然后切换用户到`example`，运行一下update试试：

```bash
sudo yum update 
```

如果可以运行，说明一切正常，大功告成！

## 第二部，简单美化终端

因为毕竟是服务器，终端只需简洁、好用即可，不需要太过花哨。我们采用`zsh`+`oh-my-zsh`的组合来让终端变得有颜色，且有简单的提示。

首先下载`zsh`

```bash
sudo yum -y install zsh
```

然后下载`oh-my-zsh`，安装非常简单只需一行代码

```bash
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

根据提示完成安装后，首先修改主题，我最爱使用主题 `ys`：

```bash
vim ~/.zshrc
```

找到 `ZSH_THEME` 字段，将原本的 `robbyshell` 修改为 `ys` 即可。

在之后安装`zsh-autosuggestions`、`zsh-syntax-highlighting`，安装过程如下：

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

然后打开`.zshrc`，在`plugin`字段中加入两个插件：

```shell
plugins=([plugins...] zsh-syntax-highlighting zsh-autosuggestions)
```

然后就是一步到位，应用更改的时候啦！

```bash
source ~/.zshrc
```

之后我们的终端就变得相对好看啦！

## 安装装逼神器neofetch

`Neofetch`可以以更全面的方式来显示输出详实的 Linux 系统信息，顺便可以和小伙伴装逼一波。（主要是他也很好看啊嘤嘤嘤）

最早我一般在CentOS上通过编译安装它，但是现在在安装`neofetch`之前，是需要使用最新的Redhat系包管理软件`dnf`，然后就可以很方便的安装啦！

```bash
sudo yum install dnf
sudo yum install dnf-plugins-core
sudo dnf -y copr enable konimex/neofetch
sudo dnf install neofetch
```

之后就可以通过终端输入`neofetch`截图给小伙伴啦！

## 安装终端复用工具TMUX

TMUX（terminal multiplexer）是Linux上的终端复用神器，可从一个屏幕上管理多个终端（准确说是伪终端）。使用该工具，用户可以连接或断开会话，而保持终端在后台运行。类似的工具还有screen。

首先安装依赖：

```bash
sudo yum install byacc bison
```

之后，下载TMUX的源码

```bash
cd /usr/local/src
sudo git clone https://github.com/tmux/tmux.git
```

编译并安装：

```bash
cd tmux
sudo sh autogen.sh
sudo ./configure
sudo make
sudo make install
```

## 安装Python3

作为一个“脚本小子”（是的，不是低端黑客的蔑称，只是爱写脚本实现自动化），以后立志要做DevOPs的人，首先要用Python尝试把服务器自动化。

然而CentOS自带了Python2，想要使用Python3写脚本就需要编译安装Python3了。

首先，安装一些工具组和依赖

```bash
yum -y groupinstall "Development tools"
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
yum install libffi-devel -y
```

不由得想到了当初体验RHEL的时候连依赖都需要编译安装，没准还要找到依赖的依赖的恐怖日子了= =

接下来是下载源码包，我的习惯是将源码包放在`/usr/local/src`中

```bash
cd /usr/local/src/
sudo wget https://www.python.org/ftp/python/3.8.0/Python-3.8.0.tar.xz 
```

此时你可以喝杯咖啡，看一集番，如果还没完事可以看看淘宝，注意SSH连接别断了。

之后可以解压，要注意的是`tar.xz`的解压方法和`tar.gz`不太一样，需要先解压成`.tar`然后再进一步解压：

```bash
sudo xz -d Python-3.8.0.tar.xz
sudo tar xvf python-3.8.0.tar
sudo cd Python-3.8.0
```

最后就是编译老一套，只不过因为不在root用户上所以要加上sudo

```bash
mkdir /usr/local/python3
sudo ./configure --prefix=/usr/local/python3
sudo make 
sudo make install
```

然后检测一下是否成功：

```bash
python3 -v
pip3 -v
```

需要卸载的时候只需执行如下即可：

```bash
rpm -qa|grep python3|xargs rpm -ev --allmatches --nodeps
whereis python3 |xargs rm -frv
```

## 总结

拿到新服务器我也就只会干这些了，在之后还会进行vim的配置，由于太过冗长所以我会新开一篇文档。
