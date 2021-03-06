+++
author = "zs"
title = "关于前端中的 AST"
date = "2022-05-10"
lastmod = "2022-05-16"
description = ""
tags = [
    "AST",
]
+++

我们先来看一段 JSON：

乍一看根本不知道要表示什么，很**抽象**，我们知道 JSON 可以用来存储**信息**。

而下面这段 JSON 就是 `let a = 10;` 的抽象表示，也就是**抽象语法树（AST）**。

```json
{
  "type": "Program",
  "start": 0,
  "end": 11,
  "body": [
    {
      "type": "VariableDeclaration",
      "start": 0,
      "end": 11,
      "declarations": [
        {
          "type": "VariableDeclarator",
          "start": 4,
          "end": 10,
          "id": {
            "type": "Identifier",
            "start": 4,
            "end": 5,
            "name": "a"
          },
          "init": {
            "type": "Literal",
            "start": 8,
            "end": 10,
            "value": 10,
            "raw": "10"
          }
        }
      ],
      "kind": "let"
    }
  ],
  "sourceType": "module"
}
```

## AST 是什么？

在计算机科学中，抽象语法树（Abstract Syntax Tree, AST），或简称语法树（Syntax tree），是源代码语法结构的一种抽象表示。它以树状的形式表现编程语法的语法结构，树上的每个节点都表示源代码中的一种结构。

## 如何进行转换

将 js 源码转换成 抽象语法树ast 的工具叫 **JS Parser 解析器**。

解析器也有多种，常见的有 uglifyjs、exprima、acorn等

有一个可视化工具 [AST explorer](https://astexplorer.net/)，可以直观地展示代码转换后的结果。type 字段表示的是节点的类型，比如 `Identifier`, `Literal`，然后通过对节点类型的判断分析进行需要的操作。

解析过程包括两部分：词法分析（Lexical Analysis）和语法分析（Syntax Analysis）。

词法分析，将代码按照最小的单元进行拆分，比如关键字、标识符、运算符、数字、字符串等。

语法分析，将拆分的各单元按照相互之间的关系组成一个树形结构。

相同的代码，使用不同的 Parser 解析器得到的结果是一样的，因为有一套 ast 解析规范 - [estree](https://github.com/estree/estree)

## 为什么要转换成 AST

从定义中我们得知，AST 是代码的抽象表示，转换成 AST 的形式，方便我们使用工具对源码进行批量的修改操作，比如代码的格式化、代码补全、打包等。都需要把源码转换成 AST 后再进行操作。

## 关于 [acorn-walk](https://github.com/acornjs/acorn/tree/master/acorn-walk)

>一个 ESTree 格式的抽象语法树遍历器

### 安装

通过 `npm` 是最简单的安装 acorn 的方法：
```shell
npm install acorn-walk
```

另外，你也可以下载源码构建你自己的 acorn：
```shell
git clone https://github.com/acornjs/acorn.git
cd acorn
npm install
```

### 接口

通过对语法树进行递归的算法被存储为一个对象，每个树节点类型的属性包含一个函数，该函数可以递归这些节点。这有几种方法可以运行这个遍历器。

**simple`(node, visitors, base, state)`** 在一棵树上做“简单”的遍历。

`node` 参数需要是 AST 格式。

`visitors` 是一个对象，其属性名称要与 ESTree 规范中的节点类型相对应。这些属性应该包含将要与节点对象一起调用的函数，如果适用的话，还应包括当时的状态。

最后两个参数是可选的。`base` 是一个遍历算法，`state` 是一个起始状态。默认的遍历器会简单地访问所有的语句和表达式，而不会产生有意义的状态。（使用状态的一个例子：跟踪树上每个节点的范围。）

```js
const acorn = require("acorn");
const walker = require("acorn-walk");

const source = `function fn(arg) {
	return arg;
}`;
const parse = acorn.parse(source, { ecmaVersion: 2020 });
walker.simple(parse, {
  Identifier(node) { // 标识符类型
    console.log(node);
  },
  Literal(node) { // 字面量类型
    console.log(node);
  },
  FunctionDeclaration(node) { // 识别函数类型
    console.log(node.params[0].name); // 用递归改进
  }
});
```

**ancestor`(node, visitors, base, state)`** 在一棵树上做“简单”的遍历。建立一个祖先节点的数组（包括当前节点），并将该数组作为第三个参数作为回调传递。

```js
const acorn = require("acorn")
const walk = require("acorn-walk")

walk.ancestor(acorn.parse("foo('hi')"), {
  Literal(_, ancestors) {
    console.log("This literal's ancestors are:", ancestors.map(n => n.type));
    // ["Program", "ExpressionStatement", "CallExpression", "Literal"]
  }
})
```

**recursive`(node, state, functions, state)`** 执行一个“递归”的遍历，其中 walker 函数负责在其目标节点的子节点上继续遍历。`state` 是一个起始状态，`functions` 需要包含一个将节点类型映射到 walker 函数的对象，这个函数是用 `(node, state, c)`作为参数调用的，并且可以通过用 `(node, state)` 参数调用 `c` 参数从而在一个子节点上继续遍历。可选的 `base` 参数为那些没有在函数对象中处理的节点类型提供备用的遍历函数。如果没有给出，将使用默认的遍历器。

**make`(functions, base)`** 通过使用遍历器中的 `functions`建立一个新的遍历器对象，并通过从`base`中获取默认值来填补缺失的函数。

**full`(node, callback, base, state)`** 在一棵树上进行完全的遍历，用每个节点的参数`(node, state, type)`调用回调。

**fullAncestor`(node, callback, base, state)`** 在一棵树上进行完全的遍历，建立一个祖先节点的数组（包括当前节点），并将该数组作为第三个参数传递给回调。
```js
const acorn = require("acorn");
const walker = require("acorn-walk");

walker.full(acorn.parse("1 + 1"), node => {
  console.log(`There's a ${node.type} node at ${node.ch}`)
})
```

**findNodeAt`(node, start, end, test, base, state)`** 尝试在树中从给定的起点到终点处找到一个满足`test`的节点。`start` 和 `end` 可以是 `null` （作为通配符）或数字。 `test` 可以是一个字符串（表示一个节点类型）或一个函数，它接收`(nodeType, node)`参数并返回一个布尔值，表示这个节点是否存在。`base` 和 `state`是可选的，用来指定一个自定义遍历器。节点从内到外进行测试，所以如果有两个节点与边界相匹配，将优先选择内侧的节点。

**findNodeAround`(node, pos, test, base, state)`** 与`findNodeAt`很相似，但会匹配任何存在于指定位置周围的节点。

**findNodeAfter`(node, pos, test, base, state)`** 与`findNodeAround`类似，但会匹配给定位置之后的所有节点（在内部节点之前测试外部节点）。

>翻译完了，但不懂的地方还是太多，所以得写一个小插件实践一下🤔
