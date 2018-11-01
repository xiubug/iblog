---
title: 深入探究 setTimeout
date: 2018-07-01 21:20:27
tags:
    - setTimeout
categories:
    - 前端教程
---

## 基本使用

```javascript
setTimeout(function|string, number);
```

setTimeout 方法接收两个参数，第一个参数为回调函数或字符串，第二个参数为触发时间（单位：毫秒）

```javascript
setTimeout( function() {
  console.log('console after 1 second');
}, 1000);
```
上面这段代码将会在 1 秒后在控制台打印出 console after 1 second

```javascript
setTimeout( 
  'console.log("console after 2 seconds")',
  2000 
);
```
当第一个参数为字符串，而不是函数时会怎样呢？setTimeout 方法会将这个字符串解析为一段 js 代码，然后在 2 秒后执行这段代码。如果这个字符串无法被解析为 js 代码，将会报错。

接下来我们再看看另外一个例子，这是一位群友的疑惑点：

```javascript
// 示例一：先输出 setTimeout，然后每 1s 输出一次 setTimeout
function show() {
  console.log('setTimeout');
	setTimeout(show, 1000);
}
show();

// 示例二：先输出 setTimeout，然后每 1s 输出一次 setTimeout
function show() {
	console.log('setTimeout');
	setTimeout('show()', 1000);
}
show();

// 示例三：无任何输出
function show() {
	console.log('setTimeout');
	setTimeout('show', 1000);
}
show();

// 示例四：show会被立刻执行，被延迟执行的是show执行后的返回值。所以show会不断的重复调用直到堆栈爆满，从而引发内存溢出
function show() {
	console.log('setTimeout');
	setTimeout(show(), 1000);
}
show();

```

## 返回值

```javascript
var timeout = setTimeout(function() {
    console.log('this is a timeout')
}, 1000);

console.log(timeout, typeof timeout);

// 1, "number"
// this is a timeout
```

用变量 timeout 接收 setTimeout() 的返回值，然后将 timeout 打印出来，会发现 timeout 的值是 1，类型是number

为什么 setTimeout() 的返回值会是一个数值类型呢？是不是每一个 setTimeout() 的返回值都会是1？

```javascript
var timeout1 = setTimeout(function() {
    //
}, 1000);

var timeout2 = setTimeout(function() {
    //
}, 1000);

var timeout3 = setTimeout(function() {
    //
}, 1000);

console.log('timeout1---', timeout1);
console.log('timeout2---', timeout2);
console.log('timeout3---', timeout3);

// timeout1--- 1
// timeout2--- 2
// timeout3--- 3
```

从上面的代码中能够发现，每一个 setTimeout() 的返回值都不同，返回值并不都是1，而是都对应着唯一的一个值

这个值其实就是对应的 setTimtout() 的 ID，随着当前页面定时器的不断增多，当需要对某一个定时器做操作时，通过 ID 就能够确定到该定时器。

## 如何结束/阻止一个 setTimeout 的执行

实际项目中，添加一个定时器以后，在其回调函数还未执行之前，满足某些条件时可能需要阻止该回调的执行，也就是取消一个定时器，那这时候该怎么做呢？javascript 已经为我们提供了现成的方法：

```javascript
clearTimeout( timeout );
```

上面这个方法就可以阻止一个定时器的执行，它接收一个参数，这个参数就是需要取消的定时器的 ID，也就是该定时器的返回值

```javascript
var timeout = setTimeout(function() {
    alert('this is a timeout')
}, 1000);

clearTimeout( timeout );
```
上面代码中的定时器timeout在一秒后并不会执行，因为已经通过clearTimeout( timeout )取消了它的执行

## 实现异步编程

我们使用异步编程很重要的一个目的就是为了不因为耗时任务而阻塞其他 js 代码的执行。我们知道 alert 会阻塞 js 代码的执行，这是因为 js 是单线程的，弹出框出现后如果不对其进行操作就无法执行后面的代码（类似的 confirm 也是）

```javascript
alert('this is an alert box');
var test = 'this is a text string';
console.log(test);
```
上面的代码在弹出框出现后如果不点击确定，将永远不会执行后面的代码

