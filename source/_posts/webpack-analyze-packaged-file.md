---
title: webpack 打包后文件分析
date: 2018-06-12 07:19:45
tags:
    - webpack
categories:
    - webpack
---

`版本：v4.17.1`

## 前言

Webpack 是目前使用最为火热的打包工具，没有之一，每月有数百万的下载量，为成千上万的网站和应用提供支持。我们网页拥有着复杂的JavaScript代码和一大堆依赖包，webpack 把我们的项目当做一个整体，通过一个给定的入口文件（如：index.js），Webpack将从这个文件开始分析我们的项目的所有依赖文件，然后经过一系列处理，最后打包为一个（或多个）浏览器可识别的JavaScript文件。可我们是否想过 Webpack 输出的 bundle.js 是什么样子的吗？ 为什么原来一个个的模块文件被合并成了一个单独的文件？为什么输出文件（如：bundle.js）能直接运行在浏览器中？现在我们就来详细的分析 webpack 打包后文件。

## 打包源码

``` js
// webpack.config.js
const path = require('path');

module.exports = {
    devtool: "source-map",
    mode: 'development',
    entry: './main.js',
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'bundle.js',
    }
}

// main.js
import helloWorld from './src/index1';
import worldHello from './src/index2';

console.log(helloWorld, worldHello);
export default './main.js';

// index1.js
export default 'Hello World';

// index2.js
export default 'World Hello';
```

## 分析打包后文件

``` js
// webpackBootstrap 启动函数
// modules 即为存放所有模块的对象，对象中的每一个属性都是一个函数
(function(modules) {
    // 安装过的模块都存放在这里面，作用是把已经加载过的模块缓存在内存中，提升性能
    var installedModules = {};
    // 加载参数对象中每一个模块，moduleId 为要加载模块对象的 key
    // 函数作用和 Node.js 中 require 语句相似
    function __webpack_require__(moduleId) {
        // 如果需要加载的模块已经被加载过，就直接从内存缓存中返回
        if(installedModules[moduleId]) {
            return installedModules[moduleId].exports;
        }

        // 如果缓存中不存在需要加载的模块，就新建一个模块，并把它存在缓存中
        // webpack1的时候都是全称，现在估计为了省点空间，都变成了id => i, load => l
        var module = installedModules[moduleId] = {
            // 加载模块对象的 key
            i: moduleId,
            // 该模块是否已经加载完毕
            l: false,
            // 该模块的导出值
            exports: {}
        };

        // 从 modules 中获取 key 为 moduleId 的模块对应的函数
        // 再调用这个函数，同时把函数需要的参数传入
        modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

        // 把这个模块标记为已加载
        module.l = true;

        // 返回这个模块的导出值
        return module.exports;
    }

    // 在源文件中，直接使用__webpack_modules__，生成文件用__webpack_require__.m替换
    __webpack_require__.m = modules;

    // 暴露module缓存
    __webpack_require__.c = installedModules;

    // 为harmory exports 定义 getter function, configurable=false表明，此属性不能修改
    // 例如export const，由于是常量，需要用__webpack_require__.d进行定义
    __webpack_require__.d = function(exports, name, getter) {
        if(!__webpack_require__.o(exports, name)) {
            Object.defineProperty(exports, name, { enumerable: true, get: getter });
        }
    };

    // define __esModule on exports
    __webpack_require__.r = function(exports) {
        if(typeof Symbol !== 'undefined' && Symbol.toStringTag) {
            Object.defineProperty(exports, Symbol.toStringTag, { value: 'Module' });
        }
        Object.defineProperty(exports, '__esModule', { value: true });
    };

    // create a fake namespace object
    // mode & 1: value is a module id, require it
    // mode & 2: merge all properties of value into the ns
    // mode & 4: return value when already ns object
    // mode & 8|1: behave like require
    __webpack_require__.t = function(value, mode) {
        if(mode & 1) value = __webpack_require__(value);
        if(mode & 8) return value;
        if((mode & 4) && typeof value === 'object' && value && value.__esModule) return value;
        var ns = Object.create(null);
        __webpack_require__.r(ns);
        Object.defineProperty(ns, 'default', { enumerable: true, value: value });
        if(mode & 2 && typeof value != 'string') for(var key in value) __webpack_require__.d(ns, key, function(key) { return value[key]; }.bind(null, key));
            return ns;
    };

    // 兼容 non-harmony 模块，这些模块如果设了__esModule属性，则被标记为non-harmony
    __webpack_require__.n = function(module) {
        var getter = module && module.__esModule ?
            function getDefault() { return module['default']; } :
            function getModuleExports() { return module; };
        __webpack_require__.d(getter, 'a', getter);
        return getter;
    };

    // Object.prototype.hasOwnProperty.call
    __webpack_require__.o = function(object, property) { return Object.prototype.hasOwnProperty.call(object, property); };

    // Webpack 配置中的 publicPath，用于加载被分割出去的异步代码
    __webpack_require__.p = "";

    // 使用 __webpack_require__ 去加载 key 为 ./main.js 的模块，并且返回该模块导出的内容
    // key 为 ./main.js 的模块就是 main.js 对应的文件，也就是执行入口模块
    // __webpack_require__.s 的含义是启动模块对应的 key
    return __webpack_require__(__webpack_require__.s = "./main.js");
})({
// 所有的模块都存放在了一个对象里，根据每个模块在对象的 key 来区分和定位模块
"./main.js": (function(module, __webpack_exports__, __webpack_require__) {
        "use strict";
        __webpack_require__.r(__webpack_exports__);
        var _src_index1__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/index1.js");
        var _src_index2__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__("./src/index2.js");
        console.log(_src_index1__WEBPACK_IMPORTED_MODULE_0__["default"], _src_index2__WEBPACK_IMPORTED_MODULE_1__["default"]);
        __webpack_exports__["default"] = ('./main.js');
    }),

"./src/index1.js": (function(module, __webpack_exports__, __webpack_require__) {
        "use strict";
        __webpack_require__.r(__webpack_exports__);
        __webpack_exports__["default"] = ('Hello World');
    }),

"./src/index2.js": (function(module, __webpack_exports__, __webpack_require__) {
        "use strict";
        __webpack_require__.r(__webpack_exports__);
        __webpack_exports__["default"] = ('World Hello');
    })
});
```
上面这段 js 就是使用 webpack 编译后的代码，其中就包含了 webpack 的运行时代码，其中就是关于模块的实现。

