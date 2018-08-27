---
title: React 源码全方位剖析
date: 2018-08-12 22:10:52
tags:
    - react
categories:
    - react
top: 100
---

`版本：v16.4.3-alpha.0`

## 前言

当时在各种前端框架或库充斥市场的情况下，出现了大量优秀的框架，比如 Backbone、Angular、Knockout、Ember 这些框架大都采用了 MV* 的理念，把数据与视图分离。而就在这样纷繁复杂的时期，React 诞生于 Facebook 的内部项目，因为该公司对市场上所有 JavaScript MVC 框架，都不满意，就决定自己写一套，用来架设 Instagram 的网站。做出来以后，发现这套东西很好用，就在2013年5月开源了。所谓知其然还要知其所以然，加上 React 真是一天一改，如果现在不看，以后也真的很难看懂了。目前社区有很多 React 的源码剖析文章，趁着最近工作不忙，我打算分享一下 React 源码，并自形成一个系列，欢迎一起交流。在开始之前我们先做以下几点约定：

**第一：**目前分析的版本是 React 的最新版本 16.4.3-alpha.0；
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
module.exports = React.default ? React.default : React;
```

上述代码中执行`import React from 'react'`时，其实引入的就是这里提供的对象。这里需要说明一点：**这里为什么会导出 `React.default ? React.default : React`？**
>首先 Rollup 支持 amd、cjs、es、iife 和 umd 模块打包，我们在`packages/src/react.js`使用 export default 进行默认导出，这里使用 require 进行导入。当我们打包成 cjs 模块，我们都知道 require 是 CommonJS 的模块导入方式，不支持模块的默认导出，导入的结果其实是一个含 default 属性的对象，因此需要使用 React.default 来获取实际的 React 对象。当我们打包成 umd 模块时，其中包含 es，这里由于 babel解析器 的功劳，它令(ES6)import === (CommonJS)require，导入的结果其实是不含 default 属性的对象，因此直接使用 React 来获取实际的 React 对象。

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

那么至此，我们应该对 React 是什么有一个直观的认识，它本质上就是一个 JSON 对象，至于 React 能做什么，它是怎么做的，我们会在后面的章节一一剖析它们。

## 主要概念

### 渲染



### Hello World

在 React 中我们可以采用十分简洁的语法来声明式的将数据渲染为 DOM：
``` js
ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```
这段代码在页面中渲染了一个 “Hello, world!” 标题。

React核心内容只涉及如何定义组件，并不涉及具体的组件渲染（即输出用户界面），这需要额外引入渲染模块，以渲染React定义的组件：

## 高级指南

### 插槽(Portals)

Portals 提供了一种很好的方法，将子节点渲染到父组件 DOM 层次结构之外的 DOM 节点。

## Fiber 架构

### 背景
我们都知道浏览器渲染引擎是单线程的，在 React15.x 及之前版本，从setState开始到渲染完成整个过程是不受控制且连续不中断完成的，由于该过程将会占用整个线程，则其他任务都会被阻塞，如样式计算、界面布局以及许多情况下的绘制等，如果需要渲染的是一个很大、层级很深的组件，这可能就会使用户感觉明显卡顿，比如更新一个组件需要1毫秒，如果有200个组件要更新，那就需要200毫秒，在这200毫秒的更新过程中，浏览器唯一的主线程在专心运行更新操作，无暇去做其他任何事情。想象一下，在这200毫秒内，用户往一个input元素中输入点什么，敲击键盘也不会获得响应，因为渲染输入按键结果也是浏览器主线程的工作，但是浏览器主线程被React占用，抽不出空，最后的结果就是用户敲了按键看不到反应，等React更新过程结束之后，咔咔咔那些按键一下子出现在input元素里了

为了解决这个问题，React 团队经过两年多的努力，重构了 React 的核心算法，重构的产物就是Fiber reconciler，并在 v16 版本发布了这个新的特性。为了区别前后的 reconciler，通常将 v16 之前的称为stack reconciler，重写后的称为fiber reconciler，简称为Fiber。

### 调和器
。。这个版本的调和器可以称为**栈调和器（Stack Reconciler）**。Stack Reconcilier 的主要缺陷就是**不能暂停渲染任务，也不能切分任务，更无法有效平衡组件更新渲染与动画相关任务间的执行顺序（即不能划分任务优先级），这样就很有可能导致重要任务卡顿，动画掉帧等问题。**

React16 版本提出了一个更先进的调和器，它允许渲染过程可分段完成，而不必一次性完成，在渲染期间可返回到主进程控制执行其他任务。这是通过计算部分组件树的变更，并暂停渲染更新，询问主进程是否有更高需求的绘制或者更新任务需要执行，这些高需求的任务完成后才开始渲染。这一切的实现是在代码层引入了一个新的数据结构：**Fiber对象**，每一个组件实例对应有一个fiber实例，此fiber实例负责管理组件实例的更新，渲染任务及与其他fiber实例的通信。
这个先进的调和器叫做**纤维调和器（Fiber Reconciler）**，它提供的新功能主要有：
**一：**把可中断的任务拆分成小任务；
**二：**可重用各分阶段任务，对正在做的工作调整优先次序；
**三：**可以在父子组件任务间前进后退切换任务，以支持React执行过程中的布局刷新；
**四：**支持 render 方法返回多个元素；
**五：**对异常边界处理提供了更好的支持；

## 本文不断更新中


