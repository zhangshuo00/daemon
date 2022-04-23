+++
author = "zs"
title = "JavaScript 是如何工作的"
date = "2022-04-21"
lastmod = "2022-04-22"
description = ""
tags = [
    "basic",
]
+++

>* 原文地址：[How JavaScript Works🔥 🤖 [Visually Explained]](https://dev.to/narottam04/how-javascript-works-visually-explained-269j)
> * 原文作者：Narottam04

这篇博客将解释 JavaScript 如何在浏览器中执行代码，我们将通过 gif 动画来学习它。读完这篇文章你将距离成为一名摇滚明星开发者更近一步。

## 执行上下文

>**“JavaScript 中的一切都发生在执行上下文中。“**

我希望每个人都能记住这句话，因为这非常重要。你可以把执行上下文当作一个大的容器，在浏览器想要运行一些 JavaScript 代码时就会去里面调用。

在这个容器中有两个组件。
1. 内存组件（Memory component） 
2. 代码组件（Code component）

内存组件也被称为变量环境。变量和函数以键值对的形式存储在变量环境中。代码组件是容器中的一个地方，代码在这里被逐行执行。这个代码组件也有一个花哨的名字，“执行线程”。我认为这听起来很酷!

**JavaScript是一种同步的、单线程的语言。** 这是因为它一次只能执行一条命令，而且是按照特定的顺序。

## 代码的执行

让我们看一个简单的例子，

```js
var a = 2;
var b = 4;

var sum = a + b;
console.log(sum);
```

在这个例子中，我们初始化了两个变量，**a** 和 **b**，分别存储 `2` 和 `4`。然后我们把 a 和 b 的值相加并存储到变量 **sum** 中。

让我们看看 JavaScript 是如何在浏览器中执行这段代码的。

![1](http://qinius.acrosstheuniverse.top/images/jq3ufd0eru2ceax067m9.gif)

浏览器创建了一个全局的执行上下文，包括内存组件和代码组件两部分。

浏览器将分两个阶段执行JavaScript代码：
1> 内存创建阶段
2> 代码执行阶段

在内存创建阶段，JavaScript 会扫描所有的代码，为代码中的所有变量和函数分配内存。对于变量，JavaScript 会在内存创建阶段存储 `undefined`，对于函数，它会保留整个函数代码，我们将在下面的例子中看到。

![2](http://qinius.acrosstheuniverse.top/images/4ty49vslo873hpehxdrw.gif)

现在在第二阶段，即代码执行阶段，开始逐行扫描整个代码。当它遇到`var a = 2`时，它将 **2** 分配给内存中的 `a`。直到现在，`a` 的值是 **undefined**。

同样地，它对 `b` 做了相同的事情。它将 **4** 分配给 `b`。现在，在最后一步，它将 `sum` 打印在控制台中，然后销毁全局执行环境（因为我们的代码已经完成）。

## 在执行上下文中如何调用函数？

当你和其他编程语言中的函数比较时，你会发现 JavaScript 中函数的工作方式和它们有些不同。

让我们看个简单的例子：

```js
var n = 2;

function square(sum) {
    var ans = sum * sum;
    return ans;
}

var square2 = square(n);
var square4 = square(4);
```

上面的例子中有一个函数 **square**，它接收一个数字类型的参数并返回数字的平方。

当我们运行代码时，JavaScript 将创建一个全局执行上下文并为第一阶段的所有变量和函数分配内存，如下图所示。

对于函数，它将把整个函数存储在内存中。

![3](http://qinius.acrosstheuniverse.top/images/68nk5l6806bax94k0tky.gif)

当 JavaScript 运行函数时，它将在全局执行上下文中创建一个执行上下文。

当它遇到 `var a = 2` 时，它把 **2** 分配给内存中的 `n`。第 2 行是一个函数，由于该函数在内存执行阶段已经被分配了内存，它将直接跳到第 6 行。

`square2` 变量将调用 `square` 函数，而 JavaScript 将创建一个新的执行上下文。

![4](http://qinius.acrosstheuniverse.top/images/zvfyis150o3i7bn1x6hy.gif)

`square` 函数的新的执行上下文将在内存创建阶段为该函数中存在的所有变量分配内存。

![5](http://qinius.acrosstheuniverse.top/images/e67rsojvcqmowwj3w75b.gif)

在给函数内的所有变量分配内存后，它将逐行执行代码。然后得到 `num` 的值，即第一个值为 **2** 的变量，之后计算出 `ans`。`ans` 计算完毕后，返回需要分配给 `square2` 的值。

一旦这个函数返回了值，这个函数的执行上下文将被销毁，因为已经完成了它的工作。

![6](http://qinius.acrosstheuniverse.top/images/b2zu35q2as6uy57qve9q.gif)

现在它将对第7行的square4变量进行类似的程序，如下所示。

![7](http://qinius.acrosstheuniverse.top/images/q7wlgf8uj91cpglpvh0z.gif)

一旦所有代码执行完毕，全局执行上下文也将被销毁，这就是 JavaScript 在幕后执行代码的方式。

## 调用堆栈

当在 JavaScript 中调用一个函数时，JavaScript 会创建一个执行上下文。当在函数内部再嵌套函数时，执行上下文将变得很复杂。

![8](http://qinius.acrosstheuniverse.top/images/idywyfc19t2vsf1nyww1.png)

JavaScript 在调用栈的帮助下管理代码执行上下文的创建和删除。

堆栈（有时被称为 "下推栈"）是一个有序的集合，新增的项和现有项的移除总是发生在同一端，比如，一摞书。

调用堆栈是一种在调用多个函数的脚本中跟踪其位置的机制。

```js
function a() {
    function insideA() {
        return true;
    }
    insideA();
}
a();
```

我们创建了一个函数 `a`，它调用另一个函数 `insideA`，返回 **true**。我知道这段代码很蠢，什么都没做，但它会帮助我们理解 JavaScript 如何处理回调函数。

![9](http://qinius.acrosstheuniverse.top/images/03bry7soja8z3ad143ry.gif)

JavaScript 将创建一个全局的执行上下文，全局执行上下文将为函数 `a` 分配内存，并在代码执行阶段调用 `function a`。

为 `function a` 创建了一个执行上下文，该执行上下文位于调用栈中的全局执行上下文之上。

`function a` 将会给 `function insideA` 分配内存并调用它，接着为 `function insideA` 创建一个执行上下文，并置于 `function a` 的调用栈之上。

现在，这个 `function insideA` 将返回 **true** 并将从调用堆栈中删除。

由于 `function a` 中没有代码，它的执行上下文将被从调用栈中删除。

最后，全局执行上下文也将从调用栈中删除。