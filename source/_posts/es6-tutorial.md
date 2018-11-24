---
title: es6 教程
tags:
  - note
categories:
  - es6
date: 2018-11-23 21:39:37
---

## 基本用法
### Symbol
Symbol 是EC6规格所支持的一种新的数据类型

#### 作用
* 作为属性名避免属性名冲突
* 替代代码中多次使用的字符串（例如：abc），多次使用的字符串在代码中不易维护，而这时候定义一个对象的属性（属性名用Symbol格式），值为abc，就可以作为全局变量来使用了。
* 由于以Symbol值作为名称的属性，不会被常规方法遍历得到。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法。
* 这个有时，我们希望重新使用同一个Symbol值，Symbol.for方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值。

### Set 和 Map 数据结构
#### WeakMap
##### 概念
按照MDN上的说明：
> WeakMap 对象是键/值对的集合，且其中的键是弱引用的。其键只能是对象，而值则可以是任意的。

从这段描述来看，我们可以大致推断出，WeakMap与Map的主要区别在于两点：
* WeakMap对key的引用是弱引用
* WeakMap的key只能是对象null除外），不接受其他类型的值作为键名

``` js
// WeakMap 可以使用 set 方法添加成员
const wm1 = new WeakMap();
const key = {foo: 1};
wm1.set(key, 2);
wm1.get(key) // 2

// WeakMap 也可以接受一个数组，
// 作为构造函数的参数
const k1 = [1, 2, 3];
const k2 = [4, 5, 6];
const wm2 = new WeakMap([[k1, 'foo'], [k2, 'bar']]);
wm2.get(k2) // "bar"
```

##### 特性
具体而言，WeakMap大致有如下一些明显的特性：
**1、WeakMap的key只能是对象（null除外），不接受其他类型的值作为键名**
例如：
```js
var m = new WeakMap();
var k = {};
// 设置键值对
m.set(k, 1);
// 取值
m.get(k);   //1
// 非对象的key会报错
m.set(1, 2)
// TypeError: 1 is not an object!
m.set(Symbol(), 2)
// TypeError: Invalid value used as weak map key
m.set(null, 2)
// TypeError: Invalid value used as weak map key
```
在上例中，我们使用了对象k作为WeakMap的key，设置了value为1。到下面取值的时候，只能使用同一个对象k去取。也就是说WeakMap是按照key的引用来对value进行存取的。如果将数值1和Symbol值作为 WeakMap 的键名，都会报错。

> 关于“key只能使用对象”和“value的查找是通过比较key的引用”这两个命题，其实是互为因果的，本质上是一个先有鸡还是先有蛋的问题：
正因为WeakMap只能使用对象作为key，所以取值的时候对key进行查找也只能按对象引用进行查找。
正因为WeakMap在查找的时候只能按对象引用进行查找，所以只能使用对象作为key，否则存进去的值根本无法查找取出。

**2、key中的对象保持弱引用，不计入垃圾回收机制**
弱引用正是WeakMap中“Weak”的含义。熟悉JavaScript的朋友都知道引用是怎么回事，简单地说，当一个对象被引用的时候，往往意味着它正在被使用，或者在将来有可能会被使用。此时对象不会被垃圾回收机制回收掉。
WeakMap的设计目的在于，有时我们想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用。请看下面的例子。
```js
const e1 = document.getElementById('foo');
const e2 = document.getElementById('bar');
const arr = [
  [e1, 'foo 元素'],
  [e2, 'bar 元素'],
];

```
上面代码中，e1和e2是两个对象，我们通过arr数组对这两个对象添加一些文字说明。这就形成了arr对e1和e2的引用。

一旦不再需要这两个对象，我们就必须手动删除这个引用，否则垃圾回收机制就不会释放e1和e2占用的内存。
``` js
// 不需要 e1 和 e2 的时候
// 必须手动删除引用
arr [0] = null;
arr [1] = null;
```
上面这样的写法显然很不方便。一旦忘了写，就会造成内存泄露。

WeakMap 就是为了解决这个问题而诞生的，它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

基本上，如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。一个典型应用场景是，在网页的 DOM 元素上添加数据，就可以使用WeakMap结构。当该 DOM 元素被清除，其所对应的WeakMap记录就会自动被移除。
```js
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```
上面代码中，先新建一个 Weakmap 实例。然后，将一个 DOM 节点作为键名存入该实例，并将一些附加信息作为键值，一起存放在 WeakMap 里面。这时，WeakMap 里面对element的引用就是弱引用，不会被计入垃圾回收机制。

也就是说，上面的 DOM 节点对象的引用计数是1，而不是2。这时，一旦消除对该节点的引用，它占用的内存就会被垃圾回收机制释放。Weakmap 保存的这个键值对，也会自动消失。

总之，WeakMap的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap结构有助于防止内存泄漏。

注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。
``` js
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```
上面代码中，键值obj是正常引用。所以，即使在 WeakMap 外部消除了obj的引用，WeakMap 内部的引用依然存在。

##### 语法
WeakMap 与 Map 在 API 上的区别主要是两个，一是没有遍历操作（即没有keys()、values()和entries()方法），也没有size属性。因为没有办法列出所有键名，某个键名是否存在完全不可预测，跟垃圾回收机制是否运行相关。这一刻可以取到键名，下一刻垃圾回收机制突然运行了，这个键名就没了，为了防止出现不确定性，就统一规定不能取到键名。二是无法清空，即不支持clear方法。因此，WeakMap只有四个方法可用：get()、set()、has()、delete()。
``` js
const wm = new WeakMap();

// size、forEach、clear 方法都不存在
wm.size // undefined
wm.forEach // undefined
wm.clear // undefined
```

##### 用途
WeakMap 应用的典型场合就是 DOM 节点作为键名。

### Module 的语法
#### export 与 import 的复合写法
如果在一个模块之中，先输入后输出同一个模块，import语句可以与export语句写在一起。
``` js
export { foo, bar } from 'my_module';

// 可以简单理解为
import { foo, bar } from 'my_module';
export { foo, bar };
```
上面代码中，export和import语句可以结合在一起，写成一行。但需要注意的是，写成一行以后，foo和bar实际上并没有被导入当前模块，只是相当于对外转发了这两个接口，导致当前模块不能直接使用foo和bar。

模块的接口改名和整体输出，也可以采用这种写法。
```js
// 接口改名
export { foo as myFoo } from 'my_module';

// 整体输出
export * from 'my_module';
```

默认接口的写法如下。
``` js
export { default } from 'foo';
```

具名接口改为默认接口的写法如下。
```js
export { es6 as default } from './someModule';

// 等同于
import { es6 } from './someModule';
export default es6;
```

同样地，默认接口也可以改名为具名接口。
```js
export { default as es6 } from './someModule';
```

下面三种import语句，没有对应的复合写法。
```js
import * as someIdentifier from "someModule";
import someIdentifier from "someModule";
import someIdentifier, { namedIdentifier } from "someModule";
```

为了做到形式的对称，现在有提案，提出补上这三种复合写法。
```js
export * as someIdentifier from "someModule";
export someIdentifier from "someModule";
export someIdentifier, { namedIdentifier } from "someModule";
```