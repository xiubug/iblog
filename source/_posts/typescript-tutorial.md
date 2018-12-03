---
title: typescript 教程
tags:
  - note
categories:
  - typescript
date: 2018-11-30 23:44:49
---

## TSLint 规则
一套完整的 TSLint 规则，以及对每条规则的释义。

### 使用方法
以`tslint-config-alloy`为例：
安装：

```bash
npm install --save-dev tslint typescript tslint-config-alloy
```

在你的项目根目录下创建 `tslint.json`，并将以下内容复制到文件中：

```
{
    "extends": ["tslint-config-alloy"],
    "linterOptions": {
        "exclude": ["**/node_modules/**"]
    },
    "rules": {
        // 这里填入你的项目需要的个性化配置，比如：
        //
        // // 一个缩进必须用两个空格替代
        // // @has-fixer 可自动修复
        // // @prettier 可交由 prettier 控制
        // "indent": [true, "spaces", 2]
    }
}
```

#### CLI 中运行

使用项目依赖中的 `tslint` 脚本，指定项目路径，检查所有 ts 后缀的文件：

```bash
./node_modules/.bin/tslint --project . ./**/*.ts
```

将 `tslint` 作为 npm scripts 运行：

1. `package.json` 的 `scripts` 字段添加一条 `"tslint": "tslint --project . ./**/*.ts"`
2. 运行 `npm run tslint`

#### 与 VSCode 集成

1. 在 VSCode 中安装 tslint 插件
2. 按下 `Cmd` + `,` 或 `Ctrl` + `,`，打开设置
3. 将 `tslint.autoFixOnSave`，配置为 `true`

#### 与 Prettier 集成

Prettier 是一个专注于对代码风格进行统一格式化的工具，由于与 TSLint 的部分配置冲突，故需要使用 tslint-config-prettier 禁用掉 TSLint 的部分规则。

首先安装 prettier 和 tslint-config-prettier：

```
npm install --save-dev prettier tslint-config-prettier
```

然后为 `tslint.config` 的 `extends` 添加 `tslint-config-prettier` 即可：

```
{
    "extends": ["tslint-config-alloy", "tslint-config-prettier"],
    "linterOptions": {
        "exclude": ["**/node_modules/**"]
    },
    "rules": {
        // 这里填入你的项目需要的个性化配置，比如：
        //
        // // 一个缩进必须用两个空格替代
        // // @has-fixer 可自动修复
        // // @prettier 可交由 prettier 控制
        // "indent": [true, "spaces", 2]
    }
}
```

如果需要在 VSCode 中实现保存时修复 Prettier 的问题，则可以按照以下步骤配置：

1. VSCode 安装 Prettier - Code formatter 插件
2. 按下 `Cmd` + `,` 或 `Ctrl` + `,`，打开设置
3. 将 `tslint.formatOnSave`，配置为 `true`

Prettier 的配置文件 `prettier.config.js` 可以参考这个：

```js
// prettier.config.js or .prettierrc.js
module.exports = {
    // 一行最多 100 字符
    printWidth: 100,
    // 使用 4 个空格缩进
    tabWidth: 4,
    // 不使用缩进符，而使用空格
    useTabs: false,
    // 行尾需要有分号
    semi: true,
    // 使用单引号
    singleQuote: true,
    // jsx 不使用单引号，而使用双引号
    jsxSingleQuote: false,
    // 末尾不需要逗号
    trailingComma: 'none',
    // 大括号内的首尾需要空格
    bracketSpacing: true,
    // jsx 标签的反尖括号需要换行
    jsxBracketSameLine: false,
    // 箭头函数，只有一个参数的时候，也需要括号
    arrowParens: 'always',
    // 每个文件格式化的范围是文件的全部内容
    rangeStart: 0,
    rangeEnd: Infinity,
    // 不需要写文件开头的 @prettier
    requirePragma: false,
    // 不需要自动在文件开头插入 @prettier
    insertPragma: false,
    // 使用默认的折行标准
    proseWrap: 'preserve',
    // 根据显示样式决定 html 要不要折行
    htmlWhitespaceSensitivity: 'css',
    // 换行符使用 lf
    endOfLine: 'lf'
};
```

### 规则列表

#### TypeScript 相关

与 TypeScript 特性相关的规则。

