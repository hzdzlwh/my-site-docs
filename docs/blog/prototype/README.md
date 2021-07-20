---
title: JS原型，原型链
---

# Javascript原型，原型链，继承

原型和原型链是JavaScript中非常重要的知识点，理解原型首先要从JavaScript的对象入手，本文将逐步深入的讲解原型和原型链。

## 创建对象

### 工厂模式

工厂模式是软件工程中一种一种广泛的设计模式，这种模式抽象了创建具体对象的过程，开发人员就发明了一种函数，用函数来封装创建对象

``` js
function createPerson(name, age, job) {
  var o = new Object();
  o.name = name;
  o.age = age;
  o.job = job;
  o.sayName = function() {
    alert(this.name)
  }
  return o
}
var person1 = createPerson("Nicholas", 20, "software engineer")
var person2 = createPerson("Greg", 21, "Doctor")
```

工厂模式虽然解决了创建多个相似对象的问题，但却没有解决对象识别的问题（即怎么知道一个对象的类型），随着JavaScript的发展，又一个新的模式出现

### 构造函数模式

``` js
function Person(name, age, job) {
  this.name = name
  this.job = job
  this.age = age
  this.sayName = function() {
    alert(this.name)
  }
}

var person1 = new Person("Nicholas", 20, "software engineer")
var person2 = new Person(Greg", 21, "Doctor)
```

上面的例子中除了用Person()函数取代了createPerson()函数，我们还注意到Person()中的代码除了与createPerson()相同的部分外，还存在如下不同之处：

+ 没有显式的创建对象
+ 直接将属性和方法赋给了this
+ 没有return语句

创建自定义的构造函数意味着将来可以将它的实例标识为一种特定的类型，这也正是构造函数模式胜过工程模式的地方。

构造函数虽然好用，但也并非没有缺点，使用构造函数的主要问题就是，每个方法都要在实例上重新创建一遍，不同实例上的同名函数是不相等的，下面的代码可以证明这一点。

``` js
alert(person1.sayName == person2.sayName); // false
```
 
### 原型模式

我们创建的每个函数都有一个prototype（原型）属性，这个属性是一个指针，指向一个对象，而这个对象的用途是包含可以由特定类型的所有实例共享的属性和方法，
如果按照字面意思来理解，那么prototype就是通过调用构造函数而创建的那个对象实例的原型对象，使用原型对象的好处是可以让所有对象实例共享它所包含的属性和方法，
换句话说，不必在构造函数中定义对象实例的信息，而是可以将这些信息直接添加到原型对象中，如下面例子

``` js
function Person() {

}

Person.prototype.name = "Nicholas";
Person.prototype.age = 20;
person.prototype.job = "software engineer";
Person.prototype.sayName = function() {
  alert(this.name)
}

var person1 = new Person()
var person2 = new Person()
alert(person1.name == person2.name) // true
```

通过调用构造函数来创建新对象，而且新对象还会具有相同的属性和方法，但于构造函数模式不同的是，新对象的这些属性和方法是由所有实例共享的，换句话说，
person1，person2访问的都是同一组属性和同一个sayName函数，要理解原型模式的工作原理必须先理解ECMAScript中原型对象的性质。

**下面重点来了**

理解原型对象

无论什么时候，只要创建一个新函数，就会根据一组特定规则为该函数创建一个Prototype属性，这个属性指向函数的原型对象，在默认情况下，所有原型对象都会自动获得一个constructor属性，指向构造函数

**原型对象的问题**

**原型模式也不是没有缺点，首先，它省略了为构造函数传递初始化参数这一环节，结果所有实例在默认情况下都将取得相同的属性值，虽然这会在某种程度上带来一些不方便，但还不是原型最大的问题，原型模式最大的问题所在是由其共享的本质所导致的。**

原型中所有属性是被很多实例共享的，这种共享对函数来说非常合适，对于哪些包含基本值的属性倒也说得过去，通过在实例上添加一个同名属性，可以隐藏原型中的对应属性。然而，对于包含引用类型值的属性来说，问题就比较突出了，来看下面的例子。


