---
title: Shell 开发汇总
tags:
  - note
categories:
  - shell
date: 2018-10-31 21:09:06
---

## Shell 教程

Shell 是一个用 C 语言编写的程序，它是用户使用 Linux 的桥梁。Shell 既是一种命令语言，又是一种程序设计语言。Shell 是指一种应用程序，这个应用程序提供了一个界面，用户通过这个界面访问操作系统内核的服务。

Ken Thompson 的 sh 是第一种 Unix Shell，Windows Explorer 是一个典型的图形界面 Shell。

### Shell 脚本

Shell 脚本（shell script），是一种为 shell 编写的脚本程序。业界所说的 shell 通常都是指 shell 脚本，但读者朋友要知道，shell 和 shell script 是两个不同的概念。

### Shell 环境
Shell 编程跟 java、php 编程一样，只要有一个能编写代码的文本编辑器和一个能解释执行的脚本解释器就可以了。Linux 的 Shell 种类众多，常见的有：

## 常见问题

### bash: /usr/local/bin: Permission denied
原因：数据权限问题

解决办法：
``` bash
$ sudo chmod 755 /usr/local/bin

$ sudo chmod -R 755 /usr/local/bin # 递归下面
```
