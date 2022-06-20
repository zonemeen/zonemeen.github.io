---
title: JavaScript系列之作用域和作用域链
date: 2021-05-18
permalink: /blog/2021/05/18/javascript-of-scopes-and-scopeChains/
---

在上一篇[《JavaScript 系列之变量对象》](https://zhuanlan.zhihu.com/p/69142071)中，我们已经知道一个执行上下文的数据（函数的形参、函数及变量声明）作为属性储存在变量对象中。

此外，我们也知道每次进入上下文时都会创建变量对象并填充初始值，并且值会在代码执行阶段进行更新，现在就对执行上下文做更深一步的了解。

先来回顾一下关于执行上下文的三个阶段生命周期：

![](/images/scopechain/1.jpg)

本章将专门介绍与执行上下文创建阶段直接相关的另一个细节——**作用域链**。

### 作用域(Scope)

在介绍作用域链前，有必要先来了解一下被称为作用域(Scope)的特性，那什么是作用域呢？

作用域就是在运行时代码中不同部分中函数和变量的可访问性。可能这句话并不太好理解，我们先来看段代码：

```javascript
function fn() {
  var inVariable = "inner variable";
  console.log(inVariable); // inner variable
}

fn();
console.log(inVariable); // Uncaught ReferenceError: inVariable is not defined
```

从上面的代码中我们可以很直观地体会作用域的概念，变量`inVariable`在全局作用域没有声明，所以在全局作用域下直接取值会报错。所以我们可以这样理解：**作用域就像一个地头蛇，我的地盘我做主，让属于自己域内的变量不会轻易外泄出去**。也就是说**作用域最大的用处就是隔离变量，不同作用域下同名变量不会有冲突**。这几句话总比前面那句好理解多了吧。

关于 JavaScript 中的作用域类型，**ES6 之前 JavaScript 并没有块级作用域，只有全局作用域和函数作用域**。ES6 的到来，为我们提供了‘块级作用域’,可通过新增命令 let 和 const 来体现：

- 全局作用域  —  变量可以随处访问
- 函数作用域—  变量可以在定义它们的函数的边界内访问
- 块级作用域—变量可以在定义它们的块中访问，块由 { 和 } 分隔

### 全局作用域和函数作用域

```javascript
const global = "global scoped";

function fn() {
  const global = "function scoped";
  console.log(global); // function scoped
}

fn();
console.log(global); // global scoped
```

从上面例子可以看出全局作用域和函数作用域的作用范围，即使全局变量在函数内部分配了不同的值，它也只保留在同一函数的边界内，互相并不影响，我们也不会因使用相同的变量名而出错。再来看个例子加深理解：

```javascript
const global = "global scoped";
const anotherGlobal = "also global scoped";

function fn() {
  const global = "function scoped";
  console.log(global); // function scoped
  const scoped = "also function scoped";

  function inner() {
    console.log(scoped); // also function scoped
    console.log(anotherGlobal); // also global scoped
  }

  inner();
}

console.log(global); // global scoped
console.log(anotherGlobal); // also global scoped

fn();
inner(); // Uncaught ReferenceError: inner is not defined
```

在这里我们可以看到 `inner()` 函数可以访问在其父函数中声明的变量—`fn()`。每当我们需要函数内部的变量时，引擎将首先在当前函数作用域内查找它。如果它没有当前函数作用域内找到它，它将继续上升，向上一级查找，直到它找到全局作用域内的变量，如果找不到变量，我们将得到一个 ReferenceError。格外注意**函数内层作用域可以访问外层作用域的变量，反之则不行**。

除了上面所讲的最外层函数外面定义的变量拥有全局作用域，全局作用域还有一种特殊的出现场合：就是**所有末声明直接赋值的变量将自动声明为拥有全局作用域的变量**：

```javascript
function fn() {
  variable = "undeclared variable";
  var inVariable = "inner variable";
}

fn();
console.log(variable); // undeclared variable
console.log(inVariable); // Uncaught ReferenceError: inVariable is not defined
```

全局作用域有个弊端：如果我们写了很多行 JavaScript 代码，变量定义都没有用函数包括，那么它们就全部都在全局作用域中，这样就会污染全局命名空间，容易引起命名冲突。同时意外的全局变量还会引起内存泄漏，所以在编程时，尽量避免全局变量的使用，以便后期更快地调试。

还有值得注意的是：**块语句（大括号“｛｝”中间的语句），如 `if` 和 `switch` 条件语句或 `for` 和 `while` 循环语句，不像函数，它们不会创建一个新的作用域**。在块语句中定义的变量将保留在它们已经存在的作用域中。比如：

```javascript
if (true) {
  // 'if' 条件语句块不会创建一个新的作用域
  var name = "miqilin"; // name 依然在全局作用域中
}

console.log(name); // miqilin
```

JS 的初学者经常需要花点时间才能习惯变量提升，而如果不理解这种特有行为，就可能导致 bug 出现 。正因为如此， ES6 引入了块级作用域，让变量的生命周期更加可控。

### 块级作用域

在 ES6 中，我们得到了两个新的变量声明关键字 - `let`和`const`。它们和`var`之间的主要区别在于，使用 ES6 关键字声明的变量是块作用域，这意味着它们仅在它们定义的代码块中可用。块级作用域在如下情况被创建：

1. 在一个函数内部
2. 在一个代码块（由一对花括号包裹）内部

`let` 声明的语法与 `var` 的语法一致。你基本上可以用 `let` 来代替 `var` 进行变量声明，但会将变量的作用域限制在当前代码块中。块级作用域有以下几个特点：

- **声明变量不会提升到代码块顶部**

`let`/`const`创建的变量不会像使用`var`声明的变量那样被提升到顶部，因此你需要手动将 `let`/`const` 声明放置到顶部，以便让变量在整个代码块内部可用。比如：

```javascript
cosole.log(name); // Uncaught ReferenceError: cosole is not defined
const name = "miqilin";
```

所以确保代码没有引用错误的一种方法是确保只使用`let`和`const`进行变量声明。

- **禁止重复声明**

如果一个标识符已经在代码块内部被定义，那么在此代码块内使用同一个标识符再进行 `let` 声明就会抛出错误。比如：

```javascript
var count = 10;
let count = 20; // Uncaught SyntaxError: Identifier 'count' has already been declared
```

上面例子中`count` 变量被前后声明了两次：第一次使用 `var` ，另一次使用 `let` 。因为 `let` 不能在同一作用域内重复声明一个已有标识符，此处的 `let` 声明就会抛出错误。但如果在嵌套的作用域内使用 `let` 声明一个同名的新变量，则不会抛出错误：

```javascript
var count = 10;
// 不会抛出错误
if (condition) {
  let count = 20;
  // 其他代码
}
```

- **循环中的绑定块作用域的妙用**

开发者可能最希望实现`for`循环的块级作用域了，因为可以把声明的计数器变量限制在循环内，比如：

```javascript
for (let i = 0; i < 10; i++) {
  // ...
}
console.log(i);
// ReferenceError: i is not defined
```

上面代码中，因为用`let`声明计数器`i`，只在`for`循环体内有效，所以在循环体外引用就会报错。

```javascript
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上面代码中，变量`i`是`var`命令声明的，在全局范围内都有效，所以全局只有一个变量`i`。每一次循环，变量`i`的值都会发生改变，而循环内被赋给数组`a`的函数内部的`console.log(i)`，里面的`i`指向的就是全局的`i`。也就是说，所有数组`a`的成员里面的`i`，指向的都是同一个`i`，导致运行时输出的是最后一轮的`i`的值，也就是 10。

如果换使用 let，声明的变量仅在块级作用域内有效，最后输出的是 6。

```javascript
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

上面代码中，变量`i`是`let`声明的，当前的`i`只在本轮循环有效，所以每一次循环的`i`其实都是一个新的变量，所以最后输出的是 6。你可能会问，如果每一轮循环的变量 i 都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 i 时，就在上一轮循环的基础上进行计算。

另外，`for`循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。

```javascript
for (let i = 0; i < 5; i++) {
  let i = "abc";
  console.log(i);
}
// abc
// abc
// abc
// abc
// abc
```

上面代码正确运行，输出了 5 次 abc。这表明函数内部的变量 i 与循环变量 i 不在同一个作用域，有各自单独的作用域。

### 作用域链(Scope Chain)

上面用一大篇幅来讲解作用域，其实在里面就有涉及到作用域链的知识了。简单的来说，当查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级(词法层面上的父级)执行上下文的变量对象中查找，一直找到全局上下文的变量对象，也就是全局对象。这样由多个执行上下文的变量对象构成的链表就叫做**作用域链**。看下面一个例子：

```javascript
function a() {
  function b() {
    console.log(myVar);
  }

  var myVar = 2;
  b();
}

var myVar = 1;
a(); // 2
b(); // Uncaught ReferenceError: b is not defined
```

最后加以执行`a()`和`b()`，这时候我们会发现两件事：

1.执行`a()`会得到 2 的结果：之所以会有这样的结果，是因为当我们执行`function a`里面的`function b`时，因为在`function b`里面它找不到`myVar`这个变量，因此它开始往它的外层去搜寻，而这时候它的父级作用域是`function a`，在`function a`里面它便找到了`myVar = 2`，因此它就不再往外部环境 (`myVar = 1`)去找了，直接返回了 2 这样的结果。

2.`b()`会得到`b is not defined`的结果：之所以`b`会是`not defined`（记得是`not defined`不是`undefined`哦！)，是因为这时候在最外层的全局上下文（`global execution context`）中，找不到`function b`。

而从`b() --> a() --> global execution context`这样的链，就称为**作用域链（Scope Chain）**：

![](/images/scopechain/2.jpg)

如果我们把`function a`里面对于`myVar`的声明拿掉的话，它才会继续往外层搜寻`myVar`，直到找到全局作用域中的声明`myVar = 1`，这时候才会返回 1 的结果。

```javascript
function a() {
  function b() {
    console.log(myVar);
  }

  //var myVar = 2;
  b();
}

var myVar = 1;
a(); // 1
```

如果我们更进一步的把全局作用域中，对于`myVar`的声明也拿掉，那么现在在全局作用域中也找不到`myVar`这个变量了，也就是说，在这整个作用域链中都找不到`myVar`，因此可想而知，最后的结果是`not defined`。

```javascript
function a() {
  function b() {
    console.log(myVar);
  }

  //var myVar = 2;
  b();
}

//var myVar = 1;
a(); // Uncaught ReferenceError: myVar is not defined
```

**如果觉得文章对你有些许帮助，欢迎在[我的 GitHub 博客](https://github.com/miqilin21/miqilin21.github.io)点赞和关注，感激不尽！**
