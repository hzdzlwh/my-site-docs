---
title: 闭包
---

## 作用域和闭包

作用域和闭包是JS中非常重要的概念，两者又是密不可分的，理解闭包必须从理解作用域开始


### 作用域是什么

作用域是一套规则，这套规则规定了变量存储在哪？程序需要时如何找到它们？

作用域有两种主要的工作模型，第一种最为普遍，被大多数编程语言所采用的词法作用域，JS正式采用的词法作用域。另一种叫做动态作用域，仍有一些编程语言在使用，比如Bash脚本

理解词法作用域需要知道编译原理，一段源代码在执行前会经历三个步骤，统称为编译
+ 分词/词法分析（Tokenizing/Lexing）

>这个过程会将由字符组成的字符串分解成有意义的代码块，这些代码块被称为词法单元，例如：var a = 2; 这行代码通常会被分解成下面词法单元：var、a、=、2、;。

+ 解析/语法分析（Parsing）

>这个过程是将词法单元转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树，这个树被称为 “抽象语法树”（Abstract Syntax Tree, AST）

+ 代码生成
>将AST转换为可执行代码的过程被称为代码生成。这个过程于语言、目标平台等信息相关。

简单来说，词法作用域就是定义在词法阶段的作用域。换句话说，词法作用域是由你在写代码时将变量和块作用域写在哪里来决定的，因此词法分析处理代码时会保持作用域不变

### 小试牛刀
考虑以下代码：

``` js
function foo(a) {
  var b = a * 2;

  function bar(c) {
    console.log(a, b, c);
  }

  bar(b * 3)
}
foo(2); // 2, 4, 12
```
在这个例子中有三个逐级嵌套的作用域。

1. 全局作用域，只有一个标识符：foo
2. foo创建的作用域，其中有三个标识符：a, bar, 和 b
3. bar所创建的作用域，其中包含一个标识符：c

执行console.log时，并查找a,b,c三个变量的引用，首先从最内部的作用域，也就是bar函数作用域开始查找，引擎无法在这里找到a, 因此会去上一级foo的作用域中继续查找，在这里找到了a，因此就使用这个a。

查找作用域的二个原则：
1. 作用域查找从运行时所处的最内部作用域开始，逐级向外
2. 作用域会在找到第一个匹配的标识符时停止

### 再理解词法作用域

词法作用域意味着作用域是由书写代码时函数声明的位置来决定的，编译的词法分析阶段基本上能够知道全部的标识符在哪里以及是如何声明的，从而能够预测在执行过程中如何对它进行查找

通过示例代码来说明：

```js
function foo() {
  console.log(a); // 2
}

funtion bar() {
  var a = 3;
  foo();
}

var a  = 2;
bar();
```
a 的查找过程是：
1. 现在foo中找，找不到
2. 到foo的上层作用域找，**注意 foo 的上层作用域是全局作用域**，而不是bar，因为foo函数定义在全局（此时已经确定了查找规则），bar中只是调用了foo函数而已

词法作用域让foo()中的a引用到了全局作用域中的a, 因此会打印2

### 千呼万唤始出来
这门语言中一个非常重要但又难于掌握，近乎神话的概念：闭包

下面我们来看一段代码，清晰地展示了闭包

```js
function foo() {
  var a = 2;

  funcion bar() {
    console.log(a)
  }

  return bar;
}

var baz = foo()

baz(); // 2, 这就是闭包的效果
```

函数bar()的词法作用域能够访问foo()的内部作用域，然而我们将bar函数本身当作一个值类型进行传递，在foo()执行后，其返回值（也就是内部的bar()函数）赋值给变量baz并调用baz()，bar()显然可以被执行，但是在这个例子中，它在自己定义的词法作用域 *以外* 的地方执行

在foo() 执行后，通常会期待foo()的整个内部作用域被销毁，因此我们知道引擎有垃圾回收机制用来释放不再使用的内存，由于看上去foo()的内容不会再被使用，所以很自然的考虑会对其进行回收。

而闭包的 “神奇” 之处正是可以阻止这件事情的发生，事实上内部作用域依然存在，因此没有被回收，谁在使用这个内部作用域呢？原来是bar()在使用

bar()依然保持着对改作用域的引用，而这个引用就叫闭包

### 犹抱琵琶半遮面

前面的代码为了解释如何使用闭包而人为的在结构上进行修饰，实际上闭包绝不仅仅那么的好玩，现在来搞懂下面的事实

```js
var a = 2;

(function IIFE() {
  console.log(a) // 2
})()
```

虽然这段代码可以正常执行，但严格来讲它并不是闭包，为什么？因为函数（IIFE()）并不是在它本身的词法作用域外执行

循环和闭包

