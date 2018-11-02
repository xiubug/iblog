---
title: redux-thunk 源码全方位剖析
date: 2018-09-09 20:59:49
tags:
  - source
categories:
  - redux-thunk
---

### 前言
redux 的核心概念很简单：将需要修改的 state 都储存在 store 里，发起一个 action 用来描述发生了什么，用 reducers 描述 action 如何改变 state tree，创建 store 的时候需要传入 reducer，真正能改变 store 中数据的是 API store.dispatch。
纯粹使用 redux 时，我们需要 dispatch 的是一个 action 对象。当我们使用 redux-thunk 后，我们 dispatch 的是一个 function，redux-thunk 中间件会自动调用这个 function，并且传递 dispatch 方法作为其第一个参数，于是我们就能在这个 function 内根据我们的请求状态：开始，请求中，请求成功／失败，dispatch 我们期望的任何 action 了，这也是为什么它能支持异步 dispatch action，自然的请求的逻辑就需要放到这里面调用了。换言之，redux-thunk 改写了 dispatch API，使其具备接受一个函数作为参数的能力，从而达到 middleware 的效果，即在 redux 的 dispatch action => reducer => store 这个流程中，在 action 被发起之后，到达 reducer 之前的扩展点，加入相关操作，比如发生请求、log信息等。一句话：redux-thunk 就是对 store.dispatch() 的增强。

### 示例
以动态收藏功能为例:
**纯粹使用 redux：**
```js
// action.js
const addFavor = (id) => ({
  type: 'FAVOR',
  id:id
})

// component
fetchFavor({id: id}).then(res => { dispatch(addFavor(id)) })
```
可以看到，我们在请求以后的回调函数中 dispatch action 去同步 redux store 中的状态。

**使用 redux-thunk：**
``` js
// store.js
import {createStore, combineReducers, applyMiddleware} from 'redux';
import * as reducers from './reducers';
import thunk from 'redux-thunk';

var store = createStore(
	combineReducers(reducers),
	applyMiddleware(thunk)
);

export default store;

// action.js
const addFavor = (id) => {
  return function (dispatch, getState) {
    reqFavor({id: id}).then(res => { 
      dispatch({
        type: "FAVOR",
        id:id
      })
    })
  }
}

// component
dispatch(addFavor(id))
```
改变以后，从功能层面上来说，两者并无差别，都可以满足业务场景需求。但除此之外我们可以发现:
**一：**dispatch 接受的参数由一个 PlainObject 变为一个函数
**二：**我们把请求的异步操作从 dispatch action 这个 redux 流程外塞到的流程里，这看起来将异步操作内聚到这个流程中，无论是从逻辑上理解（这很 middleware！）还是项目代码开发维护（区分异步与同步状态管理流程进行维护管理）上都是很大的改进
**三：**如果项目中有多处需要实现收藏功能，我们可以节省很多冗余代码，不用到处在 dispatch 外层套上 reqLike(id).then。。。

直接将 thunk 中间件引入，作为 applyMiddleware 参数，然后传入 createStore 方法，就完成了 store.dispatch() 的功能增强，这样就可以进行一些异步的操作了。其中 applyMiddleware 是 Redux 的一个原生方法，将所有中间件组成一个数组，依次执行，中间件多了可以当做参数依次传进去。
``` js
const store = createStore(
    reducers, 
    applyMiddleware(thunk, logger)
);
```

### 源码
了解了 redux-thunk 的基本概念以及应用后，我们一起看看源码加深下理解，源码十分精巧。在了解 redux-thunk 源码之前，我们很有必要先看看 redux 源码中 applyMiddleware 的部分：
``` js
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        `Dispatching while constructing your middleware is not allowed. ` +
          `Other middleware would not be applied to this dispatch.`
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
我们将 thunk 作为参数传入之后，直接返回了一个函数，这个函数作为 enhancer 传入 redux 源码中的 createStore 函数中：
``` js
export default function createStore(reducer, preloadedState, enhancer) {
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }

    return enhancer(createStore)(reducer, preloadedState)
  }
}
```
在 redux 源码中的 createStore 函数中，enhancer 被执行，传入参数 createStore，又紧接着执行其返回的函数，传入 reducer 和 preloadedState。接下来，我们进入 applyMiddleware 和 thunk 的关键部分，上面 applyMiddleware 接受的最初的 (...middlewares) 参数其实就是 thunk，thunk 会被执行，并且传入参数 getState 和 dispatch：
``` js
//传入到 thunk 的参数
const middlewareAPI = {
  getState: store.getState,
  dispatch: (action) => dispatch(action)
}
//在 map 中执行 thunk
chain = middlewares.map(middleware => middleware(middlewareAPI))
//重新改写 dispatch
dispatch = compose(...chain)(store.dispatch)
```
那么上面的chain是什么呢，我们现在就可以去看 redux-thunk 的源码了：
``` js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
redux-thunk 中间件 export default 的就是 createThunkMiddleware() 过的 thunk，再看 createThunkMiddleware 这个函数，返回的是一个柯里化过的函数。我们将上述代码编译成ES5的代码看一看：
``` js
function createThunkMiddleware(extraArgument) {
    return function({ dispatch, getState }) {
      // 这里返回的函数就是chain
      return function(next) {
        // 这里返回的函数就是改写的dispatch
        return function(action) {
          if (typeof action === 'function') {
              return action(dispatch, getState, extraArgument);
          }

          return next(action);
        };
      }
    }
}
```
从源码我们可以看出，chain 就是以 next 作为形参的匿名函数，至于 compose 只是不断传递每个函数的返回值给下一个执行函数，然后依次去执行它所有传入的函数而已，它源码中的注释说的很清楚：`For example, compose(f, g, h) is identical to doing (...args) => f(g(h(...args)))`。
我们这里的 chain 只是一个函数而已，所以很简单，就是执行 chain，并且传入 store.dispatch 作为 next 就行。

接下来，进入最后一步，改写了 dispatch，最终变为:
``` js
function (action) {
  if (typeof action === 'function') {
    return action(dispatch, getState, extraArgument);
  }
  // next为之前传入的store.dispatch,即改写前的dispatch
  return next(action);
};

```
如果传入的参数是函数，则执行函数，否则还是跟之前一样 dispatch(PlainObject)。

### 总结

redux-thunk 实现了相关异步流程内聚到 redux 的流程中，实现 middleware 的功能，也便于项目的开发与维护，避免冗余代码。而实现的方式便是改写 redux 中的 dispatch API，使其可以除 PlainObject 外，接受一个函数作为参数。

