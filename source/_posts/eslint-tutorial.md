---
title: ESlint 开发汇总
date: 2018-05-12 21:05:14
tags: 
    - note
categories:
    - eslint
---

## 前言
ESLint 最初是由 Nicholas C. Zakas 于2013年6月创建的开源项目。它是在 ECMAScript/JavaScript 代码中识别和报告模式匹配的工具，目标是保证代码的一致性和避免错误。在许多方面，它和 JSLint、JSHint 相似，除了少数的例外：
**一：**ESLint 使用[Espree](https://github.com/eslint/espree)解析 JavaScript。
**二：**ESLint 使用 AST 去分析代码中的模式。
**三：**ESLint 是完全插件化的。每一个规则都是一个插件并且我们可以在运行时添加更多的规则。

## 安装
在安装 ESLint 之前请确保满足以下几个条件：Node.js（> = 4.x），npm version 2+。有两种方式安装 ESLint： 全局安装和本地安装。我们建议全局安装 ESLint：
``` bash
$ npm install -g eslint
```
接下来设置一个 eslint 配置文件：
``` bash
$ eslint --init
```

## 配置规则

### root
默认情况下，ESLint 会在所有父级目录里寻找配置文件，一直到根目录。如果我们想要所有项目都遵循一个特定的约定时，这将会很有用，但有时候会导致意想不到的结果。为了将 ESLint 限制到一个特定的项目，在我们项目根目录下的`package.json`文件或者`.eslintrc.*`文件里的`eslintConfig`字段下设置`"root": true`。ESLint 一旦发现配置文件中有`"root": true`，它就会停止在父级目录中寻找。

### parserOptions 
解析器选项可以在`.eslintrc.*`文件使用`parserOptions`属性设置。可用的选项有：
`sourceType` - 设置为`"script"`（默认）或`"module"`（如果我们的代码是 ECMAScript 模块)。

### parser
指定一个不同的解析器，以下解析器与 ESLint 兼容：
**1、**[Esprima](https://github.com/jquery/esprima)
**2、**[Babel-ESLint](https://github.com/babel/babel-eslint)一个对Babel解析器的包装，使其能够与 ESLint 兼容。
**3、**[typescript-eslint-parser](https://github.com/eslint/typescript-eslint-parser)一个把 TypeScript 转换为 ESTree 兼容格式的解析器

注意，在使用自定义解析器时，为了让 ESLint 在处理非 ECMAScript 5 特性时正常工作，配置属性 parserOptions 仍然是必须的。解析器会被传入 parserOptions，但是不一定会使用它们来决定功能特性的开关。

### env
环境变量，可用的环境包括：
**browser：**浏览器环境中的全局变量。
**node：**Node.js 全局变量和 Node.js 作用域。
**jest：**Jest 全局变量。

### extends
继承一些标准库

## 常见问题

### eslint 支持async、await

``` bash
$ npm install babel-eslint --save-dev 
```
在`eslint`的配置文件中增加`"parser": "babel-eslint"`。

### Do not use 'new' for side effects

代码如下：
```javascript
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
```

报错：

![not-use-new.png](/images/eslint-tutorial/not-use-new.png)

原因：刪除了以下注释。这句注释可以绕过规则检测：

```javascript
/* eslint-disable no-new */
```

在new Vue()上方加上句注釋即可：
```javascript
/* eslint-disable no-new */
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: { App }
})
```

### vue-cli构建的项目，eslint一直报CRLF/LF的linebreak错误

如题，vue在构建项目的时候选择了airbnb规则，同时项目构建后被windows的unix bash工具pull并且push过，这之后在windows上进行开发，就开始一直报

```javascript
Expected linebreaks to be 'CRLF' but found 'LF'
```

这样的错误，后经查是一种强制统一方式，并且解决方法是

```javascript
linebreak-style: ["error", "windows"]
```

强制使用windows方式，我将之添加到了项目根目录下的 .eslintrc.js 文件中的rule字段下：
```javascript
// add your custom rules here
  'rules': {
    // don't require .vue extension when importing
    'import/extensions': ['error', 'always', {
      'js': 'never',
      'vue': 'never'
    }],
    // allow optionalDependencies
    'import/no-extraneous-dependencies': ['error', {
      'optionalDependencies': ['test/unit/index.js']
    }],
    // try to fix the line break problem
    'linebreak-style': ["error", "windows"],
    // allow debugger during development
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0
  }
```

结果无效，现有问题二个：
* 是否是因为系统环境不同而造成了某种强制转换才会引发如上的错误？
* 如何选择性的关闭eslint某个功能（linebreak检查）？

问题1:
*不同的操作系统下，甚至是不同编辑器，不同工具处理过的文件可能都会导致换行符的改变。


问题2：
* 项目根目录下有.eslintrc.js文件，在配置文件中修改rule配置项，如下：

```javascript
// 统一换行符，"\n" unix(for LF) and "\r\n" for windows(CRLF)，默认unix
// off或0: 禁用规则
'linebreak-style': 'off'
```
