---
title: redux 源码全方位剖析
date: 2018-06-29 23:11:43
tags:
    - redux
categories:
    - react
---

`版本：v4.0.0`

### 前言
受2014年Facebook的`Flux架构模式`以及`函数式编程语言Elm`启发，Dan Abramov在2015年创建了 Redux。很快，Redux因其体小精悍（只有2kB）且没有任何依赖短时间内成为最热门的前端架构。

Redux 是可预测的状态管理框架，它很好的解决多交互，多数据源的诉求。Redux 设计之初，作者就严格遵循三个设计理念原则：
**单一数据源：**整个应用的 state 都被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。store 可以看做是数据存储的一个容器，在这个容器里面，只会维护唯一一个 state tree。store 会给定4种基础操作API：`dispatch(action)，getState()，replaceReducer(nextReducer)，subscribe(listener)`。根据单一数据源原则，所有数据会通过`store.getState()`方法调用获取。
**state只读：**根据 state 只读原则，数据变更会通过 store，dispatch(action) 方法，Action 可以理解为变更数据的信息载体，type 是变更数据的唯一标志，payload 是用来携带需要变更的数据，格式大致为：`const action = { type: 'xxx', payload: 'yyy' }`；Reducer 是个纯函数，负责根据 action.type 获取需要变更的数据，然后计算 state 数值。格式为：`reducer: prevState => action => newState`。
**使用纯函数变更state值：**Reducer 只是一些纯函数，它接收先前的 state 和 action，并返回新的 state。

正常的一个同步数据流为：view 层触发 actionCreator，actionCreator 通过 store.dispatch(action) 方法变更 reducer。但是面对多种多样的业务场景，同步数据流方式显然无法满足。对于改变reducer的异步数据操作，就需要用到中间件的概念，如图所示：

![img1.png](/images/redux-source-analysis/img1.jpeg)

### 源码结构
Redux 的源码结构很简单，源码都在 src 目录下，其目录结构如下：
``` 
src
├── utils ---------------------------------------- 工具函数
├── applyMiddleware.js --------------------------- 加载 middleware
├── bindActionCreators.js ------------------------ 生成将 action creator 包裹在 dispatch 里的函数
├── combineReducers.js --------------------------- 合并 reducer 函数
├── compose.js ----------------------------------- 组合函数
├── createStore.js ------------------------------- 创建一个 Redux store 来储存应用中所有的 state
├── index.js ------------------------------------- 入口 js
```

### 源码入口
index.js 是整个代码的入口，其代码如下：
``` js
import createStore from './createStore'
import combineReducers from './combineReducers'
import bindActionCreators from './bindActionCreators'
import applyMiddleware from './applyMiddleware'
import compose from './compose'
import warning from './utils/warning'
import __DO_NOT_USE__ActionTypes from './utils/actionTypes'

function isCrushed() {}

if (
    process.env.NODE_ENV !== 'production' &&
    typeof isCrushed.name === 'string' &&
    isCrushed.name !== 'isCrushed'
) {
  warning(
    'You are currently using minified code outside of NODE_ENV === "production". ' +
      'This means that you are running a slower development build of Redux. ' +
      'You can use loose-envify (https://github.com/zertosh/loose-envify) for browserify ' +
      'or setting mode to production in webpack (https://webpack.js.org/concepts/mode/) ' +
      'to ensure you have the correct code for your production build.'
  )
}

export {
  createStore,
  combineReducers,
  bindActionCreators,
  applyMiddleware,
  compose,
  __DO_NOT_USE__ActionTypes
}

```
入口代码很简单，首先`isCrushed`函数主要是为了验证在非生产环境下`Redux`是否被压缩？如果被压缩了，`isCrushed.name !== 'isCrushed'` 就等于 `true`，这样就会给开发者一个`warn`提示。最后暴露`createStore`、`combineReducers`、`bindActionCreators`、`applyMiddleware`、`compose` 这几个接口给开发者使用，接下来我们逐一解析这几个 API。

