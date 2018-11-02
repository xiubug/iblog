---
title: React 源码全方位剖析
date: 2018-08-12 22:10:52
tags:
  - source
categories:
  - react
top: 100
---

`版本：v16.5.2`

## 前言

当时在各种前端框架或库充斥市场的情况下，出现了大量优秀的框架，比如 Backbone、Angular、Knockout、Ember 这些框架大都采用了 MV* 的理念，把数据与视图分离。而就在这样纷繁复杂的时期，React 诞生于 Facebook 的内部项目，因为该公司对市场上所有 JavaScript MVC 框架，都不满意，就决定自己写一套，用来架设 Instagram 的网站。做出来以后，发现这套东西很好用，就在2013年5月开源了。所谓知其然还要知其所以然，加上 React 真是一天一改，如果现在不看，以后也真的很难看懂了。目前社区有很多 React 的源码剖析文章，趁着最近工作不忙，我打算分享一下 React 源码，并自形成一个系列，欢迎一起交流。在开始之前我们先做以下几点约定：

**第一：**目前分析的版本是 React 的最新版本 16.5.2；
**第二：**React web应用是最常见的，也是最易于理解的，所以该源码均围绕 React web应用剖析；
**第三：**我尽可能站在我自己的角度去剖析，当然我会借鉴社区比较优秀的文章，同时面对大家的拍砖，我无条件接受，也很乐意与大家一起交换意见，努力写好该 React 源码系列；
**第四：**如果有幸您读到该 React 源码系列，感觉写得还行，还望收藏、分享或打赏。

## 前置知识

我们从这一章开始即将分析 React 的源码，在分析源码之前我们很有必要介绍一些前置知识如flow、Rollup等。除此之外，我们最好已经用过 React 做过实际项目，对 React 的思想有了一定的了解，对绝大部分的 API 都已经有使用，同时，我们应该有一定的HTML、CSS、JavaScript、ES6+、node & npm等功底，并对代码调试有一定的了解。

如果具备了以上条件，并且对 React 的实现原理很感兴趣，那么就可以开始 React 的底层学习了，对它的实现细节一探究竟。

### Flow - JavaScript静态类型检查工具

