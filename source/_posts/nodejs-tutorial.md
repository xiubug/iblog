---
title: nodejs 开发汇总
tags:
  - nodejs
categories:
  - nodejs
date: 2018-04-28 22:10:50
---
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
module.filename：开发期间，该行代码所在的文件。
__filename：始终等于 module.filename。
__dirname：开发期间，该行代码所在的目录。
process.cwd()：运行node的工作目录，可以使用  cd /d 修改工作目录。
require.main.filename：用node命令启动的module的filename, 如 node xxx，这里的filename就是这个xxx。

require()方法的坐标路径是：module.filename；fs.readFile()的坐标路径是：process.cwd()。

## 常见问题

### Node.js 连接MySQL时 出现 connect ECONNREFUSED 127.0.0.1:3306

原因：数据库服务没有打开。

解决办法：开启 MySQL服务。

### nodemon 不断重启

解决办法：以管理员身份打开命令行执行命令，如果没有帮助，重启电脑。
