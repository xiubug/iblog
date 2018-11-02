---
title: JavaScript模块打包工具Rollup——完全入门指南
date: 2018-08-04 23:04:53
tags:
  - note
categories:
  - rollup
---

`版本：v0.63.5。`

Rollup 是前端模块化的一个打包工具，可以将小块代码编译成大块复杂的代码，例如 library 或应用程序。简单地说，它可以从一个入口文件开始，将所有使用的模块根据命令或者根据 Rollup 配置文件打包成一个目标文件，并且 Rollup 会自动过滤掉那些没有被使用过的函数或变量，从而使代码最小化，如果想使用直接导入这一个目标文件即可，因此 Rollup 极其适合构建一个工具库。

这里提到 Rollup 的两个特别重要的特性，第一个就是它使用了 ES2015 的模板标准，这意味着你可以直接使用 import 和 export 而不需要引入 babel。另一个重要特性叫做 tree-shaking，这个特性可以帮助你将无用代码（即没有使用的代码）从最终的目标文件中过滤掉。举个简单的例子，我们在 foo.js 文件定义了 f1 和 f2 两个方法，然后在入口文件 index.js 只引入了 foo.js 文件中的 f1 方法，那么在最后打包 index.js 文件时，Rollup 就不会将 f2 方法打包到最终文件中。（这个特性是基于 ES6 模块的静态分析的，也就是说，只有 export 而没有 import 的变量是不会被打包到最终代码中的）。

## 入门

### 创建第一个 bundle

