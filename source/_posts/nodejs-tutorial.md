---
title: nodejs 开发汇总
tags:
  - nodejs
categories:
  - nodejs
date: 2018-04-28 22:10:50
---
## 前言

### 什么是NodeJS？
JavaScript是一门脚本语言，它需要一个运行环境。就好像PHP需要Apache，Java需要Tomcat等等，而NodeJS之前，JavaScript运行环境是浏览器，也就是JavaScript在网页中才能跑起来。NodeJS之后JavaScript又多了一个运行环境，就是NodeJS。NodeJS 是基于Chrome V8引擎的 JavaScript 运行环境。NodeJS使用事件驱动，非阻塞型I/O。NodeJS的包管理生态是 NPM，是现在世界上最大的开源程序包库。
由于NodeJS的最底层是Chrome的V8引擎，然后libuv封装了`一些I/O的线程池管理和网络的I/O操作`，这部分是C/C++写的。简单来说NodeJS可以控制系统文件的读写，网络的输入输出，所以NodeJS又可以被单纯的认为是一个可以运行 JavaScript 的服务器。

### NodeJS和VueJS，ReactJS，还有AngularJS的区别
这些工具的起源和诞生几乎浓缩了前端的发展历程，因为 NodeJS 可以读写文件，监听网络输入输出，所以 NodeJS 和 VueJS、ReactJS、AngularJS 有非常本质的区别。NodeJS 是可以运行 JavaScript 的环境，剩下三个是用 JavaScript 写的库。

### 一般NodeJS被用在哪里？

至于纯前端，大多用于辅助开发。我们所用的vue-cli等脚手架，或者webpack，gulp，grunt等构建工具都是用node写得。它为我们提升效率、打包、构建、维护等。

### NodeJS 还能做啥？
用JS做服务器： Express / EggJS / HAPI / Koa 等。

用JS做移动端混合应用：PhoneGap / Cordova / Ionic 等。

用JS做移动端原生应用：React-Native / NativeScript / WEEX 等。

## 常见用法
### Node.js —— 实现修改完代码自动重启
#### nodemon
使用第三方命令行工具解决频繁修改代码重启服务器问题：nodemon。nodemon是一个基于Node.js开发的第三方命令行工具。

**全局安装nodemon：**
``` bash
$ npm install -g nodemon
```

**使用：**
把输入cmd的node改为nodemon，比如node app.js改为ndoemon app.js。只要是通过nodemon启动的服务，它会监视文件变化，当文件发生变化时会自动重启服务器。

### module.filename、__filename、__dirname、process.cwd()和require.main.filename 解惑
**module.filename：**开发期间，该行代码所在的文件。
**__filename：**始终等于 module.filename。
**__dirname：**开发期间，该行代码所在的目录。
**process.cwd()：**运行node的工作目录，可以使用  cd /d 修改工作目录。
**require.main.filename：**用node命令启动的module的filename, 如 node xxx，这里的filename就是这个xxx。

require()方法的坐标路径是：module.filename；fs.readFile()的坐标路径是：process.cwd()。

## 常见问题

### bash: /usr/local/bin: Permission denied
原因：数据权限问题

解决办法：
``` bash
$ sudo chmod 755 /usr/local/bin

$ sudo chmod -R 755 /usr/local/bin # 递归下面
```

### Node.js 连接MySQL时 出现 connect ECONNREFUSED 127.0.0.1:3306

原因：数据库服务没有打开。

解决办法：开启 MySQL服务。

### nodemon 不断重启

解决办法：以管理员身份打开命令行执行命令，如果没有帮助，重启电脑。