```js
for(var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

正常情况下，我们对这段代码的行为预期是分别输出数字1-5，每秒一次，每次一个，但实际上，这段代码在运行时会以每秒一次输出五次6

这是为什么？

首先解释6从哪里来，循环的终止条件是i的值为6，因此输出显示的是循环结束时i的最终值

仔细想一下，延迟函数的回调会在循环结束时才执行，事实上当定时器运行时，所有的回调函数依然是在循环结束后才被执行，因此会每次输出一个6

这里引出一个更深入的问题，代码中到底有什么缺陷导致它的行为同语义所暗示的不一致呢？

缺陷是我们试图假设循环中的每个迭代在运行时都会给自己捕获一个i的副本，但是根据作用域的原理，实际情况是尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都 **被封闭在一个共享的全局作用域中**， 因此实际上只会有一个i

这样的话，当然所有函数共享一个i的引用

如何解决缺陷？

我们需要更多的闭包作用域，特别是在循环的过程中每个迭代都需要一个闭包作用域

IIFE会通过声明并立即执行一个函数来创建作用域

``` js
for (var i = 1; i<= 5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j)
    }, j * 1000)
  })(i)
}
```

在迭代内使用IIFE会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代都会包含一个具有正确值的变量供我们访问

### 换汤不换药

React hooks 中的闭包

useEffect 中的闭包

``` js
function WatchCount() {
  const [count, setCount] = useState(0);

  useEffect(function() {
    setInterval(function log() {
      console.log(`Count is: ${count}`);
    }, 2000);
  }, []);

  return (
    <div>
      {count}
      <button onClick={() => setCount(count + 1) }>
        Increase
      </button>
    </div>
  );
}
```

多次点击button后，控制台的打印Count is 0, 实际上count已经被加了多次

为什么会这样呢？

在第一次渲染的时候count被初始化为0，组件被挂载后，useEffect调用setInterval(log, 2000)每隔2秒打印一次，闭包log捕获的count变量是0。即使count被增加多次，log闭包函数仍旧使用count=0的初始化渲染的值。

修复上面的问题

``` js
useEffect(function() {
  const id = setInterval(function log() {
    console.log(`Count is: ${count}`);
  }, 2000);
  return function() {
    clearInterval(id);
  }
}, [count]);
```

正确的设置依赖，useEffect随着count变化更新闭包

useState 中的闭包

``` js
function DelayedCount() {
  const [count, setCount] = useState(0);

  function handleClickAsync() {
    setTimeout(function delay() {
      setCount(count + 1);
    }, 1000);
  }

  return (
    <div>
      {count}
      <button onClick={handleClickAsync}>Increase async</button>
    </div>
  );
}
```

快速点击button两次，count的值仍旧是1，而不是2

每次点击后延迟1秒调用delay函数，delay函数捕获的count始终是0，两个delay函数都是闭包，更新相同的value：setCount(count + 1) = setCount(0 + 1) = setCount(1)，两次点击后的闭包delay捕获的仍旧是更新前的count：0

为了修复这个问题，我们用函数的方式setCount(count => count + 1)更新count， 回调函数返回一个新的state依据之前的state。


Vue源码中的闭包

``` js
function defineReactive(obj, key, value) {
  return Object.defineProperty(obj, key, {
    get() {
        return value;
    },
    set(newVal) {
        value = newVal;
    }
  })
}
```

value是函数中的一个形参，属于私有变量，但是为什么在外部可以使用value, 或给vulue赋值呢？

根据闭包的特性，内层函数可以引用外层函数的变量，当存在这种引用关系的时候，变量不会垃圾回收机制回收，当你获取这个变量的时候实际上调用的是内层的get函数，当你设置这个变量时实际上调用的是set函数，都是对value形参的操作


Redux 源码中的闭包

``` js
function applyMiddleware(...middlewares) {
  return (createStore) => (reducer) => {
    const store = createStore(reducer)
    let dispatch = store.dispatch

    const midApi = {
      getState: store.getState,
      dispatch: (action) => dispatch(action) // 闭包的使用 捕获修改后dispatch
    }
    const chain = middlewares.map(middleware => middleware(midApi))

    // 修改后的dispatch
    dispatch = compose(...chain)(store.dispatch)

    return { ...store, dispatch }
 }
}
```

上面 midApi 中的 dispatch 属性为什么没有写成下面这样呢？

``` js
const midApi = {
    getState: store.getState,
    dispatch: dispatch
  }
```

首先要明白 Redux 使用了中间件后目的就是重写 dispatch 方法，要保证传给中间函数（middleware）的 dispatch 都是修改后的 dispath，否则有些中间件可能就不生效，为了保证每次传给中间件函数（middleware）中的 dispatch 都是修改后的 dispatch 这个地方必须写成一个闭包的形式，而不能直接用 store.dispatch。


### reference

[1] 《你不知道的Javascript系列》