``` js
function Person() {

}

Person.prototype = {
  constructor: Person,
  name: "Nicholas",
  age: 20,
  job: "software engineer",
  friends: ["Shelby", "Court"],
  sayName: function() { alert(this.name) }
}

var person1 = new Person()
var person2 = new Person()

person1.friends.push("Van");

alert(person1.friends) // "Shelby, Court Van"
alert(person2.friends) // "Shelby, Court Van"
alert(person1.friends === preson2.friends) // true
```

Person.prototype对象有一个名为frends的属性，该属性是一个数组，然后创建了两个实例，接着，修改了person1.friends引用的数组，向数组中添加了一个字符串，由于friends存在于Person.prototype而非person1中，所以刚刚的修改也会通过person2.friends反映出来，假如我门的初衷就是所有实例共享一个数组，那么对这个结果没什么可说的。可是实例一般都是要有自己的全部属性，这也正是很少单独使用原型模式的原因所在。

组合使用构造函数模式和原型模式

创建自定义类型的最常用方式，就是组合使用构造函数模式于原型模式，构造函数模式用于定义实例属性，而原型模式用于定义方法和共享的属性，结果，每个实例都会有自己的一份实例属性的副本，但同时又共享着对方法的引用，最大限度地节省了内存，另外，这种混成模式还支持向构造函数传递参数，可谓是集两种模式之长。下面从写前面的例子。

``` js
function Person(name, age, job) {
  this.name = name
  this.age = age
  this.job = job
  this.friends = ["Shelby", "Court"]
}

Person.prototype = {
  constructor: Person,
  sayName: function() {
    alert(this.name)
  }
}

var person1 = new Person("Nicholas", 20, "software engineer")
var person2 = new Person("Greg", 21, "Doctor");
person1.friends.push("Van")

alert(person1.friends) // "Shelby, Court, Van"
alert(person2.friends) // "Shelby, Court"
alert(person1.friends === person2.friends) // false
alert(person1.sayName === person2.sayName) // true
```

在这个例子中，实例属性都是在构造函数中定义的，而所有实例共享的属性constructor和方法sayName则是在原型中定义的，而修改person1.friends并不会影响到person2.friends，因为他们分别引用了不同的数组。

上半部分通过创建对象一步步的分析问题，解决问题，引出了原型的概念，下半部分将通过继承，引出原型链

### 继承

ECMAScript中主要是靠原型链实现继承的

原型链

原型链的概念：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针，那么，假如我们让原型对象等于另一个类型的实例，结果会怎样呢？显然，此时的原型对象将包含一个指向另一个原型的指针，响应地，另一个原型中也包含着一个指向另一个构造函数的指针，假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例和原型的链条，这就是所谓的原型链的基本概念。

组合继承

组合继承也叫经典继承，指的是将原型链和借用构造函数的技术组合到一块，从而发挥二者之长的一种继承模式，其背后的思路是使用原型链实现对原型属性和方法的继承，而通过借用构造函数来实现对实例属性的继承，这样，既通过在原型上定义方法实现了函数的复用，又能保证每个实例都有它自己的属性。

``` js
function SuperType(name) {
  this.name = name
  this.colors = ["red", "blue", "green"]
}

SuperType.prototype.sayName = function() {
  alert(this.name)
}

function SubType(name, age) {
  SuperType.call(this, name)
  this.age = age
}
SubType.prototype = new SuperType()
SubType.prototype.constructor = SubType
SubType.prototype.sayAge = function() {
  alert(this.age)
}

var instance1 = new SuperType("Nicholas", 20)
instance1.colors.push("black")
alert(instance1.colors) // "red", "blue", "green", "black"
instance1.sayName() // "Nicholas"
instance1.sayAge() // 20

var instance2 = new SuperType("Greg", 21)
alert(instance2.colors) // "red", "blue", "green"
instance2.sayName() // "Greg"
instance2.sayAge() // 21
```
