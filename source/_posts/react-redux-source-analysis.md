---
title: react-redux 源码全方位剖析
date: 2018-06-27 20:55:16
tags:
  - source
categories:
  - redux
---

### 前言
作为前端攻城狮，我们大多都有使用过 Redux，即便没有使用，我相信听肯定听说过。Redux 是一款可预测的状态管理框架，主要提供一个数据存储中心，供外部访问、修改等，因此 Redux 本身和 React 没有什么本质关系。那么我们如何比较优雅的在 React 使用 Redux？传统方法我们可以在应用初始化时，将 store 挂载在 window 上：`window.store = createStore(reducer)`，然后在需要的地方使用 Redux API：`store.getState()`、`store.dispatch`、`store.subscribe`。。。现在虽然各个子组件都能够访问 store 了，但这里 store 就变成了全局变量。由于全局变量有诸多的缺点，我们不妨换个思路，把 store 直接集成到 React 应用的顶层 props 里面，然后通过 React 钩子 Context，各个子组件就能访问到顶层 props，例如：
``` jsx
// 入口文件 index.js
ReactDOM.render(
	<WrapComponent store={store}>
		<App />
	</WrapComponent>,
	document.getElementById('root'))
);
```
现在各个子组件就能够访问到 store 了，这就是我们 react-redux 的设计思路。react-redux 库提供了顶层组件：Provider，然后通过 React 钩子 Context 向应用注入 store，同时它还利用 connect 高阶方法把 Redux API：`store.getState()`、`store.dispatch`、`store.subscribe`等封装起来，使子组件对 store 毫无感知，就好像没有 store 存在一样，然后根据 store state 和组件自身 props 计算得到新props。我们稍后会详细分析 react-redux 的具体实现，在分析源码之前，我们先看一下 react-redux 的源码结构。

### 源码结构
react-redux 的源码结构很简单，源码都在 src 目录下，其目录结构如下：
``` 
src
├── components ----------------------------------- 工具函数
│   ├── connectAdvanced.js ----------------------- git钩子的目录
│   ├── Provider.js ------------------------------ 别名配置文件
├── connect -------------------------------------- 加载 middleware
│   ├── connect.js ------------------------------- git钩子的目录
│   ├── mapDispatchToProps.js -------------------- 别名配置文件
│   ├── mapStateToProps.js ----------------------- Rollup 构建文件
│   ├── mergeProps.js ---------------------------- Rollup 构建配置的文件
│   ├── selectorFactory.js ----------------------- 生成发布通知
│   ├── verifySubselectors.js -------------------- 获取 weex 版本
│   ├── wrapMapToProps.js ------------------------ 自动发布新版本weex脚本
├── utils ---------------------------------------- 生成将 action creator 包裹在 dispatch 里的函数
│   ├── isPlainObject.js ------------------------- git钩子的目录
│   ├── PropTypes.js ----------------------------- 别名配置文件
│   ├── shallowEqual.js -------------------------- Rollup 构建文件
│   ├── Subscription.js -------------------------- Rollup 构建配置的文件
│   ├── verifyPlainObject.js --------------------- 生成发布通知
│   ├── warning.js ------------------------------- 获取 weex 版本
│   ├── wrapActionCreators.js -------------------- 自动发布新版本weex脚本
├── index.js ------------------------------------- 入口 js
```
### 源码入口
index.js 是整个代码的入口，其代码如下：
``` js
import Provider, { createProvider } from './components/Provider'
import connectAdvanced from './components/connectAdvanced'
import connect from './connect/connect'

export { Provider, createProvider, connectAdvanced, connect }
```
入口代码很简单，暴露 Provider，createProvider，connectAdvanced，connect 这几个接口给开发者使用，接下来我们逐一解析这几个 API。

### Provider
``` js
import { Component, Children } from 'react'
import PropTypes from 'prop-types'
import { storeShape, subscriptionShape } from '../utils/PropTypes'
import warning from '../utils/warning'

let didWarnAboutReceivingStore = false
function warnAboutReceivingStore() {
  if (didWarnAboutReceivingStore) {
    return
  }
  didWarnAboutReceivingStore = true

  warning(
    '<Provider> does not support changing `store` on the fly. ' +
    'It is most likely that you see this error because you updated to ' +
    'Redux 2.x and React Redux 2.x which no longer hot reload reducers ' +
    'automatically. See https://github.com/reduxjs/react-redux/releases/' +
    'tag/v2.0.0 for the migration instructions.'
  )
}

export function createProvider(storeKey = 'store') {
    const subscriptionKey = `${storeKey}Subscription`

    class Provider extends Component {
        getChildContext() {
          return { [storeKey]: this[storeKey], [subscriptionKey]: null }
        }

        constructor(props, context) {
          super(props, context)
          this[storeKey] = props.store;
        }

        render() {
          return Children.only(this.props.children)
        }
    }

    if (process.env.NODE_ENV !== 'production') {
      Provider.prototype.componentWillReceiveProps = function (nextProps) {
        if (this[storeKey] !== nextProps.store) {
          warnAboutReceivingStore()
        }
      }
    }

    Provider.propTypes = {
        store: storeShape.isRequired,
        children: PropTypes.element.isRequired,
    }
    Provider.childContextTypes = {
        [storeKey]: storeShape.isRequired,
        [subscriptionKey]: subscriptionShape,
    }

    return Provider
}

export default createProvider()
```