[Flow](https://flow.org/) 是 facebook 出品的 JavaScript 静态类型检查工具，它与 Typescript 不同的是，它可以部分引入，不需要完全重构整个项目，所以对于一个已有一定规模的项目来说，迁移成本更小，也更加可行。除此之外，Flow 可以提供实时增量的反馈，通过运行 Flow server 不需要在每次更改项目的时候完全从头运行类型检查，提高运行效率。可以简单总结为：**对于新项目，可以考虑使用 TypeScript 或者 Flow，对于已有一定规模的项目则建议使用 Flow 进行较小成本的逐步迁移来引入类型检查。React 的源码利用了 Flow 做了静态类型检查，所以了解 Flow 有助于我们阅读源码。**

#### 为什么用静态类型检查工具 Flow

JavaScript 是动态类型语言，它的灵活性有目共睹，但是过于灵活的副作用就是很容易就写出非常隐蔽的隐患代码，在编译期甚至运行时看上去都不会报错，但是可能会发生各种各样奇怪的和难以解决的bug。

类型检查是当前动态类型语言的发展趋势，所谓类型检查，就是在编译期尽早发现（由类型错误引起的）bug，又不影响代码运行（不需要运行时动态检查类型），使编写 JavaScript 具有和编写 Java 等强类型语言相近的体验。

项目越复杂就越需要通过工具的手段来保证项目的维护性和增强代码的可读性。React 源码在 ES2015 的基础上，除了用 ESLint 保证代码风格之外，也引入了 Flow 做静态类型检查。之所以选择 Flow，**最根本原因应该和 Vue 一样，还是在于工程上成本和收益的考量。** 大致体现在以下几点：

**第一点：**使用 Flow 可以一个一个文件地迁移，如果使用 TypeScript，则需要全部替换，成本极高，短期内并不现实；
**第二点：**Babel 和 ESLint 都有对应的 Flow 插件以支持语法，可以完全沿用现有的构建配置，非常小成本的改动就可以拥有静态类型检查的能力；
**第三点：**更贴近 ES 规范。除了 Flow 的类型声明之外，其他都是标准的 ES。万一哪天不想用 Flow 了，用`babel-plugin-transform-flow-strip-types `转一下，就得到符合规范的 ES；
**第四点：**在需要的地方保留 ES 的灵活性，并且对于生成的代码尺寸有更好的控制力 (rollup / 自定义 babel 插件）。

#### 如何用静态类型检查工具 Flow

在这里我们就简单说一说 Flow 的用法，其他用法可以参考[Flow官网](https://flow.org/)（可能需要 VPN，非常不稳定），有时间我会详细写一篇 Flow 使用指南。

Flow 仅仅是一个用于检查的工具，安装使用都很方便，使用时注意以下3点即可：

1.将 Flow 安装到我们的项目中。
2.确保编译之后的代码移除了 Flow 相关的语法。
3.在需要检查的地方增加了 Flow 相关的类型注解。

**第一点：将Flow增加到我们的项目中**

安装最新版本的 Flow：
``` bash
$ npm install --save-dev flow-bin
```
安装完成之后在 package.json 文件中增加执行脚本：

``` json
{
  // ...
  "scripts": {
    "your-script-name": "flow",
    // ...
  },
  // ...
}
```

然后初始化 Flow：

``` bash
$ npm run flow init
```

执行完成后，Flow 会在终端输出以下内容：

```
> yourProjectName@1.0.0 flow /yourProjectPath
> flow "init"
```

然后在根目录下生成一个名为 .flowconfig 的文件，打开之后是这样的：

``` 
[ignore]

[include]

[libs]

[lints]

[options]

[strict]
```
基本上，配置文件没有什么特殊需求是不用去配置的，Flow 默认涵盖了当前目录之后的所有文件。[include] 用于引入项目之外的文件。例如：

```
[include]

../otherProject/a.js

[libs]
```

它会将和当前项目平级的 otherProject/a.js 文件纳入进来。详细配置文件请看[官网](https://flow.org/en/docs/config/)。

**第二点：编译之后的代码移除 Flow 相关的语法**

Flow 在 JavaScript 语法的基础上使用了一些注解（annotation）进行了扩展。因此浏览器无法正确的解读这些 Flow 相关的语法，我们必须在编译之后的代码中（最终发布的代码）将增加的 Flow 注解移除掉。具体方法需要看我们使用了什么样的编译工具。下面将说明一些 React 开发常用的编译工具：

**方式一：**create-react-app

如果我们的项目是使用[create-react-app](https://github.com/facebook/create-react-app)直接创建的，那么移除 Flow 语法的事项就不用操心了，create-react-app 已经帮我们搞定了这个事。

**方式二：**Babel

如果使用 Babel 我们需要安装一个 Babel 对于 Flow 的 preset：
``` bash
$ npm install --save-dev babel-preset-flow
```

然后，我们需要在项目根目录[Babel 的配置文件 .babelrc](http://babeljs.io/docs/en/babelrc/)中添加一个 Flow 相关的 preset：

``` json
{
  "presets": [
    "flow",
    //other config
  ]
}
```
**方式三：**flow-remove-types

如果我们既没有使用 create-react-app 也没使用 Babel 作为语法糖编译器，那么可以使用 [flow-remove-types](https://github.com/flowtype/flow-remove-types) 这个工具在发布之前移除 Flow 代码。

**第三点：在需要检查的地方增加 Flow 相关的类型注解**

如果我们了解 C++/C# 的元编程或者 Java 的 Annotation，那么理解 Flow 的 Annotation 就会非常轻松。大概就是在文件、方法、代码块之前增加一个注解（Annotation）用来告知 Flow 的执行行为。

首先，Flow 只检查包含`// @flow`注解的文件，所以如果需要检查，我们需要这样编写我们的文件，首先我们写一个正确的示例：

``` js
/* @flow */

function add(x: number, y: number): number {
  return x + y
}

add(22, 11)
```

运行 Flow 终端会打印出以下内容：

``` 
> yourProjectName@1.0.0 flow /yourProjectPath
> flow "init"

Found 0 errors
```

承接上面代码，我们把代码修改成带有检查错误的例子：

``` js
/* @flow */

function add(x: number, y: number): number {
  return x + y
}

add("Hello", 11)
```

运行 Flow 终端会打印出以下内容：

```
> yourProjectName@1.0.0 flow /yourProjectPath
> flow "init"

Error ------------------------------------------------------------------------------------- src/platforms/web/mnr.js:8:5

Cannot call `add` with `"Hello"` bound to `x` because string [1] is incompatible with number [2].

   src/platforms/web/mnr.js:8:5
   8| add("Hello", 11)
          ^^^^^^^ [1]

References:
   src/platforms/web/mnr.js:4:17
   4| function add(x: number, y: number): number {
                      ^^^^^^ [2]



Found 1 error
```

到这里，Flow 已经算是安装成功了，接下来的事是要增加各种注解以加强类型限定或者参数检测。之后的内容将简要介绍 flow 的类型检查方式。

#### Flow 的类型检查方式

现在我们就说说 Flow 常用的2种类型检查方式：
**类型推断**：通过变量的执行上下文来推断出变量类型，然后根据这些推断来检查类型。
**类型注释**：事先注释好我们期望的类型，Flow 会基于这些注释来检查。

**第一种方式：类型推断**

此方式不需要编写任何代码即可进行类型检查，最小化开发者的工作量，它也不会强制我们改变开发习惯，因为它会自动推断出变量的类型，这就是所谓的类型推断，Flow 最重要的特性之一。

通过一个简单例子说明一下：
``` js
/*@flow*/

function split(str) {
  return str.split(' ')
}

split(11)
```
Flow 检查上述代码后会报错，因为函数 split 期待的参数是字符串，而我们输入的是数字。

**第二种方式：类型注释**

如上所述，类型推断是 Flow 最有用的特性之一，不需要编写任何代码就能进行类型检查。但在某些特定的场景下，使用类型注释可以提供更好更明确的检查依据。

看看以下代码：
``` js
/*@flow*/

function add(x, y){
  return x + y
}

add('Hello', 11)
```

Flow 根据类型推断检查上述代码时检查不出任何错误，因为从语法层面考虑， + 既可以用在字符串上，也可以用在数字上，我们并没有明确指出 add() 的参数必须为数字。在这种情况下，我们可以借助类型注释来指明期望的类型。类型注释是以冒号 : 开头，可以在函数参数，返回值，变量声明中使用。如果我们在上段代码中使用类型注释，就会变成如下：
``` js
/*@flow*/

function add(x: number, y: number): number {
  return x + y
}

add('Hello', 11)
```
现在 Flow 就能检查出错误，因为函数参数的期待类型为数字，而我们提供了字符串。上面的例子是针对函数的类型注释。接下来我们来看看 Flow 能支持的一些常见的类型注释:

**第一种：**数组
```js
/*@flow*/

var arr: Array<number> = [1, 2, 3]

arr.push('Hello')
```

数组类型注释的格式是 Array<T>，T 表示数组中每项的数据类型。在上述代码中，arr 是每项均为数字的数组。如果我们给这个数组添加了一个字符串，Flow 能检查出错误。

**第二种：**类和对象

```js
/*@flow*/

class Bar {
  x: string;           // x 是字符串
  y: string | number;  // y 可以是字符串或者数字
  z: boolean;

  constructor(x: string, y: string | number) {
    this.x = x
    this.y = y
    this.z = false
  }
}

var bar: Bar = new Bar('hello', 4)

var obj: { a: string, b: number, c: Array<string>, d: Bar } = {
  a: 'hello',
  b: 11,
  c: ['hello', 'world'],
  d: new Bar('hello', 3)
}
```

类的类型注释格式如上，可以对类自身的属性做类型检查，也可以对构造函数的参数做类型检查。这里需要注意的是：属性 y 的类型中间用 | 做间隔，表示 y 的类型即可以是字符串也可以是数字。

对象的注释类型类似于类，需要指定对象属性的类型。

**第三种：**Null/undefined

Flow 会检查所有的 JavaScript 基础类型—— Boolean、String、Number、null、undefined（在Flow中用void代替）。除此之外还提供了一些操作符号，例如 text : ?string，它表示参数存在“没有值”的情况，除了传递 string 类型之外，还可以是 null 或 undefined。需要特别注意的是，这里的没有值和 JavaScript 的表达式的“非”是两个概念，Flow 的“没有值”只有 null、void（undefined），而 JavaScript 表达式的“非”包含：null、undefined、0、false。

如果想任意类型 T 可以为 null 或者 undefined，只需写成如下 ?T 的格式即可：

```js
/*@flow*/

var foo: ?string = null
```
此时，foo 可以为字符串，也可以为 null。

#### Flow 在 React 源码中的应用

Flow 是 Facebook 开源的静态代码检查工具，它的作用就是在运行代码之前对 React 组件以及 Jsx 语法进行静态代码的检查以发现一些可能存在的问题。在 React v16 Fiber中的部分 TypeScript 代码只是类型声明文件和测试代码，也就是为了方便利用 TypeScript 写应用的开发者使用 React，给了接口定义和测试样例而已。

#### 小结

React 重构告诉我们，项目重构要么依赖规范，要么就得自己有绝对控制权，同时还要考量开发成本、项目收益以及整个团队的技术水平，并不是一味的什么火就用什么。这一节主要对 Flow 的认识，有助于我们后续阅读 React 的源码，这种静态类型检查的方式非常有利于大型项目源码的开发和维护。

### Rollup - 另一个前端模块化的打包工具

Rollup 是前端模块化的一个打包工具，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。简单地说，**它可以从一个入口文件开始，将所有使用的模块根据命令或者根据 Rollup 配置文件打包成一个目标文件，并且 Rollup 会自动过滤掉那些没有被使用过的函数或变量，从而使代码最小化，如果想使用直接导入这一个目标文件即可，因此 Rollup 极其适合构建一个工具库。**

这里提到 Rollup 的两个特别重要的特性，**第一个就是它使用了 ES2015 的模板标准，这意味着我们可以直接使用 import 和 export 而不需要引入 babel。另一个重要特性叫做 tree-shaking，这个特性可以帮助我们将无用代码（即没有使用的代码）从最终的目标文件中过滤掉。**举个简单的例子，我们在 foo.js 文件定义了 f1 和 f2 两个方法，然后在入口文件 index.js 只引入了 foo.js 文件中的 f1 方法，那么在最后打包 index.js 文件时，Rollup 就不会将 f2 方法打包到最终文件中（这个特性是基于 ES6 模块的静态分析的，也就是说，只有 export 而没有 import 的变量是不会被打包到最终代码中的）。

#### 为什么用前端模块化的打包工具 Rollup

之前的构建系统是基于 Gulp/Grunt+Browserify 手搓的一套工具，后来在扩展方面受限于工具，例如：

Node 环境下性能不好：频繁的`process.env.NODE_ENV`访问拖慢了`SSR 性能`，但又没办法从类库角度解决，因为`Uglify`依靠这个去除无用代码，所以`React SSR`性能最佳实践一般都有一条“`重新打包 React，在构建时去掉 process.env.NODE_ENV`”.

丢弃了过于复杂（overly-complicated）的自定义构建工具，改用更合适的 Rollup：

> It solves one problem well: how to combine multiple modules into a flat file with minimal junk code in between.

无论 Haste -> ES Module 还是 Gulp/Grunt+Browserify -> Rollup 的切换都是从非标准的定制化方案切换到标准的开放的方案，应该在“手搓”方面吸取教训，为什么业界规范的东西在我们的场景不适用，非要自己造吗？

#### 如何用前端模块化的打包工具 Rollup

关于如何使用前端模块化的打包工具 Rollup，这里就不做过多介绍了，可参考我之前写的一篇文章：[Rollup使用指南](/2018/08/04/rollup-tutorial.html)，更详细的使用文档可参考：[官网](https://www.rollupjs.com/guide/zh)。


#### Webpack 和 Rollup 有什么不同

2017年4月初，Facebook 将一个巨大的 pull 请求合并到了 React 主分支(master)中，将其现有的构建流程替换为基于 Rollup，这一举动促使一些人产生很大的疑惑“React 为什么选择 Rollup 而抛弃 webpack”，难道webpack要跌下神坛了？

Webpack 是目前使用最为火热的打包工具，没有之一，每月有数百万的下载量，为成千上万的网站和应用提供支持。相比之下，Rollup 并不起眼。但 React 并不孤单 – Vue，Ember，Preact，D3，Three.js，Moment 以及其他许多知名的库也使用 Rollup 。世界到底怎么了？为什么我们不能只有一个大众认可的 JavaScript 模块化打包工具？

Webpack 始于2012年，由 Tobias Koppers 发起，用于解决当时现有工具未解决的的一个难题：**构建复杂的单页应用程序(SPA)。**特别是 webpack 的两个特性改变了一切：

**第一个特性：**代码拆分(Code Splitting)

代码拆分也就是说我们可以将应用程序分解成可管理的代码块，可以按需加载，这意味着用户可以快速获取网站内容，而不必等到整个应用程序下载和解析完成。

**第二个特性：**各式各样的加载器（loader）

不管是图像，css，还是 html ，在 Webpack 看来一切都可作为模块，然后通过不同的加载器 loader 来加载它们。

ES6 发布之后，其中引入的模块机制使得静态分析成为了可能，于是 Rollup 发布了：其中 Rollup 有两个特别重要的特性，第一个就是它利用 ES2015 巧妙的模块设计，尽可能高效的构建出能够直接被其他 Javascript 库的。另一个重要特性叫做 tree-shaking，这个特性可以帮助我们将无用代码（即没有使用的代码）从最终的目标文件中过滤掉。

紧接着 Webpack2 发布，仿照 Rollup 增加了 tree-shaking。 在之后， Webpack3 发布，仿照 Rollup 又增加了 Scope Hoisting。在在之后， Parcel 发布了一个快速、零配置的打包工具。于是，Webpack4 仿照 Parcel 发布了。

说了这么多，工作中我们到底该用哪个工具？

对于应用使用 webpack，对于类库使用 Rollup。如果我们需要代码拆分(Code Splitting)，或者我们有很多静态资源需要处理，再或者我们构建的项目需要引入很多 CommonJS 模块的依赖，那么 webpack 是个很不错的选择。如果您的代码库是基于 ES2015 模块的，而且希望我们写的代码能够被其他人直接使用，我们需要的打包工具可能是 Rollup。

#### 小结

无论 Haste -> ES Module 还是 Gulp/Grunt+Browserify -> Rollup 的切换都是从非标准的定制化方案切换到标准的开放的方案，可以看出 React 团队也在积极拥抱标准方案并非一味造轮子。其实 Vue.js 1.0.10 就已经使用 Rollup 了，而 React v16.0 改用 Rollup 肯定也有借鉴之意，因此，好技术都是在借鉴的大背景下诞生的（Vue 就是一个典型的例子）。在这里通过对 Rollup 的认识，有助于我们了解 React 的构建以及源码目录结构。

## 项目介绍

上一章我们简单介绍了下flow、Rollup等前置知识，有兴趣的可以有针对性的学习它们。这一章我们真正的开始分析 React 源码，激动不激动？该章主要包括三小节：项目目录、源码构建、源码入口。

### 项目目录

#### 目录结构

我们主要剖析的 React 源码目录在 packages 下，在这里我们看看详细目录结构，混个眼熟：

``` js
├── build --------------------------------------- 构建后的输出目录
├── fixtures ------------------------------------ React 开发的测试用例
├── packages ------------------------------------ 源码目录，我们主要剖析目录
│   ├── create-subscription --------------------- 在组件里订阅额外数据的工具
│   ├── events ---------------------------------- 事件处理 
│   ├── interaction-tracking -------------------- 跟踪交互事件
│   ├── react ----------------------------------- 核心代码
│   ├── react-art ------------------------------- 矢量图形库
│   ├── react-dom ------------------------------- DOM 渲染相关
│   ├── react-is -------------------------------- React 元素类型相关
│   ├── react-native-renderer ------------------- react-native 渲染相关 
│   ├── react-noop-renderer --------------------- Fiber 测试相关 
│   ├── react-reconciler ------------------------ React 调制器
│   ├── react-scheduler ------------------------- 规划 React 初始化，更新等等
│   ├── react-test-renderer --------------------- 实验性的 React 渲染器
│   ├── shared ---------------------------------- 通用代码
│   ├── simple-cache-provider ------------------- 为 React 应用提供缓存
│   ├── server ---------------------------------- 服务端渲染
│   ├── sfc ------------------------------------- .vue 文件解析
│   ├── shared ---------------------------------- 整个项目通用代码
├── scripts ------------------------------------- 公共的lint，build，test和release等相关的文件
│   ├── eslint ---------------------------------- 语法规则和代码风格
│   ├── flow ------------------------------------ Flow 类型声明
│   ├── git ------------------------------------- git钩子的目录
│   ├── jest ------------------------------------ JavaScript 测试目录
│   ├── release --------------------------------- 自动发布新版本脚本
│   ├── rollup ---------------------------------- rollup 构建目录
├── .babelrc ------------------------------------ babel 配置文件
├── .editorconfig ------------------------------- 编辑器语法规范配置
├── .eslintignore ------------------------------- eslint 忽略配置 
├── .eslintrc ----------------------------------- eslint 配置文件
├── .gitattributes ------------------------------ 给 attributes 路径名的简单文本文件
├── .gitignore ---------------------------------- git 忽略配置
├── .mailmap ------------------------------------ 邮件列表档案 
├── .nvmrc -------------------------------------- nvm 配置文件
├── .prettierrc.js ------------------------------ prettierrc 配置文件
├── .watchmanconfig ----------------------------- watchman 配置文件
├── appveyor.yml -------------------------------- GitHub 托管项目的自动化集成
├── AUTHORS ------------------------------------- 开发者列表档案
├── CHANGELOG.md -------------------------------- 更新日志
├── CODE_OF_CONDUCT.md -------------------------- Code of Conduct
├── CONTRIBUTING.md ----------------------------- Contributing to React
├── dangerfile.js ------------------------------- 提高 Code Review 体验
├── netlify.toml -------------------------------- 持续集成静态网站
├── package-lock.json --------------------------- npm 加锁文件
├── package.json -------------------------------- 项目管理文件
├── README.md ----------------------------------- 项目文档
├── yarn.lock ----------------------------------- yarn 加锁文件
```

一眼望去，上面的目录结构是不是感觉很是奇怪？
**根目录下没有 src 之类的源码目录，也没有 test 这类的存放单元测试的目录，只有一个 packages 目录。这个 repository 其实是一个用 Lerna 管理的 monorepo。实际上，我们往npm上发布的几个package都来自于同一个codebase，包括react、react-dom、react-is......**

#### monorepo

通常，当我们的项目不断的迭代更新的时候，我们会根据业务或者是功能又或者是方便复用某些代码模块，把一个大的 codebase 拆成一些独立的 package 或 module，再将这些功能独立的 package 分别放入单独的 repository 中进行维护，此方式可以简单地称为`multiple repositories`。而 monorepo 则是一种相反的做法，它提倡将所有的相关 package 都放入一个 repository 来管理。

**monorepo VS multirepo（集中管理 vs 多元化）**
首先这两者都是管理组织代码的方式，顾名思义 monorepo 就是把所有的相关项目都放在一个仓库中（比如 React, Angular, Babel, Google...），multirepo 则是按模块分为多个仓库。

**multirepo：**这种管理方式可以让每个子团队拥有自己的 repo，我们可以用自己擅长的工具、workflow 等等。多元化能促使各个团队尽可能的提升自己的效率。但代价也在于会增加很多沟通成本，如果我们项目用到的库中发现了一个 bug，就必须到目标库里修复它、打包、发版本，然后再回到我们的库继续工作。在不同的仓库间，我们不仅需要处理不同的代码、工具，甚至是不同的工作流程。甚至我们只能去问维护这个仓库的人，能不能为我们做出改变，然后等着他们去解决。

**monorepo：**这种管理方式可以让不同的团队走自己的路，并不见得能提高生产力。虽然有些团队可能会找到自己最佳的工作方式，但我们的收益也会被其他团队不那么好的工作方式所抵消。相反，严格统一的管理更能提升效率，团队中的任何人都可以（并且应该也被鼓励）修改任何东西（因为修改造成的结果马上就能展现出来，）。虽然把所有的鸡蛋都放进了一个篮子里，但我们也可以更小心的照顾这个篮子。

如果我们团队选择 monorepo，那主要的挑战自然是随着项目的发展，其会变得非常庞大（因为没有根据模块或功能拆分成不同 repo）。因此会需要很多的工具来应对这样的挑战。虽然我们可能认为这是一个很糟糕的做法，但是现在这样做的开源项目和公司并不算少。

**谁在使用 monorepo**
Babel 是一个 Javascript 编译器，它可以将浏览器环境尚未支持的 Javascript 变为向下兼容的版本。因此，我们可以毫无顾虑地使用较新的 Javascript 语法和特性来提升编程的体验和效率。

其中 Babel 官方维护了众多独立的 plugin、polyfill、preset，但并未按照传统，将这些独立的模块分别放入不同的 repo。而是遵循了 monorepo 的方式，将它们放入一个相同的 repo 中。因为 Babel 认为，有效的组织一个多模块，多 repo 的项目，就像是尝试教一个刚出生的婴儿骑自行车一样。
> Juggling a multimodule project over multiple repos is like trying to teach a newborn baby how to ride a bike.

所以，Babel 采用了 lerna 来管理自己的 monorepo。

无独有偶，Cycle.js（一个函数式和响应式 Javascript 框架）的作者 André Staltz 也摒弃了一个 package 一个 repo 的做法，将 Cycle.js 的众多 package 迁移到了一个 monorepo 中。他也认为，管理多个 repo 并不是件有意思的事情。多个  repo 意味着有多个地方需要处理 issue，保持多个 repo 的 issue 标签统一，管理很多 PR 和 git 钩子等等。
> Managing multiple repos isn’t that fun. Multiple repos means multiple places to manage issues, manage issue labels (and making them consistent across repos), manage PRs, git hooks for conventions, etc.

André Staltz 并没有使用 lerna 之类的工具来实现自己的 monorepo，他自己通过 Bash sh 实现了类似于 Lerna 管理的 monorepo。

除了 Babel 和 Cycle.js 以外，React、Angular、Meteor、Ember，还包括国内饿了么的 mint-ui 等等开源项目，以及一些公司如 Google、Facebook、BBC 等也都采用了 monorepo。它到底有什么优点，这么多公司，这么多库纷纷加入。

**优点**
**一：**单个的 lint，build，test 和 release 流程；
**二：**统一的地方处理issue；
**三：**不用到处去找自己项目的repo；
**四：**方便管理版本和dependencies；
**五：**跨项目的操作和修改变得容易；
**六：**方便生成总的changelog。

**缺点**
**一：**repo 的体积变得很大；
**二：**安全问题，如何管理权限。

关于 monorepo 我们暂且就说这么多，有时间我会单独写一篇 monorepo 文章。

#### 小结

从 React 的目录设计可以看出，React 团队在项目管理比较倾向 monorepo 方式，看来这种严格统一的管理方式真的提升效率。无论 monorepo 方式，还是 multirepo 方式都是为了团队效率，因此建议还是根据团队的情况选定一种方式，尽可能的扬长避短。

### 源码构建

React v16.0 之前源码是基于`Gulp/Grunt+Browserify`构建的，而 React v16.0 是基于`Rollup`构建的，它的构建相关配置都在`scripts/rollup`目录下。

#### 构建命令

通常基于 NPM 托管的项目都会有一个 package.json 文件，实际上它是项目的描述文件，它的内容是一个标准的 JSON 对象。我们通常会配置 script 字段作为 NPM 的构建命令，React 源码构建配置如下：

``` json
{
    // ...
    "scripts": {
        "build": "npm run version-check && node ./scripts/rollup/build.js",
        // ...
        "version-check": "node ./scripts/tasks/version-check.js"
    }
    // ...
}
```
这里`build命令`，实际上先执行`version-check`命令，然后执行`node ./scripts/rollup/build.js`进行打包，其中`version-check`命令实际上是执行`node ./scripts/tasks/version-check.js`，用于检查即将构建的`bundle版本`是否完全匹配，接下来我们就来看看它实际上是如何构建的。

#### 构建过程

我们首先打开`build`命令对应的第一个 JS 文件，在`scripts/tasks/version-check.js`中：

``` js
const reactVersion = require('../../package.json').version;
const versions = {
  'packages/react/package.json': require('../../packages/react/package.json')
    .version,
  'packages/react-dom/package.json': require('../../packages/react-dom/package.json')
    .version,
  'packages/react-test-renderer/package.json': require('../../packages/react-test-renderer/package.json')
    .version,
  'packages/shared/ReactVersion.js': require('../../packages/shared/ReactVersion'),
};

let allVersionsMatch = true;
Object.keys(versions).forEach(function(name) {
  const version = versions[name];
  if (version !== reactVersion) {
    allVersionsMatch = false;
    console.log(
      '%s version does not match package.json. Expected %s, saw %s.',
      name,
      reactVersion,
      version
    );
  }
});

if (!allVersionsMatch) {
  process.exit(1);
}

```
这段代码逻辑非常简单，先获取`即将发布的以及源码核心bundle`的管理文件，再比对`即将发布的和源码核心bundle`的版本，如果不相同，给出对应的提示并结束构建，这样就保证了构建出来的 bundle 版本统一。

接下来我们打开`build`命令对应的第二个 JS 文件，在`scripts/rollup/build.js`中：
```js
async function buildEverything() {
  await asyncRimRaf('build');

  // Run them serially for better console output
  // and to avoid any potential race conditions.
  // eslint-disable-next-line no-for-of-loops/no-for-of-loops
  for (const bundle of Bundles.bundles) {
    await createBundle(bundle, UMD_DEV);
    await createBundle(bundle, UMD_PROD);
    // ...
  }
  // ...
  // ...
}

buildEverything();
```

这里通过调用`buildEverything`函数开启构建过程，`asyncRimRaf`用于删除上一次打包生成的包文件，然后循环包配置文件的配置构建出不同用途的 React 包，稍后我们再来看构建函数`createBundle`，我们先来看看包配置文件，在`scripts/rollup/bundles.js`中：
```js
const bundles = [
  /******* Isomorphic *******/
  {
    label: 'core',
    bundleTypes: [
      UMD_DEV,
      UMD_PROD,
      NODE_DEV,
      NODE_PROD,
      FB_WWW_DEV,
      FB_WWW_PROD,
    ],
    moduleType: ISOMORPHIC,
    entry: 'react',
    global: 'React',
    externals: [],
  },
    /******* React DOM *******/
  {
    label: 'dom-client',
    bundleTypes: [
      UMD_DEV,
      UMD_PROD,
      NODE_DEV,
      NODE_PROD,
      NODE_PROFILING,
      FB_WWW_DEV,
      FB_WWW_PROD,
      FB_WWW_PROFILING,
    ],
    moduleType: RENDERER,
    entry: 'react-dom',
    global: 'ReactDOM',
    externals: ['react'],
  },
  // ...
  // ...
];

module.exports = {
  bundles,
};
```
这里简单列举了一些 React 包构建的配置，其他已省略，可以看出实际上这是一个用于 Rollup 构建配置的对象数组，通过循环该对象数组构建出不同用途的 React 包。接下来我们再看一下构建函数`createBundle`，在`scripts/rollup/build.js`中：
```js
async function createBundle(bundle, bundleType) {
  if (shouldSkipBundle(bundle, bundleType)) {
    return;
  }

  const filename = getFilename(bundle.entry, bundle.global, bundleType);
  const logKey =
    chalk.white.bold(filename) + chalk.dim(` (${bundleType.toLowerCase()})`);
  const format = getFormat(bundleType);
  const packageName = Packaging.getPackageName(bundle.entry);

  let resolvedEntry = require.resolve(bundle.entry);
  if (
    bundleType === FB_WWW_DEV ||
    bundleType === FB_WWW_PROD ||
    bundleType === FB_WWW_PROFILING
  ) {
    const resolvedFBEntry = resolvedEntry.replace('.js', '.fb.js');
    if (fs.existsSync(resolvedFBEntry)) {
      resolvedEntry = resolvedFBEntry;
    }
  }

  const shouldBundleDependencies =
    bundleType === UMD_DEV || bundleType === UMD_PROD;
  const peerGlobals = Modules.getPeerGlobals(bundle.externals, bundleType);
  let externals = Object.keys(peerGlobals);
  if (!shouldBundleDependencies) {
    const deps = Modules.getDependencies(bundleType, bundle.entry);
    externals = externals.concat(deps);
  }

  const importSideEffects = Modules.getImportSideEffects();
  const pureExternalModules = Object.keys(importSideEffects).filter(
    module => !importSideEffects[module]
  );

  const rollupConfig = {
    input: resolvedEntry,
    treeshake: {
      pureExternalModules,
    },
    external(id) {
      const containsThisModule = pkg => id === pkg || id.startsWith(pkg + '/');
      const isProvidedByDependency = externals.some(containsThisModule);
      if (!shouldBundleDependencies && isProvidedByDependency) {
        return true;
      }
      return !!peerGlobals[id];
    },
    onwarn: handleRollupWarning,
    plugins: getPlugins(
      bundle.entry,
      externals,
      bundle.babel,
      filename,
      packageName,
      bundleType,
      bundle.global,
      bundle.moduleType,
      bundle.modulesToStub
    ),
    // We can't use getters in www.
    legacy:
      bundleType === FB_WWW_DEV ||
      bundleType === FB_WWW_PROD ||
      bundleType === FB_WWW_PROFILING,
  };
  const [mainOutputPath, ...otherOutputPaths] = Packaging.getBundleOutputPaths(
    bundleType,
    filename,
    packageName
  );
  const rollupOutputOptions = getRollupOutputOptions(
    mainOutputPath,
    format,
    peerGlobals,
    bundle.global,
    bundleType
  );

  console.log(`${chalk.bgYellow.black(' BUILDING ')} ${logKey}`);
  try {
    const result = await rollup(rollupConfig);
    await result.write(rollupOutputOptions);
  } catch (error) {
    console.log(`${chalk.bgRed.black(' OH NOES! ')} ${logKey}\n`);
    handleRollupError(error);
    throw error;
  }
  for (let i = 0; i < otherOutputPaths.length; i++) {
    await asyncCopyTo(mainOutputPath, otherOutputPaths[i]);
  }
  console.log(`${chalk.bgGreen.black(' COMPLETE ')} ${logKey}\n`);
}
```
上述关键的代码是`const result = await rollup(rollupConfig);`，可以看出这是通过 rollup 打包的，对于单个配置，它是遵循 Rollup 的构建规则的。其中 input 属性表示构建的入口 JS 文件地址，output.file 属性表示构建后的输出的  JS 文件地址，format 属性表示构建的格式，cjs 表示构建出来的文件遵循[CommonJS 规范](http://wiki.commonjs.org/wiki/Modules/1.1)，es 表示构建出来的文件遵循[ES Module 规范](http://exploringjs.com/es6/ch_modules.html)，umd 表示构建出来的文件遵循[UMD 规范](https://github.com/umdjs/umd)。


下面我们以配置文件的第一个`react`配置为例：

构建的入口 JS 文件地址：

``` js
let resolvedEntry = require.resolve(bundle.entry);
```
沿着`bundle.entry`我们发现它的值为`react`（在`scripts/rollup/bundles.js`中），`require.resolve`用于查询文件的完整绝对路径，也就说`react`对应的真实入口路径是`/**/**/react/packages/react/index.js`，由此不难看出所有源码都在`packages`中：

构建后的输出的 JS 文件地址：
```js
const [mainOutputPath, ...otherOutputPaths] = Packaging.getBundleOutputPaths(
    bundleType,
    filename,
    packageName
);
```
接下来我们看看`Packaging.getBundleOutputPaths`，在`scripts/rollup/packaging.js`中：
``` js
function getBundleOutputPaths(bundleType, filename, packageName) {
  switch (bundleType) {
    case NODE_DEV:
    case NODE_PROD:
    case NODE_PROFILING:
      return [`build/node_modules/${packageName}/cjs/${filename}`];
    case UMD_DEV:
    case UMD_PROD:
      return [
        `build/node_modules/${packageName}/umd/${filename}`,
        `build/dist/${filename}`,
      ];
    // ...
    default:
      throw new Error('Unknown bundle type.');
  }
}
```
从上面不难看出所有打包后的输出文件都在`build`，这就是为什么打包前先删除`build`文件了。

#### 小结

通过这一节的分析，我们可以了解到 React 的打包过程，也知道了不同作用和功能的 React 对应的入口以及最终编译生成的 JS 文件。

### 源码入口

#### React 对象

实际项目中，可以看到首先需要使用如下代码：
``` js
import React from 'react';
```
这句代码做的就是引入了React核心源码模块。而我们在源码构建一节讲到 React 的核心入口文件是`packages/react/index.js`:
``` js
'use strict';

const React = require('./src/React');

// TODO: decide on the top-level export form.
// This is hacky but makes it work with both Rollup and Jest.
module.exports = React.default || React;
```

上述代码中执行`import React from 'react'`时，其实引入的就是这里提供的对象。

这里需要说明一点：这里为什么会导出 React.default || React？（以下提到的插件都可以在源码中找到）
1. **React.default 用于 Jest 测试**
babel解析器将 es6 的 export、import等模块关键字转换成 commonjs 的规范，babel 转换 es6 的模块输出逻辑非常简单，即将所有输出都赋值给 exports。其中`packages/react/src/React.js`使用`export default`导出 React 对象，这里 babel 会将其转化`exports.default = React`，因此导入的结果其实是一个含 default 属性的对象，因此需要使用 React.default 来获取实际的 React 对象。
2. **React 用于 Rollup**
rollup-plugin-node-resolve 插件可以解决 ES6 模块的查找导入，如果npm中的包以CommonJS模块的形式出现的，我们可以使用rollup-plugin-commonjs 将CommonJS模块转换为ES6来为Rollup获得兼容（即令(ES6)import === (CommonJS)require），导入的结果其实是不含 default 属性的对象，因此直接使用 React 来获取实际的 React 对象。

在这个入口 JS 的上方我们可以找到 React 的来源：`const React = require('./src/React');`，我们来看一下这块儿的实现，它定义在`packages/react/src/React.js` 中，

```js
import ReactVersion from 'shared/ReactVersion';
import {
  REACT_ASYNC_MODE_TYPE,
  REACT_FRAGMENT_TYPE,
  REACT_PROFILER_TYPE,
  REACT_STRICT_MODE_TYPE,
  REACT_PLACEHOLDER_TYPE,
} from 'shared/ReactSymbols';
import {enableSuspense} from 'shared/ReactFeatureFlags';

import {Component, PureComponent} from './ReactBaseClasses';
import {createRef} from './ReactCreateRef';
import {forEach, map, count, toArray, only} from './ReactChildren';
import {
  createElement,
  createFactory,
  cloneElement,
  isValidElement,
} from './ReactElement';
import {createContext} from './ReactContext';
import {lazy} from './ReactLazy';
import forwardRef from './forwardRef';
import {
  createElementWithValidation,
  createFactoryWithValidation,
  cloneElementWithValidation,
} from './ReactElementValidator';
import ReactSharedInternals from './ReactSharedInternals';

const React = {
  Children: {
    map,
    forEach,
    count,
    toArray,
    only,
  },

  createRef,
  Component,
  PureComponent,

  createContext,
  forwardRef,

  Fragment: REACT_FRAGMENT_TYPE,
  StrictMode: REACT_STRICT_MODE_TYPE,
  unstable_AsyncMode: REACT_ASYNC_MODE_TYPE,
  unstable_Profiler: REACT_PROFILER_TYPE,

  createElement: __DEV__ ? createElementWithValidation : createElement,
  cloneElement: __DEV__ ? cloneElementWithValidation : cloneElement,
  createFactory: __DEV__ ? createFactoryWithValidation : createFactory,
  isValidElement: isValidElement,

  version: ReactVersion,

  __SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED: ReactSharedInternals,
};

if (enableSuspense) {
  React.Placeholder = REACT_PLACEHOLDER_TYPE;
  React.lazy = lazy;
}

export default React;
```
上述就是我们 React 的庐山真面目，实际上它的内容是一个标准的 JSON 对象，这里 React 对象里面包含什么一目了然，比如我们常用的Component、PureComponent等，由此可以看出React核心内容只包括定义组件相关的内容和API。

#### 渲染

React 的定位是一个构建用户界面的JavaScript类库，它使用JavaScript语言开发UI组件，可以使用多种方式渲染这些组件，输出用户界面，很大程度上达到了跨平台的能力：
> We don’t make assumptions about the rest of your technology stack, so you can develop new features in React without rewriting existing code.

现在的 React 在以下几个方面都发挥着很不错的效果：
1. React Web应用用户界面开发；
2. React Native App用户界面开发；
3. Node.js 服务端渲染；

在这些不同场景，渲染的主体很明显是不一样的，有诸如web应用的DOM渲染，React Native的原生View渲染，服务端字符串渲染等，要做到兼容适应多种不同渲染环境，很显然，React不能局限固定渲染UI的方式。

上一节我们讲到React核心内容只涉及如何定义组件，并不涉及具体的组件渲染（即输出用户界面），这需要引入额外渲染模块，下面以渲染React定义的组件为例：

**React DOM渲染模块：**React DOM渲染模块：将React组件渲染为DOM，然后可以被浏览器处理呈现给用户，这就是通常在web应用中引入的react-dom模块：
``` js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';

ReactDOM.render(<App />, document.getElementById('root'));
```
上述代码中，App是使用React核心模块定义的组件，然后使用react-dom渲染模块提供的render方法将其渲染为DOM输出至页面。

**React Native 渲染：**：将React组件渲染为移动端原生View，在React Native应用中引入react-native模块，它提供相应渲染方法可以渲染React组件：
``` js
import { AppRegistry } from 'react-native';
import App from './src/app.js';

AppRegistry.registerComponent('fuc', () => App);
```
上述代码中，App是React根组件，使用react-native渲染器的AppRegistry.registerComponent方法将其渲染为原生View。

**React测试渲染：**将React组件渲染为JSON树，用来完成Jest的快照测试，内容在react-test-renderer模块：
``` js
import ReactTestRenderer from 'react-test-renderer'; 
const renderer = ReactTestRenderer.create(
  <Link page="https://www.facebook.com/">Facebook</Link>
); 
console.log(renderer.toJSON()); 
// { type: 'a',
//   props: { href: 'https://www.facebook.com/' },
//   children: [ 'Facebook' ] }

```
#### 小结

那么至此，我们应该对 React 是什么有一个直观的认识，它本质上是含有诸多属性的JavaScript对象，它核心内容只涉及如何定义组件，具体的组件渲染（即输出用户界面），需要引入额外的渲染模块，渲染组件方式由环境决定，定义组件，组件状态管理，生命周期方法管理，组件更新等应该跨平台一致处理，不受渲染环境影响，这部分内容统一由调和器（Reconciler）处理，不同渲染器都会使用该模块。调和器主要作用就是在组件状态变更时，调用组件树各组件的render方法，渲染，卸载组件。至于 React 能做什么，它是怎么做的，我们会在后面的章节一一剖析它们。

## 基本概念

在正式进入流程分析之前，我们先来了解一些 React 源码内部的基本概念，读懂这些有助于我们更好地理解整个流程。

### ReactElement
我们在写 React 组件时，通常会使用JSX来描述组件，经过babel（Facebook早期有提供自己的编译器，后来Babel发展为社区jsx语法的主要编译工具）编译成对应的`React.createElement(type, props, children)`形式。

将以下代码放到[Babel](https://babeljs.io/repl/)官网上编译一下：
``` js
class App extends Component {
  render() {
    return (
      <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <p>
            Edit <code>src/App.js</code> and save to reload.
          </p>
          <a
            className="App-link"
            href="https://reactjs.org"
            target="_blank"
            rel="noopener noreferrer"
          >
            Learn React
          </a>
        </header>
        <Hello />
        <Inner text="heiheihei">
          <div>yoyoyo</div>
        </Inner>
      </div>
    );
  }
}
```
经过Babel编译后：
``` js
React.createElement(
  "div",
  { className: "App" },
  React.createElement(
    "header",
    { className: "App-header" },
    React.createElement("img", { src: logo, className: "App-logo", alt: "logo" }),
    React.createElement(
      "p",
      null,
      "Edit ",
      React.createElement(
        "code",
        null,
        "src/App.js"
      ),
      " and save to reload."
    ),
    React.createElement(
      "a",
      {
        className: "App-link",
        href: "https://reactjs.org",
        target: "_blank",
        rel: "noopener noreferrer"
      },
      "Learn React"
    )
  ),
  React.createElement(Hello, null),
  React.createElement(
    Inner,
    { text: "heiheihei" },
    React.createElement(
      "div",
      null,
      "yoyoyo"
    )
  )
);
```
可以看出我们使用JSX来编写的组件会被编译成调用React.createElement的形式。如果组件里的DOM标签的首字母为大写的时候，这个标签（类 => 自定义组件类， 函数 => 无状态组件）则会被作为参数传递给createElement；如果DOM标签的首字母为小写，则将标签名（div, span, a 等html的 DOM标签）以字符串的形式传给createElement；如果是字符串或者空的话，则直接将字符串或者null当做参数传递给createElement。接下来我们React.createElement的源码。

React.createElement的源码：
``` js
export function createElement(type, config, children) {
  let propName;

  // Reserved names are extracted
  const props = {};

  let key = null;
  let ref = null;
  let self = null;
  let source = null;

  // 将参数赋给props对象
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }

    self = config.__self === undefined ? null : config.__self;
    source = config.__source === undefined ? null : config.__source;
    // Remaining properties are added to a new props object
    for (propName in config) {
      // 跳过React保留的参数
      if (
        hasOwnProperty.call(config, propName) &&
        !RESERVED_PROPS.hasOwnProperty(propName)
      ) {
        props[propName] = config[propName];
      }
    }
  }

  // 将子元素按照顺序赋给children的数组
  // Children can be more than one argument, and those are transferred onto
  // the newly allocated props object.
  const childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    const childArray = Array(childrenLength);
    for (let i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (__DEV__) {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }

  // 对于默认的参数，判断是否有传入值，有的话直接将参数和对应的值赋给props，否则将参数和参数默认值赋给props
  // Resolve default props
  if (type && type.defaultProps) {
    const defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
  if (__DEV__) {
    if (key || ref) {
      const displayName =
        typeof type === 'function'
          ? type.displayName || type.name || 'Unknown'
          : type;
      if (key) {
        defineKeyPropWarningGetter(props, displayName);
      }
      if (ref) {
        defineRefPropWarningGetter(props, displayName);
      }
    }
  }
  return ReactElement(
    type,
    key,
    ref,
    self,
    source,
    ReactCurrentOwner.current,
    props,
  );
}

const ReactElement = function(type, key, ref, self, source, owner, props) {
  const element = {
    // Symbol类型的tag唯一标示这个对象是一个React Element类型
    // This tag allows us to uniquely identify this as a React Element
    $$typeof: REACT_ELEMENT_TYPE,

    // Built-in properties that belong on the element
    type: type,
    key: key,
    ref: ref,
    props: props,

    // Record the component responsible for creating this element.
    _owner: owner,
  };

  return element;
};
```
我们可以看出createElement基本没做什么特别的处理，最终返回了一个ReactElement对象。
ReactElement是描述DOM节点或component实例的字面级对象，ReactElement主要包含了对象类型标识（$$typeof）、DOM节点的类型（type）和属性（props）。

| key      | type        | desc|
| ------   | ------     |  ------     | 
| $$typeof | Symbol &nbsp;&nbsp;&nbsp;&#124;&nbsp;&nbsp;&nbsp; Number | 对象类型标识，用于判断当前Object属于哪一种类型的ReactElement |
| type     | Function &nbsp;&nbsp;&nbsp;&#124;&nbsp;&nbsp;&nbsp; String &nbsp;&nbsp;&nbsp;&#124;&nbsp;&nbsp;&nbsp; Symbol &nbsp;&nbsp;&nbsp;&#124;&nbsp;&nbsp;&nbsp; Number &nbsp;&nbsp;&nbsp;&#124;&nbsp;&nbsp;&nbsp; Object | 如果当前ReactElement是一个ReactComponent，那这里将是它对应的Constructor；而普通HTML标签，一般都是String|
| props     | Object | ReactElement上的所有属性，包含children这个特殊属性|

ReactElement只是保存了DOM需要的数据，并没有对应的方法来实现React提供给我们的那些功能和特性。ReactElement主要分为DOM Elements和Component Elements两种，我们称这样的对象为ReactElement。

**DOM Elements**
当节点的type属性为字符串时，它代表是普通的节点，如div，span：
``` json
{
  type: 'button',
  props: {
    className: 'button button-blue',
    children: {
      type: 'b',
      props: {
        children: 'OK!'
      }
    }
  }
}
```
**Component Elements**
当节点的type属性为一个函数或一个类时，它代表自定义的节点：
``` js
class Button extends React.Component {
  render() {
    const { children, color } = this.props;
    return {
      type: 'button',
      props: {
        className: 'button button-' + color,
        children: {
          type: 'b',
          props: {
            children: children
          }
        }
      }
    };
  }
}
// Component Elements
{
  type: Button,
  props: {
    color: 'blue',
    children: 'OK!'
  }
}
```

### ReactClass
ReactClass就是我们平时写的Component组件(类或函数)，例如上面的Button类。ReactClass实例化后调用render方法可返回DOM Element。

### ReactComponent
ReactComponent是基于ReactElement创建的一个对象，这个对象保存了ReactElement的数据的同时注入了一些方法，这些方法可以用来实现我们熟知的那些React的特性。

### ReactRoot

## 主要概念

### 首次渲染

#### 渲染入口
在 Web 项目中，如果我们要将应用渲染至页面，通常会用如下代码：

``` typescript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App'; // 应用根组件

ReactDOM.render(<App />, document.getElementById('root')); // 应用挂载容器DOM
```

这里`react-dom`是浏览器端渲染React应用的模块，通过`ReactDOM.render(component, mountNode)`可以对`自定义组件/原生DOM/字符串`进行挂载。在React16中，虽然还是通过JSX编译得到一个虚拟DOM对象，但对这些虚拟DOM对象的再加工则是发生了翻天覆地的变化。我们需要追根溯底，看它是怎么一步步转换的。我们首先找到`ReactDOM.render`，源码在`packages/react-dom/src/client/ReactDOM.js`中，有三个类似的方法：

``` typescript
const ReactDOM: Object = {
  // 新API，未来代替render
  hydrate(element: React$Node, container: DOMContainer, callback: ?Function) {
    // TODO: throw or warn if we couldn't hydrate?
    return legacyRenderSubtreeIntoContainer(
      null,
      element,
      container,
      true,
      callback,
    );
  },
  // React15的重要API，逐渐退出舞台
  render(
    element: React$Element<any>,  // react组件对象，通常是项目根组件
    container: DOMContainer, // id为root的那个dom
    callback: ?Function, // 回调函数
  ) {
    return legacyRenderSubtreeIntoContainer(
      null,
      element,
      container,
      false,
      callback,
    );
  },
  // 将组件挂载到传入的 DOM 节点上（不稳定api）
  unstable_renderSubtreeIntoContainer(
    parentComponent: React$Component<any, any>,
    element: React$Element<any>,
    containerNode: DOMContainer,
    callback: ?Function,
  ) {
    invariant(
      parentComponent != null && ReactInstanceMap.has(parentComponent),
      'parentComponent must be a valid React Component',
    );
    return legacyRenderSubtreeIntoContainer(
      parentComponent,
      element,
      containerNode,
      false,
      callback,
    );
  },
};
```
这里`ReactDOM.render/hydrate/unstable_renderSubtreeIntoContainer/unmountComponentAtNode`都是`legacyRenderSubtreeIntoContainer`方法的加壳方法。因此`ReactDOM.render`实际调用了`legacyRenderSubtreeIntoContainer`，这是一个内部API。

#### 渲染虚拟dom树
`legacyRenderSubtreeIntoContainer`从字面可以看出它大致意思就是把虚拟的dom树渲染到真实的dom容器中，我们找到`legacyRenderSubtreeIntoContainer`方法，源码在`packages/react-dom/src/client/ReactDOM.js`中：
``` typescript
// 渲染组件的子组件树至父容器
function legacyRenderSubtreeIntoContainer(
  parentComponent: ?React$Component<any, any>, // 父组件 这里为 null
  children: ReactNodeList, // element 虚拟dom树
  container: DOMContainer, // html中的dom根对象
  forceHydrate: boolean, // 服务器端渲染标识 这里为false
  callback: ?Function, // 回调函数 这里没有
) {
  // 对 container 进行校验
  invariant(
    isValidContainer(container),
    'Target container is not a DOM element.',
  );

  if (__DEV__) {
    // 开发模式render时进行检查并提供许多有用的警告和错误提示信息
    topLevelUpdateWarnings(container);
  }

  // 获取 root 对象
  let root: Root = (container._reactRootContainer: any);
  if (!root) { // 初次渲染时初始化
    // 创建一个 FiberRoot对象 并将它缓存到DOM容器的_reactRootContainer属性
    root = container._reactRootContainer = legacyCreateRootFromDOMContainer(
      container, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
      forceHydrate, // 服务器端渲染标识 这里为false
    );
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    // 初始化容器相关
    // Initial mount should not be batched.
    DOMRenderer.unbatchedUpdates(() => {
      if (parentComponent != null) {
        // 向真实dom中挂载虚拟dom
        root.legacy_renderSubtreeIntoContainer(
          parentComponent, // 父组件
          children, // 虚拟dom树
          callback, // 回调函数
        );
      } else {
        root.render(
          children, // 虚拟dom树
          callback // 回调函数
        );
      }
    });
  } else {
    if (typeof callback === 'function') {
      const originalCallback = callback;
      callback = function() {
        const instance = DOMRenderer.getPublicRootInstance(root._internalRoot);
        originalCallback.call(instance);
      };
    }
    if (parentComponent != null) {
      root.legacy_renderSubtreeIntoContainer(
        parentComponent, // 父组件
        children, // 虚拟dom树
        callback, // 回调函数
      );
    } else {
      root.render(
        children, // 虚拟dom树
        callback // 回调函数
      );
    }
  }
  // 返回根容器fiber树的根fiber实例
  return DOMRenderer.getPublicRootInstance(root._internalRoot);
}

// 源码在 packages/react-reconciler/src/ReactFiberReconciler.js 中
export function getPublicRootInstance(
  container: OpaqueRoot,
): React$Component<any, any> | PublicInstance | null {
  // 获取fiber实例
  const containerFiber = container.current;
  if (!containerFiber.child) {
    return null;
  }
  switch (containerFiber.child.tag) {
    case HostComponent:
      return getPublicInstance(containerFiber.child.stateNode);
    default:
      return containerFiber.child.stateNode;
  }
}

```
由此可见，`legacyRenderSubtreeIntoContainer`主要执行了以下几个操作：
**root：**由`legacyCreateRootFromDOMContainer`生成，该函数会生成一个`FiberRoot`对象挂载到真实的dom根节点上，有了这个对象，执行该对象上的一些方法可以将虚拟dom变成dom树挂载到根节点上。
**DOMRenderer.unbatchedUpdates：**`DOMRenderer.unbatchedUpdates`的回调执行`root.legacy_renderSubtreeIntoContainer`或`root.render`。
**root.legacy_renderSubtreeIntoContainer 和 root.render：**如果有`parentComponent`，就执行`root.render`否则执行`root.legacy_renderSubtreeIntoContainer`。

##### root
我们知道`root`是由`legacyCreateRootFromDOMContainer`生成的，我们找到`legacyCreateRootFromDOMContainer`函数，源码在`packages/react-dom/src/client/ReactDOM.js`中:
``` typescript
function legacyCreateRootFromDOMContainer(
  container: DOMContainer, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
  forceHydrate: boolean,  // 服务器端渲染标识 这里为false
): Root {
  const shouldHydrate =
    forceHydrate || shouldHydrateDueToLegacyHeuristic(container);
  // 是否需要服务器端渲染
  if (!shouldHydrate) {
    let warned = false;
    let rootSibling;
    while ((rootSibling = container.lastChild)) {
      if (__DEV__) {
        // ...
      }
      // 将dom根节点清空
      container.removeChild(rootSibling);
    }
  }
  if (__DEV__) {
    // ...
  }
  // Legacy roots are not async by default.
  const isAsync = false;
  return new ReactRoot(
    container, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
    isAsync, // 是否异步模式，默认false
    shouldHydrate // 服务器端渲染标识 这里为false
  );
}
```
我们发现该函数实际上返回的是由构造函数`ReactRoot`创建的对象。其中如果在非ssr的情况下，将dom根节点清空。我们找到构造函数`ReactRoot`，源码在`packages\react-dom\src\client\ReactDOM.js`中：
``` typescript
// 构造函数
function ReactRoot(
  container: Container, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
  isAsync: boolean, // 是否异步模式，默认false
  hydrate: boolean // 服务器端渲染标识 这里为false
) {
  // FiberRoot 对象
  const root = DOMRenderer.createContainer(
    container, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
    isAsync, // 是否异步模式，默认false
    hydrate // 服务器端渲染标识 这里为false
  );
  this._internalRoot = root;
}

// 以下几个是原型方法
// 渲染
ReactRoot.prototype.render = function(
  children: ReactNodeList, // 虚拟dom树
  callback: ?() => mixed, // 回调函数
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
  if (__DEV__) {
    warnOnInvalidCallback(callback, 'render');
  }
  if (callback !== null) {
    work.then(callback);
  }
  DOMRenderer.updateContainer(
    children, // 虚拟dom树
    root, // FiberRoot 对象
    null, // 父组件 这里为 null
    work._onCommit
  );
  return work;
};

// 销毁组件
ReactRoot.prototype.unmount = function(callback: ?() => mixed): Work {
  // ...
};
ReactRoot.prototype.legacy_renderSubtreeIntoContainer = function(
  parentComponent: ?React$Component<any, any>,
  children: ReactNodeList,
  callback: ?() => mixed,
): Work {
  const root = this._internalRoot;
  const work = new ReactWork();
  callback = callback === undefined ? null : callback;
  if (__DEV__) {
    warnOnInvalidCallback(callback, 'render');
  }
  if (callback !== null) {
    work.then(callback);
  }
  DOMRenderer.updateContainer(
    children, // 虚拟dom树
    root, // FiberRoot 对象
    parentComponent, // 父组件
    work._onCommit
  );
  return work;
};
ReactRoot.prototype.createBatch = function(): Batch {
  // ...
};
```
可以看出构造函数`ReactRoot`有render、unmount、legacy_renderSubtreeIntoContainer等原型方法外，同时还声明了一个和fiber相关的`_internalRoot`属性。其中`render`和`legacy_renderSubtreeIntoContainer`原型方法都会去执行`DOMRenderer.updateContainer`方法更新容器内容，唯一差别就是第三个参数一个传`null`，一个传`parentComponent`。`_internalRoot`是由`DOMRenderer.createContainer`生成的。我们找到`DOMRenderer.createContainer`，源码在`packages\react-reconciler\src\ReactFiberReconciler.js`中：
``` js
export function createContainer(
  containerInfo: Container, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
  isAsync: boolean, // 是否异步模式，默认false
  hydrate: boolean, // 服务器端渲染标识 这里为false
): OpaqueRoot {
  return createFiberRoot(
    containerInfo, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
    isAsync, // 是否异步模式，默认false
    hydrate // 服务器端渲染标识 这里为false
  );
}
```
接下来我们看看`createFiberRoot`是怎么将一个真实DOM变成一个Fiber对象，我们找到`createFiberRoot`，源码在 packages\react-reconciler\src\ReactFiberReconciler.js 中：
``` typescript
export function createFiberRoot(
  containerInfo: any, // ReactDOM.render(<div/>, container)的第二个参数，也就是一个元素节点
  isAsync: boolean, // 是否异步模式，默认false
  hydrate: boolean, // 服务器端渲染标识 这里为false
): FiberRoot {
  // 创建初始根组件对应的fiber实例
  const uninitializedFiber = createHostRootFiber(isAsync);

  let root;
  if (enableSchedulerTracing) {
    root = ({
      current: uninitializedFiber,
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,

      interactionThreadID: unstable_getThreadID(),
      memoizedInteractions: new Set(),
      pendingInteractionMap: new Map(),
    }: FiberRoot);
  } else {
    root = ({
      current: uninitializedFiber,
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,
    }: BaseFiberRootProperties);
  }
  uninitializedFiber.stateNode = root;
  return ((root: any): FiberRoot);
}

// 源码在 packages\react-reconciler\src\ReactFiber.js 中
// 返回一个初始根组件对应的fiber实例
export function createHostRootFiber(isAsync: boolean): Fiber {
  let mode = isAsync ? AsyncMode | StrictMode : NoContext;

  if (enableProfilerTimer && isDevToolsPresent) {
    // Always collect profile timings when DevTools are present.
    // This enables DevTools to start capturing timing at any point–
    // Without some nodes in the tree having empty base times.
    mode |= ProfileMode;
  }

  // 创建 Fiber 实例
  return createFiber(
    HostRoot, // 组件树根组件，可以嵌套
    null, 
    null, 
    mode
  );
}

// 源码在 packages\react-reconciler\src\ReactFiber.js 中
// 创建 Fiber 实例
const createFiber = function(
  tag: WorkTag, // 标记 fiber 类型
  pendingProps: mixed, // 当前处理过程中的组件props对象
  key: null | string, // 调和阶段，标识fiber，以检测是否可重用该fiber实例
  mode: TypeOfMode,
): Fiber {
  // $FlowFixMe: the shapes are exact here but Flow doesn't like constructors
  return new FiberNode(tag, pendingProps, key, mode);
};

// 源码在 packages\react-reconciler\src\ReactFiber.js 中
function FiberNode(
  tag: WorkTag,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
) {
  // Instance
  this.tag = tag;
  this.key = key;
  this.type = null;
  this.stateNode = null;

  // Fiber
  this.return = null;
  this.child = null;
  this.sibling = null;
  this.index = 0;

  this.ref = null;

  this.pendingProps = pendingProps;
  this.memoizedProps = null;
  this.updateQueue = null;
  this.memoizedState = null;
  this.firstContextDependency = null;

  this.mode = mode;

  // Effects
  this.effectTag = NoEffect;
  this.nextEffect = null;

  this.firstEffect = null;
  this.lastEffect = null;

  this.expirationTime = NoWork;
  this.childExpirationTime = NoWork;

  this.alternate = null;

  if (enableProfilerTimer) {
    this.actualDuration = 0;
    this.actualStartTime = -1;
    this.selfBaseDuration = 0;
    this.treeBaseDuration = 0;
  }

  if (__DEV__) {
    this._debugID = debugCounter++;
    this._debugSource = null;
    this._debugOwner = null;
    this._debugIsCurrentlyTiming = false;
    if (!hasBadMapPolyfill && typeof Object.preventExtensions === 'function') {
      Object.preventExtensions(this);
    }
  }
}
```
由此可知，`react-dom`渲染模块调用`createContainer`创建容器、根fiber实例、FiberRoot对象等。所有`Fiber`对象都是`FiberNode`的实例，它有许多种类型，通过tag来标识，其中内部有很多方法来生成Fiber对象：
* createFiberFromElement：type为类，无状态函数，元素标签名
* createFiberFromFragment：type为React.Fragment
* createFiberFromText：在JSX中表现为字符串，数字
* createFiberFromPortal：用于 createPortal
* createFiberRoot：用于ReactDOM.render的根节点

这里`createFiberRoot`就是创建了一个普通对象，里面`current`属性引用`fiber`对象，`containerInfo`属性引用`ReactDOM.render(<div/>, container)`的第二个参数，也就是一个元素节点，然后`fiber`对象的`stateNode`引用普通对象`root`。在React15中，`stateNode`应该是一个组件实例或真实DOM，最后返回普通对象`stateNode`。现在我们回顾下调用`reactDOM.render`传入的`container`，在执行过程中附加了哪些有用的东西：
``` js
container = { // 就是我们传入的那个真实dom
  _reactRootContainer: { // legacyCreateRootFromDOMContainer
    _internalRoot: { // DOMRenderer.createContainer
      current:{}  // new FiberNode
    }
  }
} 
```

##### unbatchedUpdates
我们找到`DOMRenderer.unbatchedUpdates`，源码在`packages\react-reconciler\src\ReactFiberScheduler.js`中：
``` js
// 正在批量更新标识
let isBatchingUpdates: boolean = false;
// 未批量更新标识
let isUnbatchingUpdates: boolean = false;
// 非批量更新操作
function unbatchedUpdates<A, R>(fn: (a: A) => R, a: A): R {
  // 如果正在批量更新
  if (isBatchingUpdates && !isUnbatchingUpdates) {
    // 未批量更新设为true
    isUnbatchingUpdates = true;
    try {
      // 运行入参函数且返回执行结果
      return fn(a);
    } finally {
      // 仍旧将未批量更新设为false
      isUnbatchingUpdates = false;
    }
  }
  // 不管是否在批量更新流程中，都执行入参函数
  return fn(a);
}
```
由此可知`unbatchedUpdates`无论如何都会执行入参函数，其中`isBatchingUpdates`和`isUnbatchingUpdates`初始值都是false。`DOMRenderer.unbatchedUpdates`的回调执行`root.legacy_renderSubtreeIntoContainer`或`root.render`。

#### 更新容器内容
从`legacyRenderSubtreeIntoContainer`函数里可以看出，无论怎样判断，最终都会到`root.legacy_renderSubtreeIntoContainer`和`root.render`两个方法，而这两个方法的核心就是`DOMRenderer.updateContainer`，无非就是传不传父组件这点区别。我们找到`DOMRenderer.updateContainer`，源码在`packages\react-reconciler\src\ReactFiberReconciler.js`中：
``` typescript
export function updateContainer(
  element: ReactNodeList, // ReactDOM.render函数的第一个参数，泛指各种虚拟DOM
  container: OpaqueRoot, // ReactDOM.render函数的第二个参数，也就是一个元素节点
  parentComponent: ?React$Component<any, any>, // parentComponent为之前的根组件，现在它为null
  callback: ?Function, // 回调函数
): ExpirationTime {
  // createFiberRoot中创建的fiber对象
  const current = container.current;
  const currentTime = requestCurrentTime();
  // 获取任务到期时间
  const expirationTime = computeExpirationForFiber(currentTime, current);
  return updateContainerAtExpirationTime(
    element, // ReactDOM.render函数的第一个参数，泛指各种虚拟DOM
    container, // ReactDOM.render函数的第二个参数，也就是一个元素节点
    parentComponent, // 父组件
    expirationTime, // 任务到期时间
    callback, // 回调函数
  );
}

// 源码在 packages\react-reconciler\src\ReactFiberScheduler.js 中
// 计算fiber的到期时间
function computeExpirationForFiber(currentTime: ExpirationTime, fiber: Fiber) {
  let expirationTime;
  if (expirationContext !== NoWork) {
    // 显示设置过期上下文
    expirationTime = expirationContext;
  } else if (isWorking) {
    if (isCommitting) {
      // 在提交阶段的更新任务
      // 需要明确设置同步优先级（Sync Priority）
      expirationTime = Sync;
    } else {
      // 在渲染阶段发生的更新任务
      // 需要设置为下一次渲染时间的到期时间优先级
      expirationTime = nextRenderExpirationTime;
    }
  } else {
    // 不在任务执行阶段，需要计算新的过期时间
    if (fiber.mode & AsyncMode) {
      if (isBatchingInteractiveUpdates) {
        // This is an interactive update
        expirationTime = computeInteractiveExpiration(currentTime);
      } else {
        // 异步更新
        expirationTime = computeAsyncExpiration(currentTime);
      }
      // 如果我们正处于渲染树的中间, 请不要在已经呈现的相同过期时间内更新。
      if (nextRoot !== null && expirationTime === nextRenderExpirationTime) {
        expirationTime += 1;
      }
    } else {
      // 同步更新
      expirationTime = Sync;
    }
  }
  if (isBatchingInteractiveUpdates) {
    // This is an interactive update. Keep track of the lowest pending
    // interactive expiration time. This allows us to synchronously flush
    // all interactive updates when needed.
    if (expirationTime > lowestPriorityPendingInteractiveExpirationTime) {
      lowestPriorityPendingInteractiveExpirationTime = expirationTime;
    }
  }
  return expirationTime;
}

// 根据渲染优先级更新dom
export function updateContainerAtExpirationTime(
  element: ReactNodeList, // ReactDOM.render函数的第一个参数，泛指各种虚拟DOM
  container: OpaqueRoot, // ReactDOM.render函数的第二个参数，也就是一个元素节点
  parentComponent: ?React$Component<any, any>,   // parentComponent为之前的根组件，现在它为null
  expirationTime: ExpirationTime, // 期望的任务到期时间
  callback: ?Function, // 回调函数
) {
  // TODO: If this is a nested container, this won't be the root.
  // 引用fiber对象
  const current = container.current;

  if (__DEV__) {
    // ...
  }

  // 获得上下文对象
  const context = getContextForSubtree(parentComponent);
  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }
  // 下一步：schedule:安排, Root: 根, Update:更新
  return scheduleRootUpdate(
    current, // fiber对象
    element, // ReactDOM.render函数的第一个参数，泛指各种虚拟DOM
    expirationTime, // 期望的任务到期时间
    callback // 回调函数
  );
}

// 源码在 packages\react-reconciler\src\ReactFiberReconciler.js 中
// 获得上下文对象
function getContextForSubtree(
  parentComponent: ?React$Component<any, any>,
): Object {
  if (!parentComponent) {
    return emptyContextObject;
  }

  const fiber = ReactInstanceMap.get(parentComponent);
  const parentContext = findCurrentUnmaskedContext(fiber);

  if (fiber.tag === ClassComponent) {
    const Component = fiber.type;
    if (isLegacyContextProvider(Component)) {
      return processChildContext(fiber, Component, parentContext);
    }
  } else if (fiber.tag === ClassComponentLazy) {
    const Component = getResultFromResolvedThenable(fiber.type);
    if (isLegacyContextProvider(Component)) {
      return processChildContext(fiber, Component, parentContext);
    }
  }

  return parentContext;
}
```
`updateContainer`的源码很简单，通过`computeExpirationForFiber`获得计算优先级，然后丢给`updateContainerAtExpirationTime`，这里`updateContainerAtExpirationTime`其实相当于什么都没做，通过`getContextForSubtree`（这里`getContextForSubtree`因为一开始`parentComponent`是不存在的，于是返回一个空对象。注意，这个空对象可以重复使用，不用每次返回一个新的空对象，这是一个很好的优化）获得上下文对象，然后分配给`container.context`或`container.pendingContext`，最后一起丢给`scheduleRootUpdate`。

#### 开始更新
我们找到`scheduleRootUpdate`，源码在`packages/react-reconciler/src/ReactFiberReconciler.js`中：
``` typescript
// 进行根节点更新
function scheduleRootUpdate(
  current: Fiber, // 引用fiber对象
  element: ReactNodeList, // 虚拟dom树
  expirationTime: ExpirationTime, // 任务到期时间
  callback: ?Function, // 回调函数
) {
  if (__DEV__) {
    // ...
  }
  
  // 返回一个包含以上属性的update对象
  const update = createUpdate(expirationTime);
  // Caution: React DevTools currently depends on this property
  // being called "element".
  // 将虚拟dom树放入payload 
  update.payload = {element};

  callback = callback === undefined ? null : callback;
  if (callback !== null) {
    warningWithoutStack(
      typeof callback === 'function',
      'render(...): Expected the last optional `callback` argument to be a ' +
        'function. Instead received: %s.',
      callback,
    );
    update.callback = callback;
  }
  // 开始队列更新
  enqueueUpdate(current, update); 
  // 调用调度器API：scheduleWork(...)来调度fiber任务
  scheduleWork(
    current, // fiber实例
    expirationTime // 任务到期时间
  );
  return expirationTime;
}

// 创建一个包含以上属性的update对象
export function createUpdate(expirationTime: ExpirationTime): Update<*> {
  return {
    expirationTime: expirationTime,

    tag: UpdateState,
    payload: null,
    callback: null,

    next: null,
    nextEffect: null,
  };
}
```
`scheduleRootUpdate`是将用户的传参封装成一个`update`对象, 其中`update`对象有`payload`对象，它就是相当于React15中 的setState的第一个state传参，但现在`payload`中把`children`也放进去了。然后添加更新任务至fiber：`enqueueUpdate(...)`，现在我们找到`enqueueUpdate`，源码在`packages/react-reconciler/src/ReactUpdateQueue.js`中：
``` typescript
export function enqueueUpdate<State>(fiber: Fiber, update: Update<State>) {
  // alternate 主要用来保存更新过程中各版本更新队列，方便崩溃或冲突时回退
  const alternate = fiber.alternate;
  // 创建两个独立的更新队列
  let queue1;
  let queue2;
  if (alternate === null) {
    // 只存在一个 fiber
    queue1 = fiber.updateQueue;
    queue2 = null;
    if (queue1 === null) {
      // 如果不存在，则创建一个更新队列
      queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
    }
  } else {
    // 两个所有者
    queue1 = fiber.updateQueue;
    queue2 = alternate.updateQueue;
    if (queue1 === null) {
      if (queue2 === null) {
        // 如果两个都不存在，则创建两个新的
        queue1 = fiber.updateQueue = createUpdateQueue(fiber.memoizedState);
        queue2 = alternate.updateQueue = createUpdateQueue(
          alternate.memoizedState,
        );
      } else {
        // queue1 不存在，queue2 存在，queue1 根据 queue2 创建
        queue1 = fiber.updateQueue = cloneUpdateQueue(queue2);
      }
    } else {
      if (queue2 === null) {
        // queue2 不存在，queue1 存在，queue2 根据 queue1 创建
        queue2 = alternate.updateQueue = cloneUpdateQueue(queue1);
      } else {
        // 全都有
      }
    }
  }
  if (queue2 === null || queue1 === queue2) {
    // 只存在一个更新队列
    appendUpdateToQueue(queue1, update);
  } else {
    // 如果任意更新队列为空，则需要将更新添加至两个更新队列
    if (queue1.lastUpdate === null || queue2.lastUpdate === null) {
      appendUpdateToQueue(queue1, update);
      appendUpdateToQueue(queue2, update);
    } else {
      // 如果2个更新队列均非空，则添加更新至第一个队列，并更新另一个队列的尾部更新项
      appendUpdateToQueue(queue1, update);
      queue2.lastUpdate = update;
    }
  }

  if (__DEV__) {
    if (
      (fiber.tag === ClassComponent || fiber.tag === ClassComponentLazy) &&
      (currentlyProcessingQueue === queue1 ||
        (queue2 !== null && currentlyProcessingQueue === queue2)) &&
      !didWarnUpdateInsideUpdate
    ) {
      warningWithoutStack(
        false,
        'An update (setState, replaceState, or forceUpdate) was scheduled ' +
          'from inside an update function. Update functions should be pure, ' +
          'with zero side-effects. Consider using componentDidUpdate or a ' +
          'callback.',
      );
      didWarnUpdateInsideUpdate = true;
    }
  }
}

// 创建一个更新队列
export function createUpdateQueue<State>(baseState: State): UpdateQueue<State> {
  const queue: UpdateQueue<State> = {
    baseState,
    firstUpdate: null,
    lastUpdate: null,
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,
    firstEffect: null,
    lastEffect: null,
    firstCapturedEffect: null,
    lastCapturedEffect: null,
  };
  return queue;
}

// clone 一个更新队列
function cloneUpdateQueue<State>(
  currentQueue: UpdateQueue<State>,
): UpdateQueue<State> {
  const queue: UpdateQueue<State> = {
    baseState: currentQueue.baseState,
    firstUpdate: currentQueue.firstUpdate,
    lastUpdate: currentQueue.lastUpdate,

    // TODO: With resuming, if we bail out and resuse the child tree, we should
    // keep these effects.
    firstCapturedUpdate: null,
    lastCapturedUpdate: null,

    firstEffect: null,
    lastEffect: null,

    firstCapturedEffect: null,
    lastCapturedEffect: null,
  };
  return queue;
}

// 更新队列
function appendUpdateToQueue<State>(
  queue: UpdateQueue<State>,
  update: Update<State>,
) {
  // Append the update to the end of the list.
  if (queue.lastUpdate === null) {
    // Queue is empty
    queue.firstUpdate = queue.lastUpdate = update;
  } else {
    queue.lastUpdate.next = update;
    queue.lastUpdate = update;
  }
}
```
这里`enqueueUpdate`是一个链表，然后根据`fiber`的状态创建一个或两个列队对象，再接下来调用调度器API：`scheduleWork(...)`来调度fiber任务，现在我们看一下如何处理更新的。

#### 处理更新
我们找到`scheduleWork`，源码在`packages/react-reconciler/src/ReactFiberScheduler.js`中：
``` typescript
function scheduleWork(fiber: Fiber, expirationTime: ExpirationTime) {
  // 记录调度器的执行状态
  recordScheduleUpdate();

  if (__DEV__) {
    if (fiber.tag === ClassComponent || fiber.tag === ClassComponentLazy) {
      const instance = fiber.stateNode;
      warnAboutInvalidUpdates(instance);
    }
  }

  const root = scheduleWorkToRoot(fiber, expirationTime);
  if (root === null) {
    if (
      __DEV__ &&
      (fiber.tag === ClassComponent || fiber.tag === ClassComponentLazy)
    ) {
      warnAboutUpdateOnUnmounted(fiber);
    }
    return;
  }

  if (enableSchedulerTracing) {
    const interactions = __interactionsRef.current;
    if (interactions.size > 0) {
      const pendingInteractionMap = root.pendingInteractionMap;
      const pendingInteractions = pendingInteractionMap.get(expirationTime);
      if (pendingInteractions != null) {
        interactions.forEach(interaction => {
          if (!pendingInteractions.has(interaction)) {
            // Update the pending async work count for previously unscheduled interaction.
            interaction.__count++;
          }

          pendingInteractions.add(interaction);
        });
      } else {
        pendingInteractionMap.set(expirationTime, new Set(interactions));

        // Update the pending async work count for the current interactions.
        interactions.forEach(interaction => {
          interaction.__count++;
        });
      }

      const subscriber = __subscriberRef.current;
      if (subscriber !== null) {
        const threadID = computeThreadID(
          expirationTime,
          root.interactionThreadID,
        );
        subscriber.onWorkScheduled(interactions, threadID);
      }
    }
  }

  if (
    !isWorking &&
    nextRenderExpirationTime !== NoWork &&
    expirationTime < nextRenderExpirationTime
  ) {
    // This is an interruption. (Used for performance tracking.)
    interruptedBy = fiber;
    resetStack();
  }
  markPendingPriorityLevel(root, expirationTime);
  if (
    // If we're in the render phase, we don't need to schedule this root
    // for an update, because we'll do it before we exit...
    !isWorking ||
    isCommitting ||
    // ...unless this is a different root than the one we're rendering.
    nextRoot !== root
  ) {
    const rootExpirationTime = root.expirationTime;
    requestWork(root, rootExpirationTime);
  }
  if (nestedUpdateCount > NESTED_UPDATE_LIMIT) {
    // Reset this back to zero so subsequent updates don't throw.
    nestedUpdateCount = 0;
    invariant(
      false,
      'Maximum update depth exceeded. This can happen when a ' +
        'component repeatedly calls setState inside ' +
        'componentWillUpdate or componentDidUpdate. React limits ' +
        'the number of nested updates to prevent infinite loops.',
    );
  }
}

function scheduleWorkToRoot(fiber: Fiber, expirationTime): FiberRoot | null {
  // 更新 fiber实例的过期时间
  if (
    fiber.expirationTime === NoWork ||
    fiber.expirationTime > expirationTime
  ) {
    // 若fiber实例到期时间大于期望的任务到期时间，则更新fiber到期时间
    fiber.expirationTime = expirationTime;
  }
  let alternate = fiber.alternate;
  // 同时更新alternate fiber的到期时间
  if (
    alternate !== null &&
    (alternate.expirationTime === NoWork ||
      alternate.expirationTime > expirationTime)
  ) {
    // 若alternate fiber到期时间大于期望的任务到期时间，则更新fiber到期时间
    alternate.expirationTime = expirationTime;
  }
  let node = fiber.return;
  // fiber.return 为空，说明到达组件树顶部
  if (node === null && fiber.tag === HostRoot) {
    // 确保是组件树根组件并获取FiberRoot实例
    return fiber.stateNode;
  }
  while (node !== null) {
    alternate = node.alternate;
    if (
      node.childExpirationTime === NoWork ||
      node.childExpirationTime > expirationTime
    ) {
      node.childExpirationTime = expirationTime;
      if (
        alternate !== null &&
        (alternate.childExpirationTime === NoWork ||
          alternate.childExpirationTime > expirationTime)
      ) {
        alternate.childExpirationTime = expirationTime;
      }
    } else if (
      alternate !== null &&
      (alternate.childExpirationTime === NoWork ||
        alternate.childExpirationTime > expirationTime)
    ) {
      alternate.childExpirationTime = expirationTime;
    }
    if (node.return === null && node.tag === HostRoot) {
      return node.stateNode;
    }
    node = node.return;
  }
  return null;
}
```
这里`scheduleWork`主要进行虚拟DOM（fiber树）的更新。`scheduleWork`的最开头有一个`recordScheduleUpdate`方法，我们找到`recordScheduleUpdate`，源码在`packages\react-reconciler\src\ReactDebugFiberPerf.js`中：
``` typescript
export function recordScheduleUpdate(): void {
  if (enableUserTimingAPI) { // 全局变量，默认为true
    if (isCommitting) { // 全局变量，默认为false, 没有进入分支
      hasScheduledUpdateInCurrentCommit = true;
    }
    // 全局变量，默认为null，没有没有进入分支
    if (
      currentPhase !== null &&
      currentPhase !== 'componentWillMount' &&
      currentPhase !== 'componentWillReceiveProps'
    ) {
      hasScheduledUpdateInCurrentPhase = true;
    }
  }
}
```
`recordScheduleUpdate`主要用来记录调度器的执行状态，如注释所示，它现在相当于什么都没有做。

##### requestWork

``` typescript
function requestWork(root: FiberRoot, expirationTime: ExpirationTime) {
  addRootToSchedule(root, expirationTime);
  if (isRendering) {
    // Prevent reentrancy. Remaining work will be scheduled at the end of
    // the currently rendering batch.
    return;
  }

  if (isBatchingUpdates) {
    // Flush work at the end of the batch.
    if (isUnbatchingUpdates) {
      // ...unless we're inside unbatchedUpdates, in which case we should
      // flush it now.
      nextFlushedRoot = root;
      nextFlushedExpirationTime = Sync;
      performWorkOnRoot(root, Sync, true);
    }
    return;
  }

  // TODO: Get rid of Sync and use current time?
  if (expirationTime === Sync) {
    performSyncWork();
  } else {
    scheduleCallbackWithExpirationTime(root, expirationTime);
  }
}
```

##### performWork
``` typescript
function performWork(minExpirationTime: ExpirationTime, dl: Deadline | null) {
  deadline = dl;

  // Keep working on roots until there's no more work, or until we reach
  // the deadline.
  findHighestPriorityRoot();

  if (deadline !== null) {
    recomputeCurrentRendererTime();
    currentSchedulerTime = currentRendererTime;

    if (enableUserTimingAPI) {
      const didExpire = nextFlushedExpirationTime < currentRendererTime;
      const timeout = expirationTimeToMs(nextFlushedExpirationTime);
      stopRequestCallbackTimer(didExpire, timeout);
    }

    while (
      nextFlushedRoot !== null &&
      nextFlushedExpirationTime !== NoWork &&
      (minExpirationTime === NoWork ||
        minExpirationTime >= nextFlushedExpirationTime) &&
      (!deadlineDidExpire || currentRendererTime >= nextFlushedExpirationTime)
    ) {
      performWorkOnRoot(
        nextFlushedRoot,
        nextFlushedExpirationTime,
        currentRendererTime >= nextFlushedExpirationTime,
      );
      findHighestPriorityRoot();
      recomputeCurrentRendererTime();
      currentSchedulerTime = currentRendererTime;
    }
  } else {
    while (
      nextFlushedRoot !== null &&
      nextFlushedExpirationTime !== NoWork &&
      (minExpirationTime === NoWork ||
        minExpirationTime >= nextFlushedExpirationTime)
    ) {
      performWorkOnRoot(nextFlushedRoot, nextFlushedExpirationTime, true);
      findHighestPriorityRoot();
    }
  }

  // We're done flushing work. Either we ran out of time in this callback,
  // or there's no more work left with sufficient priority.

  // If we're inside a callback, set this to false since we just completed it.
  if (deadline !== null) {
    callbackExpirationTime = NoWork;
    callbackID = null;
  }
  // If there's work left over, schedule a new callback.
  if (nextFlushedExpirationTime !== NoWork) {
    scheduleCallbackWithExpirationTime(
      ((nextFlushedRoot: any): FiberRoot),
      nextFlushedExpirationTime,
    );
  }

  // Clean-up.
  deadline = null;
  deadlineDidExpire = false;

  finishRendering();
}
```

##### performWorkOnRoot
``` typescript
function performWorkOnRoot(
  root: FiberRoot,
  expirationTime: ExpirationTime,
  isExpired: boolean,
) {
  invariant(
    !isRendering,
    'performWorkOnRoot was called recursively. This error is likely caused ' +
      'by a bug in React. Please file an issue.',
  );

  isRendering = true;

  // Check if this is async work or sync/expired work.
  if (deadline === null || isExpired) {
    // Flush work without yielding.
    // TODO: Non-yieldy work does not necessarily imply expired work. A renderer
    // may want to perform some work without yielding, but also without
    // requiring the root to complete (by triggering placeholders).

    let finishedWork = root.finishedWork;
    if (finishedWork !== null) {
      // This root is already complete. We can commit it.
      completeRoot(root, finishedWork, expirationTime);
    } else {
      root.finishedWork = null;
      // If this root previously suspended, clear its existing timeout, since
      // we're about to try rendering again.
      const timeoutHandle = root.timeoutHandle;
      if (enableSuspense && timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout;
        // $FlowFixMe Complains noTimeout is not a TimeoutID, despite the check above
        cancelTimeout(timeoutHandle);
      }
      const isYieldy = false;
      renderRoot(root, isYieldy, isExpired);
      finishedWork = root.finishedWork;
      if (finishedWork !== null) {
        // We've completed the root. Commit it.
        completeRoot(root, finishedWork, expirationTime);
      }
    }
  } else {
    // Flush async work.
    let finishedWork = root.finishedWork;
    if (finishedWork !== null) {
      // This root is already complete. We can commit it.
      completeRoot(root, finishedWork, expirationTime);
    } else {
      root.finishedWork = null;
      // If this root previously suspended, clear its existing timeout, since
      // we're about to try rendering again.
      const timeoutHandle = root.timeoutHandle;
      if (enableSuspense && timeoutHandle !== noTimeout) {
        root.timeoutHandle = noTimeout;
        // $FlowFixMe Complains noTimeout is not a TimeoutID, despite the check above
        cancelTimeout(timeoutHandle);
      }
      const isYieldy = true;
      renderRoot(root, isYieldy, isExpired);
      finishedWork = root.finishedWork;
      if (finishedWork !== null) {
        // We've completed the root. Check the deadline one more time
        // before committing.
        if (!shouldYield()) {
          // Still time left. Commit the root.
          completeRoot(root, finishedWork, expirationTime);
        } else {
          // There's no time left. Mark this root as complete. We'll come
          // back and commit it later.
          root.finishedWork = finishedWork;
        }
      }
    }
  }

  isRendering = false;
}
```

##### renderRoot
``` typescript
function renderRoot(
  root: FiberRoot,
  isYieldy: boolean,
  isExpired: boolean,
): void {
  invariant(
    !isWorking,
    'renderRoot was called recursively. This error is likely caused ' +
      'by a bug in React. Please file an issue.',
  );
  isWorking = true;
  ReactCurrentOwner.currentDispatcher = Dispatcher;

  const expirationTime = root.nextExpirationTimeToWorkOn;

  // Check if we're starting from a fresh stack, or if we're resuming from
  // previously yielded work.
  if (
    expirationTime !== nextRenderExpirationTime ||
    root !== nextRoot ||
    nextUnitOfWork === null
  ) {
    // Reset the stack and start working from the root.
    resetStack();
    nextRoot = root;
    nextRenderExpirationTime = expirationTime;
    nextUnitOfWork = createWorkInProgress(
      nextRoot.current,
      null,
      nextRenderExpirationTime,
    );
    root.pendingCommitExpirationTime = NoWork;

    if (enableSchedulerTracing) {
      // Reset this flag once we start rendering a new root or at a new priority.
      // This might indicate that suspended work has completed.
      // If not, the flag will be reset.
      nextRenderIncludesTimedOutPlaceholder = false;

      // Determine which interactions this batch of work currently includes,
      // So that we can accurately attribute time spent working on it,
      // And so that cascading work triggered during the render phase will be associated with it.
      const interactions: Set<Interaction> = new Set();
      root.pendingInteractionMap.forEach(
        (scheduledInteractions, scheduledExpirationTime) => {
          if (scheduledExpirationTime <= expirationTime) {
            scheduledInteractions.forEach(interaction =>
              interactions.add(interaction),
            );
          }
        },
      );

      // Store the current set of interactions on the FiberRoot for a few reasons:
      // We can re-use it in hot functions like renderRoot() without having to recalculate it.
      // We will also use it in commitWork() to pass to any Profiler onRender() hooks.
      // This also provides DevTools with a way to access it when the onCommitRoot() hook is called.
      root.memoizedInteractions = interactions;

      if (interactions.size > 0) {
        const subscriber = __subscriberRef.current;
        if (subscriber !== null) {
          const threadID = computeThreadID(
            expirationTime,
            root.interactionThreadID,
          );
          try {
            subscriber.onWorkStarted(interactions, threadID);
          } catch (error) {
            // Work thrown by an interaction tracing subscriber should be rethrown,
            // But only once it's safe (to avoid leaveing the scheduler in an invalid state).
            // Store the error for now and we'll re-throw in finishRendering().
            if (!hasUnhandledError) {
              hasUnhandledError = true;
              unhandledError = error;
            }
          }
        }
      }
    }
  }

  let prevInteractions: Set<Interaction> = (null: any);
  if (enableSchedulerTracing) {
    // We're about to start new traced work.
    // Restore pending interactions so cascading work triggered during the render phase will be accounted for.
    prevInteractions = __interactionsRef.current;
    __interactionsRef.current = root.memoizedInteractions;
  }

  let didFatal = false;

  startWorkLoopTimer(nextUnitOfWork);

  do {
    try {
      workLoop(isYieldy);
    } catch (thrownValue) {
      if (nextUnitOfWork === null) {
        // This is a fatal error.
        didFatal = true;
        onUncaughtError(thrownValue);
      } else {
        if (__DEV__) {
          // Reset global debug state
          // We assume this is defined in DEV
          (resetCurrentlyProcessingQueue: any)();
        }

        const failedUnitOfWork: Fiber = nextUnitOfWork;
        if (__DEV__ && replayFailedUnitOfWorkWithInvokeGuardedCallback) {
          replayUnitOfWork(failedUnitOfWork, thrownValue, isYieldy);
        }

        // TODO: we already know this isn't true in some cases.
        // At least this shows a nicer error message until we figure out the cause.
        // https://github.com/facebook/react/issues/12449#issuecomment-386727431
        invariant(
          nextUnitOfWork !== null,
          'Failed to replay rendering after an error. This ' +
            'is likely caused by a bug in React. Please file an issue ' +
            'with a reproducing case to help us find it.',
        );

        const sourceFiber: Fiber = nextUnitOfWork;
        let returnFiber = sourceFiber.return;
        if (returnFiber === null) {
          // This is the root. The root could capture its own errors. However,
          // we don't know if it errors before or after we pushed the host
          // context. This information is needed to avoid a stack mismatch.
          // Because we're not sure, treat this as a fatal error. We could track
          // which phase it fails in, but doesn't seem worth it. At least
          // for now.
          didFatal = true;
          onUncaughtError(thrownValue);
        } else {
          throwException(
            root,
            returnFiber,
            sourceFiber,
            thrownValue,
            nextRenderExpirationTime,
          );
          nextUnitOfWork = completeUnitOfWork(sourceFiber);
          continue;
        }
      }
    }
    break;
  } while (true);

  if (enableSchedulerTracing) {
    // Traced work is done for now; restore the previous interactions.
    __interactionsRef.current = prevInteractions;
  }

  // We're done performing work. Time to clean up.
  isWorking = false;
  ReactCurrentOwner.currentDispatcher = null;
  resetContextDependences();

  // Yield back to main thread.
  if (didFatal) {
    const didCompleteRoot = false;
    stopWorkLoopTimer(interruptedBy, didCompleteRoot);
    interruptedBy = null;
    // There was a fatal error.
    if (__DEV__) {
      resetStackAfterFatalErrorInDev();
    }
    // `nextRoot` points to the in-progress root. A non-null value indicates
    // that we're in the middle of an async render. Set it to null to indicate
    // there's no more work to be done in the current batch.
    nextRoot = null;
    onFatal(root);
    return;
  }

  if (nextUnitOfWork !== null) {
    // There's still remaining async work in this tree, but we ran out of time
    // in the current frame. Yield back to the renderer. Unless we're
    // interrupted by a higher priority update, we'll continue later from where
    // we left off.
    const didCompleteRoot = false;
    stopWorkLoopTimer(interruptedBy, didCompleteRoot);
    interruptedBy = null;
    onYield(root);
    return;
  }

  // We completed the whole tree.
  const didCompleteRoot = true;
  stopWorkLoopTimer(interruptedBy, didCompleteRoot);
  const rootWorkInProgress = root.current.alternate;
  invariant(
    rootWorkInProgress !== null,
    'Finished root should have a work-in-progress. This error is likely ' +
      'caused by a bug in React. Please file an issue.',
  );

  // `nextRoot` points to the in-progress root. A non-null value indicates
  // that we're in the middle of an async render. Set it to null to indicate
  // there's no more work to be done in the current batch.
  nextRoot = null;
  interruptedBy = null;

  if (nextRenderDidError) {
    // There was an error
    if (hasLowerPriorityWork(root, expirationTime)) {
      // There's lower priority work. If so, it may have the effect of fixing
      // the exception that was just thrown. Exit without committing. This is
      // similar to a suspend, but without a timeout because we're not waiting
      // for a promise to resolve. React will restart at the lower
      // priority level.
      markSuspendedPriorityLevel(root, expirationTime);
      const suspendedExpirationTime = expirationTime;
      const rootExpirationTime = root.expirationTime;
      onSuspend(
        root,
        rootWorkInProgress,
        suspendedExpirationTime,
        rootExpirationTime,
        -1, // Indicates no timeout
      );
      return;
    } else if (
      // There's no lower priority work, but we're rendering asynchronously.
      // Synchronsouly attempt to render the same level one more time. This is
      // similar to a suspend, but without a timeout because we're not waiting
      // for a promise to resolve.
      !root.didError &&
      !isExpired
    ) {
      root.didError = true;
      const suspendedExpirationTime = (root.nextExpirationTimeToWorkOn = expirationTime);
      const rootExpirationTime = (root.expirationTime = Sync);
      onSuspend(
        root,
        rootWorkInProgress,
        suspendedExpirationTime,
        rootExpirationTime,
        -1, // Indicates no timeout
      );
      return;
    }
  }

  if (enableSuspense && !isExpired && nextLatestAbsoluteTimeoutMs !== -1) {
    // The tree was suspended.
    if (enableSchedulerTracing) {
      nextRenderIncludesTimedOutPlaceholder = true;
    }
    const suspendedExpirationTime = expirationTime;
    markSuspendedPriorityLevel(root, suspendedExpirationTime);

    // Find the earliest uncommitted expiration time in the tree, including
    // work that is suspended. The timeout threshold cannot be longer than
    // the overall expiration.
    const earliestExpirationTime = findEarliestOutstandingPriorityLevel(
      root,
      expirationTime,
    );
    const earliestExpirationTimeMs = expirationTimeToMs(earliestExpirationTime);
    if (earliestExpirationTimeMs < nextLatestAbsoluteTimeoutMs) {
      nextLatestAbsoluteTimeoutMs = earliestExpirationTimeMs;
    }

    // Subtract the current time from the absolute timeout to get the number
    // of milliseconds until the timeout. In other words, convert an absolute
    // timestamp to a relative time. This is the value that is passed
    // to `setTimeout`.
    const currentTimeMs = expirationTimeToMs(requestCurrentTime());
    let msUntilTimeout = nextLatestAbsoluteTimeoutMs - currentTimeMs;
    msUntilTimeout = msUntilTimeout < 0 ? 0 : msUntilTimeout;

    // TODO: Account for the Just Noticeable Difference

    const rootExpirationTime = root.expirationTime;
    onSuspend(
      root,
      rootWorkInProgress,
      suspendedExpirationTime,
      rootExpirationTime,
      msUntilTimeout,
    );
    return;
  }

  // Ready to commit.
  onComplete(root, rootWorkInProgress, expirationTime);
}
```

##### completeRoot
``` typescript
function completeRoot(
  root: FiberRoot,
  finishedWork: Fiber,
  expirationTime: ExpirationTime,
): void {
  // Check if there's a batch that matches this expiration time.
  const firstBatch = root.firstBatch;
  if (firstBatch !== null && firstBatch._expirationTime <= expirationTime) {
    if (completedBatches === null) {
      completedBatches = [firstBatch];
    } else {
      completedBatches.push(firstBatch);
    }
    if (firstBatch._defer) {
      // This root is blocked from committing by a batch. Unschedule it until
      // we receive another update.
      root.finishedWork = finishedWork;
      root.expirationTime = NoWork;
      return;
    }
  }

  // Commit the root.
  root.finishedWork = null;

  // Check if this is a nested update (a sync update scheduled during the
  // commit phase).
  if (root === lastCommittedRootDuringThisBatch) {
    // If the next root is the same as the previous root, this is a nested
    // update. To prevent an infinite loop, increment the nested update count.
    nestedUpdateCount++;
  } else {
    // Reset whenever we switch roots.
    lastCommittedRootDuringThisBatch = root;
    nestedUpdateCount = 0;
  }
  commitRoot(root, finishedWork);
}
```

#### 提交更新
处理完更新后需要确认提交更新至渲染模块，然后渲染模块才能将更新渲染至DOM。
``` typescript
function commitRoot(root: FiberRoot, finishedWork: Fiber): void {
  isWorking = true;
  isCommitting = true;
  startCommitTimer();

  invariant(
    root.current !== finishedWork,
    'Cannot commit the same tree as before. This is probably a bug ' +
      'related to the return field. This error is likely caused by a bug ' +
      'in React. Please file an issue.',
  );
  const committedExpirationTime = root.pendingCommitExpirationTime;
  invariant(
    committedExpirationTime !== NoWork,
    'Cannot commit an incomplete root. This error is likely caused by a ' +
      'bug in React. Please file an issue.',
  );
  root.pendingCommitExpirationTime = NoWork;

  // Update the pending priority levels to account for the work that we are
  // about to commit. This needs to happen before calling the lifecycles, since
  // they may schedule additional updates.
  const updateExpirationTimeBeforeCommit = finishedWork.expirationTime;
  const childExpirationTimeBeforeCommit = finishedWork.childExpirationTime;
  const earliestRemainingTimeBeforeCommit =
    updateExpirationTimeBeforeCommit === NoWork ||
    (childExpirationTimeBeforeCommit !== NoWork &&
      childExpirationTimeBeforeCommit < updateExpirationTimeBeforeCommit)
      ? childExpirationTimeBeforeCommit
      : updateExpirationTimeBeforeCommit;
  markCommittedPriorityLevels(root, earliestRemainingTimeBeforeCommit);

  let prevInteractions: Set<Interaction> = (null: any);
  if (enableSchedulerTracing) {
    // Restore any pending interactions at this point,
    // So that cascading work triggered during the render phase will be accounted for.
    prevInteractions = __interactionsRef.current;
    __interactionsRef.current = root.memoizedInteractions;
  }

  // Reset this to null before calling lifecycles
  ReactCurrentOwner.current = null;

  let firstEffect;
  if (finishedWork.effectTag > PerformedWork) {
    // A fiber's effect list consists only of its children, not itself. So if
    // the root has an effect, we need to add it to the end of the list. The
    // resulting list is the set that would belong to the root's parent, if
    // it had one; that is, all the effects in the tree including the root.
    if (finishedWork.lastEffect !== null) {
      finishedWork.lastEffect.nextEffect = finishedWork;
      firstEffect = finishedWork.firstEffect;
    } else {
      firstEffect = finishedWork;
    }
  } else {
    // There is no effect on the root.
    firstEffect = finishedWork.firstEffect;
  }

  prepareForCommit(root.containerInfo);

  // Invoke instances of getSnapshotBeforeUpdate before mutation.
  nextEffect = firstEffect;
  startCommitSnapshotEffectsTimer();
  while (nextEffect !== null) {
    let didError = false;
    let error;
    if (__DEV__) {
      invokeGuardedCallback(null, commitBeforeMutationLifecycles, null);
      if (hasCaughtError()) {
        didError = true;
        error = clearCaughtError();
      }
    } else {
      try {
        commitBeforeMutationLifecycles();
      } catch (e) {
        didError = true;
        error = e;
      }
    }
    if (didError) {
      invariant(
        nextEffect !== null,
        'Should have next effect. This error is likely caused by a bug ' +
          'in React. Please file an issue.',
      );
      captureCommitPhaseError(nextEffect, error);
      // Clean-up
      if (nextEffect !== null) {
        nextEffect = nextEffect.nextEffect;
      }
    }
  }
  stopCommitSnapshotEffectsTimer();

  if (enableProfilerTimer) {
    // Mark the current commit time to be shared by all Profilers in this batch.
    // This enables them to be grouped later.
    recordCommitTime();
  }

  // Commit all the side-effects within a tree. We'll do this in two passes.
  // The first pass performs all the host insertions, updates, deletions and
  // ref unmounts.
  nextEffect = firstEffect;
  startCommitHostEffectsTimer();
  while (nextEffect !== null) {
    let didError = false;
    let error;
    if (__DEV__) {
      invokeGuardedCallback(null, commitAllHostEffects, null);
      if (hasCaughtError()) {
        didError = true;
        error = clearCaughtError();
      }
    } else {
      try {
        commitAllHostEffects();
      } catch (e) {
        didError = true;
        error = e;
      }
    }
    if (didError) {
      invariant(
        nextEffect !== null,
        'Should have next effect. This error is likely caused by a bug ' +
          'in React. Please file an issue.',
      );
      captureCommitPhaseError(nextEffect, error);
      // Clean-up
      if (nextEffect !== null) {
        nextEffect = nextEffect.nextEffect;
      }
    }
  }
  stopCommitHostEffectsTimer();

  resetAfterCommit(root.containerInfo);

  // The work-in-progress tree is now the current tree. This must come after
  // the first pass of the commit phase, so that the previous tree is still
  // current during componentWillUnmount, but before the second pass, so that
  // the finished work is current during componentDidMount/Update.
  root.current = finishedWork;

  // In the second pass we'll perform all life-cycles and ref callbacks.
  // Life-cycles happen as a separate pass so that all placements, updates,
  // and deletions in the entire tree have already been invoked.
  // This pass also triggers any renderer-specific initial effects.
  nextEffect = firstEffect;
  startCommitLifeCyclesTimer();
  while (nextEffect !== null) {
    let didError = false;
    let error;
    if (__DEV__) {
      invokeGuardedCallback(
        null,
        commitAllLifeCycles,
        null,
        root,
        committedExpirationTime,
      );
      if (hasCaughtError()) {
        didError = true;
        error = clearCaughtError();
      }
    } else {
      try {
        commitAllLifeCycles(root, committedExpirationTime);
      } catch (e) {
        didError = true;
        error = e;
      }
    }
    if (didError) {
      invariant(
        nextEffect !== null,
        'Should have next effect. This error is likely caused by a bug ' +
          'in React. Please file an issue.',
      );
      captureCommitPhaseError(nextEffect, error);
      if (nextEffect !== null) {
        nextEffect = nextEffect.nextEffect;
      }
    }
  }

  isCommitting = false;
  isWorking = false;
  stopCommitLifeCyclesTimer();
  stopCommitTimer();
  onCommitRoot(finishedWork.stateNode);
  if (__DEV__ && ReactFiberInstrumentation.debugTool) {
    ReactFiberInstrumentation.debugTool.onCommitWork(finishedWork);
  }

  const updateExpirationTimeAfterCommit = finishedWork.expirationTime;
  const childExpirationTimeAfterCommit = finishedWork.childExpirationTime;
  const earliestRemainingTimeAfterCommit =
    updateExpirationTimeAfterCommit === NoWork ||
    (childExpirationTimeAfterCommit !== NoWork &&
      childExpirationTimeAfterCommit < updateExpirationTimeAfterCommit)
      ? childExpirationTimeAfterCommit
      : updateExpirationTimeAfterCommit;
  if (earliestRemainingTimeAfterCommit === NoWork) {
    // If there's no remaining work, we can clear the set of already failed
    // error boundaries.
    legacyErrorBoundariesThatAlreadyFailed = null;
  }
  onCommit(root, earliestRemainingTimeAfterCommit);

  if (enableSchedulerTracing) {
    __interactionsRef.current = prevInteractions;

    let subscriber;

    try {
      subscriber = __subscriberRef.current;
      if (subscriber !== null && root.memoizedInteractions.size > 0) {
        const threadID = computeThreadID(
          committedExpirationTime,
          root.interactionThreadID,
        );
        subscriber.onWorkStopped(root.memoizedInteractions, threadID);
      }
    } catch (error) {
      // It's not safe for commitRoot() to throw.
      // Store the error for now and we'll re-throw in finishRendering().
      if (!hasUnhandledError) {
        hasUnhandledError = true;
        unhandledError = error;
      }
    } finally {
      if (!nextRenderIncludesTimedOutPlaceholder) {
        // Clear completed interactions from the pending Map.
        // Unless the render was suspended or cascading work was scheduled,
        // In which case– leave pending interactions until the subsequent render.
        const pendingInteractionMap = root.pendingInteractionMap;
        pendingInteractionMap.forEach(
          (scheduledInteractions, scheduledExpirationTime) => {
            // Only decrement the pending interaction count if we're done.
            // If there's still work at the current priority,
            // That indicates that we are waiting for suspense data.
            if (
              earliestRemainingTimeAfterCommit === NoWork ||
              scheduledExpirationTime < earliestRemainingTimeAfterCommit
            ) {
              pendingInteractionMap.delete(scheduledExpirationTime);

              scheduledInteractions.forEach(interaction => {
                interaction.__count--;

                if (subscriber !== null && interaction.__count === 0) {
                  try {
                    subscriber.onInteractionScheduledWorkCompleted(interaction);
                  } catch (error) {
                    // It's not safe for commitRoot() to throw.
                    // Store the error for now and we'll re-throw in finishRendering().
                    if (!hasUnhandledError) {
                      hasUnhandledError = true;
                      unhandledError = error;
                    }
                  }
                }
              });
            }
          },
        );
      }
    }
  }
}

// 循环执行提交更新
function commitAllHostEffects() {
  while (nextEffect !== null) {
    if (__DEV__) {
      ReactCurrentFiber.setCurrentFiber(nextEffect);
    }
    recordEffect();

    const effectTag = nextEffect.effectTag;

    if (effectTag & ContentReset) {
      commitResetTextContent(nextEffect);
    }

    if (effectTag & Ref) {
      const current = nextEffect.alternate;
      if (current !== null) {
        commitDetachRef(current);
      }
    }

    // The following switch statement is only concerned about placement,
    // updates, and deletions. To avoid needing to add a case for every
    // possible bitmap value, we remove the secondary effects from the
    // effect tag and switch on that value.
    let primaryEffectTag = effectTag & (Placement | Update | Deletion);
    switch (primaryEffectTag) {
      case Placement: {
        commitPlacement(nextEffect);
        // Clear the "placement" from effect tag so that we know that this is inserted, before
        // any life-cycles like componentDidMount gets called.
        // TODO: findDOMNode doesn't rely on this any more but isMounted
        // does and isMounted is deprecated anyway so we should be able
        // to kill this.
        nextEffect.effectTag &= ~Placement;
        break;
      }
      case PlacementAndUpdate: {
        // Placement
        commitPlacement(nextEffect);
        // Clear the "placement" from effect tag so that we know that this is inserted, before
        // any life-cycles like componentDidMount gets called.
        nextEffect.effectTag &= ~Placement;

        // Update
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      case Update: {
        const current = nextEffect.alternate;
        commitWork(current, nextEffect);
        break;
      }
      case Deletion: {
        commitDeletion(nextEffect);
        break;
      }
    }
    nextEffect = nextEffect.nextEffect;
  }

  if (__DEV__) {
    ReactCurrentFiber.resetCurrentFiber();
  }
}
```
提交更新是最后确认更新组件的阶段，现在我们看一下提交更新的主要逻辑：
``` typescript
function commitWork(current: Fiber | null, finishedWork: Fiber): void {
  if (!supportsMutation) {
    commitContainer(finishedWork);
    return;
  }

  switch (finishedWork.tag) {
    case ClassComponent:
    case ClassComponentLazy: {
      return;
    }
    case HostComponent: {
      const instance: Instance = finishedWork.stateNode;
      if (instance != null) {
        // Commit the work prepared earlier.
        const newProps = finishedWork.memoizedProps;
        // For hydration we reuse the update path but we treat the oldProps
        // as the newProps. The updatePayload will contain the real change in
        // this case.
        const oldProps = current !== null ? current.memoizedProps : newProps;
        const type = finishedWork.type;
        // TODO: Type the updateQueue to be specific to host components.
        const updatePayload: null | UpdatePayload = (finishedWork.updateQueue: any);
        finishedWork.updateQueue = null;
        if (updatePayload !== null) {
          commitUpdate(
            instance,
            updatePayload,
            type,
            oldProps,
            newProps,
            finishedWork,
          );
        }
      }
      return;
    }
    case HostText: {
      invariant(
        finishedWork.stateNode !== null,
        'This should have a text node initialized. This error is likely ' +
          'caused by a bug in React. Please file an issue.',
      );
      const textInstance: TextInstance = finishedWork.stateNode;
      const newText: string = finishedWork.memoizedProps;
      // For hydration we reuse the update path but we treat the oldProps
      // as the newProps. The updatePayload will contain the real change in
      // this case.
      const oldText: string =
        current !== null ? current.memoizedProps : newText;
      commitTextUpdate(textInstance, oldText, newText);
      return;
    }
    case HostRoot: {
      return;
    }
    case Profiler: {
      return;
    }
    case PlaceholderComponent: {
      return;
    }
    default: {
      invariant(
        false,
        'This unit of work tag should not have side-effects. This error is ' +
          'likely caused by a bug in React. Please file an issue.',
      );
    }
  }
}
```

#### 小结
至此首次渲染的执行流程为：
`ReactDOM.render`（渲染入口） => `legacyRenderSubtreeIntoContainer`（把虚拟的dom树渲染到真实的dom容器中） => `DOMRenderer.updateContainer`（更新容器内容） => `scheduleRootUpdate`（开始更新） => `scheduleWork`（处理更新） => `commitWork`（提交更新）
## 高级指南

### 插槽(Portals)

Portals 提供了一种很好的方法，将子节点渲染到父组件 DOM 层次结构之外的 DOM 节点。

## React Fiber
> React Fiber 并不是所谓的纤程（微线程、协程），而是一种基于浏览器的单线程调度算法。

我们都知道浏览器渲染引擎是单线程的，在 React15.x 及之前版本，从 setState 开始到渲染完成整个过程是不受控制且连续不中断完成的，由于该过程将会占用整个线程，则其他任务都会被阻塞，如样式计算、界面布局以及许多情况下的绘制等。如果需要渲染的是一个很大、层级很深的组件，这可能就会使用户感觉明显卡顿，比如更新一个组件需要1毫秒，如果有200个组件要更新，那就需要200毫秒，在这200毫秒的更新过程中，浏览器唯一的主线程在专心运行更新操作，无暇去做其他任何事情。想象一下，在这200毫秒内，用户往一个input元素中输入点什么，敲击键盘也不会立即获得响应，虽然渲染输入按键结果是浏览器主线程的工作，但是浏览器主线程被React占用，抽不出空，最后的结果就是用户敲了按键看不到反应，等React更新过程结束之后，咔咔咔那些按键一下子出现在input元素里了，这个版本的调和器可以称为**栈调和器（Stack Reconciler）**。Stack Reconcilier 的主要缺陷就是**不能暂停渲染任务，也不能切分任务，更无法有效平衡组件更新渲染与动画相关任务间的执行顺序（即不能划分任务优先级），这样就很有可能导致重要任务卡顿，动画掉帧等问题。**

为了解决这个问题，React 团队经过两年多的努力，提出了一个更先进的调和器，它允许渲染过程分段完成，而不必一次性完成，在渲染期间可返回到主线程控制执行其他任务。这是通过计算部分组件树的变更，并暂停渲染更新，询问主线程是否有更高需求的绘制或者更新任务需要执行，这些高需求的任务完成后再重新渲染。这一切的实现是在代码层引入了一个新的数据结构：**Fiber对象**，每一个组件实例对应有一个fiber实例，此fiber实例负责管理组件实例的更新，渲染任务及与其他fiber实例的通信，这个先进的调和器叫做**纤维调和器（Fiber Reconciler）**，它提供的新功能主要有：
**一：**把可中断的任务拆分成小任务；
**二：**可重用各分阶段任务，对正在做的工作调整优先次序；
**三：**可以在父子组件任务间前进后退切换任务，以支持React执行过程中的布局刷新；
**四：**支持 render 方法返回多个元素；
**五：**对异常边界处理提供了更好的支持；

### 调度任务（scheduleWork）
前面提到 Fiber 可以异步实现不同优先级任务的协调执行，目前在 JavaScript 中也提供了这种方式，在新版主流浏览器有两个可用API：requestIdleCallback 和 requestAnimationFrame：
**requestIdleCallback：**在线程空闲时调度执行低优先级函数。
**requestAnimationFrame：**在下一个动画帧调度执行高优先级函数。

一般网页线程执行任务时会以帧的形式划分，大部分网页控制在30-60帧是不会影响用户体验的；在两个执行帧之间，主线程通常会有一小段空闲时间，requestIdleCallback可以在这个空闲期（Idle Period）调用空闲期回调（Idle Callback），执行一些任务。
![img2.png](/images/react-source-analysis/img2.png)

而 Fiber 所做的就是需要分解渲染任务，根据优先级使用API调度，异步执行指定任务。低优先级任务由 requestIdleCallback 处理；高优先级任务，如动画相关的由 requestAnimationFrame 处理；requestIdleCallback 可以在多个空闲期调用空闲期回调，执行任务；requestIdleCallback 方法提供 deadline，即任务执行限制时间，以切分任务，避免长时间执行，阻塞UI渲染而导致掉帧；

现在我们来看一下 React 调度任务实现的源码：
``` js
// TODO: 目前，只有一个优先级别，Deferred。未来将增加额外的优先级
var DEFERRED_TIMEOUT = 5000;

// 回调被储存为一个双向循环链表
var firstCallbackNode = null;

// 是否在执行工作
var isPerformingWork = false;

var isHostCallbackScheduled = false;

var hasNativePerformanceNow =
  typeof performance === 'object' && typeof performance.now === 'function';

var timeRemaining;
if (hasNativePerformanceNow) {
  timeRemaining = function() {
    // We assume that if we have a performance timer that the rAF callback
    // gets a performance timer value. Not sure if this is always true.
    var remaining = getFrameDeadline() - performance.now();
    // 计算得到当前帧运行剩余时间
    return remaining > 0 ? remaining : 0;
  };
} else {
  timeRemaining = function() {
    // Fallback to Date.now()
    var remaining = getFrameDeadline() - Date.now();
    // 计算得到当前帧运行剩余时间
    return remaining > 0 ? remaining : 0;
  };
}

var deadlineObject = {
  timeRemaining,
  didTimeout: false,
};

function ensureHostCallbackIsScheduled() {
  // 正在执行工作
  if (isPerformingWork) {
    return;
  }
  // 使用列表中最先超时的回调
  var timesOutAt = firstCallbackNode.timesOutAt;
  if (!isHostCallbackScheduled) {
    isHostCallbackScheduled = true;
  } else {
    // 取消回调
    cancelCallback();
  }
  requestCallback(flushWork, timesOutAt);
}

// 刷新第一次回调
function flushFirstCallback(node) {
  var flushedNode = firstCallbackNode;

  // 在调用回调之前从列表中移除该节点。这样，即使回调抛出, 列表也处于一致状态。
  var next = firstCallbackNode.next;
  if (firstCallbackNode === next) {
    // 这是列表中的最后一个回调。
    firstCallbackNode = null;
    next = null;
  } else {
    var previous = firstCallbackNode.previous;
    firstCallbackNode = previous.next = next;
    next.previous = previous;
  }

  flushedNode.next = flushedNode.previous = null;

  // 现在调用回调是安全的。
  var callback = flushedNode.callback;
  callback(deadlineObject);
}

function flushWork(didTimeout) {
  isPerformingWork = true;
  deadlineObject.didTimeout = didTimeout;
  try {
    if (didTimeout) {
      // Flush all the timed out callbacks without yielding.
      while (firstCallbackNode !== null) {
        // Read the current time. Flush all the callbacks that expire at or
        // earlier than that time. Then read the current time again and repeat.
        // This optimizes for as few performance.now calls as possible.
        var currentTime = getCurrentTime();
        if (firstCallbackNode.timesOutAt <= currentTime) {
          do {
            flushFirstCallback();
          } while (
            firstCallbackNode !== null &&
            firstCallbackNode.timesOutAt <= currentTime
          );
          continue;
        }
        break;
      }
    } else {
      // Keep flushing callbacks until we run out of time in the frame.
      if (firstCallbackNode !== null) {
        do {
          flushFirstCallback();
        } while (
          firstCallbackNode !== null &&
          getFrameDeadline() - getCurrentTime() > 0
        );
      }
    }
  } finally {
    isPerformingWork = false;
    if (firstCallbackNode !== null) {
      // There's still work remaining. Request another callback.
      ensureHostCallbackIsScheduled(firstCallbackNode);
    } else {
      isHostCallbackScheduled = false;
    }
  }
}

// 调度任务，这是一个不稳定 api
function unstable_scheduleWork(callback, options) {
  var currentTime = getCurrentTime();

  var timesOutAt;
  if (
    options !== undefined &&
    options !== null &&
    options.timeout !== null &&
    options.timeout !== undefined
  ) {
    // 根据传入的 timeout 计算超时
    timesOutAt = currentTime + options.timeout;
  } else {
    // 使用默认常量计算超时
    timesOutAt = currentTime + DEFERRED_TIMEOUT;
  }

  var newNode = {
    callback,
    timesOutAt,
    next: null,
    previous: null,
  };

  // 将新回调插入列表中, 并按其超时顺序排序
  if (firstCallbackNode === null) {
    // 这是列表中的第一个回调
    firstCallbackNode = newNode.next = newNode.previous = newNode;
    ensureHostCallbackIsScheduled(firstCallbackNode);
  } else {
    var next = null;
    var node = firstCallbackNode;
    do {
      if (node.timesOutAt > timesOutAt) {
        // 在此之前, 新的回调超时
        next = node;
        break;
      }
      node = node.next;
    } while (node !== firstCallbackNode);

    if (next === null) {
      // 找不到稍后超时的回调, 这意味着新的回调在列表中具有最新的超时。
      next = firstCallbackNode;
    } else if (next === firstCallbackNode) {
      // 新回调在整个列表中具有最早的超时。
      firstCallbackNode = newNode;
      ensureHostCallbackIsScheduled(firstCallbackNode);
    }

    var previous = next.previous;
    previous.next = next.previous = newNode;
    newNode.next = next;
    newNode.previous = previous;
  }

  return newNode;
}

function unstable_cancelScheduledWork(callbackNode) {
  var next = callbackNode.next;
  if (next === null) {
    // Already cancelled.
    return;
  }

  if (next === callbackNode) {
    // This is the only scheduled callback. Clear the list.
    firstCallbackNode = null;
  } else {
    // Remove the callback from its position in the list.
    if (callbackNode === firstCallbackNode) {
      firstCallbackNode = next;
    }
    var previous = callbackNode.previous;
    previous.next = next;
    next.previous = previous;
  }

  callbackNode.next = callbackNode.previous = null;
}

// The remaining code is essentially a polyfill for requestIdleCallback. It
// works by scheduling a requestAnimationFrame, storing the time for the start
// of the frame, then scheduling a postMessage which gets scheduled after paint.
// Within the postMessage handler do as much work as possible until time + frame
// rate. By separating the idle call into a separate event tick we ensure that
// layout, paint and other browser work is counted against the available time.
// The frame rate is dynamically adjusted.

// We capture a local reference to any global, in case it gets polyfilled after
// this module is initially evaluated. We want to be using a
// consistent implementation.
var localDate = Date;

// This initialization code may run even on server environments if a component
// just imports ReactDOM (e.g. for findDOMNode). Some environments might not
// have setTimeout or clearTimeout. However, we always expect them to be defined
// on the client. https://github.com/facebook/react/pull/13088
var localSetTimeout = typeof setTimeout === 'function' ? setTimeout : undefined;
var localClearTimeout =
  typeof clearTimeout === 'function' ? clearTimeout : undefined;

// We don't expect either of these to necessarily be defined, but we will error
// later if they are missing on the client.
var localRequestAnimationFrame =
  typeof requestAnimationFrame === 'function'
    ? requestAnimationFrame
    : undefined;
var localCancelAnimationFrame =
  typeof cancelAnimationFrame === 'function' ? cancelAnimationFrame : undefined;

var getCurrentTime;

// requestAnimationFrame does not run when the tab is in the background. If
// we're backgrounded we prefer for that work to happen so that the page
// continues to load in the background. So we also schedule a 'setTimeout' as
// a fallback.
// TODO: Need a better heuristic for backgrounded work.
var ANIMATION_FRAME_TIMEOUT = 100;
var rAFID;
var rAFTimeoutID;
var requestAnimationFrameWithTimeout = function(callback) {
  // schedule rAF and also a setTimeout
  rAFID = localRequestAnimationFrame(function(timestamp) {
    // cancel the setTimeout
    localClearTimeout(rAFTimeoutID);
    callback(timestamp);
  });
  rAFTimeoutID = localSetTimeout(function() {
    // cancel the requestAnimationFrame
    localCancelAnimationFrame(rAFID);
    callback(getCurrentTime());
  }, ANIMATION_FRAME_TIMEOUT);
};

if (hasNativePerformanceNow) {
  var Performance = performance;
  getCurrentTime = function() {
    return Performance.now();
  };
} else {
  getCurrentTime = function() {
    return localDate.now();
  };
}

var requestCallback;
var cancelCallback;
var getFrameDeadline;

if (typeof window === 'undefined') { // 非浏览器环境
  var timeoutID = -1;
  requestCallback = function(callback, absoluteTimeout) {
    timeoutID = setTimeout(callback, 0, true);
  };
  cancelCallback = function() {
    clearTimeout(timeoutID);
  };
  getFrameDeadline = function() {
    return 0;
  };
} else if (window._schedMock) { // 动态注入, 仅用于测试目的。
  var impl = window._schedMock;
  requestCallback = impl[0];
  cancelCallback = impl[1];
  getFrameDeadline = impl[2];
} else {
  if (typeof console !== 'undefined') {
    if (typeof localRequestAnimationFrame !== 'function') {
      console.error(
        "This browser doesn't support requestAnimationFrame. " +
          'Make sure that you load a ' +
          'polyfill in older browsers. https://fb.me/react-polyfills',
      );
    }
    if (typeof localCancelAnimationFrame !== 'function') {
      console.error(
        "This browser doesn't support cancelAnimationFrame. " +
          'Make sure that you load a ' +
          'polyfill in older browsers. https://fb.me/react-polyfills',
      );
    }
  }

  var scheduledCallback = null;
  // 是否在执行空闲期回调
  var isIdleScheduled = false;
  var timeoutTime = -1;

  var isAnimationFrameScheduled = false;

  var isPerformingIdleWork = false;

  var frameDeadline = 0;

  // 用启发式跟踪法，从30fps（即30帧）开始调整得到的更适于当前环境的一帧限制时间；
  var previousFrameTime = 33;
  var activeFrameTime = 33;

  getFrameDeadline = function() {
    return frameDeadline;
  };

  // We use the postMessage trick to defer idle work until after the repaint.
  var messageKey =
    '__reactIdleCallback$' +
    Math.random()
      .toString(36)
      .slice(2);
  // 空闲期回调
  var idleTick = function(event) {
    if (event.source !== window || event.data !== messageKey) {
      return;
    }
    // 重置为false，表明可以调用空闲期回调
    isIdleScheduled = false;

    var currentTime = getCurrentTime();

    var didTimeout = false;
    if (frameDeadline - currentTime <= 0) {
      // 帧到期时间小于当前时间，说明已过期
      if (timeoutTime !== -1 && timeoutTime <= currentTime) {
        // 此帧已过期，且发生任务处理函数（执行具体任务，传入的回调）的超时
        // 需要执行任务处理，下文将调用；
        didTimeout = true;
      } else {
        // 帧已过期，但没有发生任务处理函数的超时，暂时不调用任务处理函数
        if (!isAnimationFrameScheduled) {
          // 当前没有调度别的帧回调函数
          // 调度下一帧
          isAnimationFrameScheduled = true;
          requestAnimationFrameWithTimeout(animationTick);
        }
        // Exit without invoking the callback.
        return;
      }
    }

    // 缓存的任务处理函数
    timeoutTime = -1;
    var callback = scheduledCallback;
    scheduledCallback = null;
    if (callback !== null) {
      isPerformingIdleWork = true;
      try {
        // 执行回调
        callback(didTimeout);
      } finally {
        isPerformingIdleWork = false;
      }
    }
  };
  // Assumes that we have addEventListener in this environment. Might need
  // something better for old IE.
  window.addEventListener('message', idleTick, false);

  // 帧回调
  var animationTick = function(rafTime) {
    isAnimationFrameScheduled = false;
    var nextFrameTime = rafTime - frameDeadline + activeFrameTime;
    if (
      nextFrameTime < activeFrameTime &&
      previousFrameTime < activeFrameTime
    ) {
      if (nextFrameTime < 8) {
        // Defensive coding. We don't support higher frame rates than 120hz.
        // If we get lower than that, it is probably a bug.
        nextFrameTime = 8;
      }
      // If one frame goes long, then the next one can be short to catch up.
      // If two frames are short in a row, then that's an indication that we
      // actually have a higher frame rate than what we're currently optimizing.
      // We adjust our heuristic dynamically accordingly. For example, if we're
      // running on 120hz display or 90hz VR display.
      // Take the max of the two in case one of them was an anomaly due to
      // missed frame deadlines.
      activeFrameTime =
        nextFrameTime < previousFrameTime ? previousFrameTime : nextFrameTime;
    } else {
      previousFrameTime = nextFrameTime;
    }
    frameDeadline = rafTime + activeFrameTime;
    if (!isIdleScheduled) {
      // 不在执行空闲期回调，表明可以调用空闲期回调
      isIdleScheduled = true;
      window.postMessage(messageKey, '*');
    }
  };

  // 自定义 模拟requestIdleCallback
  requestCallback = function(callback, absoluteTimeout) {
    // 回调函数
    scheduledCallback = callback;
    timeoutTime = absoluteTimeout;
    if (isPerformingIdleWork) {
      // 如果我们已经在执行空闲工作, 则必须抛出错误。
      // 不要等待下一帧。在新事件中继续尽快工作 ASAP。
      window.postMessage(messageKey, '*');
    } else if (!isAnimationFrameScheduled) {
      // 如果当前没有调度帧回调函数，我们需要进行一个调度帧回调函数
      // TODO: rAF 仍是 setTimeout
      isAnimationFrameScheduled = true;
      // 初始开始执行帧回调 
      requestAnimationFrameWithTimeout(animationTick);
    }
  };

  cancelCallback = function() {
    scheduledCallback = null;
    isIdleScheduled = false;
    timeoutTime = -1;
  };
}

export {
  unstable_scheduleWork,
  unstable_cancelScheduledWork,
  getCurrentTime as unstable_now,
};

```
### Fiber与组件

我们已经知道了Fiber的功能及其主要特点，那么其如何和组件联系，并且如何实现效果的呢，以下几点可以概括：
1. React应用中的基础单元是组件，应用以组件树形式组织，渲染组件；
2. Fiber调和器基础单元则是fiber（调和单元），应用以fiber树形式组织，应用Fiber算法；
3. 组件树和fiber树结构对应，一个组件实例有一个对应的fiber实例；
4. Fiber负责整个应用层面的调和，fiber实例负责对应组件的调和；

**注意Fiber与fiber的区别，Fiber是指调和器算法，fiber则是调和器算法组成单元，和组件与应用关系类似，每一个组件实例会有对应的fiber实例负责该组件的调和。**

### Fiber数据结构

截止目前，我们对Fiber应该有了初步的了解，在具体介绍Fiber的实现与架构之前，准备先简单介绍一下Fiber的数据结构，数据结构能一定程度反映其整体工作架构。
其实，一个fiber就是一个JavaScript对象，以键值对形式存储了一个关联组件的信息，包括组件接收的props，维护的state，最后需要渲染出的内容等。接下来我们将介Fiber对象的主要属性。

#### FiberRoot 对象
FiberRoot 对象主要用来管理组件树组件的更新进程，同时记录组件树挂载的DOM容器相关信息。
``` js
export type FiberRoot = {
  // fiber节点的容器元素相关信息，通常会直接传入容器元素
  containerInfo: any,
  // 仅用于持久更新
  pendingChildren: any,
  // 当前fiber树中激活状态（正在处理）的fiber节点
  current: Fiber,
  // 从提交中暂停的最早和最新的优先级级别
  earliestSuspendedTime: ExpirationTime,
  latestSuspendedTime: ExpirationTime,
  // 不知道要暂停的最早和最新的优先级级别。
  earliestPendingTime: ExpirationTime,
  latestPendingTime: ExpirationTime,
  // 由已解决的承诺 pinged 的最新优先级级别, 并可以重试
  latestPingedTime: ExpirationTime,

  // 如果引发错误, 并且队列中没有其他更新, 我们尝试在处理前再一次从根中渲染错误。
  didError: boolean,

  pendingCommitExpirationTime: ExpirationTime,
  // 已完成的工作正在进行的 HostRoot 已准备好提交
  finishedWork: Fiber | null,
  // setTimeout 返回的超时句柄。如果它被一个新的取代了。用于取消挂起的超时, 
  timeoutHandle: TimeoutHandle | NoTimeout,
  // 顶部上下文对象, 由 renderSubtreeIntoContainer 使用
  context: Object | null,
  pendingContext: Object | null,
  // 确定我们是否应该尝试在初始加载使用 hydrate
  +hydrate: boolean,
  // 此节点剩余的任务到期时间
  // TODO: Lift this into the renderer
  nextExpirationTimeToWorkOn: ExpirationTime,
  expirationTime: ExpirationTime,
  // 顶级批次的列表。此列表指示是否应推迟提交，也包含完成回调。
  // TODO: Lift this into the renderer
  firstBatch: Batch | null,
  // 多组件树FirberRoot对象以单链表存储链接，指向下一个需要调度的FiberRoot
  nextScheduledRoot: FiberRoot | null,
};
```

**创建FiberRoot实例**
``` js
export function createFiberRoot(
  containerInfo: any,
  isAsync: boolean,
  hydrate: boolean,
): FiberRoot {
  // 创建初始根组件对应的fiber实例
  const uninitializedFiber = createHostRootFiber(isAsync);

  let root;
  if (enableSchedulerTracing) {
    root = ({
      // 根组件对应的fiber实例，一直用它
      current: uninitializedFiber,
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,

      interactionThreadID: unstable_getThreadID(),
      memoizedInteractions: new Set(),
      pendingInteractionMap: new Map(),
    }: FiberRoot);
  } else {
    root = ({
      current: uninitializedFiber,
      containerInfo: containerInfo,
      pendingChildren: null,

      earliestPendingTime: NoWork,
      latestPendingTime: NoWork,
      earliestSuspendedTime: NoWork,
      latestSuspendedTime: NoWork,
      latestPingedTime: NoWork,

      didError: false,

      pendingCommitExpirationTime: NoWork,
      finishedWork: null,
      timeoutHandle: noTimeout,
      context: null,
      pendingContext: null,
      hydrate,
      nextExpirationTimeToWorkOn: NoWork,
      expirationTime: NoWork,
      firstBatch: null,
      nextScheduledRoot: null,
    }: BaseFiberRootProperties);
  }
  // 组件树根组件fiber实例的stateNode指向FiberRoot对象
  uninitializedFiber.stateNode = root;

  return ((root: any): FiberRoot);
}

// 创建返回一个初始根组件对应的fiber实例
export function createHostRootFiber(isAsync: boolean): Fiber {
  let mode = isAsync ? AsyncMode | StrictMode : NoContext;

  if (enableProfilerTimer && isDevToolsPresent) {
    // Always collect profile timings when DevTools are present.
    // This enables DevTools to start capturing timing at any point–
    // Without some nodes in the tree having empty base times.
    mode |= ProfileMode;
  }

  // 创建fiber
  return createFiber(HostRoot, null, null, mode);
}
```

#### Fiber对象

[Fiber对象](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiber.js)的定义在`packages/react-reconciler/src/ReactFiber.js`中：
``` js
// 一个Fiber对象作用于一个组件
export type Fiber = {|
  // 标记fiber类型tag
  tag: TypeOfWork,

  // 唯一标识
  key: null | string,

  // fiber对应的function/class/module类型组件名.
  type: any,

  // fiber所在组件树的根组件FiberRoot对象
  stateNode: any,

  // 处理完当前fiber后返回的fiber，
  // 返回当前fiber所在fiber树的父级fiber实例
  return: Fiber | null,

  // fiber树结构相关链接
  child: Fiber | null,
  sibling: Fiber | null,
  index: number,

  // The ref last used to attach this node.
  // I'll avoid adding an owner field for prod and model that as functions.
  ref: null | (((handle: mixed) => void) & {_stringRef: ?string}) | RefObject,

  // 当前处理过程中的组件props对象
  pendingProps: any, // This type will be more specific once we overload the tag.
  // 缓存的之前组件props对象
  memoizedProps: any, // The props used to create the output.

  // 组件状态更新及对应回调函数的存储队列
  updateQueue: UpdateQueue<any> | null,

  // The state used to create the output
  memoizedState: any,

  // A linked-list of contexts that this fiber depends on
  firstContextDependency: ContextDependency<mixed> | null,

  // Bitfield that describes properties about the fiber and its subtree. E.g.
  // the AsyncMode flag indicates whether the subtree should be async-by-
  // default. When a fiber is created, it inherits the mode of its
  // parent. Additional flags can be set at creation time, but after that the
  // value should remain unchanged throughout the fiber's lifetime, particularly
  // before its child fibers are created.
  mode: TypeOfMode,

  // Effect
  effectTag: TypeOfSideEffect,

  // Singly linked list fast path to the next fiber with side-effects.
  nextEffect: Fiber | null,

  // The first and last fiber with side-effect within this subtree. This allows
  // us to reuse a slice of the linked list when we reuse the work done within
  // this fiber.
  firstEffect: Fiber | null,
  lastEffect: Fiber | null,

  // 更新任务的最晚执行时间
  expirationTime: ExpirationTime,

  // This is used to quickly determine if a subtree has no pending changes.
  childExpirationTime: ExpirationTime,

  // fiber的版本池，即记录fiber更新过程，便于恢复
  alternate: Fiber | null,

  // Conceptual aliases  
  // workInProgress : Fiber ->  alternate The alternate used for reuse happens  
  // to be the same as work in progress.

  // Time spent rendering this Fiber and its descendants for the current update.
  // This tells us how well the tree makes use of sCU for memoization.
  // It is reset to 0 each time we render and only updated when we don't bailout.
  // This field is only set when the enableProfilerTimer flag is enabled.
  actualDuration?: number,

  // If the Fiber is currently active in the "render" phase,
  // This marks the time at which the work began.
  // This field is only set when the enableProfilerTimer flag is enabled.
  actualStartTime?: number,

  // Duration of the most recent render time for this Fiber.
  // This value is not updated when we bailout for memoization purposes.
  // This field is only set when the enableProfilerTimer flag is enabled.
  selfBaseDuration?: number,

  // Sum of base times for all descedents of this Fiber.
  // This value bubbles up during the "complete" phase.
  // This field is only set when the enableProfilerTimer flag is enabled.
  treeBaseDuration?: number,

  // Conceptual aliases
  // workInProgress : Fiber ->  alternate The alternate used for reuse happens
  // to be the same as work in progress.
  // __DEV__ only
  _debugID?: number,
  _debugSource?: Source | null,
  _debugOwner?: Fiber | null,
  _debugIsCurrentlyTiming?: boolean,
|};
```
1. type & key：同React元素的值；
2. type：描述fiber对应的React组件；
  1. 对于组合组件：值为function或class组件本身；
  2. 对于原生组件（div等）：值为该元素类型字符串；
3. key：调和阶段，标识fiber，以检测是否可重用该fiber实例；
4. child & sibling：组件树，对应生成fiber树，类比的关系；
5. pendingProps & memoizedProps：分别表示组件当前传入的及之前的props；
6. return：返回当前fiber所在fiber树的父级fiber实例，即当前组件的父组件对应的fiber；
7. alternate：fiber的版本池，即记录fiber更新过程，便于恢复重用；
8. workInProgress：正在处理的fiber，概念上叫法，实际上没有此属性；

##### alternate fiber
可以理解为一个fiber版本池，用于交替记录组件更新（切分任务后变成多阶段更新）过程中fiber的更新，因为在组件更新的各阶段，更新前及更新过程中fiber状态并不一致，在需要恢复时（如，发生冲突），即可使用另一者直接回退至上一版本fiber。
>1. 使用alternate属性双向连接一个当前fiber和其work-in-progress，当前fiber实例的alternate属性指向其work-in-progress，work-in-progress的alternate属性指向当前稳定fiber；
2. 当前fiber的替换版本是其work-in-progress，work-in-progress的交替版本是当前fiber；
3. 当work-in-progress更新一次后，将同步至当前fiber，然后继续处理，同步直至任务完成；
4. work-in-progress指向处理过程中的fiber，而当前fiber总是维护处理完成的最新版本的fiber。

##### 创建Fiber实例

创建fiber实例即返回一个带有上一小节描述的诸多属性的JavaScript对象，FiberNode即根据传入的参数构造返回一个初始化的对象：
``` js
const createFiber = function(
  tag: TypeOfWork,
  pendingProps: mixed,
  key: null | string,
  mode: TypeOfMode,
): Fiber {
  // $FlowFixMe: the shapes are exact here but Flow doesn't like constructors
  return new FiberNode(tag, pendingProps, key, mode);
};
```

创建alternate fiber以处理任务的实现如下：
``` js
// 创建一个alternate fiber处理任务
export function createWorkInProgress(
  current: Fiber,
  pendingProps: any,
  expirationTime: ExpirationTime,
): Fiber {
  let workInProgress = current.alternate;
  if (workInProgress === null) {
    // We use a double buffering pooling technique because we know that we'll
    // only ever need at most two versions of a tree. We pool the "other" unused
    // node that we're free to reuse. This is lazily created to avoid allocating
    // extra objects for things that are never updated. It also allow us to
    // reclaim the extra memory if needed.
    workInProgress = createFiber(
      current.tag,
      pendingProps,
      current.key,
      current.mode,
    );
    workInProgress.type = current.type;
    workInProgress.stateNode = current.stateNode;

    if (__DEV__) {
      // DEV-only fields
      workInProgress._debugID = current._debugID;
      workInProgress._debugSource = current._debugSource;
      workInProgress._debugOwner = current._debugOwner;
    }

    workInProgress.alternate = current;
    current.alternate = workInProgress;
  } else {
    workInProgress.pendingProps = pendingProps;

    // We already have an alternate.
    // Reset the effect tag.
    workInProgress.effectTag = NoEffect;

    // The effect list is no longer valid.
    workInProgress.nextEffect = null;
    workInProgress.firstEffect = null;
    workInProgress.lastEffect = null;

    if (enableProfilerTimer) {
      // We intentionally reset, rather than copy, actualDuration & actualStartTime.
      // This prevents time from endlessly accumulating in new commits.
      // This has the downside of resetting values for different priority renders,
      // But works for yielding (the common case) and should support resuming.
      workInProgress.actualDuration = 0;
      workInProgress.actualStartTime = -1;
    }
  }

  // Don't touching the subtree's expiration time, which has not changed.
  workInProgress.childExpirationTime = current.childExpirationTime;
  if (pendingProps !== current.pendingProps) {
    // This fiber has new props.
    workInProgress.expirationTime = expirationTime;
  } else {
    // This fiber's props have not changed.
    workInProgress.expirationTime = current.expirationTime;
  }

  workInProgress.child = current.child;
  workInProgress.memoizedProps = current.memoizedProps;
  workInProgress.memoizedState = current.memoizedState;
  workInProgress.updateQueue = current.updateQueue;
  workInProgress.firstContextDependency = current.firstContextDependency;

  // These will be overridden during the parent's reconciliation
  workInProgress.sibling = current.sibling;
  workInProgress.index = current.index;
  workInProgress.ref = current.ref;

  if (enableProfilerTimer) {
    workInProgress.selfBaseDuration = current.selfBaseDuration;
    workInProgress.treeBaseDuration = current.treeBaseDuration;
  }

  return workInProgress;
}
```

#### Fiber类型
上一小节，Fiber对象中有个tag属性，标记fiber类型，而fiber实例是和组件对应的，所以其类型基本上对应于组件类型，在`packages/shared/ReactWorkTags.js`中：
``` js
export type TypeOfWork = | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 | 16;

export const FunctionalComponent = 0; // 函数式组件
export const FunctionalComponentLazy = 1;
export const ClassComponent = 2; // Class类组件
export const ClassComponentLazy = 3;
export const IndeterminateComponent = 4; // Before we know whether it is functional or class
export const HostRoot = 5; // 组件树根组件，可以嵌套
export const HostPortal = 6; // 子树。可以是一个入口点不同的渲染器。
export const HostComponent = 7; // 标准组件，如地div， span等
export const HostText = 8; // 文本
export const Fragment = 9;  // 片段
export const Mode = 10;
export const ContextConsumer = 11;
export const ContextProvider = 12;
export const ForwardRef = 13;
export const ForwardRefLazy = 14;
export const Profiler = 15;
export const PlaceholderComponent = 16; // placeholder（占位符）
```
在调度执行任务的时候会根据不同类型fiber，即fiber.tag值进行不同处理。

#### FiberRoot对象

FiberRoot对象，主要用来管理组件树组件的更新进程，同时记录组件树挂载的DOM容器相关信息，在`packages/react-reconciler/src/ReactFiberRoot.js`中：
``` js
export type FiberRoot = {
  // fiber节点的容器元素相关信息，通常会直接传入容器元素
  containerInfo: any,
  // Used only by persistent updates.
  pendingChildren: any,
  // 当前fiber树中激活状态（正在处理）的fiber节点，
  current: Fiber,

  // The following priority levels are used to distinguish between 1)
  // uncommitted work, 2) uncommitted work that is suspended, and 3) uncommitted
  // work that may be unsuspended. We choose not to track each individual
  // pending level, trading granularity for performance.
  //
  // The earliest and latest priority levels that are suspended from committing.
  earliestSuspendedTime: ExpirationTime,
  latestSuspendedTime: ExpirationTime,
  // The earliest and latest priority levels that are not known to be suspended.
  earliestPendingTime: ExpirationTime,
  latestPendingTime: ExpirationTime,
  // The latest priority level that was pinged by a resolved promise and can
  // be retried.
  latestPingedTime: ExpirationTime,

  // If an error is thrown, and there are no more updates in the queue, we try
  // rendering from the root one more time, synchronously, before handling
  // the error.
  didError: boolean,

  pendingCommitExpirationTime: ExpirationTime,
  // 准备好提交的已处理完成的work-in-progress
  finishedWork: Fiber | null,
  // Timeout handle returned by setTimeout. Used to cancel a pending timeout, if
  // it's superseded by a new one.
  timeoutHandle: TimeoutHandle | NoTimeout,
  // Top context object, used by renderSubtreeIntoContainer
  context: Object | null,
  pendingContext: Object | null,
  // Determines if we should attempt to hydrate on the initial mount
  +hydrate: boolean,
  // Remaining expiration time on this root.
  // TODO: Lift this into the renderer
  nextExpirationTimeToWorkOn: ExpirationTime,
  expirationTime: ExpirationTime,
  // List of top-level batches. This list indicates whether a commit should be
  // deferred. Also contains completion callbacks.
  // TODO: Lift this into the renderer
  firstBatch: Batch | null,
  // 多组件树FirberRoot对象以单链表存储链接，指向下一个需要调度的FiberRoot
  nextScheduledRoot: FiberRoot | null,
};
```

##### 创建FiberRoot实例

``` js
import {
  ClassComponent,
  HostRoot,
  Mode,
} from 'shared/ReactTypeOfWork';
// 创建返回一个初始根组件对应的fiber实例
export function createHostRootFiber(isAsync: boolean): Fiber {
  let mode = isAsync ? AsyncMode | StrictMode : NoContext;

  if (enableProfilerTimer && isDevToolsPresent) {
    // Always collect profile timings when DevTools are present.
    // This enables DevTools to start capturing timing at any point–
    // Without some nodes in the tree having empty base times.
    mode |= ProfileMode;
  }
  // 创建fiber
  return createFiber(HostRoot, null, null, mode);
}

export function createFiberRoot(
  containerInfo: any,
  isAsync: boolean,
  hydrate: boolean,
): FiberRoot {
  // 创建初始根组件对应的fiber实例
  const uninitializedFiber = createHostRootFiber(isAsync);
  // 组件树根组件的FiberRoot对象
  const root = {
    // 根组件对应的fiber实例
    current: uninitializedFiber,
    containerInfo: containerInfo,
    pendingChildren: null,

    earliestPendingTime: NoWork,
    latestPendingTime: NoWork,
    earliestSuspendedTime: NoWork,
    latestSuspendedTime: NoWork,
    latestPingedTime: NoWork,

    didError: false,

    pendingCommitExpirationTime: NoWork,
    finishedWork: null,
    timeoutHandle: noTimeout,
    context: null,
    pendingContext: null,
    hydrate,
    nextExpirationTimeToWorkOn: NoWork,
    expirationTime: NoWork,
    firstBatch: null,
    nextScheduledRoot: null,
  };
  // 组件树根组件fiber实例的stateNode指向FiberRoot对象
  uninitializedFiber.stateNode = root;
  return root;
}
```

#### ReactChildFiber

在生成组件树的FiberRoot对象后，会为子组件生成各自的fiber实例，这一部分由[ReactChildFiber模块]()实现，在`packages/react-reconciler/src/ReactChildFiber.js`中：

``` js
// 调和（处理更新）子fibers
export const reconcileChildFibers = ChildReconciler(true);
// 挂载（初始化）子fibers
export const mountChildFibers = ChildReconciler(false);
```
而ChildReconciler方法所做的则是根据传入参数判断是调用初始化子组件fibers逻辑还是执行调和已有子组件fibers逻辑。

ChildReconciler方法，返回reconcileChildFibers方法：
1. 判断子级传递内容的数据类型，执行不同的处理，这也对应着我们写React组件时传递props.children时，其类型可以是对象或数组，字符串，是数字等；
2. 然后具体根据子组件类型，调用不同的具体调和处理函数；
3. 最后返回根据子组件创建或更新得到的fiber实例；

``` js
function reconcileChildFibers(
    returnFiber: Fiber,
    currentFirstChild: Fiber | null,
    newChild: any,
    expirationTime: ExpirationTime,
  ): Fiber | null {
    // This function is not recursive.
    // If the top level item is an array, we treat it as a set of children,
    // not as a fragment. Nested arrays on the other hand will be treated as
    // fragment nodes. Recursion happens at the normal flow.

    // Handle top level unkeyed fragments as if they were arrays.
    // This leads to an ambiguity between <>{[...]}</> and <>...</>.
    // We treat the ambiguous cases above the same.
    const isUnkeyedTopLevelFragment =
      typeof newChild === 'object' &&
      newChild !== null &&
      newChild.type === REACT_FRAGMENT_TYPE &&
      newChild.key === null;
    if (isUnkeyedTopLevelFragment) {
      newChild = newChild.props.children;
    }

    // Handle object types
    const isObject = typeof newChild === 'object' && newChild !== null;

    if (isObject) {
      // 子组件实例类型，以Symbol符号表示的
      switch (newChild.$$typeof) {
        case REACT_ELEMENT_TYPE:
          return placeSingleChild(
            reconcileSingleElement(
              returnFiber,
              currentFirstChild,
              newChild,
              expirationTime,
            ),
          );
         // React组件调用
        case REACT_PORTAL_TYPE:
          return placeSingleChild(
            reconcileSinglePortal(
              returnFiber,
              currentFirstChild,
              newChild,
              expirationTime,
            ),
          );
      }
    }

    if (typeof newChild === 'string' || typeof newChild === 'number') {
      return placeSingleChild(
        reconcileSingleTextNode(
          returnFiber,
          currentFirstChild,
          '' + newChild,
          expirationTime,
        ),
      );
    }

    if (isArray(newChild)) {
      return reconcileChildrenArray(
        returnFiber,
        currentFirstChild,
        newChild,
        expirationTime,
      );
    }

    if (getIteratorFn(newChild)) {
      return reconcileChildrenIterator(
        returnFiber,
        currentFirstChild,
        newChild,
        expirationTime,
      );
    }

    if (isObject) {
      throwOnInvalidObjectType(returnFiber, newChild);
    }

    if (__DEV__) {
      if (typeof newChild === 'function') {
        warnOnFunctionType();
      }
    }
    if (typeof newChild === 'undefined' && !isUnkeyedTopLevelFragment) {
      // If the new child is undefined, and the return fiber is a composite
      // component, throw an error. If Fiber return types are disabled,
      // we already threw above.
      switch (returnFiber.tag) {
        case ClassComponent:
        case ClassComponentLazy: {
          if (__DEV__) {
            const instance = returnFiber.stateNode;
            if (instance.render._isMockFunction) {
              // We allow auto-mocks to proceed as if they're returning null.
              break;
            }
          }
        }
        // Intentionally fall through to the next case, which handles both
        // functions and classes
        // eslint-disable-next-lined no-fallthrough
        case FunctionalComponent: {
          const Component = returnFiber.type;
          invariant(
            false,
            '%s(...): Nothing was returned from render. This usually means a ' +
              'return statement is missing. Or, to render nothing, ' +
              'return null.',
            Component.displayName || Component.name || 'Component',
          );
        }
      }
    }

    // Remaining cases are all treated as empty.
    return deleteRemainingChildren(returnFiber, currentFirstChild);
  }

  return reconcileChildFibers;
}
```

### Fiber架构

在学习Fiber的时候，我尝试去阅读源码，发现通过这种方式很难快速理解，学习Fiber，而先了解调和器是干什么的及调和器在React中的存在形式，然后再学习Fiber的结构及算法实现思路，明白从组件被定义到渲染至页面它需要做什么，这也是本篇文章的组织形式。

#### 优先级（ExpirationTime VS PriorityLevel）

我们已经知道Fiber可以切分任务并设置不同优先级，那么是如何实现划分优先级的呢，其表现形式什么呢？

##### ExpirationTime

Fiber切分任务并调用requestIdleCallback和requestAnimationFrameAPI，保证渲染任务和其他任务，在不影响应用交互，不掉帧的前提下，稳定执行，而实现调度的方式正是给每一个fiber实例设置到期执行时间，不同时间即代表不同优先级，到期时间越短，则代表优先级越高，需要尽早执行。
> 所谓的到期时间（ExpirationTime），是相对于调度器初始调用的起始时间而言的一个时间段；调度器初始调用后的某一段时间内，需要调度完成这项更新，这个时间段长度值就是到期时间值。

Fiber提供[ReactFiberExpirationTime模块](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberExpirationTime.js)实现到期时间的定义，在`packages/react-reconciler/src/ReactFiberExpirationTime.js`中：
``` js
export const NoWork = 0; // 没有任务等待处理
export const Sync = 1; // 同步模式，立即处理任务
export const Never = MAX_SIGNED_31_BIT_INT; // 1073741823 Max 31: Math.pow(2, 30) - 1 

const UNIT_SIZE = 10; // 过期时间单元（ms）
const MAGIC_NUMBER_OFFSET = 2; // 到期时间偏移量

// 以ExpirationTime特定单位（1单位=10ms）表示的到期执行时间
export function msToExpirationTime(ms: number): ExpirationTime {
  // 总是增加一个偏移量，在ms<10时与Nowork模式进行区别
  return ((ms / UNIT_SIZE) | 0) + MAGIC_NUMBER_OFFSET;
}

// 以毫秒表示的到期执行时间
export function expirationTimeToMs(expirationTime: ExpirationTime): number {
  return (expirationTime - MAGIC_NUMBER_OFFSET) * UNIT_SIZE;
}

// 向上取整（整数单位到期执行时间）
// precision范围精度：弥补任务执行时间误差
function ceiling(num: number, precision: number): number {
  return (((num / precision) | 0) + 1) * precision;
}

// 计算处理误差时间在内的到期时间
function computeExpirationBucket(
  currentTime,
  expirationInMs,
  bucketSizeMs,
): ExpirationTime {
  return (
    MAGIC_NUMBER_OFFSET +
    ceiling(
      currentTime - MAGIC_NUMBER_OFFSET + expirationInMs / UNIT_SIZE,
      bucketSizeMs / UNIT_SIZE,
    )
  );
}

export const LOW_PRIORITY_EXPIRATION = 5000;
export const LOW_PRIORITY_BATCH_SIZE = 250;

export function computeAsyncExpiration(
  currentTime: ExpirationTime,
): ExpirationTime {
  return computeExpirationBucket(
    currentTime,
    LOW_PRIORITY_EXPIRATION,
    LOW_PRIORITY_BATCH_SIZE,
  );
}

// We intentionally set a higher expiration time for interactive updates in
// dev than in production.
//
// If the main thread is being blocked so long that you hit the expiration,
// it's a problem that could be solved with better scheduling.
//
// People will be more likely to notice this and fix it with the long
// expiration time in development.
//
// In production we opt for better UX at the risk of masking scheduling
// problems, by expiring fast.
export const HIGH_PRIORITY_EXPIRATION = __DEV__ ? 500 : 150;
export const HIGH_PRIORITY_BATCH_SIZE = 100;

export function computeInteractiveExpiration(currentTime: ExpirationTime) {
  return computeExpirationBucket(
    currentTime,
    HIGH_PRIORITY_EXPIRATION,
    HIGH_PRIORITY_BATCH_SIZE,
  );
}

```

该模块提供的功能主要有：
1. Sync：同步模式，在UI线程立即执行此类任务，如动画反馈等；
2. 异步模式：
  1. 转换：到期时间特定单位和时间单位（ms）的相互转换；
  2. 计算：计算包含允许误差在内的到期时间；

##### PriorityLevel

其实在15.x版本中出现了对于任务的优先层级划分，[ReactPriorityLevel模块](https://github.com/facebook/react/blob/15.6-dev/src/renderers/shared/fiber/ReactPriorityLevel.js)，在`/src/renderers/shared/fiber/ReactPriorityLevel.js`中：
```js
export type PriorityLevel = 0 | 1 | 2 | 3 | 4 | 5;

module.exports = {
  NoWork: 0, // No work is pending.
  SynchronousPriority: 1, // For controlled text inputs. Synchronous side-effects.
  AnimationPriority: 2, // Needs to complete before the next frame.
  HighPriority: 3, // Interaction that needs to complete pretty soon to feel responsive.
  LowPriority: 4, // Data fetching, or result from updating stores.
  OffscreenPriority: 5, // Won't be visible but do the work in case it becomes visible.
};
```
相对于PriorityLevel的简单层级划分，在16.x版本中使用的则是ExpirationTime的到期时间方式表示任务的优先级，可以更好的对任务进行切分，调度。

#### 调度器（Scheduler）
前面介绍调和器主要作用就是在组件状态变更时，调用组件树各组件的render方法，渲染，卸载组件，而Fiber使得应用可以更好的协调不同任务的执行，调和器内关于高效协调的实现，我们可以称它为调度器（Scheduler）。
>顾名思义，调度器即调度资源以执行指定任务，React应用中应用组件的更新与渲染，需要占用系统CPU资源，如果不能很好的进行资源平衡，合理调度，优化任务执行策略，那很容易造成CPU这一紧缺资源的消耗和浪费，容易造成页面卡顿，动画掉帧，组件更新异常等诸多问题，就像城市交通调度一样，如果不能有效调度，交通状况很可能将拥堵不堪。

在React 15.x版本中，组件的状态变更将直接导致其子组件树的重新渲染，新版本Fiber算法将在调度器方面进行全面改进，主要的关注点是：
1. 合并多次更新：没有必要在组件的每一个状态变更时都立即触发更新任务，有些中间状态变更其实是对更新任务所耗费资源的浪费，就比如用户发现错误点击时快速操作导致组件某状态从A至B再至C，这中间的B状态变更其实对于用户而言并没有意义，那么我们可以直接合并状态变更，直接从A至C只触发一次更新；
2. 任务优先级：不同类型的更新有不同优先级，例如用户操作引起的交互动画可能需要有更好的体验，其优先级应该比完成数据更新高；
3. 推拉式调度：基于推送的调度方式更多的需要开发者编码间接决定如何调度任务，而拉取式调度更方便React框架层直接进行全局自主调度；

[源码](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js)在`packages/react-reconciler/src/ReactFiberScheduler.js`中：

``` js
export {
  requestCurrentTime,
  computeExpirationForFiber,
  captureCommitPhaseError,
  onUncaughtError,
  renderDidSuspend,
  renderDidError,
  retrySuspendedRoot,
  markLegacyErrorBoundaryAsFailed,
  isAlreadyFailedLegacyErrorBoundary,
  scheduleWork,
  requestWork,
  flushRoot,
  batchedUpdates,
  unbatchedUpdates,
  flushSync,
  flushControlled,
  deferredUpdates,
  syncUpdates,
  interactiveUpdates,
  flushInteractiveUpdates,
  computeUniqueAsyncExpiration,
};
```
如上调度器主要输出API为实现调度任务，拉取更新，延迟更新等功能。

##### 调度器与优先级

调度器如何切分任务划分优先级的呢？在React调和算法中，任务由fiber实例描述，所以要划分任务优先级，等效于设置fiber的到期时间（expirationTime），调度器内提供了computeExpirationForFiber方法以计算某一个fiber的到期时间，[源码](https://github.com/facebook/react/blob/master/packages/react-reconciler/src/ReactFiberScheduler.js)在`packages/react-reconciler/src/ReactFiberScheduler.js`中：

``` js
import {
  NoWork,
  Sync,
  Never,
  msToExpirationTime,
  expirationTimeToMs,
  computeAsyncExpiration,
  computeInteractiveExpiration,
} from './ReactFiberExpirationTime';

function computeExpirationForFiber(currentTime: ExpirationTime, fiber: Fiber) {
  let expirationTime;
  if (expirationContext !== NoWork) {
    // An explicit expiration context was set;
    expirationTime = expirationContext;
  } else if (isWorking) {
    if (isCommitting) {
      // 在提交阶段的更新任务 需要明确设置同步优先级（Sync Priority）
      expirationTime = Sync;
    } else {
      // 在渲染阶段发生的更新任务
      // 需要设置为下一次渲染时间的到期时间优先级
      expirationTime = nextRenderExpirationTime;
    }
  } else {
    // No explicit expiration context was set, and we're not currently
    // performing work. Calculate a new expiration time.
    if (fiber.mode & AsyncMode) {
      if (isBatchingInteractiveUpdates) {
        // This is an interactive update
        expirationTime = computeInteractiveExpiration(currentTime);
      } else {
        // This is an async update
        expirationTime = computeAsyncExpiration(currentTime);
      }
      // If we're in the middle of rendering a tree, do not update at the same
      // expiration time that is already rendering.
      if (nextRoot !== null && expirationTime === nextRenderExpirationTime) {
        expirationTime += 1;
      }
    } else {
      // 同步更新，设置为同步标记
      expirationTime = Sync;
    }
  }
  if (isBatchingInteractiveUpdates) {
    // This is an interactive update. Keep track of the lowest pending
    // interactive expiration time. This allows us to synchronously flush
    // all interactive updates when needed.
    if (
      lowestPendingInteractiveExpirationTime === NoWork ||
      expirationTime > lowestPendingInteractiveExpirationTime
    ) {
      lowestPendingInteractiveExpirationTime = expirationTime;
    }
  }
  return expirationTime;
}
```
1. 若当前处于任务提交阶段（更新提交至DOM渲染）时，设置当前fiber到期时间为Sync，即同步执行模式；
2. 若处于DOM渲染阶段时，则需要延迟此fiber任务，将fiber到期时间设置为下一次DOM渲染到期时间；
3. 若不在任务执行阶段，则需重新设置fiber到期时间：
  1. 若明确设置useSyncScheduling且fiber.internalContextTag值不等于AsyncUpdates，则表明是同步模式，设置为Sync；
  2. 否则，调用computeAsyncExpiration方法重新计算此fiber的到期时间；

``` js
// 重新计算当前时间（ExpirationTime单位表示）
function recalculateCurrentTime() {  
  const ms = now() - startTime;  
  // ExpirationTime单位表示的当前时间  
  // 时间段值为 now() - startTime（起始时间）  
  mostRecentCurrentTime = msToExpirationTime(ms);  
  return mostRecentCurrentTime;
} 

// 计算异步任务的到期时间
function computeAsyncExpiration() {  
  // 计算得到ExpirationTime单位的当前时间  
  // 聚合相似的更新在一起  
  // 更新应该在 ~1000ms，最多1200ms内完成  
  const currentTime = recalculateCurrentTime();  
  // 对于每个fiber的期望到期时间的增值，最大值为1000ms 
  const expirationMs = 1000;  
  // 到期时间的可接受误差时间，200ms 
  const bucketSizeMs = 200;  
  // 返回包含误差时间在内的到期时间  
  return computeExpirationBucket(currentTime, expirationMs, bucketSizeMs);
}

```
对于每一个fiber我们期望的到期时间参数是1000ms，另外由于任务执行时间误差，接受200ms误差，最后计算得到的到期时间默认返回值为ExpirationTime单位。

##### 任务调度

上一节介绍了调度器主要提供computeExpirationForFiber等方法支持计算任务优先级（到期时间），接下来介绍调度器如何调度任务。
> React应用更新时，Fiber从当前处理节点，层层遍历至组件树根组件，然后开始处理更新，调用前面的requestIdleCallback等API执行更新处理。

主要调度逻辑实现在scheduleWork：

1. 通过fiber.return属性，从当前fiber实例层层遍历至组件树根组件；
2. 依次对每一个fiber实例进行到期时间判断，若大于传入的期望任务到期时间参数，则将其更新为传入的任务到期时间；
3. 调用requestWork方法开始处理任务，并传入获取的组件树根组件FiberRoot对象和任务到期时间；

``` js

```

### 渲染与调和

在调和阶段，不涉及任何DOM处理，在处理完更新后，需要渲染模块将更新渲染至DOM，这也是React应用中虚拟DOM（Virtual DOM）的概念，即所有的更新计算都基于虚拟DOM，计算完后才将优化后的更新渲染至真实DOM。Fiber使用requestIdleCallbackAPI更高效的执行渲染更新的任务，实现任务的切分。

## 本文不断更新中


