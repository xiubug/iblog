---
title: 深聊import、require、export、module.exports
date: 2018-07-05 20:55:39
tags:
    - require
    - import
categories:
    - es6
    - nodejs
---

## 前言

ES6标准发布后，module成为标准，标准的使用是以export指令导出接口，以import引入模块，但是在我们一贯的node模块中，我们采用的是CommonJS规范，使用require引入模块，使用module.exports导出接口，甚至有时候也会常常看到两者互用的场景。只有把这些语法搞清楚才能在未来的标准编程游刃有余。

## webpack 模块化

webpack 本身维护了一套模块系统，这套模块系统兼容了所有前端历史进程下的模块规范，包括 amd commonjs es6 等，本文主要针对 commonjs es6 规范进行说明。模块化的实现其实就在最后编译的文件内。

### 我们以一个小 demo 为例

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

### demo 打包后文件

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

## babel 的作用

按理说 webpack 的模块化方案已经很好的将 es6 模块化转换成 webpack 的模块化，但是其余的 es6 语法还需要做兼容性处理。babel 专门用于处理 es6 转换 es5。当然这也包括 es6 的模块语法的转换。
其实两者的转换思路差不多，区别在于 webpack 的原生转换 可以多做一步静态分析，使用tree-shaking 技术。
>babel 能提前将 es6 的 import 等模块关键字转换成 commonjs 的规范。这样 webpack 就无需再做处理，直接使用 webpack 运行时定义的 __webpack_require__ 处理。

### 导出模块

es6 的导出模块写法：
``` js
export default 123;

export const a = 123;

const b = 3;
const c = 4;
export { b, c };
```
babel 会将这些统统转换成 commonjs 的 exports：
``` js
exports.default = 123;
exports.a = 123;
exports.b = 3;
exports.c = 4;
exports.__esModule = true;
```
babel 转换 es6 的模块输出逻辑非常简单，即将所有输出都赋值给 exports，并带上一个标志 __esModule 表明这是个由 es6 转换来的 commonjs 输出。

babel将模块的导出转换为commonjs规范后，也会将引入 import 也转换为 commonjs 规范。即采用 require 去引用模块，再加以一定的处理，符合es6的使用意图。

### 引入 default
```js
import a from './a.js';
```
在es6中 import a from './a.js' 的本意是想去引入一个 es6 模块中的 default 输出。

通过babel转换后得到`var a = require(./a.js)`，require得到的对象却是整个对象，肯定不是 es6 语句的本意，所以需要对 a 做些改变。

我们在导出提到，default 输出会赋值给导出对象的default属性。

``` js
exports.default = 123;
```

所以 babel 加了个`help _interopRequireDefault`函数：
``` js
function _interopRequireDefault(obj) {
    return obj && obj.__esModule
        ? obj
        : { 'default': obj };
}

var _a = require('assert');
var _a2 = _interopRequireDefault(_a);

var a = _a2['default'];
```
所以这里最后的 a 变量就是 require 的值的 default 属性。如果原先就是commonjs规范的模块，那么就是那个模块的导出对象。

### 引入 * 通配符
我们使用`import * as a from './a.js'`es6语法的本意是想将 es6 模块的所有命名输出以及defalut输出打包成一个对象赋值给a变量。

已知以 commonjs 规范导出：
``` js
exports.default = 123;
exports.a = 123;
exports.b = 3;
exports.__esModule = true;
```
那么对于 es6 转换来的输出通过 var a = require('./a.js') 导入这个对象就已经符合意图。

所以直接返回这个对象：
``` js
if (obj && obj.__esModule) {
   return obj;
}
```
如果本来就是 commonjs 规范的模块，导出时没有default属性，需要添加一个default属性，并把整个模块对象再次赋值给default属性：
``` js
function _interopRequireWildcard(obj) {
    if (obj && obj.__esModule) {
        return obj;
    }
    else {
        var newObj = {}; // (A)
        if (obj != null) {
            for (var key in obj) {
                if (Object.prototype.hasOwnProperty.call(obj, key))
                    newObj[key] = obj[key];
            }
        }
        newObj.default = obj;
        return newObj;
    }
}

```

### import { a } from './a.js'
直接转换成 require('./a.js').a 即可。

### 小结
经过上面的转换分析，我们得知即使我们使用了 es6 的模块系统，如果借助 babel 的转换，es6 的模块系统最终还是会转换成 commonjs 的规范。所以我们如果是使用 babel 转换 es6 模块，混合使用 es6 的模块和 commonjs 的规范是没有问题的，因为最终都会转换成 commonjs。

## babel5 & babel6

我们在上文 babel 对导出模块的转换提到，es6 的 export default 都会被转换成 exports.default，即使这个模块只有这一个输出。

我们经常会使用 es6 的 export default 来输出模块，而且这个输出是这个模块的唯一输出，我们会误以为这种写法输出的是模块的默认输出。
``` js
// a.js
export default 123;
```
``` js
// b.js 错误

var foo = require('./a.js')
```
在使用`require`进行引用时，我们也会误以为引入的是a文件的默认输出。结果这里需要改成`var foo = require('./a.js').default`。

这个场景在写 webpack 代码分割逻辑时经常会遇到。
``` js
require.ensure([], (require) => {
   callback(null, [
     require('./src/pages/profitList').default,
   ]);
});
```
这是 babel6 的变更，在 babel5 的时候可不是这样的。

![img1.jpeg](module-import-require/img1.jpeg)

在 babel5 时代，大部分人在用 require 去引用 es6 输出的 default，只是把 default 输出看作是一个模块的默认输出，所以 babel5 对这个逻辑做了 hack，如果一个 es6 模块只有一个 default 输出，那么在转换成 commonjs 的时候也一起赋值给 module.exports，即整个导出对象被赋值了 default 所对应的值。

这样就不需要加 default，require('./a.js') 的值就是想要的 default值。但这样做是不符合 es6 的定义的，在es6 的定义里，default 只是个名字，没有任何意义。

``` js
export default = 123;
export const a = 123;
```
这两者含义是一样的，分别为输出名为 default 和 a 的变量。

还有一个很重要的问题，一旦 a.js 文件里又添加了一个具名的输出，那么引入方就会出麻烦。

``` js
// a.js
export default 123;

export const a = 123; // 新增

// b.js 
var foo = require('./a.js');

// 由之前的 输出 123
// 变成 { default: 123, a: 123 }
```
所以 babel6 去掉了这个hack，这是个正确的决定，升级 babel6 后产生的不兼容问题 可以通过引入[babel-plugin-add-module-exports](https://www.npmjs.com/package/babel-plugin-add-module-exports)解决。

