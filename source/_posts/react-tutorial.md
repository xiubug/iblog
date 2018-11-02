---
title: react 开发汇总
tags:
  - note
categories:
  - react
date: 2018-06-12 23:38:24
---

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