### createStore.js
createStore.js 是 Redux 最重要的一个 API ，它负责创建一个 Redux store 来储存应用中所有的 state，整个应用中应有且仅有一个 store。现在我们来看一下 createStore 源代码：
``` js
import $$observable from 'symbol-observable'

// 私有 action
import ActionTypes from './utils/actionTypes'
import isPlainObject from './utils/isPlainObject'

export default function createStore(reducer, preloadedState, enhancer) {

    // 判断接受的参数个数，来指定 reducer、preloadedState 和 enhancer
    if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
        enhancer = preloadedState
        preloadedState = undefined
    }

    // 如果 enhancer 存在且是个合法的函数，就调用 enhancer，否则抛出错误提示
    if (typeof enhancer !== 'undefined') {
        if (typeof enhancer !== 'function') {
            throw new Error('Expected the enhancer to be a function.')
        }

        return enhancer(createStore)(reducer, preloadedState)
    }

    if (typeof reducer !== 'function') {
        throw new Error('Expected the reducer to be a function.')
    }
    // 储存当前的 currentReducer
    let currentReducer = reducer
    // 储存当前的状态
    let currentState = preloadedState
    // 储存当前的监听函数列表
    let currentListeners = []
    // 储存下一个监听函数列表
    let nextListeners = currentListeners
    let isDispatching = false

    // 这个函数可以根据当前监听函数的列表生成新的下一个监听函数列表引用
    function ensureCanMutateNextListeners() {
        if (nextListeners === currentListeners) {
            nextListeners = currentListeners.slice()
        }
    }

  // 读取由 store 管理的状态树
  function getState() {
    if (isDispatching) {
      throw new Error(
        'You may not call store.getState() while the reducer is executing. ' +
          'The reducer has already received the state as an argument. ' +
          'Pass it down from the top reducer instead of reading it from the store.'
      )
    }

    return currentState
  }

  function subscribe(listener) {
    // 判断传入的参数是否为函数
    if (typeof listener !== 'function') {
      throw new Error('Expected the listener to be a function.')
    }

    if (isDispatching) {
      throw new Error(
        'You may not call store.subscribe() while the reducer is executing. ' +
          'If you would like to be notified after the store has been updated, subscribe from a ' +
          'component and invoke store.getState() in the callback to access the latest state. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    let isSubscribed = true

    ensureCanMutateNextListeners()
    nextListeners.push(listener)

    return function unsubscribe() {
      if (!isSubscribed) {
        return
      }

      if (isDispatching) {
        throw new Error(
          'You may not unsubscribe from a store listener while the reducer is executing. ' +
            'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
        )
      }

      isSubscribed = false

      ensureCanMutateNextListeners()
      const index = nextListeners.indexOf(listener)
      nextListeners.splice(index, 1)
    }
  }

  function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }
    // 判断 action 是否有 type｛必须｝ 属性
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }
    // 如果正在 dispatch 则抛出错误
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }
    // 对抛出 error 的兼容，但是无论如何都会继续执行 isDispatching = false 的操作
    try {
      isDispatching = true
      // 使用 currentReducer 来操作传入 当前状态和 action，放回处理后的状态
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }

  // 判断参数是否是函数类型
  function replaceReducer(nextReducer) {
    if (typeof nextReducer !== 'function') {
      throw new Error('Expected the nextReducer to be a function.')
    }

    currentReducer = nextReducer
    dispatch({ type: ActionTypes.REPLACE })
  }

  function observable() {
    const outerSubscribe = subscribe
    return {
      subscribe(observer) {
        if (typeof observer !== 'object' || observer === null) {
          throw new TypeError('Expected the observer to be an object.')
        }

        function observeState() {
          if (observer.next) {
            observer.next(getState())
          }
        }

        observeState()
        const unsubscribe = outerSubscribe(observeState)
        return { unsubscribe }
      },

      [$$observable]() {
        return this
      }
    }
  }

  dispatch({ type: ActionTypes.INIT })

  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable
  }
}
```
这里我们首先要讲一下`ActionTypes`对象，它是 Redux 的私有 action，不允许外界触发，用来初始化 store 的状态树和改变 reducers 后初始化 store 的状态树。接下来我们从不同角度着重来讲一下 createStore 函数：