开始前，需要安装[Node.js](https://nodejs.org)，这样才可以使用[npm](https://npmjs.com)；还需要了解如何使用[command line](https://www.codecademy.com/learn/learn-the-command-line)。

使用 Rollup 最简单的方法是通过 Command Line Interface （或 CLI）。先全局安装 Rollup （之后会介绍如何在项目中进行安装，更便于打包，但现在不用担心这个问题）。在命令行中输入以下内容：

``` bash
$ npm install rollup --global
```

现在可以运行 rollup 命令了。试试吧~

``` bash
$ rollup
```
由于没有传递参数，所以 Rollup 打印出了使用说明。这和运行 rollup --help 或 rollup -h 的效果一样。

我们来创建一个简单的项目：

``` bash
$ mkdir -p my-rollup-project/src
$ cd my-rollup-project
```
首先，我们需要个入口文件。将以下代码粘贴到新建的文件 src/main.js 中：

``` js
// src/main.js
import { foo1 } from './foo.js';

export default function () {
  foo1();
}
```
之后创建入口文件引用的 foo.js 模块:
```js
// src/foo.js
export function foo1() {
    console.log('function foo1')
}

export function foo2() {
    console.log('function foo2')
}
```
现在可以创建 bundle 了：
```bash
$ rollup src/main.js -o bundle.js -f cjs
```
-o 表示打包后输出的文件路径，在 -o 后面的 bundle.js 就是我们最终生成的打包文件了（其实这里我们省略了参数 -i，用来表示入口文件的路径， Rollup 是会把没有加参数的文件默认是入口文件）；-f 选项（--output.format 的缩写）指定了所创建 bundle 的类型（默认使用 es 模块标准来对文件进行打包）——这里是 CommonJS（在 Node.js 中运行）。现在我们看一下输出文件 bundle.js：
```js
'use strict';

function foo1() {
    console.log('function foo1');
}

// src/main.js

function main () {
  foo1();
}

module.exports = main;

```
恭喜，你已经用 Rollup 完成了第一个 bundle。

### 使用配置文件

在项目中创建一个名为 rollup.config.js 的文件，增加如下代码：
``` js
// rollup.config.js 

// 1. output 为对象
export default {
  input: 'src/main.js',
  output: {
    file: 'bundle.js',
    format: 'cjs'
  }
};

// 2. output 为数组
export default {
    input: 'src/main.js',
    output: [{
      file: 'dist/bundle.cjs.js',
      format: 'cjs'
    }, {
      file: 'dist/bundle.umd.js',
      name: 'moduleName',
      format: 'umd'
    }, {
      file: 'dist/bundle.es.js',
      format: 'es'
    }]
};

// 整个配置为数组
export default [{
  input: 'src/main.js',
  output: {
    file: 'dist/bundle.cjs1.js',
    format: 'cjs'
  }
}, {
  input: 'src/main.js',
  output: [{
    file: 'dist/bundle.cjs2.js',
    format: 'cjs'
  }, {
    file: 'dist/bundle.umd2.js',
    name: 'moduleName',
    format: 'umd'
  }, {
    file: 'dist/bundle.es2.js',
    format: 'es'
  }]
}]
```

input 表示打包的入口文件，output 表示打包输出文件的配置（如果要输出多个，可以是一个数组，如果是数组，Rollup 会把每一个数组元素当成一个配置输出结果，因此可以在一个配置文件内设置多种输出配置），file 表示输出文件的名称路径，format 表示要打包成的模块类型。若使用 iife 或 umd 模块类型打包，需要添加属性moduleName，用来表示模块的名称；若用 amd 模块打包，可以配置 amd 相关的参数（使用 umd 模块模式时，也会使用到 amd 相关配置参数）：
``` js
amd: {
    id: 'amd-name',   // amd具名函数名称
    define: 'def'     // 用来代替define函数的函数名称
}
```

我们用 --config 或 -c 来使用配置文件：

``` bash
$ rollup -c
```
同样的命令行选项将会覆盖配置文件中的选项：
``` bash
$ rollup -c -o bundle-2.js
```
在这里我们发现配置文件使用了 ES6 语法，这是因为 Rollup 本身会处理配置文件 ，所以可以使用 export default 语法——代码不会经过 Babel 等类似工具编译，所以只能使用所用 Node.js 版本支持的 ES2015 语法。

如果愿意的话，也可以指定与默认 rollup.config.js 文件不同的配置文件：
``` bash
$ rollup --config rollup.config.dev.js
$ rollup --config rollup.config.prod.js
```

当然，我们也可以在 package.json 文件中编写 npm scripts 命令：

``` js
"build": "rollup -c"
```

针对不同模板类型我们简单编写几个命令：
```js
"build:amd": "rollup index.js -f amd -o ./dist/dist.amd.js",
"build:cjs": "rollup index.js -f cjs -o ./dist/dist.cjs.js",
"build:es": "rollup index.js -f es -o ./dist/dist.es.js",
"build:iife": "rollup index.js -f iife -n result -o ./dist/dist.iife.js",
"build:umd": "rollup index.js -f umd -n result -o ./dist/dist.umd.js",
"build": "npm run build:amd && npm run build:cjs && npm run build:es && npm run build:iife && npm run build:umd"
```
在这里我们发现在设置模块为 iife（立即执行函数）和 umd 时，还加上了一个参数 -n，这是为了事先设定模块的名称，才能让其他人通过这个模块名称引用。

### 使用ES6编写代码

许多开发人员在他们的项目中使用[Babel](https://babeljs.io/)，以便他们可以使用未被浏览器和 Node.js 支持的将来版本的 JavaScript 特性。Rollup 虽然支持了解析 import 和 export 两种语法，但是不会解析其他不被支持 JavaScript 特性，使用 Babel 和 Rollup 的最简单方法是使用[rollup-plugin-babel](https://github.com/rollup/rollup-plugin-babel)。 安装它：
``` bash
$ npm i -D babel-core rollup-plugin-babel rollup-plugin-node-resolve
```

编写 Rollup 配置文件 rollup.config.js:
``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';

export default {
    input: 'src/main.js',
    output: {
      file: 'dist/bundle.js',
      format: 'cjs'
    },
    plugins: [
      json(),
      resolve(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      })
    ]
};
```

这里有几个特别注意的地方。首先，我们设置 "modules": false，否则 Babel 会在 Rollup 有机会做处理之前，将我们的模块转成 CommonJS，导致 Rollup 的一些处理失败。

其次，我们使用 external-helpers 插件，它允许 Rollup 在包的顶部只引用一次 “helpers”，而不是每个使用它们的模块中都引用一遍（这是默认行为）。

第三，我们将 .babelrc 文件放在 src 中，而不是根目录下。 这允许我们对于不同的任务有不同的 .babelrc 配置，比如像测试，如果我们以后需要的话 - 通常为单独的任务单独配置会更好。

现在，在我们运行 rollup 之前，我们需要安装 latest preset 和 external-helpers 插件：

``` bash
$ npm i -D babel-preset-latest babel-plugin-external-helpers
```

现在我们用 es6 编辑 src / main.js：

``` js
// src/main.js
import { version } from '../package.json';

export default () => {
  console.log('version：' + version);
}
```

运行 Rollup npm run build，检查打包后的 bundle：
``` js
'use strict';

var version = "0.0.1";

// src/main.js

var main$1 = (function () {
  console.log('version：' + version);
});

module.exports = main$1;
```

## 配置文件

### 配置参数

#### external：为rollup设置外部模块和全局变量

平时开发中，我们经常会引入一些第三方模块，但是在使用的时候，我们又不想把它们打包到一个文件里，想让它们作为单独的模块（或文件）来使用，方便浏览器进行缓存，这个时候就需要使用配置文件中的 external 属性了。

我们这边以 jquery 为例，在开始使用之前，我们先安装它：
``` bash
$ npm i jquery --save-dev
```

编写 Rollup 配置文件 rollup.config.js，加入external配置:
``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs'

export default {
    input: 'src/main.js',
    output: {
      file: 'dist/bundle.js',
      format: 'cjs'
    },
    plugins: [
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      })
    ],
    external: ['jquery']
};
```

external 用来表示一个模块是否要被当成外部模块使用，属性的值可以是一个字符串数组或一个方法，当传入的是一个字符串数组时，所有数组内的模块名称都会被当成是外部模块，不会被打包到最终文件中。当传入的是一个方法时，方法有一个参数 id，表示解析的模块的名称，我们可以自定义解析方式，若是要当做外部模块不打包到最终文件中，则返回 true，若要一起打包到最终文件中，则返回 false。

#### globals

globals 的值是一个对象，key表示使用的模块名称（npm 模块名），value 表示在打包文件中引用的全局变量名，在这里我们就是把jquery模块的全局变量名设置为jQuery，重新打包。

编写 Rollup 配置文件 rollup.config.js，加入globals配置:
``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs'

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        name: 'result',
        format: 'iife',
        globals: {
            jquery: 'jQuery'
        }
    },
    plugins: [
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      })
    ],
    external: ['jquery']
};
```

运行 Rollup npm run build，检查打包后的 bundle：

``` js
var result = (function (jQuery) {
  'use strict';

  jQuery = jQuery && jQuery.hasOwnProperty('default') ? jQuery['default'] : jQuery;

  var version = "0.0.1";

  // src/main.js

  var main$1 = (function () {
      console.log(jQuery);
      console.log('version：' + version);
  });

  return main$1;

}(jQuery));
```

在重新打包出来的文件中，我们发现最后传入的参数已经由 $ 变为了 jQuery，而且 Rollup 也没有输出提示信息。

#### paths

有时候我们可能会使用 CDN 上的 js 文件，但是又不想在本地安装一个相同的模块（也有可能没有对应的模块），可能在版本升级的时候会产生一些问题，这个时候我们就需要使用 Rollup 的 paths 属性了，这个属性可以帮你把依赖的文件地址注入到打包后的文件里。

编写 Rollup 配置文件 rollup.config.js，加入 paths 配置:

``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs'

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        name: 'result',
        format: 'amd',
        globals: {
            jquery: 'jQuery'
        },
        paths: {
            jquery: 'https://cdn.bootcss.com/jquery/3.2.1/jquery.js'
        }
    },
    plugins: [
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      })
    ],
    external: ['jquery']
};
```

