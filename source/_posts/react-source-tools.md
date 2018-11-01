---
title: React 背后的工具体系
date: 2018-08-30 20:10:11
tags:
    - react
categories:
    - 前端教程
uncr: true
---

北京时间2017年9月27日，Facebook 官方发布了 React v16.0。相较于之前的 v15.x 版本，v16 发生了很大的变化。

### React v16.0 API 变化

**1.render 函数支持返回数组和字符串：**我们终于不需要再将多个同级元素包裹在一个冗余的 DOM 元素中了，但每个同级元素还是需要唯一的 key 值以便 React 进行更新，而且在未来版本，React 可能还会提供一个特殊的 jsx 片段来支持无 key 值的 DOM 元素。

**2.更好的异常处理：**在之前版本的 React 中，某个组件在 Render 阶段的运行错误可能会 break 掉整个应用，而且抛出的异常信息含义也非常模糊，难以确定错误的发生位置。在 v16.0 中，如果某个组件在执行 render 或其他生命周期函数时出错，整个组件将被从根节点上移除掉，方便开发者快速定位异常组件。在定位到异常组件后，开发者可以为该组件添加 componentDidCatch 方法，并在这个方法中为组件定义一个备用视图用于渲染异常状态下的组件。当然，在这个新的生命周期函数中，开发者也可以获得更加有帮助的错误信息进行 debug。这被称作组件的错误边界，大家可以理解为组件层面的 try catch 声明。

**3.新的组件类型 portals：**ReactDOM.createPortal(child, container) 可以将子组件直接渲染到当前容器组件 DOM 结构之外的任意 DOM 节点中，这将使得开发对话框，浮层，提示信息等需要打破当前 DOM 结构的组件更为方便。

**4.更好的服务端渲染：**与之前 renderToString 方法不同，新版本提供的 renderToNodeStream 将返回 Readable，可以持续产生字节流（a stream of bytes）并在下一部分的 document 生成之前将之前已生成的部分 document 传回给客户端。通常来讲，新的服务端渲染将比老的快3倍以上。在 document 到达客户端之后，新版本的 react 也将不会再去将客户端的初次渲染结果与服务端的渲染结果进行比较，而是尽可能地去重用相同的 DOM 元素。

**5.支持自定义 DOM 元素：**新版本将不会再抛出不支持的 DOM 元素错误，而是将所有开发者自定义的 DOM 元素都传递到相应的 DOM 节点上。

**6.更小的打包大小：**总体体积减少 30%

react is 5.3 kb (2.2 kb gzipped), 老版本 20.7 kb (6.9 kb gzipped)
react-dom is 103.7 kb (32.6 kb gzipped), 老版本 141 kb (42.9 kb gzipped)
react + react-dom is 109 kb (34.8 kb gzipped), 老版本 161.7 kb (49.8 kb gzipped)

**7.MIT 许可：**除了最新的 16.0 版本外，Facebook 还发布了使用 MIT 许可的 15.6.2 版本，以方便无法立刻升级的使用者。

**8.新的核心架构 Fiber：**React v16.0 使用了 Fiber 作为底层架构。正是得益于 Fiber，返回数组和字符串及错误边界等功能才变得可能。Fiber 相较于之前最大的不同是它支持**异步渲染（async rendering），**这意味着 React 可以在更细的粒度上控制组件的绘制过程，从最终的用户体验来讲，用户可以体验到更流畅交互及动画体验。而因为异步渲染涉及到 React 的方方面面甚至未来，在 16.0 版本中 React 还暂时没有启用，并将在未来几个月陆续推出。

其实，以上种种变化都离不开背后构建工具的变化。

### React 构建工具

``` bash
# 开发工具
ES Module, Flow, ESLint, Prettier, Yarn workspace, HUBOT(GitHub Bot), [x]Haste, [x]CommonJS Module

# 构建工具
Rollup, Closure Compiler, Error Code System, React DevTools, [x]Gulp/Grunt+Browserify

# 测试工具
Jest, Prettier

# 发布工具
npm

```

以上前面带`[x]`的表示之前在用，React v16.0 已不再使用。简单说一下上面的工具都有什么作用？

