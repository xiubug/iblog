---
title: Linux下登录MySQL
tags:
  - login
categories:
  - mysql
date: 2018-05-22 22:01:27
---

格式：mysql -h主机地址 -u用户名－p用户密码

### 一、连接到本机上的MYSQL：
* 1、开启MySQL服务后，一般可以直接键入命令mysql -uroot -p，回车后提示你输密码：
```js
mysql -uroot -p
```
### 二、连接到远程主机上的MySQL：
* 1、假设远程主机的IP为：10.0.0.1，用户名为root,密码为123。则键入以下命令（注：u与root可以不用加空格，其它也一样）：

```js
mysql -h10.0.0.1 -uroot -p123
```