#### 参数
它可以接受三个参数：reducer、preloadedState、enhancer：
**reducer：**函数，返回下一个状态，接受两个参数：当前状态 和 触发的 action；
**preloadedState：**它是 state 的初始值，可以随意指定，比如服务端渲染的初始状态，但是如果使用 combineReducers 来生成 reducer，那必须保持状态对象的 key 和 combineReducers 中的 key 相对应，另外实际上它并不仅仅是扮演着一个 initialState 的角色，如果我们第二个参数是函数类型，createStore 会认为我们忽略了 preloadedState 而传入了一个enhancer；
**enhancer：**可选参数，一个组合 store creator 的高阶函数，可以翻译成 store 的增强器，顾名思义，就是增强 store 的功能。一般指定为第三方的中间件，时间旅行，持久化等等，返回一个新的强化过的 store creator，这个函数通常用 Redux 提供的 applyMiddleware 函数来生成。

根据传入参数的个数和类型，判断 reducer、preloadedState、enhancer。

#### 返回值
调用完函数的返回值：dispatch、subscribe、getState、replaceReducer 和 [$$observable]，这就是我们开发中主要使用的几个接口。

#### enhancer
如果`enhancer`参数存在且是个合法的函数，那么就调用`enhancer`函数。`enhancer`实际上是一个高阶函数，它的参数是创建`store`的函数`createStore`，返回值是一个可以创建功能更加强大的`store`的函数(enhanced store creator)，这和 React 中的高阶组件的概念很相似。store enhancer 函数的结构一般如下：
``` js
function enhancerCreator() {
  return createStore => (...args) => {
    // do something based old store
    // return a new enhanced store
  }
}
```
注意，`enhancerCreator`是用于创建`enhancer store`的函数，也就是说`enhancerCreator`的执行结果才是一个`enhancer store`。`...args`参数代表创建`store`所需的参数，也就是`createStore`接收的参数，实际上就是`（reducer, [preloadedState], [enhancer]）`。

现在，我们来创建一个`enhancer store`，用于输出发送的`action`的信息和`state`的变化：
``` js
// logging.js（store enhancer）
export default function logging() {
  return createStore => (reducer, initialState, enhancer) => {
    const store = createStore(reducer, initialState, enhancer)
    function dispatch(action) {
      console.log(`dispatch an action: ${JSON.stringify(action)}`);
      const res = store.dispatch(action);
      const newState = store.getState();
      console.log(`current state: ${JSON.stringify(newState)}`);
      return res;
    }
    return {...store, dispatch}
  }
}
```
`logging()`改变了`store dispatch`的默认行为，在每次发送`action`前后，都会输出日志信息，然后在创建`store`上，使用`logging()`这个store enhancer:
``` js
// store.js
import { createStore, combineReducers } from 'redux';
import * as reducer from '../reducer';
import logging from '../logging';

//创建一个 Redux store 来以存放应用中所有的 state，应用中应有且仅有一个 store。

var store = createStore(
	combineReducers(reducer),
	logging()
);

export default store;
```

#### getState
``` js
// 读取由 store 管理的状态树
function getState() {
  if (isDispatching) {
    throw new Error(
      'You may not call store.getState() while the reducer is executing. ' +
        'The reducer has already received the state as an argument. ' +
        'Pass it down from the top reducer instead of reading it from the store.'
    )
  }

  return currentState
}
```
这个函数可以获取当前的状态，createStore 中的 currentState 储存当前的状态树，这是一个闭包，这个参数会持久存在，并且所有的操作状态都是改变这个引用，getState 函数返回当前的 currentState。

