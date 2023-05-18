---
title: 深入浅出 function 与 class
category:
  - 笔记
tags:
  - 代码
  - JavaScript
date: 2022-11-01 13:10:53
---
在 JavaScript 中，生成实例对象有着 `new Function` 与 `new Class` 两种写法。

尽管我们知道，传统 function 构造函数是 ES5 中的写法，而 class 则是在 ES6 新提出的关键字，那么这两种写法到底有着什么样的区别？

<!-- more -->

## class 的由来

在 ES5 中，与传统的面向对象语言（比如 C++ 和 Java）相比，JavaScript 在生成实例对象的写法上有着很大的不同，例如下面的例子。

```javascript
function Phone(brand) {
  this.brand = brand;
}

Phone.prototype.call = function () {
  console.log('hello world from', this.brand);
};

var xiaomi = new Phone('xiaomi');
xiaomi.call(); // => hello world from xiaomi
```

各类变量中，只有 function 函数才有 prototype 属性，原型属性上又有着 constructor() 方法，而这很容易让新学习这门语言的程序员感到困惑。

```javascript
function Phone() { }

console.log(Phone.prototype); // => {constructor: ƒ}
```

ES6 提供了更接近传统语言的写法，引入了 class 这个概念，作为对象的模板。通过 `class` 关键字，可以定义类。

基本上，ES6 的 class **可以看作** 只是一个 `语法糖`，它的绝大部分功能，ES5 都可以做到，新的 class 写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。上面的代码用 ES6 的 class 改写，就是下面这样。

```javascript
class Phone {
  constructor(brand) {
    this.brand = brand;
  }

  call() {
    console.log(`hello world from ${this.brand}`);
  }
}

const huawei = new Phone('huawei');
huawei.call(); // => hello world from huawei
```

这种新的 class 写法，本质上与文章开头的 ES5 的构造函数 Phone 是一致的，ES6 的类，**可以看作** 构造函数的另一种写法。

```javascript
class Phone { }

console.log(typeof Phone); // => "function"
console.log(Phone === Phone.prototype.constructor); // => true
```

上面代码表明，类的数据类型就是函数，类本身就指向构造函数。使用的时候，也是直接对类使用 `new` 关键字，跟构造函数的用法完全一致。

## constructor() 方法

constructor() 方法是类的默认方法，通过 new 关键字生成对象实例时，自动调用该方法。一个类必须有 constructor() 方法，如果没有显式定义，一个空的 constructor() 方法会被默认添加。

```javascript
class Phone { }

// 等价于
class Phone {
  constructor() { }
}
```

上面代码中，定义了一个空的类 Phone，JavaScript 引擎会自动为它添加一个空的 constructor() 方法，该方法默认返回实例对象（this）

## 语法糖，但没完全语法糖

不管是百度也好，各大培训机构也好，人们都常说 class 是一个语法糖，因为完全可以在不使用 class 的情况下用 function 实现同样的效果。

不过，这两者间 **并不能完全划等号**，它们也有着些许不同。通过 class 创建的函数具有特殊的内部属性标记 `[[IsClassConstructor]]: true`。编程语言会在许多地方检查该属性。

例如，与普通函数不同，必须使用 new 来调用它。这是它与普通构造函数的一个主要区别，前者不用 new 也可以执行。

```javascript
class Phone { }

// Uncaught TypeError: Class constructor Phone cannot be invoked without 'new'
Phone();
```

类始终都执行在严格模式下。比如，构造函数，静态方法，原型方法，getter 和 setter 都在严格模式下执行。

```javascript
class Phone {
  constructor() {
    var a, x, y;
    var r = 10;

    // Uncaught SyntaxError: Strict mode code may not include a with statement
    with (Math) {
      a = PI * r * r;
      x = r * cos(PI);
      y = r * sin(PI / 2);
    }
  }
}
```

类方法不可枚举，类定义将 prototype 中的所有方法的 `enumerable` 标志设置为 false。

```javascript
function PhoneFnc(brand) {
  this.brand = brand;
}

PhoneFnc.prototype.call = function () {
  console.log('hello world from', this.brand);
};

class PhoneCls {
  constructor(brand) {
    this.brand = brand;
  }

  call() {
    console.log(`hello world from ${this.brand}`);
  }
}

console.log(PhoneFnc.prototype); // => {constructor: ƒ, call: ƒ}
console.log(PhoneCls.prototype); // => {constructor: ƒ, call: ƒ}
console.log(Object.keys(PhoneFnc.prototype)); // => ['call']
console.log(Object.keys(PhoneCls.prototype)); // => []
```

## 实例对象

类的属性和方法，除非 **显式定义** 在其本身 this 上，否则都是定义在 \_\_proto\_\_ 原型上。

```javascript
class Phone {
  constructor(brand) {
    this.brand = brand;
  }

  call() {
    console.log(`hello world from ${this.brand}`);
  }
}

const huawei = new Phone('huawei');

huawei.call(); // => hello world from huawei
huawei.hasOwnProperty('brand'); // => true
huawei.hasOwnProperty('call'); // => false
huawei.__proto__.hasOwnProperty('call') // => true
```

上面代码中，brand 是实例对象 huawei 自身的属性（因为定义在 this 对象上），所以 hasOwnProperty() 方法返回 true，而 call() 是原型对象的属性（因为定义在 Phone 类上），所以 hasOwnProperty() 方法返回false。这些都与 ES5 的行为保持一致。

与 ES5 一样，类的所有实例共享一个原型对象。

