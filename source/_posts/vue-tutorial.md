---
title: Vue 开发汇总
tags:
  - note
categories:
  - vue
date: 2018-04-05 23:19:18
---

## 教程
### 高阶组件
高阶组件(HOC)是 React 生态系统的常用词汇，React 中代码复用的主要方式就是使用高阶组件，并且这也是官方推荐的做法。而 Vue 中复用代码的主要方式是使用 mixins，并且在 Vue 中很少提到高阶组件的概念，这是因为在 Vue 中实现高阶组件并不像 React 中那样简单，原因在于 React 和 Vue 的设计思想不同，但并不是说在 Vue 中就不能使用高阶组件，只不过在 Vue 中使用高阶组件所带来的收益相对于 mixins 并没有质的变化。本篇文章主要从技术性的角度阐述 Vue 高阶组件的实现，且会从 React 与 Vue 两者的角度进行分析。

#### 从 React 说起
起初 React 也是使用 mixins 来完成代码复用的，比如为了避免组件不必要的重复渲染我们可以在组件中混入 PureRenderMixin：
```js
const PureRenderMixin = require('react-addons-pure-render-mixin')
const MyComponent = React.createClass({
  mixins: [PureRenderMixin]
})
```
后来 React 抛弃了这种方式，进而使用 shallowCompare：
```js
```