#### subscribe
``` js
function subscribe(listener) {
  // 判断传入的参数是否为函数
  if (typeof listener !== 'function') {
    throw new Error('Expected the listener to be a function.')
  }

  if (isDispatching) {
    throw new Error(
      'You may not call store.subscribe() while the reducer is executing. ' +
        'If you would like to be notified after the store has been updated, subscribe from a ' +
        'component and invoke store.getState() in the callback to access the latest state. ' +
        'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
    )
  }

  let isSubscribed = true

  ensureCanMutateNextListeners()
  nextListeners.push(listener)

  return function unsubscribe() {
    if (!isSubscribed) {
      return
    }

    if (isDispatching) {
      throw new Error(
        'You may not unsubscribe from a store listener while the reducer is executing. ' +
          'See https://redux.js.org/api-reference/store#subscribe(listener) for more details.'
      )
    }

    isSubscribed = false

    ensureCanMutateNextListeners()
    const index = nextListeners.indexOf(listener)
    nextListeners.splice(index, 1)
  }
}
```
这个函数可以给 store 的状态添加订阅监听函数，一旦调用`dispatch`，所有的监听函数就会执行；`nextListeners`就是储存当前监听函数的列表，调用`subscribe`，传入一个函数作为参数，那么就会给`nextListeners`列表`push`这个函数；同时调用`subscribe`函数会返回一个`unsubscribe`函数，用来解绑当前传入的函数，同时在`subscribe`函数定义了一个`isSubscribed`标志变量来判断当前的订阅是否已经被解绑，解绑的操作就是从`nextListeners`列表中删除当前的监听函数。

#### dispatch
``` js
function dispatch(action) {
    if (!isPlainObject(action)) {
      throw new Error(
        'Actions must be plain objects. ' +
          'Use custom middleware for async actions.'
      )
    }
    // 判断 action 是否有 type｛必须｝ 属性
    if (typeof action.type === 'undefined') {
      throw new Error(
        'Actions may not have an undefined "type" property. ' +
          'Have you misspelled a constant?'
      )
    }
    // 如果正在 dispatch 则抛出错误
    if (isDispatching) {
      throw new Error('Reducers may not dispatch actions.')
    }
    // 对抛出 error 的兼容，但是无论如何都会继续执行 isDispatching = false 的操作
    try {
      isDispatching = true
      // 使用 currentReducer 来操作传入 当前状态和 action，放回处理后的状态
      currentState = currentReducer(currentState, action)
    } finally {
      isDispatching = false
    }

    const listeners = (currentListeners = nextListeners)
    for (let i = 0; i < listeners.length; i++) {
      const listener = listeners[i]
      listener()
    }

    return action
  }
```

这个函数是用来触发状态改变的，它接受一个 action 对象作为参数，然后 reducer 根据 action 的属性以及当前 store 的状态来生成一个新的状态，赋予当前状态，改变 store 的状态；即`currentState = currentReducer(currentState, action)`；这里的`currentReducer`是一个函数，它接受两个参数：当前状态 和 action，然后返回计算出来的新的状态；然后遍历`nextListeners`列表，调用每个监听函数。

#### replaceReducer
``` js
// 判断参数是否是函数类型
function replaceReducer(nextReducer) {
  if (typeof nextReducer !== 'function') {
    throw new Error('Expected the nextReducer to be a function.')
  }

  currentReducer = nextReducer
  dispatch({ type: ActionTypes.REPLACE })
}
```
这个函数可以替换 store 当前的 reducer 函数，首先直接用`currentReducer = nextReducer`替换；然后`dispatch({ type: ActionTypes.INIT })`，用来初始化替换后 reducer 生成的初始化状态并且赋予 store 的状态。

#### observable
``` js
function observable() {
  const outerSubscribe = subscribe
  return {
    subscribe(observer) {
      if (typeof observer !== 'object' || observer === null) {
        throw new TypeError('Expected the observer to be an object.')
      }

      function observeState() {
        if (observer.next) {
          observer.next(getState())
        }
      }

      observeState()
      const unsubscribe = outerSubscribe(observeState)
      return { unsubscribe }
    },

    [$$observable]() {
      return this
    }
  }
}
```
对于这个函数，是不直接暴露给开发者的，它提供了给其他观察者模式/响应式库的交互操作。

