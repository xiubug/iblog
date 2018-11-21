---
title: 理解JavaScript Promise
date: 2018-05-25 23:32:08
tags:
    - promise
categories:
    - es6
---

### 背景
从JavaScript中的异步任务说起，典型的异步任务，是一个setTimeout调用。
```js
setTimeout(() => {
  // 爱干啥干啥
}, 1000)
```
通常用JS写异步任务的时候，会分成两个部分：主过程和后续过程，在主过程执行成功后，触发后续过程执行。比如，在实际编程中经常需要使用AJAX向服务器请求数据，成功获取到数据后，才开始处理数据。于是代码分成获取数据部分和处理数据部分，像下面这样：
``` js
getData((data) => {
  // 处理data
})
```
上面这两个处理异步任务的编程方式都是采用的回调函数的形式。
现在假设有多个异步任务，且任务间有依赖关系（一个任务需要拿到另一个任务成功后的结果才能开始执行）的时候，回调的方式写出来的代码就会像下面这样：
```js
getData1(data1 => {
  getData2(data1, data2 => {
    getData3(data2, data3 => {
      getData4(data3, data4 => {
        getData5(data4, data5 => {
          // 终于取到data5了
        })
      })
    })
  })
})
```
这种代码被称为回调地狱或者回调金字塔。
假设上面的任务，想要换一下执行顺序，代码修改起来，就比较麻烦了。如果内容复杂，阅读代码的时候跳来跳去，也让人头大。
如果用promise改写一下：
```js
// 先把getData们都转成返回promise对象的函数

// 然后
getData1()
.then(getData2)
.then(getData3)
.then(getData4)
.then(getData5)
.then(data => {
  // 取到最终data了
})
```
这样的代码，是线性的，符合人的阅读习惯，代码表示的流程清晰，便于阅读。

### Promise 概念
Promise 是异步编程的一种解决方案，比传统的解决方案——回调函数和事件——更合理和更强大。它由社区最早提出和实现，ES6 将其写进了语言标准，统一了用法，原生提供了Promise对象。所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件（通常是一个异步操作）的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。从写法上说，Promise 就是一种用来写JavaScript编程中的异步代码的方式。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理。

### Promise 用法
首先要认清最基本的用法。一般学习Promise看到的第一段代码是这样：
```js
let p = new Promise((resolve, reject) => {
  // 做一些事情
  // 然后在某些条件下resolve，或者reject
  if (/* 条件随便写^_^ */) {
    resolve()
  } else {
    reject()
  }
})

p.then(() => {
    // 如果p的状态被resolve了，就进入这里
}, () => {
    // 如果p的状态被reject
})
```
第一段调用了Promise构造函数，第二段是调用了promise实例的.then方法。
**构造实例**
* 构造函数接受一个函数作为参数
* 调用构造函数得到实例p的同时，作为参数的函数会立即执行
* 参数函数接受两个回调函数参数resolve和reject
* 在参数函数被执行的过程中，如果在其内部调用resolve，会将p的状态变成fulfilled，或者调用reject，会将p的状态变成rejected

**调用.then**
* 调用.then可以为实例p注册两种状态回调函数
* 当实例p的状态为fulfilled，会触发第一个函数执行
* 当实例p的状态为rejected，则触发第二个函数执行

上面这样构造promise实例，然后调用.then.then.then的编写代码方式，就是promise。其基本模式是：
* 将异步过程转化成promise对象
* 对象有3种状态
* 通过.then注册状态的回调
* 已完成的状态能触发回调

采用这种方式来处理编程中的异步任务，就是在使用promise了，所以promise就是一种异步编程模式。

#### Promise 状态
首先，promise实例有三种状态：
* pending（待定）
* fulfilled（已执行）
* rejected（已拒绝）
fulfilled和rejected有可以说是已成功和已失败，这两种状态又归为已完成状态

#### resolve和reject
调用resolve和reject能将分别将promise实例的状态变成fulfilled和rejected，只有状态变成已完成（即fulfilled和rejected之一），才能触发状态的回调。

#### Promise API
promise的内容分为构造函数、实例方法和静态方法：
* 1个构造函数： new Promise
* 2个实例方法：.then 和 .catch
* 4个静态方法：Promise.all、Promise.race、Promise.resolve和Promise.reject

下面逐个讲下他们的作用：
* new Promise能将一个异步过程转化成promise对象。先有了promise对象，然后才有promise编程方式。
* .then用于为promise对象的状态注册回调函数。它会返回一个promise对象，所以可以进行链式调用，也就是.then后面可以继续.then。在注册的状态回调函数中，可以通过return语句改变.then返回的promise对象的状态，以及向后面.then注册的状态回调传递数据；也可以不使用return语句，那样默认就是将返回的promise对象resolve。
* .catch用于注册rejected状态的回调函数，同时该回调也是程序出错的回调，即如果前面的程序运行过程中出错，也会进入执行该回调函数。同.then一样，也会返回新的promise对象。
* 调用Promise.resolve会返回一个状态为fulfilled状态的promise对象，参数会作为数据传递给后面的状态回调函数。
* Promise.reject与Promise.resolve同理，区别在于返回的promise对象状态为rejected。


