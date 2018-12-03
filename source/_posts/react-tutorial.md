---
title: react 开发汇总
tags:
  - note
categories:
  - react
date: 2018-06-12 23:38:24
---

## 教程

### React v16
北京时间2017
 
### 组件复用
#### Mixin 混入模式
最原始的一种复用方式应该就是 Mixin。通过将公用逻辑封装为一个 Mixin，通过注入的方式进行组件间的复用。「ps: 该方式不仅用于组件，也流行于各种 css 预处理器中」。

在 React 中，通过`React.createClass()`方式创建的组件可使用 Mixin 模式，而在 ES6 的伪类模式下，并不支持 Mixin 模式，官方推荐用组合或者高阶组件方式实现复用，废话不多说，使用方式如下：
```js
//mixin
const mixinPart = {
  mixinFunc() {
    console.log('this mixin!');
    return 'this mixin!';
  }
};

const Contacts = React.createClass({
  mixins: [mixinPart],
  render() {
    return (
      <div>{this.mixinFunc()}</div>
    );
  }
});
// => 'this mixin!';
```
而在 Vue 中，使用逻辑类似：
```js
//mixin
const mixinPart = {
  created() {
    console.log('this mixin!');
    return 'this mixin!';
  }
};

const Component = Vue.extend({
  mixins: [mixinPart]
});
const component = new Component(); // => "this mixin!"
```
Mixin 模式给予组件公共抽象与复用能力，但另一方面也具有大量的局限性。由于 Mixin 是侵入式的，因此修改了 Mixin 相当于修改了原组件。其次，在混入过程中，对于相同键值对象与函数的相互覆盖与合并，容易导致各种意外产生。因此使用过程中必须对 Mixin 内部实现有一定了解。强大的灵活性导致了在大型项目中 Mixin 的难维护。

#### 高阶组件
高阶组件（High Order Component）这个概念最早是 React 社区提出，借鉴函数式中的高阶函数，提出通过传入一个组件，操作后返回一个新组件的方式进行复用。

