---
title: redux 源码全方位剖析
date: 2018-06-29 23:11:43
tags:
    - redux
categories:
    - redux
---

`版本：v4.0.0`

### 前言
受2014年Facebook的 Flux 架构模式以及函数式编程语言Elm启发，Dan Abramov在2015年创建了 Redux。很快，Redux因其体小精悍（只有2kB）且没有任何依赖短时间内成为最热门的前端架构。

### 概念
Redux 是可预测的状态管理框架，它很好的解决多交互，多数据源的诉求。Redux 设计之初，作者就严格遵循三个设计理念原则：
**单一数据源：**整个应用的 state 都被存储在一棵object tree中，并且这个object tree只存在于唯一一个store中。Store 可以看做是数据存储的一个容器。在这个容器里面，只会维护唯一的一个State Tree。Store 会给定4种基础操作方法：`dispatch(action)，getState()，replaceReducer(nextReducer)，subscribe(listener)`。根据单一数据源原则，所有数据会通过`store.getState()`方法调用获取。
**State只读：**根据State只读原则，数据变更会通过store，dispatch(action)方法。Action可以理解为变更数据的信息载体。type是变更数据的唯一标志，payload是用来携带需要变更的数据。格式为：`const action = { type: 'xxx', payload: 'yyy' }`; Reducer是个纯函数。负责根据获取action.type的内容，计算state数值。格式为：`reducer: prevState => action => newState`。
**使用纯函数变更state值：**Reducer只是一些纯函数，它接收先前的state和action，并返回新的state.

正常的一个同步数据流为：view层触发actionCreator，actionCreator通过store.dispatch(action)方法, 变更reducer。但是面对多种多样的业务场景，同步数据流方式显然无法满足。对于改变reducer的异步数据操作，就需要用到中间件的概念，如图所示：

![img1.png](redux-source-analysis/img1.png)

### createStore