#### 初始化 store 的状态
最后执行`dispatch({ type: ActionTypes.INIT })`，用来根据 reducer 初始化 store 的状态。

### compose.js
`compose`可以接受一组函数参数，然后从右到左来组合多个函数（这是函数式编程中的方法），最后返回一个组合函数。现在我们来看一下 compose 源代码：
``` js
/**
 * For example, compose(f, g, h) is identical to doing
 * (...args) => f(g(h(...args))).
 */
export default function compose(...funcs) {
  if (funcs.length === 0) {
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }

  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}
```
`compose`其作用是把一系列的函数，组装生成一个新的函数，并且从后到前，后面参数的执行结果作为其前一个的参数，当需要把多个 store 增强器 依次执行的时候，需要用到它。

#### 参数
**(...funcs)：**需要合成的多个函数。每个函数都接收一个函数作为参数，然后返回一个函数。

#### 返回值
**(Function)：**从右到左把接收到的函数合成后的最终函数。

### applyMiddleware.js
> It provides a third-party extension point between dispatching an action, and the moment it reaches the reducer.

这是 redux 作者 Dan 对 middleware 的描述，middleware 提供了一个分类处理 action 的机会，在 middleware 中我们可以检阅每一个流过的 action，挑选出特定类型的 action 进行相应操作，给我们改变 action 的机会。
Redux middleware 的设计十分特殊，是一个层层包裹的匿名函数，实际上这是函数式编程中的柯里化，一种使用匿名单参数函数来实现多参数函数的方法，柯里化的 middleware 结构好处在于：
**一：**易串联，柯里化函数具有延迟执行的特性，通过不断柯里化形成的 middleware 可以累积参数，配合组合的方式，很容易形成 pipeline 来处理数据流。
**二：**共享 store，在 applyMiddleware 执行过程中，store 还是旧的，但是因为闭包的存在，applyMiddleware 完成后，所有的 middlewares 内部拿到的 store 是最新且相同的。

redux 提供了 applyMiddleware 这个 api 来加载 middleware。现在我们来看一下 applyMiddleware 源代码：
``` js
import compose from './compose'

export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }
    // 暴漏 getState 和 dispatch 供第三方中间件使用
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // middlewareAPI 作为每个 middleware 的参数依次执行一遍，最终返回函数组成的数组
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 用 compose 组合函数生成新的 dispatch
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
由上我们可以发现 applyMiddleware 的结构也是一个多层柯里化的函数，借助 compose，applyMiddleware 可以用来和其他插件一起加强 createStore 函数。

#### 参数
我们在 createStore 小节中其实就用提及过 applyMiddleware：
``` js
// 如果 enhancer 存在且是个合法的函数，就调用 enhancer，否则抛出错误提示
if (typeof enhancer !== 'undefined') {
  if (typeof enhancer !== 'function') {
    throw new Error('Expected the enhancer to be a function.')
  }

  return enhancer(createStore)(reducer, preloadedState)
}
```
这里 enhancer 其实就等于 applyMiddleware(mid1, mid2, mid3, ...)，因此我们创建一个 store 实际上就变成如下方式了：
``` js
applyMiddleware(mid1, mid2, mid3, ...)(createStore)(reducer, preloadedState);
```
由上述代码可知 applyMiddleware 陆续可以接受四个参数：
**[md1, mid2, mid3, ...]：** middlewares 数组；
**createStore：**Redux 原生的 createStore；
**reducer：**函数，返回下一个状态；
**preloadedState：**state 的初始值。
接下来，我们看一下 applyMiddleware 用这些参数都做了什么？
``` js
const store = createStore(...args)
const middlewareAPI = {
  getState: store.getState,
  dispatch: (...args) => dispatch(...args)
}

