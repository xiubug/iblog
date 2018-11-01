---
title: webpack 开发汇总
date: 2018-05-08 22:11:50
tags:
    - webpack
categories:
    - webpack笔记
---

## 常见用法

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

### Node 环境变量 process.env.NODE_ENV 之webpack应用
文档中说：
> DefinePlugin 在原始的源码中执行查找和替换操作，在导入的代码中，任何出现 process.env.NODE_ENV的地方都会被替换为”production”。因此，形如if (process.env.NODE_ENV !== 'production') console.log('……') 的代码就会等价于 if (false) console.log('……') 并且最终通过UglifyJS等价替换掉。
也就是说，webpack config文件中定义的变量：
``` js
new webpack.DefinePlugin({
  'process.env.NODE_ENV': JSON.stringify('production')
})
```
是为了我们将要打包的文件中用的。那如何在webpack config文件中使用 process.env.NODE_ENV 呢？答案是corss-env。

#### 接下来进入主题，开始配置 webpack.config.js:
``` js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

const config = {
    entry:path.join(__dirname,"src/app.js"),
    output:{
        path:path.join(__dirname,"dist"),
        filename:"bundle.js"
    },
    plugins:[
        new HtmlWebpackPlugin()
    ]

}
module.exports = config
console.log("process.env.NODE_ENV 的值是(webpack.config.js)："+ process.env.NODE_ENV)
```
然后新建文件 src/app.js：
``` js
console.log("app test")

console.log("process.env.NODE_ENV 的值是(app.js)："+ process.env.NODE_ENV)
```
一切准备好后，给package.json加一行 ："build": "webpack"
``` json
"scripts": {
  "build":"webpack"
}
```
由于没有进行任何配置，所以上面的输出中给出的信息是：
``` js
process.env.NODE_ENV 的值是(webpack.config.js)：undefined
```
通过浏览器访问/dist/index.html，控制台有如下信息输出：
```js
app test 
process.env.NODE_ENV 的值是(app.js)：undefined
```
也就是说，在/src/app.js里，process.env.NODE_ENV 也未定义。

#### 通过webpack -p参数控制
在package.json里增加一行：
``` json
"scripts": {
  "build":"webpack",
  "build-p":"webpack -p"
}
```

然后执行： npm run build-p 命令行输出没有任何变化，仍然是：
``` js
process.env.NODE_ENV 的值是(webpack.config.js)：undefined
```
但通过浏览器访问/dist/index.html，控制台有如下信息输出：
``` js
app test 
process.env.NODE_ENV 的值是(app.js)：production
```

也就是说，通过webpack -p，然process.env.NODE_ENV值传递给app.js了（webpack.config.js并未获取到~）

#### 通过 webpack.DefinePlugin 定义
接着看，假设webpack.config.js是基本定义，针对上线产品，额外定义了webpack.config.prod.js，然后通过webpack-merge合并两个配置文件webpack.config.prod.js如下：
``` js
const webpack = require('webpack')
const merge = require('webpack-merge')

const config = require("./webpack.config.js")

module.exports = merge(config,{
    plugins:[
        new webpack.DefinePlugin({
            'process.env.NODE_ENV': JSON.stringify('production')
        }),
        new webpack.optimize.UglifyJsPlugin({
            compress: process.env.NODE_ENV === 'production'
        })
    ]
})
console.log("process.env.NODE_ENV 的值是(webpack.config.prod.js)："+ process.env.NODE_ENV)
```
可以看到，在这个文件里，我们通过webpack.DefinePlugin定义了process.env.NODE_ENV，现在在package.json里增加一行，并通过–config指定配置文件：
``` json
"scripts": {
  "build":"webpack",
  "build-p":"webpack -p",
  "build-prod":"webpack --config webpack.config.prod.js"
},
```
然后执行：npm run build-prod，命令行里有如下输出：
``` js
process.env.NODE_ENV 的值是(webpack.config.js)：undefined 
process.env.NODE_ENV 的值是(webpack.config.prod.js)：undefined
```
通过浏览器访问/dist/index.html，控制台有如下信息输出：
``` js
app test 
process.env.NODE_ENV 的值是(app.js)：production
```
这次没有用webpack -p参数，而是在webpack.config.prod.js里通过webpack.DefinePlugin定义了process.env.NODE_ENV，取得了相当的效果。

#### 在config文件里获取NODE_ENV值

解决了在app.js获取NODE_ENV的值，如何在webpack配置文件里获取NODE_ENV的值呢，这样就可以根据不同的值定义相关的参数了，如上所述，答案是：corss-env，在package.json里增加一行：
``` json
"scripts": {
  "build":"webpack",
  "build-p":"webpack -p",
  "build-prod":"webpack --config webpack.config.prod.js",
  "build-cross-env":"cross-env NODE_ENV=production webpack"
}
```
这里执行：npm run build-cross-env，命令行里会得到：
``` js
process.env.NODE_ENV 的值是(webpack.config.js)：production
```
通过浏览器访问/dist/index.html，控制台有如下信息输出：
``` js
app test 
process.env.NODE_ENV 的值是(app.js)：undefined
```
可以看到，通过cross-env NODE_ENV=production，然信息传递给了webpack的配置文件，但app.js并没有获取到。

#### 很自然的想到，如果里要在配置文件里和业务代码里，都获取到NODE_ENV，那将3、4结合起来：
``` json
"scripts": {
  "build":"webpack",
  "build-p":"webpack -p",
  "build-prod":"webpack --config webpack.config.prod.js",
  "build-cross-env":"cross-env NODE_ENV=production webpack",
  "build-cross-env-with-prod":"cross-env NODE_ENV=production webpack  --config webpack.config.prod.js"
}
```
运行： npm run build-cross-env-with-prod，命令行中有显示：
``` js
process.env.NODE_ENV 的值是(webpack.config.js)：production 
process.env.NODE_ENV 的值是(webpack.config.prod.js)：production
```
通过浏览器访问/dist/index.html，控制台有如下信息输出：
``` js
app test 
process.env.NODE_ENV 的值是(app.js)：production
```

## 常见问题

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

### loader 加载顺序
loader 的加载顺序是从右往左的。为啥是从右往左，而不从左往右，那是因为Webpack选择了compose方式，而不是pipe的方式而已。
``` js
compose : require("style-loader!css-loader!sass-loader!./my-styles.sass");

pipe : require("./my-styles.sass!sass-loader!css-loader!style-loader");
```
