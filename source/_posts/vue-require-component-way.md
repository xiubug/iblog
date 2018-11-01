---
title: Vue 组件注册时什么情况在require后面加上default
date: 2018-05-20 22:20:51
tags:
    - vue
categories:
    - 前端教程
---

今天团队内部一起回顾之前项目时，发现两个记忆中相同配置的项目，但其中一个项目可以直接用这样的代码注册组件：
``` js
Vue.component('Menu', require("./components/common/Menu.vue"));
```

然而另外一个项目却需要这样注册组件：
``` js
Vue.component('Menu', require("./components/common/Menu.vue").default);
```

否则的话就会报错:
``` 
Failed to mount component: template or render function not defined
```

**这到底是怎么回事？**

首先 webpack 支持 CommonJS、AMD 和 ES6模块打包。当我们用 .vue 单文件写组件时，在 script 标签内使用的是 ES6 的语法且使用 export default 进行默认导出。然而，require 是 CommonJS 的模块导入方式，不支持模块的默认导出，因此导入的结果其实是一个含 default 属性的对象，因此需要使用 .default 来获取实际的组件，当然我们也可以使用 ES6 的 import 语句，如果使用 import，需要给定一个变量名，所有 import 语句必须统一放在模块的开头。相反，如果 .vue 文件中使用 CommonJS 或 AMD 模块化语法，使用 module.exports 对象进行导出，那么使用 require 导入时就不需要使用 .default 来获取。