开发时，按照 ES Module 机制编写源码，用 flow 进行类型检查、ESLint 语法规则和代码风格的检查、Prettier 来统一代码风格，借助 Yarn workspace 处理模块依赖，HUBOT(GitHub Bot) 检查PR；

开发过程中，用 Jest 单元测试，Prettier 来统一代码风格

开发完成，用 Rollup + Closure Compiler 构建，利用 Error Code System 机制实现生产环境错误追踪，React DevTools 侧面辅助 bundle 检查；

最后通过 npm 发布新 package。

### 开发工具

#### ES Module

React 16 之前的版本都用 CommonJS Module 定义，例如：
``` js
'use strict';

module.exports = require('./lib/React');
```
React 16 为什么选择使用 ES Module，有以下几个原因：

**1.编译期发现模块导入/导出问题：**我们都知道使用 CommonJS Module 的 require 一个未定义的方法时，不调用我们是发现不了错误的。而 ES Module 由于静态的模块机制，import 与 export 必须按名匹配，否则编译构建就会出错。 

**2.更小的打包大小：** 众所周知 module.exports 是对象级别导出，而ES Module 支持更细粒度的原子级导出，我们把这个特性叫做 tree-shaking，这个特性可以帮助你将无用代码（即没有使用的代码）从最终的目标文件中过滤掉。

这里只是把源码切换到了 ES Module，单元测试用例并未切换，主要原因是 CommonJS Module 对 Jest 的一些特性（resetModules）更友好，即便切换到 ES Module，在需要模块状态隔离的场景，仍然要用 require，所以切换意义不是很大。

还有 Haste，则是 React 团队自定义的模块处理工具，用来解决长相对路径的问题，例如：
``` js
// ref: react-15.5.4
var ReactCurrentOwner = require('ReactCurrentOwner');
var warning = require('warning');
var canDefineProperty = require('canDefineProperty');
var hasOwnProperty = Object.prototype.hasOwnProperty;
var REACT_ELEMENT_TYPE = require('ReactElementSymbol');
```
Haste 模块机制下模块引用不需要给出明确的相对路径，而是通过项目级唯一的模块名来自动查找，例如：
``` js
// 声明
/**
 * @providesModule ReactClass
 */

// 引用
var ReactClass = require('ReactClass');
```
从表面上解决了长路径引用的问题（并没有解决项目结构深层嵌套的根本问题），使用非标准模块机制有几个典型的坏处：

**1.与标准不和，接入标准生态中的工具时会面临适配问题**

**2.源码难读，不容易弄明白模块依赖关系**

React 16 去掉了大部分自定义的模块机制（ReactNative 里还有一小部分），采用 Node 标准的相对路径引用，长路径的问题通过重构项目结构来彻底解决，采用扁平化目录结构（同 package 下最深2级引用，跨 package 的经 Yarn 处理以顶层绝对路径引用）

#### Flow + ESLint

Flow 是 facebook 出品的 JavaScript 静态类型检查工具，所谓类型检查，就是在编译期尽早发现（由类型错误引起的）bug，又不影响代码运行（不需要运行时动态检查类型），使编写 JavaScript 具有和编写 Java 等强类型语言相近的体验。我们看一下 React 使用实例：

``` js
export type ReactElement = {
  $$typeof: any,
  type: any,
  key: any,
  ref: any,
  props: any,
  _owner: any, // ReactInstance or ReactFiber

  // __DEV__
  _store: {
    validated: boolean,
  },
  _self: React$Element<any>,
  _shadowChildren: any,
  _source: Source,
};
```

除了静态类型声明及检查外，Flow 最大的特点是**对React组件及JSX的深度支持**：
``` js
type Props = {
  foo: number,
};
type State = {
  bar: number,
};
class MyComponent extends React.Component<Props, State> {
  state = {
    bar: 42,
  };

  render() {
    return this.props.foo + this.state.bar;
  }
}
```

另外还有导出类型检查的 Flow “魔法”，用来校验 mock 模块的导出类型是否与源模块一致：
```js
type Check<_X, Y: _X, X: Y = _X> = null;
(null: Check<FeatureFlagsShimType, FeatureFlagsType>);
```

