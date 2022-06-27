---
title: JavaScript系列之变量对象
date: 2021-05-12
permalink: /blog/2021/05/12/javascript-of-variable-object/
---

[[toc]]

JavaScript 编程的时候总规避不了声明变量和函数，但是解释器是如何并且在什么地方去查找这些变量和函数呢？接下来，再延续上一篇[《JavaScript 系列之执行上下文和执行栈》](https://luozongmin.com/2019/06/13/JavaScript%E7%B3%BB%E5%88%97%E4%B9%8B%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E5%92%8C%E6%89%A7%E8%A1%8C%E6%A0%88/)，通过对变量对象(Variable Object)的介绍对执行上下文有一个更深一步的了解。

上一篇文章也提到了，一个执行上下文的生命周期可以分为三个阶段：

![](/images/scopechain/1.jpg)

详细了解执行上下文对于初学者来说极为重要，因为其中涉及到了变量对象，作用域链，this 等很多 JavaScript 初学者没完全搞懂，且极为重要的概念，它关系到我们能不能真正理解 JavaScript，真正理解也能更为轻松地胜任后续工作，在后面的文章中我们会一一详细介绍，这里我们先重点了解一下**变量对象**。

### 变量对象

变量对象（Variable Object）是一个与执行上下文相关的数据作用域，存储了在上下文中定义的**变量**和**函数声明**，先来看一段代码示例：

```javascript
function foo() {
  var a = 10;
  function b() {}
  (function c() {});
  console.log(a); // 10
  console.log(b); // function b(){}
  console.log(c); // Uncaught ReferenceError: c is not defined
}

foo();
```

在上面的例子中，`foo（）`函数的变量对象包含**变量`a`**和**函数`b（）`的声明**。这里要注意的一点是，函数表达式并不像函数声明一样包含在变量对象中，在示例中所看到的那样，访问 c（）函数会导致引用错误。因为变量对象是抽象的和特殊的，它不能在代码中访问，但会由 JavaScript 引擎处理。

上面利用的是函数上下文下的变量对象来说明变量对象储存了什么，但变量对象还存在于全局上下文中，接下来就分别来聊聊全局上下文中和函数上下文中的变量对象吧。

### 全局上下文

以浏览器中为例，全局对象为`window`。 全局上下文有一个特殊的地方，它的变量对象，就是`window`全局对象，而这个特殊，在`this`指向上也同样适用，`this`也是指向`window`。

```javascript
// 以浏览器中为例，全局对象为window
// 全局上下文创建阶段
// VO 为变量对象（Variable Object）的缩写
windowEC = {
  VO: Window,
  scopeChain: {},
  this: Window,
};
```

除此之外，全局上下文的生命周期，与程序的生命周期一致，只要程序运行不结束，比如关掉浏览器窗口，全局上下文就会一直存在。其他所有的上下文环境，都能直接访问全局上下文的属性。

### 函数上下文

在上面已经提到了，变量对象存储了执行上下文中的变量和函数声明，但在函数上下文中，还多了一个`arguments(函数参数列表)`, 一个伪数组对象。

这时变量对象的**创建阶段**会包括：

1. **创建`arguments`对象**。检查当前上下文中的参数，建立该对象下的属性与属性值。
2. **检查当前上下文的函数声明，也就是使用 function 关键字声明的函数**。在变量对象中以函数名建立一个属性，属性值为指向该函数所在内存地址的引用。如果变量对象已经存在相同名称的属性，则完全替换这个属性。
3. **检查当前上下文中的变量声明**（`var` 声明的变量），默认为 `undefined`；如果变量名称跟已经声明的形式参数或函数相同，为了防止同名的函数被修改为`undefined`，则会直接跳过变量声明，原属性值不会被修改。

![](/images/variableobject/1.jpg)

对于第 3 点中的“跳过”一词想必大家会有一丝疑问？底下例子中既然按照上面的规则，变量声明的`foo`遇到函数声明的`foo`会跳过，可是为什么最后`foo`的输出结果仍然是被覆盖了？

```javascript
function foo() {
  console.log("I am function foo");
}
var foo = 10;

console.log(foo); // 10
```

理由其实很简单，因为上面的三条规则仅仅适用于变量对象的**创建过程**，也就是执行上下文的创建过程。而`foo = 10`是在执行上下文的**执行过程**中运行的，输出结果自然会是 10。对比下例：

```javascript
console.log(foo); // ƒ foo() { console.log('I am function foo') }
function foo() {
  console.log("I am function foo");
}

var foo = 10;
console.log(foo); // 10
```

为啥又是不一样的结果呢？其实它的执行顺序为：

```javascript
// 首先将所有函数声明放入变量对象中，函数声明变量提升
function foo() {
  console.log("I am function foo");
}

// 其次将所有变量声明放入变量对象中，但是因为foo已经存在同名函数，因此此时会跳过变量声明默认undefined的赋值
// var foo = undefined;

// 然后开始执行阶段代码的执行
console.log(foo); // ƒ foo() { console.log('I am function foo') }

// 在执行上下文的执行过程中运行
foo = 10;
console.log(foo); // 10
```

根据上面的规则，理解变量提升就变得十分简单了，我们也可以看出，**`function`声明会比`var`声明优先级更高一点**。为了帮助大家更好的理解变量对象，我们再结合一个简单的例子来进行探讨。

```javascript
function test() {
  console.log(a);
  console.log(foo());

  var a = 1;
  function foo() {
    return 2;
  }
}

test();

/* 结果为：
undefined
2
*/
```

根据上述的规则，理解变量提升后可以将执行顺序理解为：

```javascript
function test() {
  function foo() {
    return 2;
  }
  var a;
  console.log(a);
  console.log(foo());
  a = 1;
}

test();
```

这样是不是一目了然了呢？

当然还需要注意的是，函数未进入执行阶段之前，变量对象中的属性都不能访问！但是进入执行阶段之后，变量对象（VO）转变为了活动对象（AO），然后开始进行执行阶段的操作。

### 执行阶段

当前进入执行阶段，变量对象（VO）激活成活动对象（AO），里面的属性都能被访问了，函数会顺序执行代码，改变变量对象的属性值，此阶段的执行上下文代码会分成两个阶段进行处理：

1. 进入执行上下文
2. 执行代码

#### 进入执行上下文

当进入执行上下文时，这时候还没有执行代码。让我们看一个例子：

```javascript
function foo(a, b) {
  var c = 10;
  function d() {}
  var e = function _e() {};
  (function x() {});
}

foo(10);
```

当进入带有参数 10 的`foo`函数上下文时，AO 表现为如下：

```javascript
AO = {
    arguments: {
        0: 10,
        1: undefined,
        length: 1
    }
    a: 10,
    b: undefined,
    c: undefined,
    d: <function reference to d>,
    e: undefined,
}
```

`x` 是函数表达式，所以不在变量对象当中，`e` 变量引用的值也是函数表达式，所以变量 `e` 本身是声明，所以在变量对象当中。

#### 执行代码

这个阶段会按顺序执行代码，修改变量对象的属性值，紧接上面的例子，执行完成后 AO 如下：

```javascript
AO = {
    arguments: {
        0: 10,
        1: undefined,
        length: 1
    }
    a: 10,
    b: undefined,
    c: 10,
    d: <reference to function declaration d>,
    e: <reference to Function expression to _e>,
}
```

到这里变量对象的创建过程就介绍完了，让我们简短地总结一下：

1. 全局上下文的变量对象初始化是全局对象
2. 函数上下文的变量对象初始化只包括 `Arguments` 对象
3. 在进入执行上下文时会依次给变量对象**添加形参**、**函数声明**、**变量声明**等初始的属性值
4. 函数未进入执行阶段之前，变量对象中的属性都不能访问
5. 在执行代码阶段，会再次修改变量对象的属性值，并赋予该有的属性值

**如果觉得文章对你有些许帮助，欢迎在[我的 GitHub 博客](https://github.com/miqilin21/miqilin21.github.io)点赞和关注，感激不尽！**
