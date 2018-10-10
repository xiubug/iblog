---
title: Vue 源码全方位剖析
date: 2018-08-12 23:55:37
tags:
    - vue
categories:
    - vue
top: 99
---

`版本：v2.5.17-beta.0`

## 前言

随着这几年前端的快速发展，页面中需要实现的功能越来越复杂，DOM操作频繁，使用传统的jQuery库去频繁操作DOM时不仅消耗性能，而且各种DOM绑定后期维护时简直是一场噩梦，在开发大型项目时，模块间的依赖问题也变得十分复杂，在这个大背景下，以数据驱动和组件化思想开发的 Vue、React等JavaScript MVVM库应运而生。相比于其他库，Vue.js 提供了更加简洁、更易于理解的 API，使得我们能够快速上手，一经推出，便迅速走红。现在 Vue.js 更是火得一塌糊涂，github star 数更是超越 React。既然 Vue 如此火，我们是不是很有必要了解一下 Vue.js 背后的实现原理。

目前社区有很多 Vue.js 的源码剖析文章，当下质量比较好的有[Vue技术内幕--逐行级别的 Vue 源码分析](http://hcysun.me/vue-design/)、[Vue.js 技术揭秘](https://ustbhuangyi.github.io/vue-analysis/)，更是得到 Vue 作者本人的推荐。通过通读这两本电子书，我相信大家都能全方位了解 Vue.js 的实现原理。有人可能会有疑问，比如：既然人家写得这么好你为什么还写，直接看人家的不就好了吗，谁会看你的等等？我要说的是源码分析并不是为了彰显个人技术，更多的是为了学习，不管当下想法是否足够成熟，只要我们坚持，我们都会有收获。在开始之前我们先做以下几点约定：

**第一：**目前分析的版本是 Vue.js 的最新版本 Vue.js 2.5.17-beta.0；
**第二：**Vue web应用是最常见的，也是最易于理解的，所以该源码均围绕 Vue web应用剖析；
**第三：**我尽可能站在我自己的角度去剖析，当然我会借鉴社区比较优秀的文章，面对大家的拍砖，我无条件接受，也很乐意与大家一起交换意见，努力写好该 Vue 源码系列；
**第四：**如果有幸您读到该 Vue 源码系列，感觉写得还行，还望收藏、分享或打赏。

## 前置知识

我们从这一章开始即将分析 Vue 的源码，在分析源码之前我们很有必要介绍一些前置知识如flow、Rollup等。除此之外，我们最好已经用过 Vue 做过实际项目，对 Vue 的思想有了一定的了解，对绝大部分的 API 都已经有使用，同时，我们应该有一定的HTML、CSS、JavaScript、ES6+、node & npm等功底，并对代码调试有一定的了解。

如果具备了以上条件，并且对 Vue 的实现原理很感兴趣，那么就可以开始 Vue 的底层学习了，对它的实现细节一探究竟。

### Flow - JavaScript静态类型检查工具

[Flow](https://flow.org/) 是 facebook 出品的 JavaScript 静态类型检查工具，它与 Typescript 不同的是，它可以部分引入，不需要完全重构整个项目，所以对于一个已有一定规模的项目来说，迁移成本更小，也更加可行。除此之外，Flow 可以提供实时增量的反馈，通过运行 Flow server 不需要在每次更改项目的时候完全从头运行类型检查，提高运行效率。可以简单总结为：**对于新项目，可以考虑使用 TypeScript 或者 Flow，对于已有一定规模的项目则建议使用 Flow 进行较小成本的逐步迁移来引入类型检查。Vue 的源码利用了 Flow 做了静态类型检查，所以了解 Flow 有助于我们阅读源码。**

#### 为什么用静态类型检查工具 Flow

JavaScript 是动态类型语言，它的灵活性有目共睹，但是过于灵活的副作用就是很容易就写出非常隐蔽的隐患代码，在编译期甚至运行时看上去都不会报错，但是可能会发生各种各样奇怪的和难以解决的bug。

类型检查是当前动态类型语言的发展趋势，所谓类型检查，就是在编译期尽早发现（由类型错误引起的）bug，又不影响代码运行（不需要运行时动态检查类型），使编写 JavaScript 具有和编写 Java 等强类型语言相近的体验。

项目越复杂就越需要通过工具的手段来保证项目的维护性和增强代码的可读性。Vue.js 在做2.0重构的时候，在 ES2015 的基础上，除了 ESLint 保证代码风格之外，也引入了 Flow 做静态类型检查。之所以选择 Flow，**最根本原因作者在知乎提及过，还是在于工程上成本和收益的考量。** 大致体现在以下几点：

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

Flow 在 JavaScript 语法的基础上使用了一些注解（annotation）进行了扩展。因此浏览器无法正确的解读这些 Flow 相关的语法，我们必须在编译之后的代码中（最终发布的代码）将增加的 Flow 注解移除掉。具体方法需要看我们使用了什么样的编译工具。下面将说明一些 Vue 开发常用的编译工具：

**方式一：**Babel

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
**方式二：**flow-remove-types

如果我们既没有使用 Babel 作为语法糖编译器，那么可以使用 [flow-remove-types](https://github.com/flowtype/flow-remove-types) 这个工具在发布之前移除 Flow 代码。

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

#### Flow 在 Vue.js 源码中的应用

有时候我们想引用第三方库，或者自定义一些类型，但 Flow 并不认识，因此检查的时候会报错。为了解决这类问题，Flow 提出了一个 libdef 的概念，可以用来识别这些第三方库或者是自定义类型，而 Vue.js 也利用了这一特性。

在 Vue.js 的主目录下有 .flowconfig 文件， 它是 Flow 的配置文件。这其中的 [libs] 部分用来描述包含指定库定义的目录，默认是名为 flow-typed 的目录。

这里 [libs] 配置的是 flow，表示指定的库定义都在 flow 文件夹内。我们打开这个目录，会发现文件如下：
``` 
flow
├── compiler.js        # 编译相关
├── component.js       # 组件数据结构
├── global-api.js      # Global API 结构
├── modules.js         # 第三方库定义
├── options.js         # 选项相关
├── ssr.js             # 服务端渲染相关
├── vnode.js           # 虚拟 node 相关
```

可以看到，Vue.js 有很多自定义类型的定义，在阅读源码的时候，如果遇到某个类型并想了解它完整的数据结构的时候，可以回来翻阅这些数据结构的定义。

#### 小结

通过对 Flow 的认识，有助于我们阅读 Vue 的源码，并且这种静态类型检查的方式非常有利于大型项目源码的开发和维护。**此外，通过 Vue 重构，我们发现项目重构要么依赖规范，要么就得自己有绝对控制权，同时还要考量开发成本、项目收益以及整个团队的技术水平，并不是一味的什么火就用什么。**

### Rollup - 另一个前端模块化的打包工具

Rollup 是前端模块化的一个打包工具，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。简单地说，它可以从一个入口文件开始，将所有使用的模块根据命令或者根据 Rollup 配置文件打包成一个目标文件，并且 Rollup 会自动过滤掉那些没有被使用过的函数或变量，从而使代码最小化，如果想使用直接导入这一个目标文件即可，因此 Rollup 极其适合构建一个工具库。

这里提到 Rollup 的两个特别重要的特性，第一个就是它使用了 ES2015 的模板标准，这意味着我们可以直接使用 import 和 export 而不需要引入 babel。另一个重要特性叫做 tree-shaking，这个特性可以帮助我们将无用代码（即没有使用的代码）从最终的目标文件中过滤掉。举个简单的例子，我们在 foo.js 文件定义了 f1 和 f2 两个方法，然后在入口文件 index.js 只引入了 foo.js 文件中的 f1 方法，那么在最后打包 index.js 文件时，Rollup 就不会将 f2 方法打包到最终文件中（这个特性是基于 ES6 模块的静态分析的，也就是说，只有 export 而没有 import 的变量是不会被打包到最终代码中的）。

#### 为什么用前端模块化的打包工具 Rollup

之前 Vue 用 webpack 打包，还是会自带一个小型的动态 module 加载机制，并且每个文件是包在一个模块函数里的。Rollup 打包通过重命名 import binding 直接把所有文件的函数都放在同一个函数体里面... 所以最终出来的文件会小一些，并且初始化快个十几毫秒的样子。

#### 如何用前端模块化的打包工具 Rollup

关于如何使用前端模块化的打包工具 Rollup，这里就不做过多介绍了，可参考我之前写的一篇文章：[Rollup使用指南](/2018/08/04/rollup-tutorial.html)，更详细的使用文档可参考：[官网](https://www.rollupjs.com/guide/zh)。


#### Webpack 和 Rollup 有什么不同

Vue 从 1.0.10 开始就改用 Rollup 来打包。作者尤雨溪在知乎上也曾说过 使用 Rollup 只是用于 Vue 发布文件的构建，对用户使用没有直接影响。在这之前用 webpack 打包，还是会自带一个小型的动态 module 加载机制，并且每个文件是包在一个模块函数里的。Rollup 打包通过重命名 import binding 直接把所有文件的函数都放在同一个函数体里面... 所以最终出来的文件会小一些，并且初始化快个十几毫秒的样子。

Webpack 是目前使用最为火热的打包工具，没有之一，每月有数百万的下载量，为成千上万的网站和应用提供支持。相比之下，Rollup 并不起眼。但 Vue 并不孤单 – React，Ember，Preact，D3，Three.js，Moment 以及其他许多知名的库也使用 Rollup 。世界到底怎么了？为什么我们不能只有一个大众认可的 JavaScript 模块化打包工具？

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

通过对 Rollup 的认识，有助于我们了解 Vue 的构建以及源码目录结构。

## 项目介绍

上一章我们简单介绍了下 flow、Rollup 等前置知识，有兴趣的可以有针对性的学习它们。这一章我们真正的开始分析 Vue 源码，激动不激动？该章主要包括三小节：项目目录、源码构建、源码入口。

### 项目目录

Vue.js 的源码都在 src 目录下，其详细目录结构如下：
``` js
├── dist ---------------------------------------- 构建后的输出目录
├── examples ------------------------------------ Vue 开发的应用案例
├── flow ---------------------------------------- Flow 类型声明
├── packages ------------------------------------ 独立发布包的目录
├── scripts ------------------------------------- 构建相关的文件
│   ├── git-hooks ------------------------------- git钩子的目录
│   ├── alias.js -------------------------------- 别名配置文件
│   ├── build.js -------------------------------- Rollup 构建文件
│   ├── config.js ------------------------------- Rollup 构建配置的文件
│   ├── gen-release-note.js --------------------- 生成发布通知
│   ├── get-weex-version.js --------------------- 获取 weex 版本
│   ├── release-weex.sh ------------------------- 自动发布新版本weex脚本
│   ├── ci.sh ----------------------------------- 持续集成运行的脚本
│   ├── release.sh ------------------------------ 自动发布新版本脚本
├── src ----------------------------------------- 源码目录，我们主要剖析目录
│   ├── compiler -------------------------------- 编译相关，主要将 template 编译为 render 函数 
│   ├── core ------------------------------------ 核心代码，与平台无关的代码
│   │   ├── components -------------------------- 抽象出来的通用组件
│   │   ├── instance ---------------------------- Vue 构造函数设计相关的代码
│   │   ├── global-api -------------------------- Vue 构造函数挂载全局方法(静态方法)或属性的代码
│   │   ├── observer ---------------------------- 响应系统，包含数据观测的核心代码
│   │   ├── vdom -------------------------------- 虚拟DOM创建(creation)和打补丁(patching)的代码
├── ├── platforms ------------------------------- 平台特有的相关代码，不同平台的构建入口文件
│   │   ├── web --------------------------------- web平台
│   │   │   ├── entry-runtime.js ---------------- 不带编译器构建的入口
│   │   │   ├── entry-runtime-with-compiler.js -- 自带编译器构建的入口
│   │   │   ├── entry-compiler.js --------------- vue-template-compiler 包的入口文件
│   │   │   ├── entry-server-renderer.js -------- vue-server-renderer 包的入口文件
│   │   │   ├── entry-server-basic-renderer.js -- 输出 packages/vue-server-renderer/basic.js
│   │   ├── weex -------------------------------- 混合应用
├── ├── server ---------------------------------- 服务端渲染
│   ├── sfc ------------------------------------- .vue 文件解析
│   ├── shared ---------------------------------- 整个项目通用代码
├── test ---------------------------------------- 测试文件
├── .babelrc ------------------------------------ babel 配置文件
├── .editorconfig ------------------------------- 编辑器语法规范配置
├── .eslintignore ------------------------------- eslint 忽略配置
├── .eslintrc ----------------------------------- eslint 配置文件
├── .flowconfig --------------------------------- flow 的配置文件
├── .gitignore ---------------------------------- git 忽略配置
├── package-lock.json --------------------------- npm 加锁文件
├── package.json -------------------------------- 项目管理文件
├── README.md ----------------------------------- 项目文档
├── yarn.lock ----------------------------------- yarn 加锁文件
```
上述目录很是熟悉，根目录下 src 存放源码，test 存放单元测试，examples 作为应用案例等等，后续 Vue 团队会不会也采用 monorepo 项目组织方式这个目前不好说，但有可能。接下来我们对重点剖析的源码目录做一个简要分析：

#### compiler

compiler 目录包含 Vue.js 所有编译相关的代码。它包括把模板解析成 ast 语法树，ast 语法树优化，代码生成等功能。通常我们利用 vue-cli 去初始化我们的 Vue.js 项目的时候会询问我们用 Runtime Only 版本的还是 Runtime + Compiler 版本。下面我们来对比这两个版本：

**Runtime Only 版本：**我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。

**Runtime + Compiler 版本：**我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板，如下所示：
```js
// 需要编译器的版本
new Vue({
  template: '<div>{{ hi }}</div>'
})

// 这种情况不需要
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```
因为在 Vue.js 2.0 中，最终渲染都是通过 render 函数，如果写 template 属性，则需要编译成 render 函数，那么这个编译过程会发生运行时，所以需要带有编译器的版本。很显然，这个编译过程对性能会有一定损耗，所以通常我们更推荐使用 Runtime-Only 的 Vue.js。

#### core
core 目录包含了 Vue.js 的核心代码，包括内置组件、全局 API 封装，Vue 实例化、观察者、虚拟 DOM、工具函数等等。

#### platform
platform 目录是 Vue.js 的入口，2 个目录代表 2 个主要入口，分别打包成运行在 web 上和配合 weex 运行在 native 客户端上的 Vue.js。

#### server
server 目录主要用于服务端渲染。这部分代码是跑在服务端的 Node.js，不要和跑在浏览器端的 Vue.js 混为一谈。Vue.js 从 2.0 开始支持了服务端渲染，服务端渲染的主要工作是把组件渲染为服务器端的 HTML 字符串，然后将它们直接发送到浏览器，最后将静态标记"混合"为客户端上完全交互的应用程序。

#### sfc
sfc 目录主要用于把 .vue 文件内容解析成一个 JavaScript 对象，实际开发中我们一般用 webpack 完成该工作。

#### shared
shared 目录主要定义了一些共享的工具方法，这些工具方法不但适用于浏览器端的 Vue.js，同时也适用于服务端的 Vue.js。

#### 小结
从 Vue.js 的目录设计可以看到，作者把功能模块拆分的非常清楚，相关的逻辑放在一个独立的目录下维护，并且把复用的代码也抽成一个独立目录。

### 源码构建

Vue 源码是基于 Rollup 构建的，它的构建相关配置都在 scripts 目录下。

#### 构建命令

通常一个基于 NPM 托管的项目都会有一个 package.json 文件，实际上它是对项目的描述文件，它的内容是一个标准的 JSON 对象。我们通常会配置 script 字段作为 NPM 的构建命令，Vue 源码构建的脚本如下：
``` json
{
    // ...
    "main": "dist/vue.runtime.common.js",
    "module": "dist/vue.runtime.esm.js",
    "unpkg": "dist/vue.js",
    "jsdelivr": "dist/vue.js",
    "typings": "types/index.d.ts",
    "files": [
        "src",
        "dist/*.js",
        "types/*.d.ts"
    ],
    "sideEffects": false,
    "scripts": {
         // 构建完整版 umd 模块的 Vue
        "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev",
        // 构建运行时 cjs 模块的 Vue
        "dev:cjs": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-cjs",
        // 构建运行时 es 模块的 Vue
        "dev:esm": "rollup -w -c scripts/config.js --environment TARGET:web-runtime-esm",
        // 构建 web-server-renderer 包
        "dev:ssr": "rollup -w -c scripts/config.js --environment TARGET:web-server-renderer",
        // 构建 Compiler 包
        "dev:compiler": "rollup -w -c scripts/config.js --environment TARGET:web-compiler ",
        // ...
        "build": "node scripts/build.js",
        "build:ssr": "npm run build -- web-runtime-cjs,web-server-renderer",
        "build:weex": "npm run build -- weex",
        // ...
    },
    // ...
}
```

这里总共有 3 条命令，作用都是构建 Vue，后面 2 条是在第一条命令的基础上，添加一些环境参数。当在命令行运行`npm run build`的时候，实际上会执行`node scripts/build.js`，接下来我们就来看看它实际上是如何构建的。

#### 构建过程

我们首先打开构建命令对应的构建 JS 脚本，在`scripts/build.js`中：
``` js
// ...
let builds = require('./config').getAllBuilds()

// filter builds via command line arg
if (process.argv[2]) {
  const filters = process.argv[2].split(',')
  builds = builds.filter(b => {
    return filters.some(f => b.output.file.indexOf(f) > -1 || b._name.indexOf(f) > -1)
  })
} else {
  // filter out weex builds by default
  builds = builds.filter(b => {
    return b.output.file.indexOf('weex') === -1
  })
}

build(builds)

// ...
```
这段代码逻辑非常简单，先从配置文件读取配置，再通过命令行参数对构建配置做过滤，这样就可以构建出不同用途的 Vue.js 了。稍后我们再来看构建函数 build，我们先来看看配置文件，在`scripts/config.js`中：
``` js
const builds = {
  // Runtime only (CommonJS). Used by bundlers e.g. Webpack & Browserify
  'web-runtime-cjs': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.common.js'),
    format: 'cjs',
    banner
  },
  // Runtime+compiler CommonJS build (CommonJS)
  'web-full-cjs': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.common.js'),
    format: 'cjs',
    alias: { he: './entity-decoder' },
    banner
  },
  // Runtime only (ES Modules). Used by bundlers that support ES Modules,
  // e.g. Rollup & Webpack 2
  'web-runtime-esm': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.esm.js'),
    format: 'es',
    banner
  },
  // Runtime+compiler CommonJS build (ES Modules)
  'web-full-esm': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.esm.js'),
    format: 'es',
    alias: { he: './entity-decoder' },
    banner
  },
  // runtime-only build (Browser)
  'web-runtime-dev': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.js'),
    format: 'umd',
    env: 'development',
    banner
  },
  // runtime-only production build (Browser)
  'web-runtime-prod': {
    entry: resolve('web/entry-runtime.js'),
    dest: resolve('dist/vue.runtime.min.js'),
    format: 'umd',
    env: 'production',
    banner
  },
  // Runtime+compiler development build (Browser)
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
  // Runtime+compiler production build  (Browser)
  'web-full-prod': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.min.js'),
    format: 'umd',
    env: 'production',
    alias: { he: './entity-decoder' },
    banner
  },
  // ...
}
```
这里简单列举了一些 Vue 构建的配置，其他已省略，可以看出实际上这是一个用于 Rollup 构建配置的对象。接下来我们再看一下构建函数 build，在`scripts/build.js`中：
```js
// ...
build(builds)

function build (builds) {
  let built = 0
  const total = builds.length
  const next = () => {
    buildEntry(builds[built]).then(() => {
      built++
      if (built < total) {
        next()
      }
    }).catch(logError)
  }

  next()
}

function buildEntry (config) {
  const output = config.output
  const { file, banner } = output
  const isProd = /min\.js$/.test(file)
  return rollup.rollup(config)
    .then(bundle => bundle.generate(output))
    .then(({ code }) => {
      if (isProd) {
        var minified = (banner ? banner + '\n' : '') + uglify.minify(code, {
          output: {
            ascii_only: true
          },
          compress: {
            pure_funcs: ['makeMap']
          }
        }).code
        return write(file, minified, true)
      } else {
        return write(file, code)
      }
    })
}
```
上述关键的代码是` return rollup.rollup(config)`，可以看出这是通过 rollup 打包的，对于单个配置，它是遵循 Rollup 的构建规则的。其中 entry 属性表示构建的入口 JS 文件地址，dest 属性表示构建后的输出的 JS 文件地址，format 属性表示构建的格式，cjs 表示构建出来的文件遵循[CommonJS 规范](http://wiki.commonjs.org/wiki/Modules/1.1)，es 表示构建出来的文件遵循[ES Module 规范](http://exploringjs.com/es6/ch_modules.html)，umd 表示构建出来的文件遵循[UMD 规范](https://github.com/umdjs/umd)。

**下面我们以配置文件的`web-runtime-cjs`配置为例：**

构建的入口 JS 文件地址：

``` js
const builds = {
  // Runtime only (CommonJS). Used by bundlers e.g. Webpack & Browserify
    'web-runtime-cjs': {
        entry: resolve('web/entry-runtime.js'),
        dest: resolve('dist/vue.runtime.common.js'),
        format: 'cjs',
        banner
    },
    // ...
}
```

沿着`resolve`函数我们来看一下它的定义，在`scripts/config.js`中：

``` js
const aliases = require('./alias')
const resolve = p => {
  const base = p.split('/')[0]
  if (aliases[base]) {
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    return path.resolve(__dirname, '../', p)
  }
}
```
上述`resolve`函数实现非常简单：**它把传入的参数`p`通过`/`分割成数组并取数组第一个元素赋值给`base`。**在我们这个例子中，参数`p`的值是`web/entry-runtime.js`，那么`base`的值则为`web`。这里的`base`并不是实际的路径，它的实际路径是借助别名获取的，接下来我们来看一下别名配置的代码，在`scripts/alias`中：
``` js
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  entries: resolve('src/entries'),
  sfc: resolve('src/sfc')
}
```

很显然，这里`web`对应的真实的路径是`path.resolve(__dirname, '../', 'src/platforms/web')`，这个路径就是`src/platforms/web`，然后`resolve`函数通过`path.resolve(aliases[base], p.slice(base.length + 1)) `就得到了`Vue`源码的最终路径，它就是`src/platforms/web/entry-runtime.js`，因此，`web-runtime-cjs`配置对应的入口文件就是`src/platforms/web/entry-runtime.js`。

构建后的输出的 JS 文件地址：

``` js
const aliases = require('./alias')
const resolve = p => {
  const base = p.split('/')[0]
  if (aliases[base]) {
    return path.resolve(aliases[base], p.slice(base.length + 1))
  } else {
    return path.resolve(__dirname, '../', p)
  }
}

const builds = {
  // Runtime only (CommonJS). Used by bundlers e.g. Webpack & Browserify
    'web-runtime-cjs': {
        entry: resolve('web/entry-runtime.js'),
        dest: resolve('dist/vue.runtime.common.js'),
        format: 'cjs',
        banner
    },
    // ...
}

// alias.js
const path = require('path')

const resolve = p => path.resolve(__dirname, '../', p)

module.exports = {
  vue: resolve('src/platforms/web/entry-runtime-with-compiler'),
  compiler: resolve('src/compiler'),
  core: resolve('src/core'),
  shared: resolve('src/shared'),
  web: resolve('src/platforms/web'),
  weex: resolve('src/platforms/weex'),
  server: resolve('src/server'),
  entries: resolve('src/entries'),
  sfc: resolve('src/sfc')
}
```
由于别名配置里并没有`dist`别名配置，因此`dest`直接返回`path.resolve(__dirname,'../',dist/vue.runtime.common.js);`，因此构建后的输出的 JS 文件地址是`dist/vue.runtime.common.js`。

#### 小结

通过这一节的分析，我们可以了解到 Vue.js 的打包过程，也知道了不同作用和功能的 Vue.js 它们对应的入口以及最终编译生成的 JS 文件。尽管在实际开发过程中我们会用`Runtime Only`版本开发比较多，但为了分析 Vue 的编译过程，我们重点分析的源码是`Runtime + Compiler 的 Vue.js`。

### 源码入口

#### Vue 的定义

我们在源码构建一节讲到，在`web`应用下，我们来分析`Runtime + Compiler`构建出来的 Vue.js，它的入口是`src/platforms/web/entry-runtime-with-compiler.js`：

``` js
/* @flow */

import config from 'core/config'
import { warn, cached } from 'core/util/index'
import { mark, measure } from 'core/util/perf'

import Vue from './runtime/index'
import { query } from './util/index'
import { compileToFunctions } from './compiler/index'
import { shouldDecodeNewlines, shouldDecodeNewlinesForHref } from './util/compat'

const idToTemplate = cached(id => {
  const el = query(id)
  return el && el.innerHTML
})

const mount = Vue.prototype.$mount
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {

    // ...

    return mount.call(this, el, hydrating)
}

// ...

Vue.compile = compileToFunctions
export default Vue
```

由此可以看出，当我们在代码执行`import Vue from 'vue'`时，就是从这个入口来初始化 Vue 的。在这个入口 JS 的上方我们可以找到 Vue 的来源：`import Vue from './runtime/index'`，接下来我们来看一下这块儿的实现，在`src/platforms/web/runtime/index.js`中：
``` js
/* @flow */

import Vue from 'core/index'
import config from 'core/config'
import { extend, noop } from 'shared/util'
import { mountComponent } from 'core/instance/lifecycle'
import { devtools, inBrowser, isChrome } from 'core/util/index'

import {
  query,
  mustUseProp,
  isReservedTag,
  isReservedAttr,
  getTagNamespace,
  isUnknownElement
} from 'web/util/index'

import { patch } from './patch'
import platformDirectives from './directives/index'
import platformComponents from './components/index'

// install platform specific utils
Vue.config.mustUseProp = mustUseProp
Vue.config.isReservedTag = isReservedTag
Vue.config.isReservedAttr = isReservedAttr
Vue.config.getTagNamespace = getTagNamespace
Vue.config.isUnknownElement = isUnknownElement

// install platform runtime directives & components
extend(Vue.options.directives, platformDirectives)
extend(Vue.options.components, platformComponents)

// install platform patch function
Vue.prototype.__patch__ = inBrowser ? patch : noop

// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}

// ...

export default Vue
```

在这个 JS 的上方我们可以找到 Vue 的来源：`import Vue from 'core/index'`，剩下的都是对 Vue 这个对象的扩展，我们暂且不去分析，我们先来看一下关键代码实现的文件，在`src/core/index.js`中：
``` js
import Vue from './instance/index'
import { initGlobalAPI } from './global-api/index'
import { isServerRendering } from 'core/util/env'
import { FunctionalRenderContext } from 'core/vdom/create-functional-component'

initGlobalAPI(Vue)

Object.defineProperty(Vue.prototype, '$isServer', {
  get: isServerRendering
})

Object.defineProperty(Vue.prototype, '$ssrContext', {
  get () {
    /* istanbul ignore next */
    return this.$vnode && this.$vnode.ssrContext
  }
})

// expose FunctionalRenderContext for ssr runtime helper installation
Object.defineProperty(Vue, 'FunctionalRenderContext', {
  value: FunctionalRenderContext
})

Vue.version = '__VERSION__'

export default Vue
```
阿西吧，还没到！在这个 JS 的上方我们可以找到 Vue 的来源是`import Vue from './instance/index'`，在`src/core/instance/index.js`中，不过这里有一点需要特别说明下：`initGlobalAPI(Vue)`用于初始化全局 Vue API（我们稍后介绍）：

```js
import { initMixin } from './init'
import { stateMixin } from './state'
import { renderMixin } from './render'
import { eventsMixin } from './events'
import { lifecycleMixin } from './lifecycle'
import { warn } from '../util/index'

function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)
stateMixin(Vue)
eventsMixin(Vue)
lifecycleMixin(Vue)
renderMixin(Vue)

export default Vue
```
至此，我们终于看到了 Vue 的定义，可以看出它实际上就是一个用 Function 实现的类，我们只能通过 new Vue 去实例化它。接下来我们来分析一下上面遗留下来的问题。

#### initGlobalAPI

Vue.js 在整个初始化过程中，除了给它的原型 prototype 上扩展方法，还会给 Vue 这个对象本身扩展全局的静态方法，它的定义在`src/core/global-api/index.js`中：

``` js
export function initGlobalAPI (Vue: GlobalAPI) {
  // config
  const configDef = {}
  configDef.get = () => config
  if (process.env.NODE_ENV !== 'production') {
    configDef.set = () => {
      warn(
        'Do not replace the Vue.config object, set individual fields instead.'
      )
    }
  }
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive
  }

  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  extend(Vue.options.components, builtInComponents)

  initUse(Vue)
  initMixin(Vue)
  initExtend(Vue)
  initAssetRegisters(Vue)
}
```
这里是对 Vue 扩展的一些全局方法，有一点要注意的是，Vue.util 暴露的方法最好不要依赖，因为它可能经常会发生变化且不稳定的。

#### 小结

那么至此，我们应该对 Vue 是什么有一个直观的认识，它本质上就是一个用 Function 实现的 Class，然后在它的原型 prototype 以及它本身都扩展了一系列的方法和属性，至于 Vue 能做什么，它是怎么做的，我们会在后面的章节一一剖析它们。

## 基础

## 组件化
在`Vue.js`中，除了它内置的组件如`keep-alive`、`component`、`transition`、`transition-group`等，其它自定义组件在使用前必须注册。我们在开发过程中可能会遇到如下报错信息：
``` js
'Unknown custom element: <xxx> - did you register the component correctly?
 For recursive components, make sure to provide the "name" option.'
```
一般报这个错的原因都是我们使用了未注册的组件。Vue.js 提供了 2 种组件的注册方式：全局注册和局部注册。接下来我们从源码分析的角度来分析这两种注册方式。
### 组件注册

## 本文不断更新中


