---
title: webpack4.0 升级指南
date: 2018-08-31 22:11:50
tags:
    - webpack
categories:
    - webpack
---

## 问题指南

### webpack-cli

**Q：**由于webpack-cli从webpack包里面分离出来了，未安装webpack-cli会产生以下错误：
```
One CLI for webpack must be installed. These are recommended choices, delivered as separate packages:
 - webpack-cli (https://github.com/webpack/webpack-cli)
   The original webpack full-featured CLI.
 - webpack-command (https://github.com/webpack-contrib/webpack-command)
   A lightweight, opinionated webpack CLI.
We will use "npm" to install the CLI via "npm install -D".
Which one do you like to install (webpack-cli/webpack-command):
```
**A：**
``` bash
$ npm install webpack-cli -D
```

### mode
**Q：**未设置mode属性
```
WARNING in configuration
The 'mode' option has not been set, webpack will fallback to 'production' for this value. Set 'mode' option to 'development' or 'production' to enable defaults for each environment.
You can also set it to 'none' to disable any default behavior. Learn more: https://webpack.js.org/concepts/mode/
```
**A：**
development：开发模式，webpack会默认配置常用于开发的参数，如输出运行时的错误信息等
production：产品模式，webpack会默认配置常用语产品构建的餐宿，如压缩代码等
``` js
// 配置文件
module.exports = {
    mode: 'development'
    // mode: 'production'
}
// 命令行
webpack --mode development
webpack --mode production
```