const chain = middlewares.map(middleware => middleware(middlewareAPI))
dispatch = compose(...chain)(store.dispatch)
```
applyMiddleware 利用 createStore 和 (reducer, preloadedState) 创建了一个 store，然后 store 的 getState 方法和 dispatch 方法又分别赋值给 middlewareAPI 变量，紧接着用 middlewareAPI 作为每个 middleware 的参数依次执行一遍，执行完后，最终获得数组 chain：[f1, f2, ..., fn] 交给组合函数 compose 处理。compose 可以接受一组函数参数，然后从右到左来组合多个函数（这是函数式编程中的方法），最后返回一个组合函数，例如：
``` js
// 调用
dispatch = compose(f, g, h)(store.dispatch)
// 返回
dispatch = f(g(h(store.dispatch)))
```
这样通过调用新的 dispatch，每个 middleware 的代码就可以依次执行了。

#### 返回值
**store：**原来的store；
**dispatch：**改变后的dispatch。
### combineReducers.js
Reducer 是管理 state 的一个模块，它主要做的事情就是当项目初始化时，返回 initalState，当用户用操作时，它会根据 action 进行相应地更新。需要注意的是它是一个纯函数，换言之，它不会改变传入的 state。现在我们来看一下 combineReducers 源码（源码有删减，删除了一些验证代码）：
``` js
import ActionTypes from './utils/actionTypes'
import warning from './utils/warning'
import isPlainObject from './utils/isPlainObject'

