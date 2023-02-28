---
title: prototype、__proto__ 与 constructor 分析
category:
  - 笔记
tags:
  - 代码
  - JavaScript
date: 2022-11-02 09:34:48
---
Javascript 里面所有的数据类型都是 "对象"(object)，这一点与 Java 非常相似，所以很多人都会误以为 JavaScript 是一门面向对象的语言。

但从严格意义上来讲，Javascript 并不是一门面向对象语言，而是基于对象，也可以说是基于原型。Javascript 并没有 "类"(class) 的概念，也没有提供像抽象、继承、重载等有关面向对象语言的许多特性，那么 Javascript 是如何进行对象实例化的？

<!-- more -->

## 对象原型

可以说 JavaScript 既是一个面向对象的语言又是一个面向过程的语言。

在 JavaScript 中，通过 **在运行时** 给空对象附加方法和属性来创建对象，与编译语言如 C++ 和 Java 中常见的通过语法来定义类相反。对象构造后，它可以用作是创建相似对象的原型。

前面我们有说到，Javascript 里面所有的数据类型都是对象，其中每个对象上都有 \_\_proto\_\_ 和 constructor 这两个属性。

```javascript
const number = new Number(114514);
// 等价于 const array = new Array(19, 19, 810);
const array = [19, 19, 810];
const string = new String('哼哼哼啊');

console.log(number instanceof Object); // => true
console.log(array instanceof Object); // => true
console.log(string instanceof Object); // => true
```

函数也是对象，较为特殊的是，除了 \_\_proto\_\_ 和 constructor，函数还有 prototype 这个独有属性。prototype 是显式原型，\_\_proto\_\_ 是隐式原型。

```javascript
// 等价于 var foo = new Function();
function Foo() { };

console.log(Foo instanceof Object); // => true
console.log(Foo.__proto__ === Function.prototype); // => true
console.log(Foo.prototype); // => {constructor: ƒ Foo(), [[Prototype]]: Object}
```

对象的 \_\_proto\_\_ 属性是创建对象时自动添加的，默认值为其构造函数的 prototype。函数的 prototype 属性是定义时自动添加的，默认为 { }，一层一层的 \_\_proto\_\_ 便形成了原型链。

```javascript
function Foo() { };

Foo.prototype.test = function () {
  console.log('hello world');
};

var foo = new Foo();
foo.test(); // => hello world
```

prototype 的作用就是包含可以由特定类型的所有实例共享的属性和方法，也就是让该函数所实例化的对象们都可以找到公用的属性和方法。

值得一提的是，Object 的原型是 null，因为它是原型链的最顶端。

```javascript
console.log(Object.prototype.__proto__ === null); // => true
```

## \[\[prototype\]\] 与 \_\_proto\_\_

前面我们有说到，所有对象都有 \_\_proto\_\_ 属性，可是在浏览器里打印却显示的是 \[\[prototype\]\] 而非 \_\_proto\_\_。

其实 [[prototype]] 和 \_\_proto\_\_ 意义相同，均表示对象的内部属性，其值指向对象原型。前者在一些书籍、规范中表示一个对象的原型属性，后者则是在浏览器实现中指向对象原型。

\_\_proto\_\_ 并不是语言本身的特性，这是各大厂商具体实现时添加的私有属性。虽然目前很多现代浏览器的 JS 引擎中都提供了这个私有属性，但依旧不建议在生产中使用该属性，避免对环境产生依赖。生产环境中，建议只使用 Object.getPrototypeOf() 方法来获取实例对象的原型。

## 构造函数

constructor 属性也是对象才拥有的，它是从一个对象指向一个函数，含义就是指向该对象的构造函数。

每个对象都有构造函数，不过 Function 对象比较特殊，它的构造函数就是它自己（因为 Function 可以看成是一个函数，也可以是一个对象），所有函数最终都是由 Function() 构造函数得来，所以 constructor 属性的指向就是 Function()。

## ES6

不过在 ES6 中引入了 class 这个概念，能让定义类更接近传统写法，其本质也只是构造函数的语法糖，最后还是绕不开原型链。

有兴趣你可以查阅 [深入浅出 function 与 class](./function-%E4%B8%8E-class-%E7%9A%84%E5%8C%BA%E5%88%AB.md) 一栏

## 参考文献

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/About_JavaScript)
- [阮一峰的网络日志](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)
