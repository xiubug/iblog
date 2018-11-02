---
title: 聊聊es7的decorator修饰器
tags:
  - decorator
categories:
  - es7
date: 2018-10-21 21:08:46
---

### 类的修饰

许多面向对象的语言都有修饰器（Decorator）函数，用来修改类的行为。目前，有一个提案将这项功能，引入了ECMAScript

```javascript
@testable
class MyTestableClass {
    // ...
}
function testable(target) {
    target.isTestable = true;
}

MyTestableClass.isTestable; // true
```

上面代码中，@testable就是一个修饰器。它修改了MyTestableClass这个类的行为，为它加上了静态属性isTestable。testable函数的参数target是MyTestableClass类本身。

基本上，修饰器的行为就是下面这样。

```javascript
@decorator
class A {}

等同于

class A {}
A = decorator(A) || A;
```

也就是说，修饰器是一个对类进行处理的函数。修饰器函数的第一个参数，就是所要修饰的目标类。

```javascript
function testable(target) {
  // ...
}
```

上面代码中，testable函数的参数target，就是会被修饰的类。

如果觉得一个参数不够用，可以在修饰器外面再封装一层函数。

```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```

上面代码中，修饰器testable可以接受参数，这就等于可以修改修饰器的行为。

注意，修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。也就是说，修饰器本质就是编译时执行的函数。

前面的例子是为类添加一个静态属性，如果想添加实例属性，可以通过目标类的prototype对象操作。

```javascript
function testable(target) {
  target.prototype.isTestable = true;
}

@testable
class MyTestableClass {}

let obj = new MyTestableClass();
obj.isTestable // true
```

上面代码中，修饰器函数testable是在目标类的prototype对象上添加属性，因此就可以在实例上调用。

下面是另外一个例子。

```javascript
// mixins.js
export function mixins(...list) {
  return function (target) {
    Object.assign(target.prototype, ...list)
  }
}

// main.js
import { mixins } from './mixins'

const Foo = {
  foo() { console.log('foo') }
};

@mixins(Foo)
class MyClass {}

let obj = new MyClass();
obj.foo() // 'foo'
```

上面代码通过修饰器mixins，把Foo类的方法添加到了MyClass的实例上面。可以用Object.assign()模拟这个功能。

```javascript
const Foo = {
  foo() { console.log('foo') }
};

class MyClass {}

Object.assign(MyClass.prototype, Foo);

let obj = new MyClass();
obj.foo() // 'foo'
```

实际开发中，React 与 Redux 库结合使用时，常常需要写成下面这样。

```javascript
class MyReactComponent extends React.Component {}

export default connect(mapStateToProps, mapDispatchToProps)(MyReactComponent);
```

有了装饰器，就可以改写上面的代码。

```javascript
@connect(mapStateToProps, mapDispatchToProps)
export default class MyReactComponent extends React.Component {}
```

### static

static涉及到了ES6的class，我们定义一个组件的时候通常是定义了一个类，而static则是创建了一个属于这个类的属性或者方法。

组件则是这个类的一个实例，component的props和state是属于这个实例的，所以实例还未创建，我们又怎么可能读得到props和state呢？

总结来说static并不是react定义的，而加上static关键字，就表示该方法不会被实例继承，而是直接通过类来调用