export default function combineReducers(reducers) {
  // 根据 reducers 生成最终合法的 finalReducers：value 为 函数
  const reducerKeys = Object.keys(reducers)
  const finalReducers = {}
  for (let i = 0; i < reducerKeys.length; i++) {
    const key = reducerKeys[i]

    if (process.env.NODE_ENV !== 'production') {
      if (typeof reducers[key] === 'undefined') {
        warning(`No reducer provided for key "${key}"`)
      }
    }

    if (typeof reducers[key] === 'function') {
      finalReducers[key] = reducers[key]
    }
  }
  const finalReducerKeys = Object.keys(finalReducers)

  let unexpectedKeyCache
  if (process.env.NODE_ENV !== 'production') {
    unexpectedKeyCache = {}
  }

  let shapeAssertionError
  try {
    assertReducerShape(finalReducers)
  } catch (e) {
    shapeAssertionError = e
  }

  // 返回最终生成的 reducer
  return function combination(state = {}, action) {
    if (shapeAssertionError) {
      throw shapeAssertionError
    }

    if (process.env.NODE_ENV !== 'production') {
      const warningMessage = getUnexpectedStateShapeWarningMessage(
        state,
        finalReducers,
        action,
        unexpectedKeyCache
      )
      if (warningMessage) {
        warning(warningMessage)
      }
    }

    let hasChanged = false
    const nextState = {}
    for (let i = 0; i < finalReducerKeys.length; i++) {
      const key = finalReducerKeys[i]
      const reducer = finalReducers[key]
      const previousStateForKey = state[key]
      const nextStateForKey = reducer(previousStateForKey, action)
      if (typeof nextStateForKey === 'undefined') {
        const errorMessage = getUndefinedStateErrorMessage(key, action)
        throw new Error(errorMessage)
      }
      nextState[key] = nextStateForKey
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    // 遍历一遍验证下是否改变，然后返回原有状态值或者新的状态值
    return hasChanged ? nextState : state
  }
}
```
该函数最终返回 combination 函数，它就是真正 createStore 函数的 reducer，接受一个初始化状态和一个 action 参数；该函数每次调用大致执行以下几个操作：
**1、**`for (let i = 0; i < finalReducerKeys.length; i++) { ... }：`遍历 finalReducer（有效的 reducer 列表）；
**2、**`var previousStateForKey = state[key]：`当前遍历项的之前状态，看到这里就应该明白传入的 `reducers` 组合为什么 `key` 要和 store 里面的 state 的 `key` 相对应了；
**3、**`var nextStateForKey = reducer(previousStateForKey, action)：`当前遍历项的下一个状态；
**4、**`nextState[key] = nextStateForKey：`将 当前遍历项的下一个状态添加到 nextState；
**5、**`hasChanged = hasChanged || nextStateForKey !== previousStateForKey：`判断状态是否改变；
**6、**`return hasChanged ? nextState : state：`如果没有改变就返回原有状态，如果改变了就返回新生成的状态对象。

#### 参数
**reducers (Object): **一个对象，它的值（value）对应不同的 reducer 函数，这些 reducer 函数后面会被合并成一个。

#### 返回值
**(Function)：**它是真正 createStore 函数的 reducer，接受一个初始化状态和一个 action 参数；每次调用的时候会去遍历 finalReducer（有效的 reducer 列表），然后调用列表中每个 reducer，最终构造一个与 reducers 对象结构相同的 state 对象。

### bindActionCreators.js
Redux 中的 bindActionCreators 是通过 dispatch 将 action 包裹起来，这样就可以通过 bindActionCreators 创建方法调用 dispatch(action)。现在我们来看一下 bindActionCreators 源代码：
``` js
function bindActionCreator(actionCreator, dispatch) {
  return function() {
    return dispatch(actionCreator.apply(this, arguments))
  }
}

export default function bindActionCreators(actionCreators, dispatch) {
  // 如果是一个函数，直接返回一个 bindActionCreator 函数，即调用 dispatch 触发 action
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  if (typeof actionCreators !== 'object' || actionCreators === null) {
    throw new Error(
      `bindActionCreators expected an object or a function, instead received ${
        actionCreators === null ? 'null' : typeof actionCreators
      }. ` +
        `Did you write "import ActionCreators from" instead of "import * as ActionCreators from"?`
    )
  }
  // 遍历对象，然后设置每个遍历项的 actionCreator 生成函数，最后返回这个对象
  const keys = Object.keys(actionCreators)
  const boundActionCreators = {}
  for (let i = 0; i < keys.length; i++) {
    const key = keys[i]
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  return boundActionCreators
}
```
由此可以看出 bindActionCreators 的实现逻辑比较简单：
**一、**判断传入的参数是否是 object，如果是函数，就直接返回一个将 action creator 包裹在 dispatch 里的函数。
**二、**如果是object，就根据相应的key，生成相应的将 action creator 包裹在 dispatch 里的函数。

为了方便理解，我们用一个 TODO 例子说明下：
``` js
// actions/todo.js
export function addTodo(text) {
  return {
    type: 'ADD_TODO',
    text
  }
}

export function removeTodo(id) {
  return {
    type: 'REMOVE_TODO',
    id
  }
}
```
我们 import 后会得到如下对象：
``` js
{
   addTodo : text => 
    { 
      type: 'ADD_TODO',
      text
    },
   removeTodo : id => {
      type: 'REMOVE_TODO',
      id
    }
}
```
经过 bindActionCreator 就会变成 key 相同，值为用 dispatch 将 action creator 包裹起来的函数的对象：
``` js
{
   addTodo : text => dispatch(addTodo('text'));
   removeTodo : id => dispatch(removeTodo('id'));
}
```
由此我们发现可以通过 bindActionCreators 创建方法直接调用 dispatch(action)。

#### 参数
它可以接收两个参数：actionCreators、dispatch
**actionCretors：**可以是一个对象，也可以是一个单个函数
**dispatch：**dispatch 函数，从 store 实例中获取，用于包裹 action creator

如果只是传入一个 function，返回的也是一个 function，例如：
``` js
// actions/todo.js
export const toggleTodo = (id) => {
  return {
      type: 'TOGGLE_TODO',
      id
  };
};

```
经过 bindActionCreator：
``` js
const boundActionCreators = bindActionCreators(toggleTodo, dispatch);
```
由于 bindActionCreators 第一个参数是一个函数，结果就会变为：
``` js
const boundActionCreators  = (id) => dispatch(toggleTodo(id));
```

#### 返回值
单个函数，或者是一个对象。

### 总结
通过阅读 Redux 的源码，我们发现 Redux 设计的实在是太精妙了，完全函数式编程，依赖少，耦合低。




