---
title: Mac OS系统使用指南
date: 2018-08-11 07:25:20
tags:
  - note
categories:
  - Mac OS
---

Mac OS是一套运行于苹果Macintosh系列电脑上的操作系统。Mac OS是首个在商用领域成功的图形用户界面操作系统。现行的最新的系统版本是OS X 10.12 ，且网上也有在PC上运行的Mac系统，简称 Mac PC。
Mac系统是基于Unix内核的图形化操作系统；一般情况下在普通PC上无法安装的操作系统。由苹果公司自行开发。苹果机的操作系统已经到了OS 10，代号为MAC OS X(X为10的罗马数字写法），这是MAC电脑诞生15年来最大的变化。新系统非常可靠；它的许多特点和服务都体现了苹果公司的理念。
另外，疯狂肆虐的电脑病毒几乎都是针对Windows的，由于MAC的架构与Windows不同，所以很少受到病毒的袭击。MAC OSX操作系统界面非常独特，突出了形象的图标和人机对话。苹果公司不仅自己开发系统，也涉及到硬件的开发。
接下来，我们就总结下Mac OS常见的使用指南。

### Mac电脑使用：您的安全性偏好设置仅允许安装来自App Store和被认可的开发者的应用（解决方法）

**1. 打开dock栏里面的“系统偏好设置”；**

![img1.png](/images/macos-tutorial/img1.png)

**2. 在系统偏好设置里面，找到“安全性与隐私”选项；**

![img2.png](/images/macos-tutorial/img2.png)

**3. 在安全性与隐私里面，找到左下角的锁型图标，然后点击锁，会弹出输入电脑开机密码的窗口，输入密码之后，点击“解锁”按钮，那个锁型变为开启的锁；**

![img3.png](/images/macos-tutorial/img3.png)

![img4.png](/images/macos-tutorial/img4.png)

**4. 解锁后，如果你的电脑里面在允许从以下位置下载应用有三个选项，就在允许从以下位置下载的应用选项中选择“任何来源”，在弹出的确认框里点击“允许来自任何来源”；如果你的电脑里面允许从以下位置下载的应用中只有两个选项，那你就直接去打开你刚才需要安装的那个程序的安装包，双击重新安装，会有一个提示框，也是提示允许来自任何来源安装的，然后就可以安装成功了，最后打开即可。**

![img5.png](/images/macos-tutorial/img5.png)

### 用终端连接远程服务器
``` bash
# 登录
$ ssh -t root@121.199.61.169 -p 22

# 退出
$ control + d
```
### 隐藏/显示 隐藏文件

``` bash
# 凡是前面带有小点的隐藏文件，或者是显示淡蓝色的文件都是隐藏文件
$ shift + cmmand + .
```

### mac os 获取root/su/sudo权限的方法

方法一：  打开终端， 输入sudo -i ,  按照提示输入当前管理员用户密码，即可进入root权限

### mac os 从root账户切换到普通账户

方法一：  打开终端， 输入su - test ,  即可进入test权限

### 如何解除mac上不能安装不明开发者的软件

打开 「Launchpad」 - 「实用工具」 - 「终端」；你在终端里输入

```bash
$ sudo spctl --master-disable
```

输入password（开机密码），输完敲回车；“任何来源”就会显示了，然后勾选你要的选项；

### mysql登录退出命令

登录Mysql：“输入mysql -u帐号 -p密码 这是登陆

mysql退出：mysql > exit;

以下是实例参考下：

登录Mysql：“输入mysql -uroot -p -P3306 -h127.0.0.1”

表示超级用户名root,密码稍后输入，端口号3306（不输入P默认为3306），

主机地址127.0.0.1（若使用本机作为主机，h默认127.0.0.1）

mysql退出三种方法：

mysql > exit;

mysql > quit;

mysql > \q;

### mysql连接错误如下：ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

1. 确保环境没有mysql
``` bash
$ brew remove mysql

$ brew cleanup
```

2. 安装 
``` bash
$ brew install mysql
```

3. 启动
```bash
$ brew services start mysql
```
设置开机启动: launchctl load ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist (设置启动的命令可以通过 brew info mysql获得)

4.登录
``` bash
$ mysql -uroot
```






