---
title: Vue.js 组件间通信方式
tags:
  - vue
categories:
  - 前端教程
date: 2018-04-25 21:27:06
---
> Vue 组件通信包括：父子组件和兄弟组件间的通信。在组件化系统构建中，组件间通信必不可少的。

### 父组件向子组件通信
#### props
父组件核心传递数据代码如下：
``` html
<template>
  <child :msg="message"></child>
</template>

<script>
import child from './child.vue';

export default {
  components: {
    child
  },
  data () {
    return {
      message: 'I am Father！'
    }
  }
}
</script>
```
子组件核心接收数据代码如下：
``` html
<template>
  <div>{{ msg }}</div>
</template>

<script>
export default {
  props: {
    msg: {
      type: String,
      required: true
    }
  }
}
</script>
```
msg 为父组件给子组件设置的额外属性值，属性值需在子组件中设置 props，子组件中可直接使用 msg 变量。

#### 实例方法
父组件通过 $children 可以访问所有直接子组件（父组件的子组件的子组件不是直接子组件）。注意 $children 并不保证顺序，也不是响应式的。

### 子组件向父组件通信

#### 事件
子组件核心传递数据代码如下：
``` html
<template>
  <button @click="handleClick">传递数据</button>
</template>

<script>
export default {
  props: {
    msg: {
      type: String,
      required: true
    }
  },
  methods () {
    handleClick () {
      this.$emit("sonMsg", "I am Son！");
    }
  }
}
</script>
```
父组件核心接收数据代码如下：
``` html
<template>
    <child @sonMsg="getSonMsg"></child>
</template>

<script>
import child from './child.vue';

export default {
  components: {
    child
  },
  methods: {
    getSonMsg (msg) {
      console.log(msg); // I am Son！
    }
  }
}
</script>
```
父组件向子组件传递事件方法，子组件通过 $emit 触发事件，回调给父组件。sonMsg 为子组件触发的事件名称，父组件中 @sonMsg 为向子组件传递的事件名称，getSonMsg 为父组件事件回调方法。

#### ref
设置子组件 ref 属性的值，比如：
``` html
<!-- 子组件。 ref的值是组件引用的名称 -->
<child-component ref="aName"></child-component>
```
父组件中通过`$refs.组件名`来获得子组件，也就可以调用子组件的属性和方法了。
``` js
const child = this.$refs.aName
child.属性
child.方法()
```

#### 实例方法
子组件通过 $parent 访问父组件。

### Bus中央通信
目前中央通信是解决兄弟间通信，祖父祖孙间通信的最佳方法，不仅限于此，也可以解决父组件子组件间的相互通信。如下图（盗图）：
![img1.png](/images/vue-component-communication/img1.png)

各组件可自己定义好组件内接收外部组件的消息事件即可，不用理会是哪个组件发过来；而对于发送事件的组件，亦不用理会这个事件到底怎么发送给我需要发送的组件。

**先设置Bus**：
``` js
// bus.js 
import Vue from 'vue'
export default new Vue();
```
**发送事件的组件：**
``` js
import bus from '@/bus';
// 方法内执行下面动作
bus.$emit('child-message', this.data);
```
**组件内监听事件：**
``` js
import bus from '@/bus';

export default {
  name: 'child',
  methods: {
  },
  created() {
    bus.$on('child-message', function(data) {
      console.log('I get it');
    });
  }
};
```
Bus中央通信的方案各种情况下都可用，比较方便。

### 复杂的单页应用数据管理
当应用足够复杂情况下，我们要使用vuex进行数据管理。


