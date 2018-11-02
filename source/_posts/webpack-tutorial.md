---
title: webpack 开发汇总
date: 2018-05-08 22:11:50
tags:
  - note
categories:
  - webpack
---

## webpack 教程

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

### Webpack Code Splitting 

首先说，code splitting指什么。我们打包时通常会生成一个大的bundle.js(或者index,看你如何命名)文件，这样所有的模块都会打包到这个bundle.js文件中，最终生成的文件往往比较大。code splitting就是指将文件分割为块(chunk)，webpack使我们可以定义一些分割点(split point)，根据这些分割点对文件进行分块，并实现按需加载。

#### code splitting的意义

1、第三方类库单独打包。由于第三方类库的内容基本不会改变，可以将其与业务代码分离出来，这样就可以将类库代码缓存在客户端，减少请求。
2、按需加载。webpack支持定义分割点，通过require.ensure进行按需加载。
3、通用模块单独打包。我们代码中可能会有一些通用模块，比如弹窗、分页、通用的方法等等。其他业务代码模块常常会有引用这些通用模块。若按照2中做，则会造成通用模块重复打包。这时可以将通用模块单独打包出来。

#### 如何进行code spliting

##### 第三方类库

我们项目中常常会用到一些第三方的类库，比如jquery,bootstrap等。可以配置多入口来将第三方类库单独打包，如下：

```javascript
//在entry中添加入口
entry: {
  index: './index',
  vendor: ['jquery', 'bootstrap']
},

//在plugins中配置
plugins: [
  new webpack.optimize.CommonsChunkPlugin("vendor", "vendor.bundle.js"),
]
```

**说明 **
CommonsChunkPlugin提供两个参数，第一个参数为对应的chunk名（chunk指文件块，对应entry中的属性名），第二个参数为生成的文件名。
这个插件做了两件事：
1、将vendor配置的模块（jquery,bootstrap）打包到vendor.bundle.js中。
2、将index中存在的jquery, bootstrap模块从文件中移除。这样index中则只留下纯净的业务代码。

##### 按需加载

以基于backbone的单页面应用为例，可以在router中进行配置实现按需加载，如下：

```javascript

// router.js

var Router = Backbone.Router.extend({
  routes: {
    'a': 'a',
    'b': 'b'
  },
  
  a: function() {
    require.ensure(['./a'], (require) => {
      let a = require('./a');
      //do something
    })
  },
  
  b: function() {
    require.ensure(['./b'], (require) => {
      let b = require('./b');
      //do something
    })
  }
})
```

**说明**
如上方式将打出两个文件，a.js和b.js（当然名字会有所不同），且为按需加载。只有在访问a时，a.js才会被加载，b同理。但是这种做法存在两个问题：
1、若路由分配不合理，会打包出很多很小的文件，每个文件或许只有几k，却多了很多网络请求，得不偿失。
2、会造成通用模块的重复打包，比如a模块和b模块都引用了c模块，

```javascript
// a
import 'c' from './c'

// b
import 'c' from './c'
```

这样我们会发现打包出的a.js和b.js中都包含c模块的代码，造成了代码冗余。

对于问题1，可以通过webpack提供的插件来解决：

```javascript
//在plugins中添加该插件：
plugins: [
  new webpack.optimize.AggressiveMergingPlugin()
]
```

对于问题2:
可以按照下文中所说方式解决。

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
