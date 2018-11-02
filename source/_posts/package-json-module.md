---
title: package.json 文件中的 module 字段
date: 2018-08-08 23:47:07
tags:
  - package.json
categories:
  - nodejs
---

通常基于 NPM 托管的项目都会有一个 package.json 文件，它是项目的描述文件，它的内容是一个标准的 JSON 对象。相信大家对 package.json 常用配置肯定熟悉的不能再熟悉了，例如项目名称（name）、项目版本号（version）、项目描述 （description）、npm 命令（scripts）等等，而我们今天主要聊聊`pkg.module`字段的功能以及使用场景。

### pck.module

之前查阅了package.json的[文档](https://docs.npmjs.com/files/package.json)，并没有找到我们想要的 module 字段的定义，无意中看了一个帖子才知道它是 rollup 中最早就提出的概念 --- [pkg.module](https://github.com/rollup/rollup/wiki/pkg.module)。在这之前 npm 包大都是基于 CommonJS 规范的。当我们当 require 引入包的时候，就会根据 main 字段去查找入口文件。

而在 ES6 规范出现后，ES6 定义了一套基于 import、export 操作符的模块规范。它与 CommonJS 规范最大的区别在于 ES6 中的 import 和 export 都是静态的。静态意味着一个模块要暴露或引入的所有方法在编译阶段就能全部确定，之后不能再改变。这样做的好处就是打包工具在打包阶段就可以分析出代码中用到了某个模块中的哪几个方法。其它没有用到的方法就可以从最终的 bundle 文件中剔除掉。这样既可以减少 bundle 文件的大小，又可以提高脚本的执行速度。这个机制被称为 Tree Shaking。在这个构建思想的基础上，开发基于 ES Module 规范的包是很有必要的。

之前我们说过 CommonJS 规范的包都是以 main 字段表示入口文件了，如果 ES Module 的也用 main 字段，就会对使用者造成困扰，如果他的项目不支持打包构建，比如大多数 node 项目(尽管 node9+ 支持 ES Module)，这时库开发者的模块系统跟项目构建的模块系统的冲突，更像是一种规范上的问题。况且目前大部分仍是采用 CommonJS，所以 rollup 便使用了另一个字段：module。如下配置：
``` json
{
    "name": "mypck",
    "version": "1.0.0",
    "main": "dist/index.cjs.js",
    "module": "dist/index.esm.js"
}
```

webpack 从版本 2 开始也可以识别 pkg.module 字段。打包时，如果存在 module 字段，会优先使用，如果没找到对应的文件，则使用 main 字段，并按照 CommonJS 规范打包。所以目前主流的打包工具（webpack, rollup）都是支持 pkg.module 的，鉴于其优点，module 字段很有可能加入 package.json 的规范之中。另外，越来越多的 npm 包已经同时支持两种模块，使用者可以根据情况自行选择，并且实现也比较简单，只是模块导出的方式。

注意：虽然打包工具支持了 ES Module，但是并不意味着其他的 es6 代码可以正常使用，因为使用者并不会对我们的 npm 包做编译处理，比如 webpack rules 中 exclude: /node_modules/，所以如果不是事先约定好后编译或者没有兼容性的需求，我们仍需要用 babel 处理，从而产出兼容性更好的 npm 包。
