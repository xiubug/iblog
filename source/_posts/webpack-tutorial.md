---
title: webpack 使用指南
date: 2018-07-13 22:11:50
tags:
    - webpack
categories:
    - webpack
---

## 使用指南

### require.context
**require.context：**创建自己的（模块）上下文，这个方法有 3 个参数：要搜索的文件夹目录，是否还应该搜索它的子目录，以及一个匹配文件的正则表达式。 
``` js
require.context(directory, useSubdirectories = false, regExp = /^\.\//)
```
``` js
/**
* 创建一个
* 包含了 test 文件夹
* 不包含子目录下面的
* 所有文件名以 `.test.js` 结尾的、
* 能被 require 请求到的文件的上下文。
*/
require.context("./test", false, /\.test\.js$/);

/**
* 创建一个
* 包含了父级文件夹
* 包含子目录下面的
* 所有文件名以 `.stories.js` 结尾的
* 能被 require 请求到的文件的上下文。
*/
require.context("../", true, /\.stories\.js$/);
```

require.context模块导出（返回）一个（require）函数，这个函数可以接收一个参数：request 函数 – 这里的 request 应该是指在 require() 语句中的表达式。require.context 第一个参数不能是变量，webpack在编译阶段无法定位目录。导出的方法有 3 个属性： resolve, keys, id。
**resolve：**函数，它返回请求被解析后得到的模块 id。 
**keys：**函数，它返回一个数组，由所有可能被上下文模块处理的请求组成。 
**id：**上下文模块里面所包含的模块 id. 它可能在你使用 module.hot.accept 的时候被用到。

#### 示例一：引入多张图片

``` js
import img1 from '../../assets/img1.jpeg';
import img2 from '../../assets/img2.jpeg';
import img3 from '../../assets/img3.jpeg';
```
上面页面上需要的图片很多，这样做太麻烦了，有没有批量引入的办法呢？
``` js
const requireContext = require.context("../../assets", false, /^\.\/.*\.jpeg$/);
const images = requireContext.keys().map(requireContext);
```

#### 示例二：多个js文件
``` js
const context = require.context('./', false, /\.js$/);
const models = context.keys().filter(item => item !== './index.js').map(context);
```


## 问题指南

### webpack-cli（v4）

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

### mode（v4）
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