运行 Rollup npm run build，检查打包后的 bundle：

``` js
define(['https://cdn.bootcss.com/jquery/3.2.1/jquery.js'], function (jQuery) { 'use strict';

  jQuery = jQuery && jQuery.hasOwnProperty('default') ? jQuery['default'] : jQuery;

  var version = "0.0.1";

  // src/main.js

  var main$1 = (function () {
      console.log(jQuery);
      console.log('version：' + version);
  });

  return main$1;

});
```

可以看到 Rollup 已经把我们需要的 CDN 地址作为依赖加入到了打包文件中。

## 插件

### 使用插件

随着构建更复杂的 bundle，通常需要更大的灵活性——引入 npm 安装的模块、通过 Babel 编译代码、和 JSON 文件打交道等。为此，我们可以用 插件(plugins) 在打包的关键过程中更改 Rollup 的行为。[the Rollup wiki](https://github.com/rollup/rollup/wiki/Plugins)维护了可用的插件列表。

我们这边将以[rollup-plugin-json](https://github.com/rollup/rollup-plugin-json)的使用为例，它的作用是令 Rollup 从 JSON 文件中读取数据。

将 rollup-plugin-json 安装为开发依赖：
``` bash
$ npm install --save-dev rollup-plugin-json
```
我们用的是 --save-dev 而不是 --save，因为实际执行的代码并不依赖这个插件——只是在打包时使用。

更新 src/main.js 文件，从 package.json 而非 src/foo.js 中读取数据：
``` js
// src/main.js
import { version } from '../package.json';

export default function () {
  console.log('version：' + version);
}
```
编写 Rollup 配置文件 rollup.config.js，加入 JSON 插件：
``` js
// rollup.config.js
import json from 'rollup-plugin-json';

export default {
    input: 'src/main.js',
    output: {
      file: 'dist/bundle.js',
      format: 'cjs'
    },
    plugins: [ json() ]
};
```
npm run build 执行 Rollup。结果如下：
``` js
'use strict';

var version = "0.0.1";

// src/main.js

function main$1 () {
  console.log('version：' + version);
}

module.exports = main$1;
```

### 插件列表

#### rollup-plugin-commonjs

有时候我们会引入一些其他模块的文件（第三方的或是自己编写的），但是目前，npm 中的大多数包都是以 CommonJS 模块的形式出现的。在它们更改之前，我们需要将CommonJS模块转换为 ES2015 供 Rollup 解析。这个[rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs)插件就是用来将 CommonJS 转换成 ES2015 模块的。请注意，rollup-plugin-commonjs 应该用在其他插件转换你的模块之前 - 这是为了防止其他插件的改变破坏 CommonJS 的检测。

在我们使用之前，需要先安装它：
``` bash
$ npm i rollup-plugin-commonjs --save-dev
```
编写 Rollup 配置文件 rollup.config.js:

``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs'

export default {
    input: 'src/main.js',
    output: {
      file: 'dist/bundle.js',
      format: 'cjs'
    },
    plugins: [
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      })
    ]
};
```

编写 cjs 模块的文件：

``` js
exports.foo1 = function() {
    console.log('function foo1')
}

