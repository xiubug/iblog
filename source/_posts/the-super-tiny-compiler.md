---
title: 有史以来最小的编译器源码解析
date: 2018-09-14 06:30:22
tags:
    - the-super-tiny-compiler
categories:
    - 源码教程
---
[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)
[详细中文注释 the-super-tiny-compiler](https://github.com/sosout/astc)
## 前言
稍微接触一点前端，我们都知道现在前端“ES6即正义”，然而浏览器的支持还处于进行阶段，所以我们常常会用一个神奇的工具将 ES6 语法转换为目前支持比较广泛的 ES5 语法，这里我们所说的神奇的工具就是编译器。编译器功能非常纯粹，将字符串形式的输入语言编译成目标语言的代码字符串（以及sourcemap），常用的编译器除了我们熟知的 Babel 之外，还有 gcc。不过我们今天的主角是号称可能是有史以来最小的编译器[the-super-tiny-compiler](https://github.com/jamiebuilds/the-super-tiny-compiler)，去掉注释也就200多行代码，作者 James Kyle 更是 Babel 的活跃维护者之一。这个编译器的功能很简单，主要把 Lisp 风格的函数调用转换成 C 风格的，例如：

|  | Lisp 风格（转化前） | C 风格（转化后） |
| ------ | ------ | ------ |
| 2 + 2 | (add 2 2)  | add(2, 2) |
| 4 - 2 | (subtract 4 2)  | subtract(4, 2) |
| 2 + (4 - 2) | (add 2 (subtract 4 2))  | add(2, subtract(4, 2)) |

## 编译器工作的三个阶段
绝大多数编译器的编译过程都差不多，主要分为三个阶段：
**解析：**将代码字符串解析成抽象语法树。
**转换：**对抽象语法树进行转换操作。
**代码生成：**根据转换后的抽象语法树生成目标代码字符串。

### 解析
解析过程主要分为两部分：词法分析和语法分析。
**1、**词法分析是由词法分析器把原始代码字符串转换成一系列词法单元（token），词法单元是一个数组，由一系列描述独立语法的对象组成，它们可以是数值、标签、标点符号、运算符、括号等。
**2、**语法分析是由语法分析器将词法分析器生成的词法单元转化为能够描述语法结构（包括语法成分及其关系）的中间表示形式（Intermediate Representation）或抽象语法树（Abstract Syntax Tree），其中抽象语法树（简称AST）是个深层嵌套的对象。

我们简单看一下 the-super-tiny-compiler 的整个解析过程：
``` js
// 原始代码字符串
(add 2 (subtract 4 2))

// 词法分析转化后生成的词法单元
[
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'add'      },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'subtract' },
  { type: 'number', value: '4'        },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: ')'        },
  { type: 'paren',  value: ')'        },
]

// 语法分析转化后生成的抽象语法树（AST）
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```
### 转换
转换过程主要任务是修改 AST，即遍历解析过程生成的 AST，同时进行一系列操作，比如增/删/改节点、增/删/改属性、创建新树等，我们简单看一下 the-super-tiny-compiler 的整个转换过程：
``` js
// 原始代码字符串
(add 2 (subtract 4 2))

// 解析过程生成的 AST
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}

// 转换过程生成的 AST
{
  type: 'Program',
  body: [{
    type: 'ExpressionStatement',
    expression: {
      type: 'CallExpression',
      callee: {
        type: 'Identifier',
        name: 'add'
      },
      arguments: [{
        type: 'NumberLiteral',
        value: '2'
      }, {
        type: 'CallExpression',
        callee: {
          type: 'Identifier',
          name: 'subtract'
        },
        arguments: [{
          type: 'NumberLiteral',
          value: '4'
        }, {
          type: 'NumberLiteral',
          value: '2'
        }]
      }
    }
  }]
}
```
### 代码生成
根据转换过程生成的抽象语法树生成目标代码字符串。

## 源码实现
接下来我们根据编译器工作的三个阶段逐一分析一下 the-super-tiny-compiler 源码实现。

### 词法分析
词法分析器主要任务把原始代码字符串转换成一系列词法单元（token）。
``` js
// 词法分析器 参数：代码字符串input
function tokenizer(input) {
  // 当前正在处理的字符索引
  let current = 0;
  // 词法单元数组
  let tokens = [];

  // 遍历字符串，获得词法单元数组
  while (current < input.length) {
    let char = input[current];

    // 匹配左括号
    if (char === '(') {

      // type 为 'paren'，value 为左圆括号的对象
      tokens.push({
        type: 'paren',
        value: '('
      });

      // current 自增
      current++;

      // 结束本次循环，进入下一次循环
      continue;
    }

    // 匹配右括号
    if (char === ')') {
      tokens.push({
        type: 'paren',
        value: ')'
      });

      current++;

      continue;
    }

    // \s：匹配任何空白字符，包括空格、制表符、换页符、换行符、垂直制表符等
    let WHITESPACE = /\s/;
    // 跳过空白字符
    if (WHITESPACE.test(char)) {
      current++;
      continue;
    }

    // [0-9]：匹配一个数字字符
    let NUMBERS = /[0-9]/;
    // 匹配数值
    if (NUMBERS.test(char)) {
      let value = '';
      // 匹配连续数字，作为数值
      while (NUMBERS.test(char)) {
        value += char;
        char = input[++current];
      }
      tokens.push({
        type: 'number',
        value
      });

      continue;
    }

    // 匹配形如"abc"的字符串
    if (char === '"') {
      let value = '';

      // 跳跃左双引号
      char = input[++current];

      // 获取两个双引号之间的所有字符
      while (char !== '"') {
        value += char;
        char = input[++current];
      }

      // 跳跃右双引号
      char = input[++current];

      tokens.push({
        type: 'string',
        value
      });

      continue;
    }

    // [a-z]：匹配1个小写字符 i 模式中的字符将同时匹配大小写字母
    let LETTERS = /[a-z]/i;
    // 匹配函数名，要求只含大小写字母
    if (LETTERS.test(char)) {
      let value = '';

      // 获取连续字符
      while (LETTERS.test(char)) {
        value += char;
        char = input[++current];
      }

      tokens.push({
        type: 'name',
        value
      });

      continue;
    }

    // 无法识别的字符，抛出错误提示
    throw new TypeError('I dont know what this character is: ' + char);
  }

  // 词法分析器的最后返回词法单元数组
  return tokens;
}
```
通过遍历代码字符串，分拣出各个词素，然后构成由一系列描述独立语法的对象组成的数组的词法单元。
### 语法分析
语法分析器主要任务是将词法分析器生成的词法单元转化为能够描述语法结构（包括语法成分及其关系）的中间表示形式（Intermediate Representation）或抽象语法树（Abstract Syntax Tree）。
``` js
// 语法分析器 参数：词法单元数组
function parser(tokens) {
  // 当前正在处理的 token 索引
  let current = 0;
  // 递归遍历（因为函数调用允许嵌套），将 token 转成 AST 节点
  function walk() {
    // 获取当前 token
    let token = tokens[current];

    // 数值
    if (token.type === 'number') {
      // current 自增
      current++;

      // 生成一个 AST节点 'NumberLiteral'，用来表示数值字面量
      return {
        type: 'NumberLiteral',
        value: token.value,
      };
    }

    // 字符串
    if (token.type === 'string') {
      current++;

      // 生成一个 AST节点 'StringLiteral'，用来表示字符串字面量
      return {
        type: 'StringLiteral',
        value: token.value,
      };
    }

    // 函数
    if (token.type === 'paren' && token.value === '(') {
      // 跳过左括号，获取下一个 token 作为函数名
      token = tokens[++current];

      let node = {
        type: 'CallExpression',
        name: token.value,
        params: []
      };

      // 再次自增 `current` 变量，获取参数 token
      token = tokens[++current];

      // 右括号之前的所有token都属于参数
      while ((token.type !== 'paren') || (token.type === 'paren' && token.value !== ')')) {
        node.params.push(walk());
        token = tokens[current];
      }

      // 跳过右括号
      current++;

      return node;
    }
    // 无法识别的字符，抛出错误提示
    throw new TypeError(token.type);
  }

  // AST的根节点
  let ast = {
    type: 'Program',
    body: [],
  };

  // 填充ast.body
  while (current < tokens.length) {
    ast.body.push(walk());
  }

  // 最后返回ast
  return ast;
}
```
通过递归来将词法分析器生成的词法单元转化为能够描述语法结构的 ast。

### 遍历
``` js
// 遍历器
function traverser(ast, visitor) {
  // 遍历 AST节点数组 对数组中的每一个元素调用 `traverseNode` 函数。
  function traverseArray(array, parent) {
    array.forEach(child => {
      traverseNode(child, parent);
    });
  }

  // 接受一个 `node` 和它的父节点 `parent` 作为参数
  function traverseNode(node, parent) {
    // 从 visitor 获取对应方法的对象
    let methods = visitor[node.type];
    // 通过 visitor 对应方法操作当前 node
    if (methods && methods.enter) {
      methods.enter(node, parent);
    }

    switch (node.type) {
      // 根节点
      case 'Program':
        traverseArray(node.body, node);
        break;
      // 函数调用
      case 'CallExpression':
        traverseArray(node.params, node);
        break;
      // 数值和字符串，不用处理
      case 'NumberLiteral':
      case 'StringLiteral':
        break;

      // 无法识别的字符，抛出错误提示
      default:
        throw new TypeError(node.type);
    }
    if (methods && methods.exit) {
      methods.exit(node, parent);
    }
  }
  // 开始遍历
  traverseNode(ast, null);
}
```
通过递归遍历 AST，在遍历过程中通过 visitor 对应方法操作当前 node，这里和切面差不多。

### 转换
``` js
// 转化器，参数：AST
function transformer(ast) {
  // 创建 `newAST`，它与之前的 AST 类似，Program：新AST的根节点
  let newAst = {
    type: 'Program',
    body: [],
  };

  // 通过 _context 维护新旧 AST，注意 _context 是一个引用，从旧的 AST 到新的 AST。
  ast._context = newAst.body;

  // 通过遍历器遍历 参数：AST 和 visitor
  traverser(ast, {
    // 数值，直接原样插入新AST
    NumberLiteral: {
      enter(node, parent) {
        parent._context.push({
          type: 'NumberLiteral',
          value: node.value,
        });
      },
    },
    // 字符串，直接原样插入新AST
    StringLiteral: {
      enter(node, parent) {
        parent._context.push({
          type: 'StringLiteral',
          value: node.value,
        });
      },
    },
    // 函数调用
    CallExpression: {
      enter(node, parent) {
        // 创建不同的AST节点
        let expression = {
          type: 'CallExpression',
          callee: {
            type: 'Identifier',
            name: node.name,
          },
          arguments: [],
        };

        // 函数调用有子类，建立节点对应关系，供子节点使用
        node._context = expression.arguments;

        // 顶层函数调用算是语句，包装成特殊的AST节点
        if (parent.type !== 'CallExpression') {

          expression = {
            type: 'ExpressionStatement',
            expression: expression,
          };
        }
        parent._context.push(expression);
      },
    }
  });
  // 最后返回新 AST
  return newAst;
}
```
这里通过 _context 引用维护新旧 AST，简单方便，但会污染旧AST。

### 代码生成
``` js
// 代码生成器 参数：新 AST
function codeGenerator(node) {

  switch (node.type) {
    // 遍历 body 属性中的节点，且递归调用 codeGenerator，结果按行输出
    case 'Program':
      return node.body.map(codeGenerator)
        .join('\n');

    // 表达式，处理表达式内容，并用分号结尾
    case 'ExpressionStatement':
      return (
        codeGenerator(node.expression) +
        ';'
      );

    // 函数调用，添加左右括号，参数用逗号隔开
    case 'CallExpression':
      return (
        codeGenerator(node.callee) +
        '(' +
        node.arguments.map(codeGenerator)
          .join(', ') +
        ')'
      );

    // 标识符，数值，原样输出
    case 'Identifier':
      return node.name;
    case 'NumberLiteral':
      return node.value;

    // 字符串，用双引号包起来再输出
    case 'StringLiteral':
      return '"' + node.value + '"';

    // 无法识别的字符，抛出错误提示
    default:
      throw new TypeError(node.type);
  }
}
```
根据转换后的新AST生成目标代码字符串。

### 编译器
``` js
function compiler(input) {
  let tokens = tokenizer(input);
  let ast    = parser(tokens);
  let newAst = transformer(ast);
  let output = codeGenerator(newAst);

  return output;
}
```
编译器整个工作流程：
**1、**input  => tokenizer   => tokens
**2、**tokens => parser      => ast
**3、**ast    => transformer => newAst
**4、**newAst => generator   => output
将上面流程串起来，就构成了简单的编译器。

                       
                  
     