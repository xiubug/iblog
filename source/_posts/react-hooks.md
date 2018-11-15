---
title: React Hooks — 新一代 React API
tags:
  - hooks
categories:
  - react
date: 2018-11-14 22:08:41
---

## 前言
不得不说React等现在前端开发框架的出现为前端开发带来了极大的方便，但是在React v16.7 "Hooks"提出之前我们依旧不得不面对一些很恶心的问题：

### 组件逻辑复用
类（class）是我们开发React组件的基本单位。在抽象组件中的逻辑时，我们一般会用到高阶组件（HOC）和Render props 两种模式。然而这两种模式都存在一些相同的问题，就是当我们不管想要引入哪一种逻辑时，我们都需要对我们的代码进行比较大幅的改动。而在某些更复杂的情况下，这些模式更是会造成可怕的“嵌套黑洞（Wrapper Hell）”，导致我们不得不在众多相互嵌套的组件中追踪我们的数据流动。
我们都知道react的核心思想就是：将一个页面拆成一堆独立的，可复用的组件，并且用自上而下的单向数据流的形式将这些组件串联起来，但如果我们在大型项目中用react，我们会发现我们的项目中实际上很多react组件冗长且难以复用。尤其是那些写成class的组件，它们本身包含了状态（state），所以复用这类组件就变得很麻烦。
在 hooks 之前官方推荐的解决方式：[渲染属性（Render Props）](https://reactjs.org/docs/render-props.html)和[高阶组件（Higher-Order Components）](https://reactjs.org/docs/higher-order-components.html)。我们可简单看一下这两种模式：

渲染属性指的是使用一个值为函数的prop来传递需要动态渲染的nodes或组件。如下面的代码可以看到我们的DataProvider组件包含了所有跟状态相关的代码，而Cat组件则可以是一个单纯的展示型组件，这样一来DataProvider就可以单独复用了。
``` jsx
import Cat from 'components/cat'
class DataProvider extends React.Component {
  constructor(props) {
    super(props);
    this.state = { target: 'Zac' };
  }

  render() {
    return (
      <div>
        {this.props.render(this.state)}
      </div>
    )
  }
}

<DataProvider render={data => (
  <Cat target={data.target} />
)}/>
```
虽然这个模式叫Render Props，但不是说非用一个叫render的props不可，习惯上大家更常写成下面这种：
``` jsx
...
<DataProvider>
  {data => (
    <Cat target={data.target} />
  )}
</DataProvider>
```
高阶组件这个概念就更好理解了，说白了就是一个函数接受一个组件作为参数，经过一系列加工后，最后返回一个新的组件。看下面的代码示例，withUser函数就是一个高阶组件，它返回了一个新的组件，这个组件具有了它提供的获取用户信息的功能。
``` js
const withUser = WrappedComponent => {
  const user = sessionStorage.getItem("user");
  return props => <WrappedComponent user={user} {...props} />;
};

const UserPage = props => (
  <div class="user-container">
    <p>My name is {props.user}!</p>
  </div>
);

export default withUser(UserPage);
```
以上这两种模式看上去都挺不错的，很多库也运用了这种模式，比如我们常用的React Router。但我们仔细看这两种模式，会发现它们会增加我们代码的层级关系。最直观的体现，打开devtool看看我们的组件层级嵌套是不是很夸张吧。如果我采用 hooks 方式，把各种想要的功能写成一个一个可复用的自定义hook，当我们的组件想用什么功能时，直接在组件里调用这个hook即可，这样就简洁了许多，也没有多余的层级嵌套。

### 组件生命周期混乱
由于React生命周期的存在，我们常常需要将一些明明是逻辑强相关的代码分散地放置在组件的不同位置，造成我们的组件中出现了许多零散的、重复的代码。比如我们需要在 componentDidMount 中绑定事件、添加定时器，然后再 componentWillUnmount 中移除他们；又或者频繁地在 componentDidUpdate 中比较变化前和变化后的state来决定是否执行某些逻辑。

### 无状态组件（Function）和有状态组件（Class）选择问题
回想我们在刚开始学习React的时候一定会迷惑于Functional Component(那个时候我们叫它Stateless Component)和Class Component。两者得到的结果似乎是一致的，但是书写方式却迥然不同。有些组件可能最开始的时候使用Functional，后来发现需要加入生命周期和state的支持又不得不大费周折地改成Class。这样不清不楚的定位对许多初学者来说，无疑造成了很大的困扰。有些开发者便无脑的都使用 Class Component。

### class component 的 this 指向问题
我们用class来创建react组件时，还有一件很麻烦的事情，就是this的指向问题。为了保证this的指向正确，我们要经常写这样的代码：`this.handleClick = this.handleClick.bind(this)`，或者是这样的代码：`<button onClick={() => this.handleClick(e)}>`。一旦我们不小心忘了绑定this，各种bug就随之而来，很麻烦。

### Class Component 无法 prepack 优化
在 class里，类的属性即便内部没用到，对外部还是可访问的，所以类的属性在Uglify的时候是不会被编译的，同时如果一个类的方法没有被使用，编译器也无法将它识别出来并精简掉。

在这样的背景下，Hooks便横空出世了！

## 什么是 Hooks
`Hooks`是 React 提供的一系列新的方法（习惯上以useXXX命名）。这些方法将state、context和Class组件中的生命周期，统统抽象成了函数，使得我们在Functional组件中也能使用它们，甚至我们还可以将它们彼此进行组合，从而将特定的逻辑进一步进行抽象和封装，进一步作为npm包的形式进行发布！

接下来我们一起认识一下新增的几个方法：
* State Hook: 为组件提供访问state的能力
* Effect Hook: 监听state的变动，并在合适的时候调用
* Custom Hooks: 用户自定义的钩子，是以上两者的组合。方便用户对操作state的逻辑进行封装
* Other Hooks: 主要包括访问Context的钩子和管理复杂state的钩子

### State Hook: React.useState
**useState: (any<T>) => [state: <T>, (newState) => null]**
useState 方法比较简单，基本就是一个简化版的 setState 。每次调用会生成一个新的state，并将状态与组件绑定。来看一个简单的例子：
``` js
import { useState } from 'react';

function Example() {
  // Declare a new state variable, which we'll call "count"
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
我们再来看一下使用class component后的版本：
``` js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  render() {
    return (
      <div>
        <p>You clicked {this.state.count} times</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```
可以看出，使用hooks后，代码简单了许多。Example变成了一个函数，但这个函数却有自己的状态（count），同时它还可以更新自己的状态（setCount）。这个函数之所以这么了不得，就是因为它注入了一个hook--useState，就是这个hook让我们的函数变成了一个有状态的函数。我们分解来看到底state hooks做了什么：
#### 声明一个状态变量
```js
import { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);
```
useState是react自带的一个hook函数，它的作用就是用来声明状态变量。useState这个函数接收的参数是我们的状态初始值（initial state），它返回了一个数组，这个数组的第[0]项是当前当前的状态值，第[1]项是可以改变状态值的方法函数。所以我们做的事情其实就是：声明了一个状态变量count，把它的初始值设为0，同时提供了一个可以更改count的函数setCount。

上面这种表达形式，是借用了[es6的数组解构（array destructuring）](http://es6.ruanyifeng.com/#docs/destructuring#%E6%95%B0%E7%BB%84%E7%9A%84%E8%A7%A3%E6%9E%84%E8%B5%8B%E5%80%BC)，它可以让我们的代码看起来更简洁。

如果不用数组解构的话，可以写成下面这样。实际上数组解构是一件开销很大的事情，用下面这种写法，或者改用对象解构，性能会有很大的提升。
``` js
let _useState = useState(0);
let count = _useState[0];
let setCount = _useState[1];
```

#### 读取状态值
``` html
<p>You clicked {count} times</p>
```
是不是超简单？因为我们的状态count就是一个单纯的变量而已，我们再也不需要写成`{this.state.count}`这样了。

#### 更新状态
``` html
<button onClick={() => setCount(count + 1)}>
  Click me
</button>
```
当用户点击按钮时，我们调用setCount函数，这个函数接收的参数是修改过的新状态值。接下来的事情就交给react了，react将会重新渲染我们的Example组件，并且使用的是更新后的新的状态，即count=1。这里我们要停下来思考一下，Example本质上也是一个普通的函数，为什么它可以记住之前的状态？

#### 一个至关重要的问题
这里我们就发现了问题，通常来说我们在一个函数中声明的变量，当函数运行完成后，这个变量也就销毁了（这里我们先不考虑闭包等情况），比如考虑下面的例子：
``` js
function add(n) {
    const result = 0;
    return result + 1;
}

add(1); //1
add(1); //1
```
不管我们反复调用add函数多少次，结果都是1。因为每一次我们调用add时，result变量都是从初始值0开始的。那为什么上面的Example函数每次执行的时候，都是拿的上一次执行完的状态值作为初始值？答案是：是react帮我们记住的。至于react是用什么机制记住的，我们可以再思考一下。

### Effect Hook: React.useEffect
**useEffect: (() => {do sth...; return () => null}, []) => null**
可以说，这个方法是这次更新当中， 最难理解的一个方法。同时也是最精妙的一个。 因为通过这个方法，我们可以完成对Class Component所有关键生命周期的访问。 我们详细来看一下：

首先 useEffect 方法接收两个参数：
* 第一个参数是一个函数
这个函数会在每次组件重新update后被调用（我们也可以理解为每次render之后会调用一遍这个方法）。只使用这个方法的作用和 componentDidUpdate 差不多，比如要实现输入和document.title的双向绑定：
``` js
import React, { useState, useEffect } from 'react'

export default function Example() {
  const [value, setValue] = useState('')
  useEffect(() => {
    document.title = value
  })

  return <input value={value} onChange={e => {setValue(e.target.value)}} />
}
```
* 这个函数的返回值也是一个函数
回想一下，在使用class组件时， componentDidMount 其实是一类特殊的 componentDidUpdate —— 前者只会在第一次update时调用。所以在functional组件中，我们也可以使用useEffect来模拟 componentDidUpdate，只要区分函数的调用时机就可以。

不过在useEffect里不是这样做的。useEffect的做法更简单粗暴一些：在state更新时调用，在下一次render之前清理。useEffect方法接收一个函数作为返回值，返回的函数会在下一次render之前被调用。 (总觉得这样会不会太粗暴了一点，因为每次重新render都会绑定一次事件。
``` js
import React, { useState, useEffect } from 'react'

export default function Example() {
  const handleClick = () => {
      // do something
  }
  useEffect(() => {
    document.querySelector('#example).addEventListener('click', handleClick)
    return () => {
      document.querySelector('#example).removeEventListener('click', handleClick)  
    }
  })
  return <div id="example">Lorem</div>
}
```
* 第三个知识点是这个函数的第二个参数是一个数组
这个数组里的值可以等同于我们在写 componentDidUpdate 里的条件判断。只有当数组中包含的值变化的时候才会触发当前的useEffect。

