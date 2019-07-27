---
title: golang 环境搭建
tags:
  - golang
categories:
  - golang
date: 2019-07-27 11:20:33
---

> Golang官网下载地址：https://golang.org/dl/
## Linux 下安装 Go 环境

1. 打开官网下载地址选择对应的系统版本, 复制下载链接，这里我选择的是：`go1.12.7.linux-amd64.tar.gz`，对应下载链接：https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz

2. `cd` 进入你用来存放安装包的目录。当然你也可以直接输入cd ~，然后执行：
``` bash
$ wget https://dl.google.com/go/go1.12.7.linux-amd64.tar.gz
```

3. 下载完成，执行`tar`解压到`/usr/loacl`目录下，得到`go`文件夹：
``` bash
$ tar -C /usr/local -zxvf  go1.12.7.linux-amd64.tar.gz
```

4. 添加`/usr/loacl/go/bin`目录到`PATH`变量中，其中添加到`/etc/profile`或`$HOME/.profile`都可以：
``` bash
# 习惯用vim，没有的话可以用命令`sudo apt-get install vim`安装一个，
# 按`o(在光标所在行的下面插入新的一行。光标停在空行首，等待输入文本)`键进行编辑
$ vim /etc/profile
# 在最后一行添加
$ export GOROOT=/usr/local/go
$ export GOPATH=/mnt/go_workspaces
$ export GOBIN=$GOPATH/bin
$ export PATH=$PATH:$GOBIN:$GOROOT/bin
# 按`ESC`键跳到命令模式，然后`:wq`保存退出后`source`一下:
$ source /etc/profile
```

5. 执行`go version`，如果显示版本号，则Go环境安装成功。
```bash
$ go version
```

## Mac 下安装 Go 环境
Mac分为压缩版和安装版，他们都是64位的。压缩版和Linux的大同小异，因为Mac和Linux都是基于Unix，终端这一块基本上是相同的。
压缩版解压后，就可以和Linux一样放到一个目录下，这里也以`/usr/local/go/`为例。在配置环境变量的时候，针对所有用户和Linux是一样的，都是`/etc/profile`这个文件；针对当前用户，Mac下是`$HOME/.bash_profile`，其他配置都一样，包括编辑sudo权限和生效方式，最后在终端里测试：
```bash
$ go version
```