在 React 中的使用非常便捷，官方博客中就有相关介绍：
```js
const HOC = (WrappedComponent) => {
  const HOC_Component = (props) => {
    return (
      <React.Fragment>
        <WrappedComponent {...props} name="WrappedComponent" />
        <div>This comes from HOC Component</div>
      </React.Fragment>
    );
  };
  HOC_Component.displayName = 'HOC_Component';
  return HOC_Component;
}
const Component = (props) => {
  return <div>This message comes from Component: {props.name}</div>;
}
const Result = HOC(Component);
ReactDOM.render(<Result />, document.getElementById('root'));
// => This message comes from Component: WrappedComponent
// => This comes from HOC Component
```
Vue 虽然没有官方示例，但与 React 进行类比，Vue 中的组件最终的展现形式是函数，但在过程中，实际上是一个个对象。因此，Vue 中的高阶组件，应当是传入一个对象，最后传出一个对应对象。我们可以简单实现个上例对应的 HOC 功能：
```js
const HOC = (WrappedComponent) => {
  const HOC_Component = (props) => {
    return (
      <React.Fragment>
        <WrappedComponent {...props} name="WrappedComponent" />
        <div>This comes from HOC Component</div>
      </React.Fragment>
    );
  };
  HOC_Component.displayName = 'HOC_Component';
  return HOC_Component;
}
const Component = (props) => {
  return <div>This message comes from Component: {props.name}</div>;
}
const Result = HOC(Component);
ReactDOM.render(<Result />, document.getElementById('root'));
// => This message comes from Component: WrappedComponent
// => This comes from HOC Component
```
Vue 虽然没有官方示例，但与 React 进行类比，Vue 中的组件最终的展现形式是函数，但在过程中，实际上是一个个对象。因此，Vue 中的高阶组件，应当是传入一个对象，最后传出一个对应对象。我们可以简单实现个上例对应的 HOC 功能：
```js
const HOC = (WrappedComponent) => {
    return {
        components: {
            'wrapped-component': WrappedComponent
        },
        template: `
          <div>
              <wrapped-component name="WrappedComponent" v-bind="$attrs" />
              <div>This comes from HOC Component</div>
            </div>
          `
    };
}
const Component = {
    props: ['name'],
    template: '<div>This message comes from Component: {{ name }}</div>'
};

new Vue(HOC(Component)).$mount('#root')
// => This message comes from Component: WrappedComponent
// => This comes from HOC Component
```
高阶组件用途十分广泛，主要可以分为**属性代理**和**反向继承**两种。
属性代理具体为高阶组件可以直接获取外部传入的参数，根据需求完成变更后重新传给被包含的组件。如上例中就在原始`props`基础上为`WrappedComponent`增加了一个`name`属性，同时在原始渲染基础上增添了一行信息渲染。
```js
// 最基本的反向继承
const HOC = (WrappedComponent) => {
  return class extends WrappedComponent {
    render() {
      return super.render();
    }
  }
}
```
反向继承因为继承于 WrappedComponent，因而能够获取其 state, render 等各种组件数据，从而做到对组件的渲染和 state 状态等的干预。反向继承虽然在日常使用中遇到情况较少，但无疑是高阶组件中笔者认为的一个闪光点 ( 貌似其它方式中暂时没有可以替代的方案 )。例如，在 Vue 中有着 keep-alive 作为组件缓存，而在 React 中官方暂无类似功能，应用 data => view 的原则，一个常用的替代实现是进行状态保存，然后在需要的时候进行状态还原，在这种情况下，反向继承就是一个很好的工具。
```js
const withStateCached = (WrappedComponent) => {
  return class extends WrappedComponent {
    static getDerivedStateFromProps(nextProps, state) {
      // 进行数据的存储等
    }

    componentDidMount() {
      // 进行缓存数据的读取
    }

    render() {
      return super.render();
    }
  }
}
```
在笔者的实际项目中，更多的把高阶组件看作是一个组件工厂或者装饰者模式的应用，例如对一个基础表格元素进行多次的高阶组件的包装，添加分页、工具栏等功能，形成一个个更符合具体业务需求的新组件，达到组件复用的目的。当然，高阶组件也不是全能的，首先其对于业务耦合度较高，更适合封装一些日常业务中常用的组件。其次最重要的弊端是因为内部产生的的 Props 值固定，容易被外部传入值覆盖。如例子中，当外部也传入了一个 name 属性值时，就会根据组件的写法产生不同的覆盖方式而导致错误。

#### 渲染属性/函数子组件
为了解决高阶组件存在的问题，一种新的「Render Props」的方案被提出。该方案提供了一个叫做 render 的函数作为 Props 参数传入，在内部处理完毕后，将所需的组件信息，数据作为 render 的参数传出，从而实现更加灵活的复用逻辑。
```js
const RenderProps = ({ render, ...props }) => render(props, 'RenderPropComponent');
const Component = () => (
    <RenderProps
        render={(originProps, componentName) => (<div>From {componentName}</div>)}
    />
);

ReactDOM.render(<Component />, document.getElementById('root'));
// => From RenderPropComponent
```
在该例中，我们通过 render 函数传入了原 Props 和一个新的 name 属性，在实际使用中，重新命名 name 为 componentName，由此避开了高阶组件的弊端。