exports.foo2 = function() {
    console.log('function foo2')
}
```

npm run build 执行 Rollup。结果如下：
```js
'use strict';

var version = "0.0.1";

// src/main.js

var main$1 = (function () {
  console.log('version：' + version);
});

module.exports = main$1;

```

#### rollup-plugin-uglify

代码发布时，我们经常会把自己的代码压缩到最小，以减少网络请求中的传输文件大小。Rollup rollup-plugin-uglify 就是来帮你压缩代码的，在使用之前，我们先安装它：
``` bash
$ npm i rollup-plugin-uglify --save-dev
```

编写 Rollup 配置文件 rollup.config.js，加入 uglify 插件：

``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs';
import { uglify } from 'rollup-plugin-uglify'

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        name: 'result',
        format: 'amd',
        globals: {
            jquery: 'jQuery'
        },
        paths: {
            jquery: 'https://cdn.bootcss.com/jquery/3.2.1/jquery.js'
        }
    },
    plugins: [
      json(),
      resolve(),
      commonjs(),
      babel({
        exclude: 'node_modules/**' // 只编译我们的源代码
      }),
      uglify()
    ],
    external: ['jquery']
};
```
运行打包命令，查看打包后的目标文件，发现代码已经被压缩了。但是，压缩过的代码在 debug 时会带来很大的不便，因此我们需要在压缩代码的同时生成一个 sourceMap 文件。幸运的是，Rollup 自己就支持 sourceMap 文件的生成，不需要我们去引入其他插件，只需要在配置文件中 output 选项加上以下代码即可：

``` js 
// rollup.config.js
sourcemap: true
```

若是将 sourceMap 属性的值设置为 inline，则会将 sourceMap 的内容添加到打包文件的最后。

#### rollup-plugin-eslint

在大型工程的团队开发中，我们需要保证团队代码风格的一致性，因此需要引入 eslint，而且在打包时需要检测源文件是否符合 eslint 设置的规范，若是不符合则抛出异常并停止打包。Rollup rollup-plugin-eslint 就是用于设置代码规范，使用之前我们先安装它：

``` bash
$ npm i eslint rollup-plugin-eslint --save-dev
```

编写 eslint 配置文件 .eslintrc：

``` json
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es6": true,
        "node": true
    },
    "parserOptions": {
        "ecmaFeatures": {
            "jsx": false
        },
        "sourceType": "module"
    },
    "rules": {
        "semi": ["error","never"]
    }
}
```

编写 Rollup 配置文件 rollup.config.js，加入 eslint 插件：
``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs';
import { uglify } from 'rollup-plugin-uglify';
import { eslint } from 'rollup-plugin-eslint';

