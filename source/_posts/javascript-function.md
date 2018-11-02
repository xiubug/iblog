---
title: javascript 函数
date: 2018-07-24 21:02:47
tags:
  - function
categories:
  - javascript
---

函数是一段可以反复调用的代码块。函数还能接受输入的参数，不同的参数会返回不同的值。

## 立即执行函数

在 Javascript 中，圆括号()是一种运算符，跟在函数名之后，表示调用该函数。比如，print()就表示调用print函数。

有时，我们需要在定义函数之后，立即调用该函数。这时，我们不能在函数的定义之后加上圆括号，这会产生语法错误。
``` js
function(){ /* code */ }();
// SyntaxError: Unexpected token (
```
产生这个错误的原因是，function这个关键字即可以当作语句，也可以当作表达式。
``` js
// 语句
function f() {}

// 表达式
var f = function f() {}
```
为了避免解析上的歧义，JavaScript 引擎规定，如果function关键字出现在行首，一律解释成语句。因此，JavaScript 引擎看到行首是function关键字之后，认为这一段都是函数的定义，不应该以圆括号结尾，所以就报错了。

解决方法就是不要让function出现在行首，让引擎将其理解成一个表达式。最简单的处理，就是将其放在一个圆括号里面。
``` js
(function(){ /* code */ }());
// 或者
(function(){ /* code */ })();
```
上面两种写法都是以圆括号开头，引擎就会认为后面跟的是一个表示式，而不是函数定义语句，所以就避免了错误。这就叫做立即执行函数（Immediately-Invoked Function Expression），简称 IIFE。

注意，上面两种写法最后的分号都是必须的。如果省略分号，遇到连着两个 IIFE，可能就会报错。

``` js
// 报错
(function(){ /* code */ }())
(function(){ /* code */ }())
```
推而广之，任何让解释器以表达式来处理函数定义的方法，都能产生同样的效果，比如下面三种写法。
``` js
var i = function(){ return 10; }();
true && function(){ /* code */ }();
0, function(){ /* code */ }();
```
甚至像下面这样写，也是可以的。
``` js
!function () { /* code */ }();
~function () { /* code */ }();
-function () { /* code */ }();
+function () { /* code */ }();
```

### 定义

简单说立即执行函数就是**声明一个匿名函数、马上调用这个匿名函数**。

![img1.jpg](/images/javascript-function/img1.jpg)

上面是一个典型的立即执行函数。
**首先，**声明一个匿名函数 function(){alert('我是匿名函数')}。
**然后，**在匿名函数后面接一对括号 ()，调用这个匿名函数。

### 作用

只有一个作用：创建一个独立的作用域。

这个作用域里面的变量，外面访问不到（即避免「变量污染」）。

以一个著名的面试题为例：

``` js
var liList = ul.getElementsByTagName('li')
for(var i=0; i<6; i++){
  liList[i].onclick = function(){
    alert(i) // 为什么 alert 出来的总是 6，而不是 0、1、2、3、4、5
  }
}
```
为什么 alert 的总是 6 呢，因为 i 是贯穿整个作用域的，而不是给每个 li 分配了一个 i，如下：
![img2.jpg](/images/javascript-function/img2.jpg)

那么怎么解决这个问题呢？用立即执行函数给每个 li 创造一个独立作用域即可：
``` js
var liList = ul.getElementsByTagName('li')
for(var i=0; i<6; i++){
  !function(ii){
    liList[ii].onclick = function(){
      alert(ii) // 0、1、2、3、4、5
    }
  }(i)
}
```
在立即执行函数执行的时候，i 的值被赋值给 ii，此后 ii 的值一直不变。i 的值从 0 变化到 5，对应 6 个立即执行函数，这 6 个立即执行函数里面的 ii 「分别」是 0、1、2、3、4、5。