以上看上去复杂的代码其实是一个立即执行函数，可以简写为如下：
``` js
(function(modules) {

  // 模拟 require 语句
  function __webpack_require__() {
  }

  // 执行存放所有模块中 key 值为 ./main.js 的模块
  __webpack_require__('./main.js');

})({
    "./main.js": fn // 函数
})
```

自执行函数的入参是个对象，这个对象包含了所有的模块，包裹在函数中。
自执行函数体里的逻辑就是处理模块的逻辑。关键在于`__webpack_require__`函数，这个函数就是`require`或者是`import`的替代，我们可以看到在函数体内先定义了这个函数，然后调用了它。这里会传入一个`moduleId`，这个例子中是`./main.js`，也就是我们的入口模块`main.js`的内容。

我们再看`__webpack_require__`内执行了：
``` js
modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);
return module.exports；
```
即从入参的 modules 对象中取第一个函数进行调用，并入参：
* module
* module.exports
* webpack_require

我们再看第一个函数（即入口模块）的逻辑：

``` js
(function(module, __webpack_exports__, __webpack_require__) {
    "use strict";
    __webpack_require__.r(__webpack_exports__);
    var _src_index1__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__("./src/index1.js");
    var _src_index2__WEBPACK_IMPORTED_MODULE_1__ = __webpack_require__("./src/index2.js");
    console.log(_src_index1__WEBPACK_IMPORTED_MODULE_0__["default"], _src_index2__WEBPACK_IMPORTED_MODULE_1__["default"]);
    __webpack_exports__["default"] = ('./main.js');
})
```
我们可以看到入口模块又调用了`__webpack_require__("./src/index1.js")`和`__webpack_require__("./src/index2.js")`去引用入参数组里的第2和第3个函数。

然后会将入参的`__webpack_exports__`对象添加`default`属性，并赋值。这里的`__webpack_exports__`就是这个模块的`module.exports`通过对象的引用传参，间接的给`module.exports`添加属性，这就是webpack模块化的实现原理。

最后会将`module.exports return`出来。就完成了`__webpack_require__`函数的使命。比如在入口模块中又调用了`__webpack_require__("./src/index1.js")`，就会得到这个模块返回的`module.exports`。