export default {
    input: 'src/main.js',
    output: {
        file: 'dist/bundle.js',
        name: 'result',
        format: 'amd',
        globals: {
            jquery: 'jQuery'
        },
        paths: {
            jquery: 'https://cdn.bootcss.com/jquery/3.2.1/jquery.js'
        },
        sourcemap: true
    },
    plugins: [
        eslint({
            throwOnError: true,
            throwOnWarning: true,
            include: ['src/**'],
            exclude: ['node_modules/**']
        }),
        json(),
        resolve(),
        commonjs(),
        babel({
            exclude: 'node_modules/**' // 只编译我们的源代码
        }),
        uglify()
    ],
    external: ['jquery']
};
```

这里有两个属性需要特别说明下：throwOnError 和 throwOnWarning 设置为 true 时，如果在 eslint 的检查过程中发现了 error 或 warning，就会抛出异常，阻止打包继续执行（如果设置为 false，就只会输出 eslint 检测结果，而不会停止打包）。

如果我们使用IDE或编辑器的 eslint 插件，有时候这些插件会去检查打包完的文件，导致你的提示框里一直会有 eslint 检测到错误的消息，我们现在有两种解决方案，第一种是创建一个 .eslintignore 文件，将打包文件加进去，让 eslint 忽略这个文件，还有一种就是让 Rollup 在打包文件的开始和最后自动生成注释来阻止 eslint 检测代码，使用这种方法时，需要使用   Rollup 配置文件的两个属性：banner和footer，这两个属性会在生成文件的开头和结尾插入一段你自定义的字符串。我们利用这个属性，在打包文件的开头添加`/*eslint-disable */`注释，让 eslint 不检测这个文件。

添加banner和footer属性

``` js
banner: '/*eslint-disable */'
```

如果说 banner 和 footer 是在文件开始和结尾添加字符串，那么 intro 和 outro 就是在被打包的代码开头和结尾添加字符串了，以 iife 模式来举例，如果我们配置了这四个属性，那么输出结果就会是：
``` js
// banner字符串
(function () {
'use strict';
// intro字符串

// 被打包的代码

// outro字符串
}());
// footer字符串
```

#### rollup-plugin-replace

有时候我们会把开发/生产环境的信息直接写在源文件里面，这个时候用 intro/outro 来注入代码的方式就不适合了。这个时候我们就需要使用 rollup-plugin-replace 插件来对源代码的变量值进行替换，在使用之前，我们先安装它：

``` bash
$ npm i rollup-plugin-replace --save-dev
```

编写 Rollup 配置文件 rollup.config.js，加入 replace 插件：
``` js
// rollup.config.js
import json from 'rollup-plugin-json';
import resolve from 'rollup-plugin-node-resolve';
import babel from 'rollup-plugin-babel';
import commonjs from 'rollup-plugin-commonjs';
import { uglify } from 'rollup-plugin-uglify';
import { eslint } from 'rollup-plugin-eslint';
import { version } from '../package.json';
import replace  from 'rollup-plugin-replace';

const VERSION = process.env.VERSION || version;

const copyright = new Date().getFullYear() > 2018 ? '2018-' + new Date().getFullYear() : 2018;

const banner =
  '/*!\n' +
  ' * idebug v' + VERSION + '\n' +
  ' * (c) ' + copyright + ' Weich\n' +
  ' * Released under the MIT License.\n' +
  ' */';

// const weexFactoryPlugin = {
//     intro () {
//         return 'module.exports = function weexFactory (exports, document) {'
//     },
//     outro () {
//         return '}'
//     }
// };

export default {
    input: 'src/main.js',
    output: {
        banner: banner,
        footer: '/* my-library version ' + VERSION + ' */',
        file: 'dist/bundle.js',
        name: 'result',
        format: 'iife',
        globals: {
            jquery: 'jQuery'
        },
        paths: {
            jquery: 'https://cdn.bootcss.com/jquery/3.2.1/jquery.js'
        },
        sourcemap: true
    },
    plugins: [
        // weexFactoryPlugin,
        replace({
            __VERSION__: VERSION
        }),
        eslint({
            throwOnError: true,
            throwOnWarning: true,
            include: ['src/**'],
            exclude: ['node_modules/**']
        }),
        json(),
        resolve(),
        commonjs(),
        babel({
            exclude: 'node_modules/**' // 只编译我们的源代码
        }),
        uglify({
            output: {
              comments: function(node, comment) {
                  var text = comment.value;
                  var type = comment.type;
                  if (type == "comment2") {
                      // multiline comment
                      return /idebug|ENVIRONMENT/i.test(text);
                  }
              }
            }
          })
    ],
    external: ['jquery']
};
```

接下来就可以直接在源码中使用 `__VERSION__` 了，编写入口文件 index.js：
```js
// src/main.js
import jQuery from 'jquery'

export default () => {
    console.log(jQuery)
    console.log('version：__VERSION__' )
}
```

执行打包命令，并检查源文件里有没有被替换。

## 命令行

### 命令行的参数

`-v/--version`：打印已安装的Rollup版本号。

`-w/--watch`：我们在开发过程中，需要频繁对源文件进行修改，如果每次都自己手动输一遍打包命令，那真的是要烦死，因此，我们在 rollup 命令后面加上 -w/--watch 参数，就能让 rollup 监听文件变化，即时打包。

## 本文持续更新中