Eslint 解决了代码格式检查的问题，同时，一些有用的提示能让我们发现 bug 和无用代码（如 no-unused-vars, no-extra-bind, no-implicit-globals），例如：

```js
rules: {
  'no-unused-expressions': ERROR,
  'no-unused-vars': [ERROR, {args: 'none'}],
  // React & JSX
  // Our transforms set this automatically
  'react/jsx-boolean-value': [ERROR, 'always'],
  'react/jsx-no-undef': ERROR,
}
```

#### Prettier

Prettier 可以制定想要的代码风格，然后通过脚本或编辑器插件来一键格式化/美化代码，我发现使用 Prettier 有很多益处：

**1.代码格式化成统一风格**

**2.提交之前对有改动的部分进行格式化，也可以保存文件的时候自动统一风格。**

**3.配合持续集成，保证PR代码风格完全一致（否则build失败，并输出风格存在差异的部分）**

**4.对构建结果进行格式化，一方面提升dev bundle可读性，另外还有助于发现prod bundle中的冗余代码**

**5.开源代码开发者不需要去学习项目的代码风格。**

#### Yarn workspace

Yarn 的 workspace 特性用来解决 monorepo 的 package 依赖（作用类似于 lerna bootstrap），通过在 node_modules 下建立软链接“骗过”Node模块机制。

通过 package.json/workspaces 配置 Yarn workspaces：

``` json
{
    // ...
    "workspaces": [
        "packages/*"
    ],
    // ...
}
```