由此理念，在 React 中，延伸出函数子组件的概念，将 children 作为函数使用，更加贯彻了一切皆为组件的概念。同时在 React@16.3 版本中，FB 官方的 Context 新 API 的实现也采用了函数子组件的方式。
```js
const RenderProps = ({ children, ...props }) => children(props, name = 'RenderPropComponent');
const Component = () => (<RenderProps>
    {(originProps, componentName) => (<div>From {componentName}</div>)}
</RenderProps>);

ReactDOM.render(<Component />, document.getElementById('root'));
// => From RenderPropComponent
```
而在 Vue@2.5 后的版本中，slot-scope 的概念也有点渲染属性的影子。
```js
const RenderProps = {
    template: `<div><slot v-bind="{ name: 'RenderPropComponent' }"></slot></div>`
};

const vm = new Vue({
    el: '#root',
    components: { 'render-props': RenderProps },
    template: `
      <render-props>
        <template slot-scope="{ name }">
          <div>From Component</div>
          From {{ name }}
        </template>
      </render-props>
    `
});
// => From Component
// => From RenderPropComponent
```
#### 组件注入
组件注入（Component Injection）的概念有些类似渲染属性，都是传递一个类似 render 的函数属性，区别在于组件注入将该函数作为 React 中的无状态组件使用，而非原始的函数。
```js
const RenderProps = ({ Render, ...props }) => <Render {...props} name='RenderPropComponent' />;
const Component = () => (<RenderProps Render={({ name }) => (<div>From {name}</div>)} />);

ReactDOM.render(<Component />, document.getElementById('root'));
// => From RenderPropComponent
```
与渲染属性相比，组件注入能在 devTool 的组件树上直观的展现出内嵌的组件结构。但在另一方面，由于所有属性都被打包成了 props 传出，反而失去了渲染属性的多参数的灵活性。

## 常见问题
### render 中使用箭头函数或绑定会导致子组件重新渲染
#### 问题：
``` js
import React from 'react';
import { render } from 'react-dom';
import User from './User';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: 'Cory' }, 
        { id: 2, name: 'Meg' }, 
        { id: 3, name: 'Bob' }
      ]
    };
  }
  
  deleteUser = id => {
    this.setState(prevState => {
      return { 
        users: prevState.users.filter( user => user.id !== id)
      }
    })
  }

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
        { 
          this.state.users.map( user => {
            return <User 
              key={user.id} 
              name={user.name} 
              onDeleteClick={() => this.deleteUser(user.id)} />
          })
        }
        </ul>
      </div>
    );
  }
}

export default App;

render(<App />, document.getElementById('root'));

// user.js
import React from 'react';

// Note how the debugger below gets hit when *any* delete
// button is clicked. Why? Because the parent component
// uses an arrow function, which means this component
//
class User extends React.PureComponent {
  render() {
    const {name, onDeleteClick } = this.props
    console.log(`${name} just rendered`);
    return (
      <li>             
        <input 
          type="button" 
          value="Delete" 
          onClick={onDeleteClick} 
        /> 
        {name}
      </li>
    );
  }
}

export default User;
```
以上例子中，在 render 函数中使用了一个箭头函数将一个值传递给了 deleteUser 函数，这就是问题的所在。

每次 render 调用时，控制台上都会打印日志。User 已经被声明为 PureComponent。所以 User 应该只在 props 或者 state 改变时才会重新 render。但是，当你点击 delete 按钮时，对于每一个 User 实例，都会调用 render。

**原因在于：**父组件在 props 中传递了一个箭头函数。箭头函数在每次 render 时都会重新分配（和使用 bind 的方式相同）。所以，尽管我将 User 声明为 PureComponent，User 的父组件中的箭头函数导致 User 组件为所有的用户实例传递了一个新的函数。所以当点击任何删除按钮时，每个用户实例都会重新 render。

**结论：**
避免在 render 中使用箭头函数和绑定。否则会打破 shouldComponentUpdate 和 PureComponent 的性能优化。