```javascript
setTimeout(function() {
  alert('this is an alert box');
}, 1000);
var test = 'this is a text string';
console.log(test);

// this is a text string
```

上面的代码将 alert 放在了一个延时 1 秒的定时器中，这样就会先打印出 test，过一秒后再显示弹出框。或许你会说，本身 alert 就延时了 1 秒执行，当然不会阻塞其他的代码执行。那么你可以试着将延时1000改为0，这就表示弹出框应该是没有延时立即执行，但是你会发现实际上还是先打印出 test，再执行了 alert。为什么会这样呢？我们下面再说

在实际项目中，我们可以利用 setTimeout 的异步特性，解决一些问题，比如某个对象还未实例化，为了保证该对象在使用到时能够确保已经被实例化，就可以通过 setTimeout 来实现

## setTimeout 回调函数的执行时机

现在我们来说说为什么延时设为 0 ，回调函数却没有立即执行的问题。我们知道浏览器是基于事件循环的，其中会有多个队列，页面的渲染是一个队列，js 代码的执行也是一个队列。js 代码执行时会创建一系列的任务，而这些任务秉承着先进先出的原则被加入到队列中。但是 setTimeout 是特殊的，当执行到 setTimeout 时，js会将其拿出来放到一个单独的特殊队列中，这个队列中的任务在 js 队列还有未执行完的任务时，永远不会被执行

举个不恰当的栗子，小明想玩游戏，但是作业还没做完，由于小明是单线程的，虽然现在很想玩游戏，但是他还是给自己设定了条件：作业不做完不能玩游戏，一做完立刻玩游戏（延时为0），于是玩游戏这个任务就被小明归置到了一个特殊的任务队列里面，在作业队列任务完成之前不执行特殊队列里面玩游戏的任务，作业完成小明闲下来后立刻开始玩游戏。所以只有浏览器的 js 引擎闲下来以后，才会执行所有 setTimeout，即使延时为 0。

```javascript
var flag = true;
setTimeout(function() {
    flag = false;
}, 1000);
while(flag) {}
alert('this is an alert box');
```

问：上面的代码什么时候会显示弹出框？

答：永远都不会

上面的代码中，while是一个耗时函数，虽然setTimeout只延时了一秒执行，但是由于主队列中的while会永远的执行下去，所以setTimeout所在的队列永远不会被执行，代码会永远阻塞在while循环这边。当然，上面的这种无限循环在项目中不可能出现，而代码执行速度极快，只要不出现十分耗时的代码，定时器几乎还是能够按照我们的意愿在指定时间执行回调函数。

## 通过 setTimeout 优化用户体验

既然setTimeout必须等到主队列中的任务执行完以后才会执行，那我们在碰到一些十分耗时的代码时，是不是可以通过它来放在页面的阻塞呢？当然是可以的，将耗时代码写进setTimeout的回调，时间设置为 0，这样只要 js 引擎空闲下来就回去执行这些耗时代码，就不会阻塞页面给用户造成卡顿的体验，从而提升用户体验。

## 不易察觉的危险——内存泄漏

1. 什么是内存泄漏？

一块内存在分配使用完毕以后，既不会被再次使用，又没有被及时回收，直到程序执行完毕都始终占据着这块内存。

2. setTimeout 什么情况会导致内存泄漏？

setTimeout的第一个参数可以是函数，也可以是字符串。当传入字符串时，就会有内存泄漏产生。先看下面两个例子

```javascript
setTimeout(function test1() {
    var a = 1;
    console.log(a);
}, 0)

setTimeout((function test2() {
    var b = 1;
    console.log(b);
}).toString(), 0)

```

执行代码后，打开控制台，分别输入函数名test1和test2

```javascript
test1
// Uncaught ReferenceError: test1 is not defined

test2
// ... (打印出 test2 的函数体)
```

会发现，当第一个参数为函数时，回调函数执行完毕后，test1函数被销毁，其所使用内存也被释放；当第一个参数为字符串时，test2却始终存在，它没有被销毁，始终占据着内存，也就造成了内存泄漏，所以让我们需要使用 setTimeout时，一定要注意，第一个参数必须传入一个函数。

