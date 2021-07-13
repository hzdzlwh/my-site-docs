---
title: this
---

## this全面解析

this关键字是javascript中最复杂的机制之一，它是一个很特别的关键字，被自动定义在所有函数的作用域。

### 为什么要用this

``` js
function identify() {
  return this.name.toUpperCase()
}

function speak() {
  var greeting = "Hello, I am " + identify.call(this)
  console.log(greeting)
}

var me = {
  name: "Kyle"
}

var you = {
  name: "Reader"
}

identify.call(me) // KYLE
identify.call(you) // READER

speak.call(me) // Hello, I am KELE
speak.call(you) // Hello, I am READER

```

这段代码可以在不同的上下文对象（me和you）中重复使用函数 identify() 和 speak()，不用针对每个对象编写不同版本的函数

### this的错误认知

1. 指向自身
人们很容易把this理解成指向函数自身，那么为什么需要从函数内部引用自身呢？常见的原因是递归。

通过下面的代码我们分析一下，this并不像我们所想的那样指向函数本身

``` js
function foo(num) {
  console.log("foo: " + num)
  // 记录foo被调用次数
  this.count++
}

foo.count = 0

for (var i = 0; i < 10; i++) {
  if (i > 5) {
    foo(i)
  }
}

// foo: 6
// foo: 7
// foo: 8
// foo: 9

// foo被调用了多少次？
console.log(foo.count) // 0
```

foo()函数实际上被调用了4次，但是foo.count仍然是0

执行foo.count = 0时，的确向函数对象foo添加了一个属性count，但是函数内部代码this.count中的this并不是指向那个函数对象。

负责人的开发一定会问，那我增加的count是那个count？如果深入探究的话，就会发现这段代码在无意中创建了一个全局变量count，它的值为NaN。

2. 指向函数作用域

this在任何情况下都不指向函数的词法作用域，在JavaScript内部，作用域确实和对象类似，可见的标识符都是它的属性，但是作用域对象无法通过JavaScript代码访问，它存在于JavaScript引擎内部。

思考一下下面的代码，它试图跨越边界，使用this来隐式引用函数的词法作用域：

``` js
function foo() {
  var a = 2
  this.bar()
}

function bar() {
  console.log(this.a)
}

foo() // ReferenceError: a is not defined
```

这段代码中的错误不止一个，首先这段代码试图通过this.bar(）来引用bar()函数。这是绝对不可能成功的。调用bar()最自然的方式是省略前面的this，直接使用词法引用标识符。

此外，这段代码中试图使用this联通foo()和bar()的词法作用域，从而让bar() 可以访问foo()作用域的变量a，这是不可能实现的，你不能使用this来引用一个词法作用域内部的东西

### this到底是什么

排除了一些错误的理解后，我们来看看this到底是一种什么机制

之前我们说过**this是在运行时进行绑定的，并不是在编写是绑定**，它的上下文取决于函数调用时的各种条件，this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

### 绑定规则
我们来看看在函数的执行过程中调用位置如何决定 this 的绑定对象

1. 默认绑定

``` js
function foo() {
  console.log(this.a)
}
var a = 2
foo() // 2
```

当调用foo()时，this.a被解析成了一个全局变量a，函数调用时应用了默认绑定，因此this指向全局对象。

2. 隐式绑定

调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含。

``` js
function foo() {
  console.log(this.a)
}
var obj = {
  a: 2, 
  foo: foo
}
obj.foo() // 2
```

首先注意foo()的声明方式，及其之后是如何被当作引用属性添加到obj中的，
但是无论是直接在obj中定义还是先定义再添加为引用属性，这个函数严格来说都不属于obj对象。

然而，调用位置会使用 obj 上下文来引用函数，因此你可以说函数被调用时 obj 对象“拥 有”或者“包含”它。

3. 显示绑定

使用call(...)或apply(...)方法

``` js
function foo() { 
  console.log( this.a );
}
var obj = { a:2 };
foo.call( obj ); // 2
```

通过 foo.call(..)，我们可以在调用 foo 时强制把它的 this 绑定到 obj 上。

4. new 绑定

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作
1. 创建（或者说构造）一个全新的对象。
2. 这个新对象会被执行 [[ 原型 ]] 连接。
3. 这个新对象会绑定到函数调用的 this。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

``` js
function foo(a) {
  this.a = a
}
var bar = new foo(2)
console.log(bar.a) // 2
```

使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上

### 优先级

现在我们已经了解了函数调用中的this绑定四条规则，你需要做的就是找到函数的调用位置并判断应当应用哪条规则， 可以按照下面的顺序来判断：

1. 函数是否在 new 中调用？如果是的话this绑定的是新创建的对象
2. 函数是否通过call，apply显示绑定，如果是，this绑定的是指定的对象
3. 函数是否在某个上下文对象中调用（隐式绑定）? 如果是的话，this绑定的是那个上下文对象
4. 如果都不是的话，使用默认绑定

### this词法

上面介绍的四条规则已经可以包含所有正常函数，但是ES6中介绍了一种无法使用这些规则的函数类型：箭头函数

我们来看看箭头函数的词法作用域：
``` js
function foo() {
  return (a) => {
    console.log(this.a)
  }
}

var obj1 = { a: 2 }
var obj1 = { a: 3 }

var bar = foo.call(obj1)
bar.call(obj2) // 2, 不是3
```

foo() 内部创建的箭头函数会捕获调用foo()的this，由于foo()的this绑定到obj1，bar（引用箭头函数）的this也会绑定到obj1，箭头函数的绑定无法被修改。

### 总结

如果要判断一个运行中函数的this绑定，就需要找到这个函数的直接调用位置，找到之后就可以顺序应用下面四条规则来判断。

1. 由new调用？绑定到新创建的对象
2. 由call，apply或bind调用？绑定到指定对象
3. 由上下文对象调用？绑定到那个上下文对象
4. 默认绑定到全局

ES6 中的箭头函数并不会使用这四条绑定规则，而是根据当前词法作用域来决定this，具体来说箭头函数会继承外层函数调用的this绑定。