#### 解决办法
```js
import React from 'react';
import { render } from 'react-dom';
import User from './User';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      users: [
        { id: 1, name: 'Cory' }, 
        { id: 2, name: 'Meg' }, 
        { id: 3, name: 'Bob'}
      ],
    };
  }

  deleteUser = id => {
    this.setState(prevState => {
      return { 
        users: prevState.users.filter(user => user.id !== id) 
      };
    });
  };

  renderUser = user => {
    return <User key={user.id} user={user} onClick={this.deleteUser} />;
  }

  render() {
    return (
      <div>
        <h1>Users</h1>
        <ul>
          {this.state.users.map(this.renderUser)}
        </ul>
      </div>
    );
  }
}

render(<App />, document.getElementById('root'));

import React from "react";
import PropTypes from "prop-types";

// Note that the console.log below isn't called
// when delete is clicked on a user.
// That's because pureComponent's shallow
// comparison works properly here because
// the parent component isn't passing down
// an arrow function (which would cause this
// component to see a new function on each render)
class User extends React.PureComponent {
  onDeleteClick = () => {
    // No bind needed since we can compose the relevant data for this item here
    this.props.onClick(this.props.user.id);
  };

  render() {
    console.log(`${name} just rendered`);
    return (
      <li>
        <input 
          type="button" 
          value="Delete" 
          onClick={this.onDeleteClick} 
        />
        {this.props.user.name}
      </li>
    );
  }
}

User.propTypes = {
  user: PropTypes.object.isRequired,
  onClick: PropTypes.func.isRequired
};

export default User;
```
在 User.js 中，onDeleteClick 调用了在 props 中传递的 onClick 函数，并传递了相应的 user.id。

当你再次点击 delete 按钮时，其他的用户再也不会调用 render 了！

**总结**
为了最佳性能：
1、避免在 render 中使用箭头函数和绑定。
2、怎么做？提取子组件，或者直接传递数据给 HTML 元素。

### 函数作为React组件的方法时, 箭头函数和普通函数的区别是什么
**问题：**
```js
class App extends Component {
  a() {
    console.log(1)
  }

  a = () => {
    console.log(1)
  }
}
```
里面的两个 a 的定义有什么区别？
第一个 a 不必说，是原型方法的定义，很显然第一个写法是非法的。宽松模式下对应 ES5 就是：
```js
App.prototype.a = function() {}
```
第二个是 Stage 2 Public Class Fields 里面的写法，babel 下需要用 Class properties transform Plugin 进行转义。相当于：
```js
class App extends Component {
  constructor (...args) {
    super(...args)
    this.a = () => {
        console.log(1)
    }
  }
}
```
为什么需要第二种写法？

在 React 里面，要将类的原型方法通过 props 传给子组件，传统写法需要 bind(this)，否则方法执行时 this 会找不到：
```js
<button onClick={this.handleClick.bind(this)}></button>
// 或
<button onClick={(e) => this.handleClick(e)}></button>
```
这种写法难看不说，还会对 React 组件的 shouldComponentUpdate 优化造成影响。

这是因为 React 提供了 shouldComponentUpdate 让开发者能够控制避免不必要的 render，还提供了在 shouldComponentUpdate 自动进行  Shallow Compare 的 React.PureComponent, 继承自 PureComponent 的组件只要 props 和 state 中的值不变，组件就不会重新 render。

然而如果用了 bind this，每次父组件渲染，传给子组件的 props.onClick 都会变，PureComponent 的 Shallow Compare 基本上就失效了，除非你手动实现 shouldComponentUpdate.

使用 Public Class Fields 的这种写法，就解决了这个问题。另外还有其他若干种办法，比如先定义原型方法，然后在 constructor 里面 bind 一遍；或者使用 decorator 进行 bind 等：
```js
class A {
  constructor() {
    this.a = this.a.bind(this)
  }

  a() {}

  // or
  @bindthis
  b() {}
}
```

## 优秀写法
### 子组件事件父组件定义

```jsx
class Parent extends Component {
  constructor(props) {
    super(props);
    this.state = { }
  }
  getData = () => { // 子类的方法在父类直接使用
    // 子类的方法在父类直接使用
  }
  render() {
    return (
      <div className='parent-container'>
        <Child onClick={this.getData} />
      </div>
    )
  }
}

class Child extends Component {
  render() {
    const that = this;
    const { ...others } = that.props;
    return (
      <div className='child-container' {...others}>
        <div className='short-icon'></div>
      </div>
    )
  }
}
```
