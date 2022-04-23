+++
author = "zs"
title = "JavaScript 原型初级指南"
date = "2022-04-09"
lastmod = "2022-04-09"
description = "A Beginner's Guide to JavaScript Prototype"
tags = [
    "Basic",
]
+++

>* 原文地址：[A Beginner’s Guide to JavaScript’s Prototype](https://medium.com/free-code-camp/a-beginners-guide-to-javascript-s-prototype-9c049fe7b34)
> * 原文作者：Tyler McGinnis

对象是以键值对的形式存在的。最常用的创建对象的方法是用大括号 `{}`，之后你可以通过点符号 `.` 给对象增加属性和方法。

```js
let animal = {};
animal.name = 'Leo';
animal.energy = 10;

animal.eat = function(amount) {
    console.log(`${this.name} is eating.`);
    this.energy += amount;
}

animal.sleep = function(length) {
    console.log(`${this.name} is sleeping.`);
    this.energy += length;
}

animal.play = function(length) {
    console.log(`${this.name} is playing.`);
    this.energy -= length;
}
```

在应用程序中，我们可能需要创建不止一种「动物」。下一步是将这一逻辑封装在函数中，当创建一种新的「动物」时，也可以调用这个函数，我们把这种模式叫做 **「函数实例化」** ，并且把这个函数称为 **「构造函数」**，因为它负责构造一个新的对象。

```js
function Animal(name, energy) {
    let animal = {};
    animal.name = name;
    animal.energy = energy;

    animal.eat = function(amount) {
        console.log(`${this.name} is eating.`);
        this.energy += amount;
    }

    animal.sleep = function(length) {
        console.log(`${this.name} is sleeping.`);
        this.energy += length;
    }

    animal.play = function(length) {
        console.log(`${this.name} is playing.`);
        this.energy -= length;
    }
    return animal;
}

const leo = Animal('Leo', 7);
const snoop = Animal('Snoop', 10);
```

`“我想这应该是一个 JavaScript 的进阶教程...？” - 你的大脑`

是的，我们最后会讲到那里的。

现在无论我们想要创建一个新的「动物」（或者更宽泛地说是创建一个新的实例），只需要调用 `Animal` 函数，并且把动物的名字和体力传给它。

这种方式很好用，而且简单的让人难以置信。但是，你能从这种方式中发现什么缺点吗？最大的问题，也是我们接下来要解决的问题，于构造函数中的三种方法有关：`eat、sleep、play`。

这些方法中的每一个不仅是动态的，而且也是完全通用的。这也就是说，当创建一个新的「动物」时，我们没有必要再去重复地创建这些方法。否则只是在浪费内存，并且使每个「动物对象」变得比它需要的还要大。

你可以想到一些解决方法吗？在创建新的「动物」时，是否可以不再重新创建这些方法，而是把它们放到属于它们自己的对象中呢？然后让每个「动物」都引用这个对象。

> 我们把这种模式叫做 **`Functional Instantiation with Shared Methods 共享方法的函数实例化`** ，不那么优雅，但是一看就明白。

### 共享方法的函数实例化

```js
const animalMethods = {
    eat(mount) {
        console.log(`${this.name} is eating.`);
        this.energy += amount;
    },
    sleep(length) {
        console.log(`${this.name} is sleeping.`);
        this.energy += length;
    },
    play(length) {
        console.log(`${this.name} is playing.`);
        this.energy -= length;
    }
}

function Animal (name, energy) {
  let animal = {}
  animal.name = name
  animal.energy = energy
  animal.eat = animalMethods.eat
  animal.sleep = animalMethods.sleep
  animal.play = animalMethods.play
  return animal
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
```

通过将共享的方法移动到它们自己的对象中，然后在 Animal 方法中引用这个对象，我们解决了内存浪费和 animal 对象过大的问题。

### Object.create

让我们再用 Object.create 改善一下。简单来说，Object.create 允许我们创建一个对象，它将在查找失败时委托给另一个对象。

换句话说，Object.create 允许你创建一个对象，每当在该对象上查找属性失败时，它可以咨询另一个对象，看看其他对象是否有该属性。我们来看段代码。

```js
const parent = {
    name: 'Stacey',
    age: 35,
    heritage: 'Irish'
};

const child = Object.create(parent);
child.name = 'Ryan';
child.age = 7;

console.log(child.name); // Ryan
console.log(child.age); // 7
console.log(child.heritage); // Irish
```

在上面例子中，`child` 是用 `Object.create(parent)` 创建的，所以每当在 `child` 中查找属性失败时，JavaScript 就会把查询委托给父对象。

这就意味着，尽管 `child` 没有 `heritage` 属性，但是 `parent` 有，所以当你打印 `child.heritage` 时，你将会得到 `parent.heritage` 也就是 `Irish`。

现在我们的工具箱里又多了 `Object.create`，那么怎么用它来简化前面的 `Animal` 呢？

那么，与其把所有的共享方法一个接一个地添加给 **`animal`**，不如用 **`Object.create`** 委托给 **`animalMethods`**。

听起来非常不错，我们把这个称为 **带有共享方法和 Object.create 的函数实例化**。🙃

### 带有共享方法和 Object.create 的函数实例化

```js
const animalMethods = {
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  },
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  },
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

function Animal(name, energy) {
    let animal = Object.create(animalMethods);
    animal.name = name;
    animal.energy = energy;

    return animal;
}

const leo = Animal('Leo', 7);
const snoop = Animal('Snoop', 10);
leo.eat(10);
snoop.play(5);
```

所以现在当我们调用 `leo.eat`，JavaScript 将会在 `leo` 对象中寻找 `eat` 方法。这样并找不到，但因为我们使用了 `Object.create` -- 它将委托给 `animalMethods` 对象，在那里可以找到 `eat` 方法。

目前来说都很不错，不过我们还可以做一些提升。为了在不同的实例间共享方法，不得不管理一个单独的对象，这貌似有一点 `"hacky"`。这似乎是一个常见的功能，我们希望它在语言底层实现。事实证明是这样的，这也是你看这篇文章的原因 -- `prototype`。
>hacky `(maybe, (of a piece of computer code) providing a clumsy or inelegant solution to a particular problem.) （一段计算机代码）为特定问题提供笨拙或不优雅的解决方案。`

那么，JavaScript 中的原型到底是什么？emmm，简单来说，JavaScript 中的每个函数都有一个原型属性，它指向了一个对象。不太相信是吗？自己试一下吧。

```js
function doThing() {}
console.log(doThing.prototype) // {}
```

如果我们不是创建一个抽离的对象来管理方法（像`animalMethods`那样），而是把这些方法放到`Animal`函数的原型上呢？接下来我们要做的就是，不用`Object.create`委托给`animalMethods`，而是把它委托给`Animal.prototype`。我们把这一模式称为 **`原型实例化`**。

### 原型实例化

```js
function Animal(name, energy){
    let animal = Object.create(Animal.prototype);
    animal.name = name;
    animal.energy = energy;

    return animal;
}

Animal.prototype.eat = function(amount) {
    console.log(`${this.name} is eating.`);
    this.energy += amount;
}

Animal.prototype.sleep = function(length) {
    console.log(`${this.name} is sleeping.`);
    this.energy += length;
}

Animal.prototype.play = function(length) {
    console.log(`${this.name} is playing.`);
    this.energy -= length;
}

const leo = Animal('Leo', 7)
const snoop = Animal('Snoop', 10)
leo.eat(10)
snoop.play(5)
```

👏👏👏 希望你能有所收获。同样，prototype 只是 JavaScript 中每个函数都有的一个属性，正如在上面例子中看到的，它允许我们在一个函数的所有实例中共享方法。所有的功能都是一样的，但是现在不用为所有的方法去管理一个单独的对象，而只需要使用另一个内置于 `Animal` 函数的对象，`Animal.prototype`。

在这点上我们知道三件事情：

1. 如何创建一个构造函数；
2. 如何像构造函数的原型添加方法；
3. 如何使用 Object.create 将失败的查找委托给函数的原型；

这三项任务似乎是任何编程语言的基础性工作。
难道JavaScript真的那么糟糕，以至于没有更简单的、"内置 "的方法来完成同样的事情？
和你猜测的一样，可以通过使用 `new` 关键字来完成。

通过之前的方法，循序渐进地帮助你对 `new` 关键字有一个更深刻的理解。

回头看一下 `Animal` 这个构造函数，最重要的两个部分是创建对象和返回对象。
如果没有用 Object.create 创建对象，就不能在查找失败时委托给函数的原型；如果没有返回语句，我们就拿不到创建的对象。

```js
function Animal (name, energy) {
  let animal = Object.create(Animal.prototype);
  animal.name = name;
  animal.energy = energy;
  return animal;
}
```

`new` 的好处是：当使用 `new` 关键字调用一个函数时，这两行是在底层隐式完成的，所创建的对象称为`this`。

假设 Animal 构造函数是通过 new 关键字调用的，注释部分展示的是底层执行的内容。
```js
function Animal (name, energy) {
  // const this = Object.create(Animal.prototype);
  this.name = name;
  this.energy = energy;
  // return this;
}
const leo = new Animal('Leo', 7);
const snoop = new Animal('Snoop', 10);
```

去掉隐式执行的注释：
```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}
Animal.prototype.eat = function (amount) {
  console.log(`${this.name} is eating.`)
  this.energy += amount
}
Animal.prototype.sleep = function (length) {
  console.log(`${this.name} is sleeping.`)
  this.energy += length
}
Animal.prototype.play = function (length) {
  console.log(`${this.name} is playing.`)
  this.energy -= length
}
const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```

同样这也是有效的，`this` 对象是为我们创建的，因为我们用 new 关键字调用了构造函数。

如果你在调用函数时不使用 `new` ，`this` 对象就不会被创建，也不会被隐式返回。我们可以在下面的例子中看到这个问题。

```js
function Animal (name, energy) {
  this.name = name
  this.energy = energy
}
const leo = Animal('Leo', 7)
console.log(leo) // undefined
```

我们把这种模式称为 **`伪类实例化`** 。

如果 JavaScript 不是你的第一个编程语言，你可能会有点烦躁。

>"这只是重新创造了一个更蹩脚的类。"

对于那些不熟悉的人来说，类允许你为对象创建一个蓝图。之后只要你创建一个该类的实例，你就会得到一个包含蓝图中定义了的属性和方法的对象。

听起来很熟悉是吗？这是在上面 `Animal` 构造函数的基础上得到的。然而我们并没有用 `class` 关键字，只是用了一个普通的 JavaScript 函数来重新实现相同的功能。当然，这需要做一些额外的工作，以及一些底层的知识，但结果是一样的。

这里有一个好消息。JavaScript并不是一种死的语言。它一直在被TC-39委员会改进和补充。这意味着，即使最初版本的JavaScript不支持类，也没有理由不把它们添加到官方规范中。

事实上，这正是TC-39委员会所做的。2015年，EcmaScript（官方JavaScript规范）6发布，支持类和 `class` 关键字。让我们看看我们上面的 `Animal` 构造函数在新的类语法下会是什么样子。

```js
class Animal {
  constructor(name, energy) {
    this.name = name;
    this.energy = energy;
  }
  eat(amount) {
    console.log(`${this.name} is eating.`)
    this.energy += amount
  }
  sleep(length) {
    console.log(`${this.name} is sleeping.`)
    this.energy += length
  }
  play(length) {
    console.log(`${this.name} is playing.`)
    this.energy -= length
  }
}

const leo = new Animal('Leo', 7)
const snoop = new Animal('Snoop', 10)
```

非常简洁，不是吗？

所以如果这种新的方法可以创建类，为什么我们还要花这么长时间复习旧方法呢？因为新的方式（ `class` 关键字）是在现有方式上的「语法糖」，我们称为 `伪类模式 pseudoclassical pattern` 。为了完全理解 ES6 中类的语法，你首先要理解伪类模式。

目前为止，我们已经理解了 JavaScript 的原型的基本原理。

**Finished edit with 4/14 08:02**