| 名称        | 描述 |
| ------     | ------     | 
| [adjacent-overload-signatures](https://palantir.github.io/tslint/rules/adjacent-overload-signatures/) |重载的函数**必须**写在一起|
| [ban-types](https://palantir.github.io/tslint/rules/ban-types/)     | 禁用特定的类型 |
| [member-access](https://palantir.github.io/tslint/rules/member-access/)     | **必须**指定类的成员的可访问性 |
| [member-ordering](https://palantir.github.io/tslint/rules/member-ordering/)     | 指定类成员的排序规则 |
| [no-any](https://palantir.github.io/tslint/rules/no-any/)     | **禁止**使用any |
| [no-empty-interface](https://palantir.github.io/tslint/rules/no-empty-interface/)     | **禁止**定义空的接口 |
| [no-import-side-effect](https://palantir.github.io/tslint/rules/no-import-side-effect/)     | **禁止**导入立即执行的模块，除了`css`、`less`、`sass`、`scss` |
| [no-inferrable-types](https://palantir.github.io/tslint/rules/no-inferrable-types/)     | **禁止**给一个初始化时直接赋值为`number`、`string`或`boolean`的变量显式的指定类型 |
| [no-internal-module](https://palantir.github.io/tslint/rules/no-internal-module/)     | **禁止**使用`module`来定义命名空间 |
| [no-magic-numbers](https://palantir.github.io/tslint/rules/no-magic-numbers/)     | **禁止**使用魔法数字，仅允许使用一部分白名单中的数字 |
| [no-namespace](https://palantir.github.io/tslint/rules/no-any/)     | **禁止**使用`namespace`来定义命名空间 |
| [no-non-null-assertion](https://palantir.github.io/tslint/rules/no-non-null-assertion/)     | **禁止**使用 non-null 断言（感叹号） |
| [no-parameter-reassignment](https://palantir.github.io/tslint/rules/no-parameter-reassignment/)     | **禁止**对函数的参数重新赋值 |
| [no-reference](https://palantir.github.io/tslint/rules/no-reference/)     | **禁止**使用三斜线引入模块`/// <reference path="foo" />` |
| [no-unnecessary-type-assertion](https://palantir.github.io/tslint/rules/no-unnecessary-type-assertion/)     | **禁止**无用的类型断言 |
| [no-var-requires](https://palantir.github.io/tslint/rules/no-unnecessary-type-assertion/)     | **禁止**使用`require`来引入模块 |
| [only-arrow-functions](https://palantir.github.io/tslint/rules/only-arrow-functions/)     | **必须**使用箭头函数，除非是单独的函数声明或是命名函数 |
| [prefer-for-of](https://palantir.github.io/tslint/rules/prefer-for-of/)     | 使用`for`循环遍历数组时，如果`index`仅用于获取成员，则**必须**使用`for of`循环替代`for`循环 |
| [promise-function-async](https://palantir.github.io/tslint/rules/promise-function-async/)     | `async`函数的返回值**必须**是`Promise` |
| [typedef](https://palantir.github.io/tslint/rules/typedef/)     | 变量、函数返回值、函数参数等**必须**要有类型定义 |
| [typedef-whitespace](https://palantir.github.io/tslint/rules/typedef-whitespace/)     | 类型定义的冒号前面**必须**没有空格，后面**必须**有一个空格 |
| [unified-signatures](https://palantir.github.io/tslint/rules/unified-signatures/)     | 函数重载时，若能通过联合类型将两个函数的类型声明合为一个，则使用联合类型而不是两个函数声明 |

#### 功能性检查

找出可能的错误，以及可能会产生 bug 的编码习惯。

| 名称        | 描述 |
| ------     | ------     | 
| [await-promise](https://palantir.github.io/tslint/rules/await-promise/) |`await`**必须**接受`Promise`|
| [ban](https://palantir.github.io/tslint/rules/ban/) |禁用指定的函数或全局方法|
| [ban-comma-operator](https://palantir.github.io/tslint/rules/ban-comma-operator/) |**禁止**使用逗号操作符|
| [curly](https://palantir.github.io/tslint/rules/curly/) |`if`后面**必须**有` {`，除非是单行`if`|
| [forin](https://palantir.github.io/tslint/rules/forin/) |`for in`内部**必须**有`hasOwnProperty`|
| [import-blacklist](https://palantir.github.io/tslint/rules/import-blacklist/) |禁用指定的模块|
| [label-position](https://palantir.github.io/tslint/rules/label-position/) |只允许在`do`, `for`, `while` 或 `switch` 中使用 `label`|
| [no-arg](https://palantir.github.io/tslint/rules/no-arg/) |**禁止**使用`arguments.callee`|
| [no-bitwise](https://palantir.github.io/tslint/rules/no-bitwise/) |**禁止**使用位运算|
| [no-conditional-assignment](https://palantir.github.io/tslint/rules/no-conditional-assignment/) |**禁止**在分支条件判断中有赋值操作|
| [no-console](https://palantir.github.io/tslint/rules/no-console/) |**禁止**使用`console`|
| [no-construct](https://palantir.github.io/tslint/rules/no-construct/) |**禁止**使用`new`来生成`String`,`Number`或`Boolean`|
| [no-debugger](https://palantir.github.io/tslint/rules/no-debugger/) |**禁止**使用`debugger`|
| [no-duplicate-super](https://palantir.github.io/tslint/rules/no-duplicate-super/) |**禁止**`super`在一个构造函数中出现两次|
| [no-duplicate-switch-case](https://palantir.github.io/tslint/rules/no-duplicate-switch-case/) |**禁止**在`switch`语句中出现重复测试表达式的`case`|
| [no-duplicate-variable](https://palantir.github.io/tslint/rules/no-duplicate-variable/) |**禁止**出现重复的变量定义或函数参数名|
| [no-dynamic-delete](https://palantir.github.io/tslint/rules/no-dynamic-delete/) |**禁止**`delete`动态的值|
| [no-empty](https://palantir.github.io/tslint/rules/no-empty/) |**禁止**出现空代码块，允许`catch`是空代码块|
| [no-eval](https://palantir.github.io/tslint/rules/no-eval/) |**禁止**使用`eval`|
| [no-floating-promises](https://palantir.github.io/tslint/rules/no-floating-promises/) |函数返回值为`Promise`时，**必须**被处理|
| [no-for-in-array](https://palantir.github.io/tslint/rules/no-for-in-array/) |**禁止**对`array`使用`for in`循环|
| [no-implicit-dependencies](https://palantir.github.io/tslint/rules/no-implicit-dependencies/) |**禁止**引入`package.json`中不存在的模块|
| [no-inferred-empty-object-type](https://palantir.github.io/tslint/rules/no-inferred-empty-object-type/) |**禁止**推论出的类型是空对象类型|
| [no-invalid-template-strings](https://palantir.github.io/tslint/rules/no-invalid-template-strings/) |**禁止**在非模版字符串中出现`${}`|
| [no-invalid-this](https://palantir.github.io/tslint/rules/no-invalid-this/) |**禁止**在类外面使用`this`|
| [no-misused-new](https://palantir.github.io/tslint/rules/no-misused-new/) |**禁止**在接口中定义`constructor`，或在类中定义`new`|
| [no-null-keyword](https://palantir.github.io/tslint/rules/no-null-keyword/) |**禁止**使用`null`|
| [no-object-literal-type-assertion](https://palantir.github.io/tslint/rules/no-object-literal-type-assertion/) |**禁止**对对象字面量进行类型断言（断言成`any`是允许的）|
| [no-return-await](https://palantir.github.io/tslint/rules/no-return-await/) |**禁止**没必要的`return await`|
| [no-shadowed-variable](https://palantir.github.io/tslint/rules/no-shadowed-variable/) |**禁止**变量名与上层作用域内的定义过的变量重复|
| [no-sparse-arrays](https://palantir.github.io/tslint/rules/no-sparse-arrays/) |**禁止**在数组中出现连续的逗号，如`let foo = [,,]`|
| [no-string-literal](https://palantir.github.io/tslint/rules/no-string-literal/) |**禁止**出现`foo['bar']`，必须写成`foo.bar`|
| [no-string-throw](https://palantir.github.io/tslint/rules/no-string-throw/) |**禁止**`throw`字符串，必须`throw`一个`Error`对象|
| [no-submodule-imports](https://palantir.github.io/tslint/rules/no-submodule-imports/) |**禁止**`import`模块的子文件|
| [no-switch-case-fall-through](https://palantir.github.io/tslint/rules/no-switch-case-fall-through/) |`switch` 的 `case` 必须 `return` 或 `break`|
| [no-this-assignment](https://palantir.github.io/tslint/rules/no-this-assignment/) |**禁止**将`this`赋值给其他变量，除非是解构赋值|
| [no-unbound-method](https://palantir.github.io/tslint/rules/no-unbound-method/) |使用实例的方法时，**必须**`bind`到实例上|
| [no-unnecessary-class](https://palantir.github.io/tslint/rules/no-unnecessary-class/) |**禁止**定义没必要的类，比如只有静态方法的类|
| [no-unsafe-any](https://palantir.github.io/tslint/rules/no-unsafe-any/) |**禁止**取用一个类型为`any`的对象的属性|
| [no-unsafe-finally](https://palantir.github.io/tslint/rules/no-unsafe-finally/) |**禁止**`finally`内出现`return`,`continue`,`break`,`throw`等|
| [no-unused-expression](https://palantir.github.io/tslint/rules/no-unused-expression/) |**禁止**无用的表达式|
| [no-use-before-declare](https://palantir.github.io/tslint/rules/no-use-before-declare/) |变量`必须`先定义后使用|
| [no-var-keyword](https://palantir.github.io/tslint/rules/no-var-keyword/) |**禁止**使用`var`|
| [no-void-expression](https://palantir.github.io/tslint/rules/no-void-expression/) |**禁止**返回值为`void`类型|
| [prefer-conditional-expression](https://palantir.github.io/tslint/rules/prefer-conditional-expression/) |可以用三元表达式时，就不用`if else`|
| [prefer-object-spread](https://palantir.github.io/tslint/rules/prefer-object-spread/) |使用`{ ...foo, bar: 1 }`代替`Object.assign({}, foo, { bar: 1 })`|
| [radix](https://palantir.github.io/tslint/rules/radix/) |`parseInt`**必须**传入第二个参数|
| [restrict-plus-operands](https://palantir.github.io/tslint/rules/restrict-plus-operands/) |使用加号时，两者**必须**同为数字或同为字符串|
| [strict-boolean-expressions](https://palantir.github.io/tslint/rules/strict-boolean-expressions/) |在分支条件判断中**必须**传入布尔类型的值|
| [strict-type-predicates](https://palantir.github.io/tslint/rules/strict-type-predicates/) |**禁止**出现永远为`true`或永远为`false`的条件判断（通过类型预测出一个表达式为`true`还是`false`）|
| [switch-default](https://palantir.github.io/tslint/rules/switch-default/) |`switch`语句**必须**有`default`|
| [triple-equals](https://palantir.github.io/tslint/rules/triple-equals/) |**必须**使用`===`或`!==`，**禁止**使用`==`或`!=`|
| [typeof-compare](https://palantir.github.io/tslint/rules/typeof-compare/) |`typeof`表达式比较的对象**必须**是`'undefined'`,`'object'`,`'boolean'`,`'number'`,`'string'`,`'function'`或`'symbol'`|
| [use-default-type-parameter](https://palantir.github.io/tslint/rules/use-default-type-parameter/) |传入的类型与默认类型一致时，**必须**省略传入的类型|
| [use-isnan](https://palantir.github.io/tslint/rules/use-isnan/) |**必须**使用`isNaN(foo)`而不是`foo === NaN`|

#### 可维护性

增强代码可维护性的规则。

| 名称        | 描述 |
| ------     | ------     | 
| [cyclomatic-complexity](https://palantir.github.io/tslint/rules/cyclomatic-complexity/) |**禁止**函数的循环复杂度超过 20，详见 https://en.wikipedia.org/wiki/Cyclomatic_complexity|
| [deprecation](https://palantir.github.io/tslint/rules/deprecation/) |**禁止**使用废弃（被标识了`@deprecated`）的`API`|
| [eofline](https://palantir.github.io/tslint/rules/eofline/) |文件最后一行**必须**有一个空行|
| [indent](https://palantir.github.io/tslint/rules/indent/) |一个缩进**必须**用四个空格替代|
| [linebreak-style](https://palantir.github.io/tslint/rules/linebreak-style/) |限制换行符为 LF 或 CRLF|
| [max-classes-per-file](https://palantir.github.io/tslint/rules/max-classes-per-file/) |限制每个文件的类的数量|
| [max-file-line-count](https://palantir.github.io/tslint/rules/max-file-line-count/) |限制每个文件的行数|
| [max-line-length](https://palantir.github.io/tslint/rules/max-line-length/) |限制每行字符数|
| [no-default-export](https://palantir.github.io/tslint/rules/no-default-export/) |**禁止**使用`default export`|
| [no-duplicate-imports](https://palantir.github.io/tslint/rules/no-duplicate-imports/) |**禁止**出现重复的`import`|
| [no-require-imports](https://palantir.github.io/tslint/rules/no-require-imports/) |**禁止**使用`require`|
| [object-literal-sort-keys](https://palantir.github.io/tslint/rules/object-literal-sort-keys/) |对象字面量**必须**按`key`排序|
| [prefer-const](https://palantir.github.io/tslint/rules/prefer-const/) |申明后不再被修改的变量**必须**使用`const`来申明|
| [prefer-readonly](https://palantir.github.io/tslint/rules/prefer-readonly/) |如果私有变量只在构造函数中被赋值，则**必须**使用`readonly`修饰符|
| [trailing-comma](https://palantir.github.io/tslint/rules/trailing-comma/) |限制对象、数组、解构赋值等的最后一项末尾是否需要逗号|


#### 代码风格

与代码风格相关的规则。

| 名称        | 描述 |
| ------     | ------     | 
| [align](https://palantir.github.io/tslint/rules/align/) |变量定义需要竖向对其|
| [array-type](https://palantir.github.io/tslint/rules/array-type/) |限制**必须**使用`T[]`或`Array<T>`之中的一种来定义数组的类型|
| [arrow-parens](https://palantir.github.io/tslint/rules/arrow-parens/) |箭头函数的参数**必须**有小括号|
| [arrow-return-shorthand](https://palantir.github.io/tslint/rules/arrow-return-shorthand/) |箭头函数的函数体只有`return`语句的时候，**必须**简写|
| [binary-expression-operand-order](https://palantir.github.io/tslint/rules/binary-expression-operand-order/) |数字字面量**必须**在加号的右边，即**禁止**`1 + x`|
| [callable-types](https://palantir.github.io/tslint/rules/callable-types/) |可以简写为函数类型的接口或字面类似，**必须**简写|
| [class-name](https://palantir.github.io/tslint/rules/class-name/) |类名与接口名**必须**为驼峰式|
| [comment-format](https://palantir.github.io/tslint/rules/comment-format/) |限制单行注释的规则|
| [completed-docs](https://palantir.github.io/tslint/rules/completed-docs/) |类、函数等**必须**写注释|
| [encoding](https://palantir.github.io/tslint/rules/encoding/) |文件类型**必须**是`utf-8`|
| [file-header](https://palantir.github.io/tslint/rules/file-header/) |文件的开头**必须**有指定的字符串|
| [file-name-casing](https://palantir.github.io/tslint/rules/file-name-casing/) |约束文件命名规范|
| [import-spacing](https://palantir.github.io/tslint/rules/import-spacing/) |`import`语句中，关键字之间的间距**必须**是一个空格|
| [interface-name](https://palantir.github.io/tslint/rules/interface-name/) |接口名称**必须**已`I`开头|
| [interface-over-type-literal](https://palantir.github.io/tslint/rules/interface-over-type-literal/) |优先使用接口而不是字面类型|
| [jsdoc-format](https://palantir.github.io/tslint/rules/jsdoc-format/) |注释**必须**符合`JSDoc`规范|
| [match-default-export-name](https://palantir.github.io/tslint/rules/match-default-export-name/) |`import`的名称**必须**和`export default`的名称一致|
| [new-parens](https://palantir.github.io/tslint/rules/new-parens/) |`new`后面只**必须**有一个空格|
| [newline-before-return](https://palantir.github.io/tslint/rules/newline-before-return/) |`return`语句前**必须**有空行|
| [newline-per-chained-call](https://palantir.github.io/tslint/rules/newline-per-chained-call/) |链式调用时，每次调用都**必须**占用一行|
| [no-angle-bracket-type-assertion](https://palantir.github.io/tslint/rules/no-angle-bracket-type-assertion/) |类型断言**必须**使用`as Type`，**禁止**使用`<Type>`|
| [no-boolean-literal-compare](https://palantir.github.io/tslint/rules/no-boolean-literal-compare/) |**禁止**变量与`true`或`false`比较|
| [no-consecutive-blank-lines](https://palantir.github.io/tslint/rules/no-consecutive-blank-lines/) |**禁止**连续超过三行空行|
| [no-irregular-whitespace](https://palantir.github.io/tslint/rules/no-irregular-whitespace/) |**禁止**使用特殊空白符（比如全角空格|
| [no-parameter-properties](https://palantir.github.io/tslint/rules/no-parameter-properties/) |**禁止**给类的构造函数的参数添加修饰符|
| [no-redundant-jsdoc](https://palantir.github.io/tslint/rules/no-redundant-jsdoc/) |**禁止**JSDoc 中的冗余类型声明，因为 TypeScirpt 已经包含了大部分功能|
| [no-reference-import](https://palantir.github.io/tslint/rules/no-reference-import/) |如果已经引入过库，则**禁止**使用三斜杠引入类型定义文件|
| [no-trailing-whitespace](https://palantir.github.io/tslint/rules/no-trailing-whitespace/) |**禁止**行尾有空格|
| [no-unnecessary-callback-wrapper](https://palantir.github.io/tslint/rules/no-unnecessary-callback-wrapper/) |**禁止**没必要的函数调用，如`x => f(x)`应该简写为`f`|
| [no-unnecessary-initializer](https://palantir.github.io/tslint/rules/no-unnecessary-initializer/) |**禁止**变量定义时赋值为`undefined`|
| [no-unnecessary-qualifier](https://palantir.github.io/tslint/rules/no-unnecessary-qualifier/) |在命名空间中，可以直接使用内部变量，不需要添加命名空间前缀|
| [number-literal-format](https://palantir.github.io/tslint/rules/number-literal-format/) |小数**必须**以`0.`开头，**禁止**以`.`开头，并且不能以`0`结尾|
| [object-literal-key-quotes](https://palantir.github.io/tslint/rules/object-literal-key-quotes/) |对象的`key`**必须**用引号包起来|
| [object-literal-shorthand](https://palantir.github.io/tslint/rules/object-literal-shorthand/) |**必须**使用`a = {b}`而不是`a = {b: b}`|
| [one-line](https://palantir.github.io/tslint/rules/one-line/) |`if`后的`{`**禁止**换行|
| [one-variable-per-declaration](https://palantir.github.io/tslint/rules/one-variable-per-declaration/) |变量申明**必须**每行一个，`for`循环的初始条件中除外|
| [ordered-imports](https://palantir.github.io/tslint/rules/ordered-imports/) |`import`**必须**排序|
| [prefer-function-over-method](https://palantir.github.io/tslint/rules/prefer-function-over-method/) |类中没有使用`this`的方法应该提取成类外的函数|
| [prefer-method-signature](https://palantir.github.io/tslint/rules/prefer-method-signature/) |**必须**使用`foo(): void`而不是`foo: () => void`|
| [prefer-switch](https://palantir.github.io/tslint/rules/prefer-switch/) |当`if`中只有`=== `时，**必须**使用`switch`替换`if`|
| [prefer-template](https://palantir.github.io/tslint/rules/prefer-template/) |**必须**使用模版字符串而不是字符串连接|
| [prefer-while](https://palantir.github.io/tslint/rules/prefer-while/) |当没有初始值的时候，**必须**使用`while`而不是`for`|
| [quotemark](https://palantir.github.io/tslint/rules/quotemark/) |**必须**使用单引号，`jsx`中**必须**使用双引号|
| [return-undefined](https://palantir.github.io/tslint/rules/return-undefined/) |使用`return;`而不是`return undefined;`|
| [semicolon](https://palantir.github.io/tslint/rules/semicolon/) |行尾**必须**有分号|
| [space-before-function-paren](https://palantir.github.io/tslint/rules/space-before-function-paren/) |函数名前**必须**有空格|
| [space-within-parens](https://palantir.github.io/tslint/rules/space-within-parens/) |括号内首尾**禁止**有空格|
| [switch-final-break](https://palantir.github.io/tslint/rules/switch-final-break/) |`switch`的最后一项**禁止**有`break`|
| [type-literal-delimiter](https://palantir.github.io/tslint/rules/type-literal-delimiter/) |字面类型的每个成员都**必须**有分号|
| [variable-name](https://palantir.github.io/tslint/rules/variable-name/) |限制变量命名规则|
| [whitespace](https://palantir.github.io/tslint/rules/whitespace/) |限制空格的位置|