注意：Yarn 的实际处理与 Lerna 类似，都通过软链接来实现，只是在包管理器这一层提供 monorepo package 支持更合理一些，具体原因见[Workspaces in Yarn | Yarn Blog](https://yarnpkg.com/blog/2017/08/02/introducing-workspaces/#lerna)

``` js
import {enableUserTimingAPI} from 'shared/ReactFeatureFlags';
import getComponentName from 'shared/getComponentName';
import invariant from 'fbjs/lib/invariant';
import warning from 'fbjs/lib/warning';
```

另外，Yarn 与 Lerna 可以无缝结合，通过 useWorkspaces 选项把依赖处理部分交由 Yarn 来做，详细见[Integrating with Lerna](https://yarnpkg.com/blog/2017/08/02/introducing-workspaces/#integrating-with-lerna)


#### HUBOT

[HUBOT](https://hubot.github.com/) 是指 Github 机器人，通常用于：

**1. 持续集成、PR 触发构建/检查**
**2. 管理 Issue，关掉不活跃的讨论帖**

主要围绕 PR 与 Issue 做一些自动化的事情，比如 React 团队计划（目前还没这么做）机器人回复 PR 对 bundle size 的影响，以此督促持续优化 bundle size。

目前每次构建把 bundle size 变化输出到文件，并交由 Git 追踪变化（提交上去），例如：

``` json
{
  "bundleSizes": [
    {
      "filename": "react.development.js",
      "bundleType": "UMD_DEV",
      "packageName": "react",
      "size": 59086,
      "gzip": 16296
    },
    {
      "filename": "react.production.min.js",
      "bundleType": "UMD_PROD",
      "packageName": "react",
      "size": 7217,
      "gzip": 3050
    },
    // ...
}
```

缺点可想而知，这个json文件经常冲突，要么需要浪费精力 merge 冲突，要么就懒得提交这个自动生成的麻烦文件，导致版本滞后，所以计划通过 GitHub Bot 把这个麻烦抽离出去。

### 构建工具

#### bundle形式

React16 之前提供了两种 bundle 形式：

**第一种：**UMD 单文件，用作外部依赖。

**第二种：**CJS 散文件，用于支持自行构建 bundle（把 React 作为源码依赖）。

存在一些问题：

**一：**自行构建的版本不一致：不同的 build 环境/配置构建出的 bundle 都不一样。

**二：**bundle 性能有优化空间：用打包 App 的方式构建类库不太合适，性能上有提升余地

**三：**不利于实验性优化尝试：无法对散文件模块应用打包、压缩等优化手段

React 16 调整了 bundle 形式：

**一：**不再提供 CJS 散文件，从 npm 拿到的就是构建好的，统一优化过的 bundle。

**二：**提供 UMD 单文件与 CJS 单文件，分别用于 Web 环境与 Node 环境（SSR）。

以不可再分的类库姿态，把优化环节都收进来，摆脱 bundle 形式带来的限制。

#### Rollup

之前的构建系统是基于 Gulp/Grunt+Browserify 手搓的一套工具，后来在扩展方面受限于工具，例如：

Node 环境下性能不好：频繁的 process.env.NODE_ENV 访问拖慢了 SSR 性能，但又没办法从类库角度解决，因为 Uglify 依靠这个去除无用代码，所以 React SSR 性能最佳实践一般都有一条“重新打包 React，在构建时去掉 process.env.NODE_ENV”（当然，React 16 不需要再这样做了，原因见上面提到的bundle形式变化）

丢弃了过于复杂（overly-complicated）的自定义构建工具，改用更合适的 Rollup：

> It solves one problem well: how to combine multiple modules into a flat file with minimal junk code in between.

无论 Haste -> ES Module 还是 Gulp/Grunt+Browserify -> Rollup 的切换都是从非标准的定制化方案切换到标准的开放的方案，应该在“手搓”方面吸取教训，为什么业界规范的东西在我们的场景不适用，非要自己造吗？

#### mock module

构建时可能面临动态依赖的场景：不同的 bundle 依赖功能相似但实现存在差异的 module，例如 ReactNative 的错误提醒机制是显示个红框，而 Web 环境就是输出到 Console。

一般解法有2种：

**第一种：**运行时动态依赖（注入）：把两份都放进bundle，运行时根据配置或环境选择。

**第二种：**构建时处理依赖：多构建几份，不同的bundle含有各自需要的依赖模块。

显然构建时处理更干净一些，即 mock module，开发中不用关心这种差异，构建时根据环境自动选择具体依赖，通过手写简单的 Rollup 插件来实现：[动态依赖配置](https://github.com/facebook/react/blob/master/scripts/rollup/forks.js) + [构建时依赖替换](https://github.com/facebook/react/blob/master/scripts/rollup/plugins/use-forks-plugin.js)。

#### Closure Compiler

[google/closure-compiler](https://github.com/google/closure-compiler)是个非常强大的 minifier，有3种优化模式（compilation_level）：

**第一种：**WHITESPACE_ONLY：去除注释，多余的标点符号和空白字符，逻辑功能上与源码完全等价。

**第二种：**SIMPLE_OPTIMIZATIONS：默认模式，在 WHITESPACE_ONLY 的基础上进一步缩短变量名（局部变量和函数形参），逻辑功能基本等价，特殊情况（如 eval('localVar')按名访问局部变量和解析 fn.toString() ）除外

**第三种：**ADVANCED_OPTIMIZATIONS：在 SIMPLE_OPTIMIZATIONS 的基础上进行更强力的重命名（全局变量名，函数名和属性），去除无用代码（走不到的，用不着的），内联方法调用和常量（划算的话，把函数调用换成函数体内容，常量换成其值）

关于compilation_level的详细信息见[Closure Compiler Compilation Levels](https://developers.google.com/closure/compiler/docs/compilation_levels)

ADVANCED 模式过于强大：
```js
// 输入
function hello(name) {
  alert('Hello, ' + name);
}
hello('New user');

// 输出
alert("Hello, New user");
```
也可以在[Closure Compiler Service](https://closure-compiler.appspot.com/home)在线试玩。

迁移切换有一定风险，因此 React 用的还是 SIMPLE 模式，但后续可能有计划开启 ADVANCED 模式，充分利用 Closure Compiler 优化 bundle size。

#### Error Code System

> In order to make debugging in production easier, we’re introducing an Error Code System in 15.2.0. We developed a gulp script that collects all of our invariant error messages and folds them to a JSON file, and at build-time Babel uses the JSON to rewrite our invariant calls in production to reference the corresponding error IDs.

简言之，在 prod bundle 中把详细的报错信息替换成对应错误码，生产环境捕获到运行时错误就把错误码与上下文信息抛出来，再丢给错误码转换服务还原出完整错误信息。这样既保证了 prod bundle 尽量干净，还保留了与开发环境一样的详细报错能力。

例如生产环境下的非法 React Element 报错：

> Minified React error #109; visit https://reactjs.org/docs/error-decoder.html?invariant=109&args[]=Foo for the full message or use the non-minified dev environment for full errors and additional helpful warnings.

很有意思的技巧，确实在提升开发体验上花了不少心思。

#### envification

所谓 envification 就是分环境 build，例如：
``` js
// ref: react-16.2.0/build/packages/react/index.js
if (process.env.NODE_ENV === 'production') {
  module.exports = require('./cjs/react.production.min.js');
} else {
  module.exports = require('./cjs/react.development.js');
}
```

常用手段，构建时把 process.env.NODE_ENV 替换成目标环境对应的字符串常量，在后续构建过程中（打包工具/压缩工具）会把多余代码剔除掉。

除了 package 入口文件外，还在里面做了同样的判断作为双保险：
``` js
// ref: react-16.2.0/build/packages/react/cjs/react.development.js
if (process.env.NODE_ENV !== "production") {
  (function() {
    module.exports = react;
  })();
}
```

此外，还担心开发者误用 dev bundle 上线，所以在 React DevTools 也加了一点提醒：

> This page is using the development build of React.

#### DCE check

DCE(dead code eliminated) check 是指检查无用代码是否被正常去除。

考虑了一种特殊情况：process.env.NODE_ENV 如果是在运行时设置的话也不合理（可能存在另一环境的多余代码），所以还通过 React DevTools 做了 bundle 环境检查：

```js
// ref: react-16.2.0/packages/react-dom/npm/index.js
function checkDCE() {
  if (process.env.NODE_ENV !== 'production') {
    throw new Error('^_^');
  }
  try {
    __REACT_DEVTOOLS_GLOBAL_HOOK__.checkDCE(checkDCE);
  } catch (err) {
    console.error(err);
  }
}
if (process.env.NODE_ENV === 'production') {
  checkDCE();
}

// DevTools 即__REACT_DEVTOOLS_GLOBAL_HOOK__.checkDCE声明
checkDCE: function(fn) {
  try {
    var toString = Function.prototype.toString;
    var code = toString.call(fn);
    if (code.indexOf('^_^') > -1) {
      hasDetectedBadDCE = true;
      setTimeout(function() {
        throw new Error(
          'React is running in production mode, but dead code ' +
            'elimination has not been applied. Read how to correctly ' +
            'configure React for production: ' +
            'https://fb.me/react-perf-use-the-production-build'
        );
      });
    }
  } catch (err) { }
}
```

原理类似于 Redux 的 minified 检测，先声明一个含有 dev 环境判断的方法，在判断中包含一个标识字符串，然后运行时（通过 DevTools ）检查 fn.toString() 源码，如果含有该标识字符串就说明 DCE 失败（无用代码没在 build 过程中去除），异步 throw 出来。

关于 DCE check 的详细信息，可以参考[Detecting Misconfigured Dead Code Elimination](https://reactjs.org/blog/2017/12/15/improving-the-repository-infrastructure.html#detecting-misconfigured-dead-code-elimination)

### 测试工具

#### Jest

[Jest](https://jestjs.io/) 是由 Facebook 发布的开源的、基于[Jasmine](https://jasmine.github.io/)的 JavaScript 单元测试框架。

**为什么选择Jest？**

**第一点：**Jest 可以利用其特有的快照测试功能，通过比对 UI 代码生成的快照文件，实现对 React 等常见框架的自动测试。此外， Jest 的测试用例是并行执行的，而且只执行发生改变的文件所对应的测试，提升了测试速度。

**第二点：**安装配置简单，非常容易上手，几乎是零配置的，通过 npm 命令安装就可以直接运行了。

**第三点：**Jest 内置了测试覆盖率工具 [istanbul](https://github.com/gotwarlost/istanbul)，可以通过命令开启或者在 package.json 文件进行更详细的配置。运行 istanbul 除了会再终端展示测试覆盖率情况，还会在项目下生产一个 coverage 目录，内附一个测试覆盖率的报告，让我们可以清晰看到分支的代码的测试情况。

**第四点：**集成了断言库，不需要再引入第三方的断言库，并且非常完美的支持 React 组件化测试。

Snapshot Testing 与 UI 自动化测试的一般做法类似，对正确结果截屏作为基准（这个基准需要持续更新，所以快照文件一般随源码提交上去），后续每次改动后与之前的截图做像素级对比，存在差异则说明有问题。

另外，提到 React App 测试，还有一个更狠的：[Enzyme](http://airbnb.io/enzyme/)，可以采用Jest + Enzyme对React组件进行深度测试，更多信息请查看[Unit Testing React Components: Jest or Enzyme](https://www.codementor.io/vijayst/unit-testing-react-components-jest-or-enzyme-du1087lh8)?

关于前端UI自动化测试的一般方法，可参考[如何进行前端自动化测试？ – 张云龙的回答 – 知乎](https://www.zhihu.com/question/29922082/answer/46141819)，当然也可以在[repl.it – try-jest by @amasad在线试玩](https://repl.it/@amasad/try-jest)。

#### preventing Infinite Loops

即死循环检查，Facebook 团队不希望测试过程被死循环阻塞（React 16 递归改循环之后有很多while (true)，他们不太放心）。处理方式与死递归检查类似：限制最大深度（TTL）。通过 Babel 插件来做，在测试环境构建时注入检查：

``` js
// ref: https://github.com/facebook/react/blob/master/scripts/jest/preprocessor.js#L38
require.resolve('../babel/transform-prevent-infinite-loops'),

// ref: https://github.com/facebook/react/blob/master/scripts/babel/transform-prevent-infinite-loops.js#L37
'WhileStatement|ForStatement|DoWhileStatement': (path, file) => {
  const guard = buildGuard({
    ITERATOR: iterator,
    MAX_ITERATIONS: t.numericLiteral(MAX_ITERATIONS),
  });
  if (!path.get('body').isBlockStatement()) {
    const statement = path.get('body').node;
    path.get('body').replaceWith(t.blockStatement([guard, statement]));
  } else {
    path.get('body').unshiftContainer('body', guard);
  }
}
```
用来防护的 buildGuard 如下：

```js
const buildGuard = template(`
  if (ITERATOR++ > MAX_ITERATIONS) {
    global.infiniteLoopError = new RangeError(
      'Potential infinite loop: exceeded ' +
      MAX_ITERATIONS +
      ' iterations.'
    );
    throw global.infiniteLoopError;
  }
`);
```
注意这里使用了一个全局错误变量 global.infiniteLoopError，用来中断后续测试流程：
```js
// ref: https://github.com/facebook/react/blob/master/scripts/jest/setupTests.js#L56
 env.afterEach(() => {
  const error = global.infiniteLoopError;
  global.infiniteLoopError = null;
  if (error) {
    throw error;
  }
});
```

在每个 case 结束都看一眼是否发生死循环，防止 guard 中 throw 的错误被外层 catch 住后，测试流程仍然正常进行。

### 发布工具

#### npm publish

为了规范/简化发布流程，Facebook 团队做了以下几件事情：

**1.采用 master + feature flag 的分支策略**
**2.统一的工具化发布流程**

之前采用 stable 分支策略，发版时需要手动[cherry-pick](https://git-scm.com/docs/git-cherry-pick)，发个版要花很长时间。后来调整为直接从 master 发布，对于不想要的 breaking change，通过 feature flag 在构建时去掉，免去了手动 cherry-pick 的繁琐。

统一了工具发布流程，自动的按顺序自动执行，人工的就提示保存退出，人工处理完成后恢复之前的进度继续向下执行，大致经过以下流程：

``` bash
# 自动
$ npm run test
$ npm run build
# 人工
changelog # 更新日志
smoke test # 冒烟测试
# 自动
$ git commit # 提交更新日志
$ npm publish # 发布新包
# 人工
GitHub release # Github 发布
update site version # 更新版本
test new release # 测试新版
notify involved team # 发布通知
```

这样通过工具化可以减少很多人为失误，保证统一的发布流程。