```javascript
class Phone {
  constructor(brand) {
    this.brand = brand;
  }

  call() {
    console.log(`hello world from ${this.brand}`);
  }
}

const xiaomi = new Phone('xiaomi');
const huawei = new Phone('huawei');

console.log(xiaomi.__proto__ === huawei.__proto__); // => true
```

上面代码中，xiaomi 和 huawei 都是 Phone 的实例，它们的原型都是 Phone.prototype，所以 \_\_proto\_\_ 属性是相等的。

这也意味着，可以通过实例的 \_\_proto\_\_ 属性为 class 添加方法。

```javascript
const xiaomi = new Phone('xiaomi');
const huawei = new Phone('huawei');

xiaomi.__proto__.camera = () => console.log('LYCRA is so cooooool ♪(o∀o)っ');

xiaomi.camera(); // => "LYCRA is so cooooool ♪(o∀o)っ"
huawei.camera(); // => "LYCRA is so cooooool ♪(o∀o)っ"

const nokia = new Phone('nokia');
nokia.camera(); // => "LYCRA is so cooooool ♪(o∀o)っ"
```

上面代码在 xiaomi 的原型上添加了一个 camera() 方法，由于 xiaomi 的原型就是 huawei 的原型，因此 huawei 也可以调用这个方法。而且，此后新建的实例 nokia 也可以调用这个方法。这意味着，使用实例的 \_\_proto\_\_ 属性改写原型，必须相当谨慎，不推荐使用，因为这会改变 class 的原始定义，影响到所有实例。~~不是所有相机都叫莱卡~~

## 继承

在看过了前面的内容，相信你对 Javascript 的 `类` 已经有了一个大致的了解。但这个时候你一定有一个疑问，它是如何做到继承的？

~~呐，ES5 的继承有四样写法，你知道么？我教给你，记着！这些应该记着。将来做程序员的时候，开发要用。~~

### 原型链继承

```javascript
function Phone(brand) {
  this.brand = brand;
}

Phone.prototype.call = function () {
  console.log('hello world from', this.brand);
};

function XiaomiPhone(model) {
  this.model = model;
}

XiaomiPhone.prototype = new Phone('Xiaomi');
XiaomiPhone.prototype.slogan = function () {
  console.log('为发烧而生');
}

var xiaomi = new XiaomiPhone('13 Ultra');
```

思路：

  1. 将子类的原型对象指向父类的实例对象

缺点：

  1. 原型中包含的引用值会在 **所有实例** 之间共享
  2. 子类 **实例化** 时，无法给父类构造函数传参

### 构造函数继承

```javascript
function Phone(brand, price) {
  this.brand = brand;
  this.price = price;

  this.call = function () {
    console.log('hello world from', this.brand);
  };
}

function XiaomiPhone(model, price) {
  Phone.call(this, 'Xiaomi', price);
  this.model = model;
}

XiaomiPhone.prototype.slogan = function () {
  console.log('为发烧而生');
}

var xiaomi = new XiaomiPhone('13 Ultra', 5999);
```

思路：

  1. 让父类构造函数在 **子类内部** 运行
  2. 将父类内部的 this 指向子类的实例

缺点：

  1. 父类必须在 **构造函数中** 定义方法
  2. 子类不能访问父类 **原型上** 定义的属性

### 原型继承（经典继承）

```javascript
function Phone(brand, price) {
  this.brand = brand;
  this.price = price;
}

Phone.prototype.call = function () {
  console.log('hello world from', this.brand);
};

function XiaomiPhone(model, price) {
  this.brand = 'Xiaomi';
  this.model = model;
  this.price = price;
}

XiaomiPhone.prototype = Phone.prototype;
XiaomiPhone.prototype.constructor = Phone;
XiaomiPhone.prototype.slogan = function () {
  console.log('为发烧而生');
}

var xiaomi = new XiaomiPhone('13 Ultra');
```

思路：

  1. 让子类的原型指向父类的原型

缺点：

  1. 子类会修改父类的原型对象，造成原型链结构混乱

### 寄生继承

```javascript
function createPhone(object) {
  function Phone() { }

  Phone.prototype = object;
  Phone.prototype.call = function () {
    console.log('hello world from', this.brand);
  };
  return new Phone();
}

function createXiaomiPhone(model, price) {
  var phone = createPhone({
    brand: 'Xiaomi',
    price: price,
  });

  phone.model = model;
  phone.slogan = function () {
    console.log('为发烧而生');
  };

  return phone;
}

var xiaomi = createXiaomiPhone('13 Ultra', 5999);
```

思路：

  1. 结合工厂模式设计，创建一个封装继承的函数
  2. 在函数内部将其 **实例化**，并在对象上做属性扩展并返回

缺点：

  1. 与构造函数继承比较相似，子类只能在函数内部定义方法，由于不能做到复用从而导致低效

### ES6 extends

上述的几种方式都有着各自的优劣，在实际开发中都会采用 **组合编写** 的方式来互补，而在 ES6 发布之后，类的继承有了质的飞跃：

```javascript
class Phone {
  constructor(brand, price) {
    this.brand = brand;
    this.price = price;
  }

  call() {
    console.log('hello world from', this.brand);
  }
}

class XiaomiPhone extends Phone {
  constructor(model, price) {
    super('Xiaomi', price);

    this.model = model;
  }

  slogan() {
    console.log('为发烧而生');
  }
}
```

## 还没想好怎么写 |ू･ω･` )

> TODO

## 参考文献

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes)
- [ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/class)
