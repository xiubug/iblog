---
title: redux-thunk 源码全方位剖析
date: 2018-05-08 20:59:49
tags:
    - redux-thunk
categories:
    - redux
---

### 前言
redux 的核心概念很简单：将需要修改的 state 都储存在 store 里，发起一个 action 用来描述发生了什么，用 reducers 描述 action 如何改变 state tree。创建 store 的时候需要传入 reducer，真正能改变 store 中数据的是 store.dispatch API。
redux-thunk 是一个比较流行的 redux 异步 action 中间件，比如 action 中有 setTimeout 或者通过 fetch通用远程 API 这些场景，那么就应该使用 redux-thunk 了。redux-thunk 帮助我们统一了异步和同步 action 的调用方式，把异步过程放在 action 级别解决，对 component 没有影响。纯粹使用 redux 时，我们需要 dispatch 的是一个 action 对象，当我们使用 redux-thunk 后，我们 dispatch 的是一个 function，redux-thunk 中间件会自动调用这个 function，并且传递 dispatch 方法作为其第一个参数，于是我们就能在这个 function 内根据我们的请求状态：开始，请求中，请求成功／失败，   dispatch 我们期望的任何 action 了，这也是为什么它能支持异步 dispatch action，自然的请求的逻辑就需要放到这里面调用了。换言之，redux-thunk 就是对 store.dispatch() 的增强。

### 用法
``` js
import {createStore, combineReducers, applyMiddleware} from 'redux';
import * as reducers from './reducers';
import thunk from 'redux-thunk';

var store = createStore(
	combineReducers(reducers),
	applyMiddleware(thunk)
);

export default store;
```
直接将 thunk 中间件引入，作为 applyMiddleware 参数，然后传入 createStore 方法，就完成了 store.dispatch() 的功能增强，这样就可以进行一些异步的操作了。其中 applyMiddleware 是 Redux 的一个原生方法，将所有中间件组成一个数组，依次执行，中间件多了可以当做参数依次传进去。
``` js
const store = createStore(
    reducers, 
    applyMiddleware(thunk, logger)
);
```

### 源码

