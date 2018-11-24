---
title: React Hooks — 新一代 React API
tags:
  - hooks
categories:
  - react
date: 2018-11-14 22:08:41
---

## 前言
不得不说 React 等优秀框架的出现为前端开发带来了极大的便利，但是在`React v16.7`提出`Hooks`之前我们依旧不得不面对一些很恶心的问题：

### 组件逻辑复用麻烦且很有可能造成嵌套黑洞（Wrapper Hell）
我们都知道react的核心思想是：将一个页面拆成一堆独立的，可复用的组件，并且用自上而下的单向数据流的形式将这些组件串联起来，但如果我们在大型项目中用react，我们便会发现项目中很多react组件冗长且难以复用，尤其是那些写成class的组件，它们本身包含了状态（state），所以复用这类组件就变得很麻烦。在 hooks 之前官方推荐的解决方式：[渲染属性（Render Props）](https://reactjs.org/docs/render-props.html)和[高阶组件（Higher-Order Components）](https://reactjs.org/docs/higher-order-components.html)，我们现在简单看一下这两种模式：

#### 渲染属性（Render Props）
`渲染属性（Render Props）`是一个组件间共享代码逻辑的小技巧, 通过props传递函数来实现。组件中有一个叫做`render`的`prop`, 值是一个返回React元素的函数, 在组件内部调用这个函数渲染组件。语言描述不够直观, 我们来看一个例子：
```js
class Cat extends React.Component {
  render() {
    const mouse = this.props.mouse;
    return (
      <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
    );
  }
}

class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          使用传入函数的逻辑动态渲染
          而不是硬编码地渲染固定内容
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={mouse => (
          <Cat mouse={mouse} />
        )}/>
      </div>
    );
  }
}
```
可以看到我们的`Mouse`组件包含了所有跟状态相关的代码，而`Cat`组件则只是一个单纯的展示型组件，这样一来我们可以随处复用`Mouse`组件了。

虽然这个技巧或者说模式(Pattern)叫`Render Props`, 但不是说非用一个叫render的props来传递渲染函数, 习惯上我们更常写成下面这种：
``` jsx
<Mouse>
  {mouse => (
    <p>The mouse position is {mouse.x}, {mouse.y}</p>
  )}
</Mouse>
```
#### 高阶组件
简单说高阶组件就是一个函数接受一个组件作为参数，经过一系列加工后，最后返回一个新的组件。看下面的代码示例，withUser函数就是一个高阶组件，它返回了一个新的组件，这个组件具有了它提供的获取用户信息的功能。
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
以上这两种模式看上去都挺不错的，有许多库(比如React Router, React Motion)都有使用这些模式。但我们仔细看这两种模式，会发现它们会增加我们代码的层级关系。最直观的体现，打开devtool看看我们的组件层级嵌套是不是很夸张吧。如果我们采用 hooks 方式，把各种想要的功能写成一个一个可复用的自定义hook，当我们的组件想用什么功能时，直接在组件里调用这个hook即可，这样就简洁了许多，也没有多余的层级嵌套。

### 组件生命周期混乱
由于React生命周期的存在，我们常常将一些逻辑强相关的代码分散地放置在组件的不同位置，造成我们的组件中出现了许多零散的、重复的代码，比如我们在 componentDidMount 中绑定事件、添加定时器，然后在 componentWillUnmount 中移除他们；又或者频繁地在 componentDidUpdate 中比较变化前和变化后的state来决定是否执行某些逻辑。

### 无状态组件（Function）和有状态组件（Class）选择问题
回想我们在刚开始学习React的时候常常会疑惑该使用Functional Component(那个时候我们叫Stateless Component)还是有状态组件（Class Component），虽然两者得到的结果大致相同，但是书写方式却迥然不同。有些组件可能最开始的时候使用Functional，后来发现需要加入生命周期和state的支持又不得不大费周折地改成Class。这样不清不楚的定位对许多初学者来说，无疑造成了很大的困扰。更有些开发者便无脑的都使用 Class Component。

### Class Component 的 this 指向问题
我们用class创建react组件时，还有一件很麻烦的事情，就是this的指向问题。为了保证this的指向正确，我们要经常写这样的代码：`this.handleClick = this.handleClick.bind(this)`，或者是这样的代码：`<button onClick={() => this.handleClick(e)}>`。一旦我们不小心忘了绑定this，各种bug就随之而来，很麻烦。

### Class Component 无法 prepack 优化
在 class 里，类的属性即便内部没用到，对外部还是可访问的，所以类的属性在Uglify的时候是不会被编译的，同时如果一个类的方法没有被使用，编译器也无法将它识别出来并精简掉。

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
可以看出，使用hooks后，代码简单了许多。我们分解来看到底state hooks做了什么：
#### 声明状态变量
```js
import { useState } from 'react';

function Example() {
  const [count, setCount] = useState(0);
```
useState是react自带的一个hook函数，它的作用就是用来声明状态变量。useState这个函数接收的参数是我们的状态初始值（initial state），它返回了一个数组，这个数组的第`[0]`项是当前的状态值，第`[1]`项是可以改变状态值的函数，所以其实useState做的事情就是：声明了一个状态变量count，把它的初始值设为0，同时提供了一个可以更改count的函数setCount。上面这种表达形式，是借用了[es6的数组解构（array destructuring）](http://es6.ruanyifeng.com/#docs/destructuring#%E6%95%B0%E7%BB%84%E7%9A%84%E8%A7%A3%E6%9E%84%E8%B5%8B%E5%80%BC)，让我们的代码看起来更简洁。实际上数组解构是一件开销很大的事情，用下面这种写法，或者改用对象解构，性能会有很大的提升。如果不用数组解构的话，也可以写成下面这样：
``` js
let _useState = useState(0);
let count = _useState[0];
let setCount = _useState[1];
```

#### 使用状态值
``` html
<p>You clicked {count} times</p>
```
是不是超简单？因为我们的状态count就是一个单纯的变量而已，我们再也不需要写成`{this.state.count}`这样了。

#### 修改状态
``` html
<button onClick={() => setCount(count + 1)}>
  Click me
</button>
```
当用户点击按钮时，我们调用setCount函数，这个函数接收的参数是修改过的新状态值。接下来的事情就交给react了，react将会重新渲染我们的Example组件，并且使用的是更新后的新状态，即count=1。这里我们要停下来思考一下，Example本质上也是一个普通的函数，为什么它可以记住之前的状态？

#### React 帮忙记住之前的状态
通常来说我们在一个函数中声明的变量，当函数运行完成后，这个变量也就销毁了（这里我们先不考虑闭包等情况），比如考虑下面的例子：
``` js
function add(n) {
    const result = 0;
    return result + 1;
}

add(1); //1
add(1); //1
```
不管我们反复调用add函数多少次，结果都是1。因为每一次我们调用add时，result变量都是从初始值0开始的。那为什么上面的Example函数每次执行的时候，都是拿的上一次执行完的状态值作为初始值？答案是：是react帮我们记住的。至于react是用什么机制记住的，我们可以再思考一下。

#### React 记住状态的关键点
首先，useState是可以多次调用的，所以我们完全可以这样写：
``` js
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```
其次，useState接收的初始值没有规定一定要是string/number/boolean这种简单数据类型，它完全可以接收对象或者数组作为参数。唯一需要注意的点是，之前我们的this.setState做的是合并状态后返回一个新状态，而useState是直接替换老状态后返回新状态。最后，react也给我们提供了一个useReducer的hook，如果我们更喜欢redux式的状态管理方案的话。

从ExampleWithManyStates函数我们可以看到，useState无论调用多少次，相互之间是独立的。这一点至关重要。为什么这么说呢？

其实我们看hook的“形态”，有点类似之前被官方否定掉的Mixins这种方案，都是提供一种“插拔式的功能注入”的能力。而mixins之所以被否定，是因为Mixins机制是让多个Mixins共享一个对象的数据空间，这样就很难确保不同Mixins依赖的状态不发生冲突。

而现在我们的hook，一方面它是直接用在function当中，而不是class；另一方面每一个hook都是相互独立的，不同组件调用同一个hook也能保证各自状态的独立性，这就是两者的本质区别。

#### react是根据useState出现的顺序记住状态
还是看上面给出的ExampleWithManyStates例子，我们调用了三次useState，每次我们传的参数只是一个值（如42，‘banana’），我们根本没有告诉react这些值对应的key是哪个，那react是怎么保证这三个useState找到它对应的state呢？

答案是，react是根据useState出现的顺序来定的。我们具体来看一下：
``` js
//第一次渲染
useState(42);  //将age初始化为42
useState('banana');  //将fruit初始化为banana
useState([{ text: 'Learn Hooks' }]); //...

//第二次渲染
useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
useState('banana');  //读取状态变量fruit的值（这时候传的参数banana直接被忽略）
useState([{ text: 'Learn Hooks' }]); //...
```
假如我们改一下代码：
``` js
let showFruit = true;
function ExampleWithManyStates() {
  const [age, setAge] = useState(42);
  
  if(showFruit) {
    const [fruit, setFruit] = useState('banana');
    showFruit = false;
  }
 
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
```
这样一来，
``` js
//第一次渲染
useState(42);  //将age初始化为42
useState('banana');  //将fruit初始化为banana
useState([{ text: 'Learn Hooks' }]); //...

//第二次渲染
useState(42);  //读取状态变量age的值（这时候传的参数42直接被忽略）
// useState('banana');  
useState([{ text: 'Learn Hooks' }]); //读取到的却是状态变量fruit的值，导致报错
```
鉴于此，react规定我们必须把hooks写在函数的最外层，不能写在ifelse等条件语句当中，来确保hooks的执行顺序一致。

### Effect Hook: React.useEffect
**useEffect: (() => {do sth...; return () => null}, []) => null**
因为通过useEffect，我们可以完成对Class Component所有关键生命周期的访问。 我们详细来看一下：

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

我们在上一节的例子中增加一个新功能：
``` js
import { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似于componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 更新文档的标题
    document.title = `You clicked ${count} times`;
  });

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
我们对比看一下，如果没有hooks，我们会怎么写？
```js
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  componentDidMount() {
    document.title = `You clicked ${this.state.count} times`;
  }

  componentDidUpdate() {
    document.title = `You clicked ${this.state.count} times`;
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
我们写的有状态组件，通常会产生很多的副作用（side effect），比如发起ajax请求获取数据，添加一些监听的注册和取消注册，手动修改dom等等。我们之前都把这些副作用的函数写在生命周期函数钩子里，比如componentDidMount，componentDidUpdate和componentWillUnmount。而现在的useEffect就相当与这些声明周期函数钩子的集合体。它以一抵三。

同时，由于前文所说hooks可以反复多次使用，相互独立。所以我们合理的做法是，给每一个副作用一个单独的useEffect钩子。这样一来，这些副作用不再一股脑堆在生命周期钩子里，代码变得更加清晰。

#### useEffect做了什么？
我们再梳理一遍下面代码的逻辑：
``` js
function Example() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });
```
首先，我们声明了一个状态变量count，将它的初始值设为0。然后我们告诉react，我们的这个组件有一个副作用。我们给useEffecthook传了一个匿名函数，这个匿名函数就是我们的副作用。在这个例子里，我们的副作用是调用browser API来修改文档标题。当react要渲染我们的组件时，它会先记住我们用到的副作用。等react更新了DOM之后，它再依次执行我们定义的副作用函数。

这里要注意几点：

第一，react首次渲染和之后的每次渲染都会调用一遍传给useEffect的函数。而之前我们要用两个声明周期函数来分别表示首次渲染（componentDidMount）和之后的更新导致的重新渲染（componentDidUpdate）。

第二，useEffect中定义的副作用函数的执行不会阻碍浏览器更新视图，也就是说这些函数是异步执行的，而之前的componentDidMount或componentDidUpdate中的代码则是同步执行的。这种安排对大多数副作用说都是合理的，但有的情况除外，比如我们有时候需要先根据DOM计算出某个元素的尺寸再重新渲染，这时候我们希望这次重新渲染是同步发生的，也就是说它会在浏览器真的去绘制这个页面前发生。

#### useEffect怎么解绑一些副作用
这种场景很常见，当我们在componentDidMount里添加了一个注册，我们得马上在componentWillUnmount中，也就是组件被注销之前清除掉我们添加的注册，否则内存泄漏的问题就出现了。

怎么清除呢？让我们传给useEffect的副作用函数返回一个新的函数即可。这个新的函数将会在组件下一次重新渲染之后执行。这种模式在一些pubsub模式的实现中很常见。看下面的例子：
``` js
import { useState, useEffect } from 'react';

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // 一定注意下这个顺序：告诉react在下次重新渲染组件之后，同时是下次调用ChatAPI.subscribeToFriendStatus之前执行cleanup
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
这里有一个点需要重视！这种解绑的模式跟componentWillUnmount不一样。componentWillUnmount只会在组件被销毁前执行一次而已，而useEffect里的函数，每次组件渲染后都会执行一遍，包括副作用函数返回的这个清理函数也会重新执行一遍。所以我们一起来看一下下面这个问题。

#### 为什么要让副作用函数每次组件更新都执行一遍？
我们先看以前的模式：
```js
componentDidMount() {
  ChatAPI.subscribeToFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}

componentWillUnmount() {
  ChatAPI.unsubscribeFromFriendStatus(
    this.props.friend.id,
    this.handleStatusChange
  );
}
```
很清除，我们在componentDidMount注册，再在componentWillUnmount清除注册。但假如这时候props.friend.id变了怎么办？我们不得不再添加一个componentDidUpdate来处理这种情况：
```js
...
  componentDidUpdate(prevProps) {
    // 先把上一个friend.id解绑
    ChatAPI.unsubscribeFromFriendStatus(
      prevProps.friend.id,
      this.handleStatusChange
    );
    // 再重新注册新但friend.id
    ChatAPI.subscribeToFriendStatus(
      this.props.friend.id,
      this.handleStatusChange
    );
  }
...
```
看到了吗？很繁琐，而我们但useEffect则没这个问题，因为它在每次组件更新后都会重新执行一遍。所以代码的执行顺序是这样的：
```js
1.页面首次渲染
2.替friend.id=1的朋友注册

3.突然friend.id变成了2
4.页面重新渲染
5.清除friend.id=1的绑定
6.替friend.id=2的朋友注册
...
```

#### 怎么跳过一些不必要的副作用函数
按照上一节的思路，每次重新渲染都要执行一遍这些副作用函数，显然是不经济的。怎么跳过一些不必要的计算呢？我们只需要给useEffect传第二个参数即可。用第二个参数来告诉react只有当这个参数的值发生改变时，才执行我们传的副作用函数（第一个参数）。
```js
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 只有当count的值发生变化时，才会重新执行`document.title`这一句
```
当我们第二个参数传一个空数组[]时，其实就相当于只在首次渲染的时候执行。也就是componentDidMount加componentWillUnmount的模式。不过这种用法可能带来bug，少用。

### 还有哪些自带的Effect Hooks?
除了上文重点介绍的useState和useEffect，react还给我们提供来很多有用的hooks：
useContext
useReducer
useCallback
useMemo
useRef
useImperativeMethods
useMutationEffect
useLayoutEffect

### 怎么写自定义的Effect Hooks?
为什么要自己去写一个Effect Hooks? 这样我们才能把可以复用的逻辑抽离出来，变成一个个可以随意插拔的“插销”，哪个组件要用来，我就插进哪个组件里，so easy！看一个完整的例子，你就明白了。

比如我们可以把上面写的FriendStatus组件中判断朋友是否在线的功能抽出来，新建一个useFriendStatus的hook专门用来判断某个id是否在线。
```js
import { useState, useEffect } from 'react';

function useFriendStatus(friendID) {
  const [isOnline, setIsOnline] = useState(null);

  function handleStatusChange(status) {
    setIsOnline(status.isOnline);
  }

  useEffect(() => {
    ChatAPI.subscribeToFriendStatus(friendID, handleStatusChange);
    return () => {
      ChatAPI.unsubscribeFromFriendStatus(friendID, handleStatusChange);
    };
  });

  return isOnline;
}
```
这时候FriendStatus组件就可以简写为：
```js
function FriendStatus(props) {
  const isOnline = useFriendStatus(props.friend.id);

  if (isOnline === null) {
    return 'Loading...';
  }
  return isOnline ? 'Online' : 'Offline';
}
```
简直Perfect！假如这个时候我们又有一个朋友列表也需要显示是否在线的信息：
```js
function FriendListItem(props) {
  const isOnline = useFriendStatus(props.friend.id);

  return (
    <li style={{ color: isOnline ? 'green' : 'black' }}>
      {props.friend.name}
    </li>
  );
}
```
简直Fabulous！
