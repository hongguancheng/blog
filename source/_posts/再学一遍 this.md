---
title: 再学一遍 this
thumbnail: http://honggc.b0.upaiyun.com/blog/oaqk7qqnh_c-patrick-tomasso.jpg
date: 1483891200000
---

最近在看《你不知道的 Javascript》，看到了关于 this 的介绍，收获良多。这里分享一下书中关于 this 的介绍。

### 误解

在了解 this 的工作原理之前首先要理清我们对 this 的一些错误的认识。

#### 指向自身

我们很容易会把 this 理解成为指向函数本身，这个从英语的语法上是说的通的。然而事实并不是这样，我们看看下面这个例子：

```javascript
function foo(){
  console.log(this.a)
}

foo.a = 1

foo()
```

最后输出的结果是 undefined 并不是 1，原因是 this 并不指向函数对象而指向了global 对象（浏览器是 window）

#### 指向它的作用域

第二个常见的误解是，this 指向了函数的词法作用域。

this 在任何情况下都不会指向函数的词法作用域。词法作用域确实很像一个对象，但是作用域"对象"只存在于 JavaScript 引擎内部。

下面看一个例子：

```javascript
function foo(){
  var a = 1
  console.log(this.a)
}

foo()
```

最后的结果还是 undefined，原因跟上一个例子的原因一样。

### this 全面解析

所以 this 究竟指向哪里呢？这取决于函数的调用位置（也就是函数的调用方法）。找到调用位置之后需要应用下面提到的四条规矩，然后再看看如果多条规则同时使用时他们的优先级。

#### 默认绑定

首先要介绍的就是最常用的函数调用类型，可以把这个规则看作是无法应用其他规矩的默认规则。看下面例子：

```javascript
function foo(){
  console.log(this.a)
}

var a = 1

foo() // 1
```

在调用 foo() 时，这里的 this 会指向全局对象，所以会输出 1。

如果是用的严格模式，那么 this 会绑定到 undefined。上面的例子会报错。

#### 隐式绑定

接下来的规则是函数调用的位置是否有上下文对象，请看下面例子：

```javascript
function foo () {
  console.log(this.a)
}

var obj = {
  a: 2,
  foo: foo
}
obj.foo() // 2
```

这里的函数调用位置会使用 obj 的上下文，当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。因为调用 foo() 时 this 被绑定到 obj，因此 this.a 和 obj.a 是一样的。

##### 隐式丢失

一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定。

看下面代码：

```javascript
function foo () {
  console.log(this.a)
}
var obj = {
  a: 2,
  foo: foo
}
var bar = obj.foo; // 函数别名!
var a = 'oops, global' // a是全局对象的属性 
bar(); // "oops, global"
```

这里的 bar 实际上是引用 foo 函数本身，因此这里的 bar() 应是默认绑定。

一种更微妙、更常见并且更出乎意料的情况发生在传入回调函数时:

```javascript
function foo () {
  console.log(this.a)
}
function doFoo (fn) {
  // fn其实引用的是foo
  fn(); // <-- 调用位置!
}
var obj = {
  a: 2,
  foo: foo
}
var a = 'oops, global'; // a是全局对象的属性
doFoo(obj.foo); // "oops, global"
```

参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一个例子一样。

如果把函数传入语言内置的函数而不是传入你自己声明的函数，会发生什么呢？结果是一样的，没有区别：

```javascript
function foo () {
  console.log(this.a)
}
var obj = {
  a: 2,
  foo: foo
}
var a = 'oops, global'; // a是全局对象的属性 
setTimeout(obj.foo, 100); // "oops, global"
```

JavaScript 环境中内置的 setTimeout() 函数实现和下面的伪代码类似:

```javascript
function setTimeout(fn,delay) { 
  // 等待 delay 毫秒
  fn(); // <-- 调用位置!
}
```

#### 显式绑定

当我们分析隐式绑定时，需要在一个对象内部包含一个指向函数的属性，并通过这个属性间接引用函数，从而把 this 间接(隐式)绑定到这个对象上。那么如果我们不想这么做就可以在对象上强制调用函数呢？

一般是用 call(…) 和 apply(…)，他们的第一个参数是一个对象，它们会把这个对象绑定到 this。我们称之为显式绑定。

思考下面代码：

```javascript
function foo () {
  console.log(this.a)
}
var obj = {
  a: 2
}
foo.call(obj); // 2
```

这里通过 foo.call(…) 在 foo 调用时把 this 绑定到 obj 上。

如果你传入了一个原始值(字符串类型、布尔类型或者数字类型)来当作 this 的绑定对象，这个原始值会被转换成它的对象形式(也就是 new String(..)、new Boolean(..)或者 new Number(..))。这通常被称为“装箱”。

可惜,显式绑定仍然无法解决我们之前提出的丢失绑定问题。

##### 硬绑定

但是显式绑定的一个变种可以解决这个问题。

思考下面的代码:

```javascript
function foo () {
  console.log(this.a)
}
var obj = {
  a: 2
}
var bar = function () {
  foo.call(obj)
}
bar() // 2
setTimeout(bar, 100) // 2
// 硬绑定的 bar 不可能再修改它的 this 
bar.call( window ); // 2
```

这里我们创建了函数 bar，并在函数内手动调用了 foo.call(obj)，所以无论怎么调用 bar，它都会手动在 obj 上调用 foo。这种绑定是一种显式的强制绑定,因此我们称之为硬绑定。

硬绑定的典型应用场景就是创建一个包裹函数,传入所有的参数并返回接收到的所有值:			

```javascript
function foo (something) {
  console.log(this.a, something)
  return this.a + something
}
var obj = {
  a: 2
}
var bar = function () {
  return foo.apply(obj, arguments)
}
var b = bar(3) // 2 3 
console.log(b); // 5
```

另一种使用方法是创建一个可以重复使用的辅助函数:

```javascript
function foo (something) {
  console.log(this.a, something)
  return this.a + something
}
// 简单的辅助绑定函数 
function bind (fn, obj) {
  return function () {
    return fn.apply(obj, arguments)
  }
}
var obj = {
  a: 2
}
var bar = bind(foo, obj)
var b = bar(3) // 2 3 
console.log(b) // 5
```

由于硬绑定是一种非常常用的模式,所以在 ES5 中提供了内置的方法Function.prototype. bind,它的用法如下:

```javascript
function foo (something) {
  console.log(this.a, something)
  return this.a + something
}

var obj = {
  a: 2
}
var bar = foo.bind(obj)
var b = bar(3) // 2 3 
console.log(b); // 5
```

bind(..) 会返回一个硬编码的新函数，它会把参数设置为 this 的上下文并调用原始函数。

#### new 绑定

这是最后一条规则。使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作。

1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的 this。
4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

思考下面代码：

```javascript
function foo (a) {
  this.a = a
}
var bar = new foo(2)
console.log(bar.a); // 2
```

使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上。new 是最后一种可以影响函数调用时 this 绑定行为的方法，我们称之为 new 绑定。

### 优先级

现在我们已经了解了函数调用中 this 绑定的四条规则，你需要做的就是找到函数的调用位 置并判断应当应用哪条规则。但是,如果某个调用位置可以应用多条规则该怎么办？为了解决这个问题就必须给这些规则设定优先级，这就是我们接下来要介绍的内容。

毫无疑问，默认绑定的优先级是四条规则中最低的，所以我们可以先不考虑它。

隐式绑定和显式绑定哪个优先级更高？我们来测试一下:

```javascript
function foo () {
  console.log(this.a)
}
var obj1 = {
  a: 2,
  foo: foo
}
var obj2 = {
  a: 3,
  foo: foo
}
obj1.foo() // 2
obj2.foo() // 3
obj1.foo.call(obj2) // 3
obj2.foo.call(obj1); // 2
```

可以看到，显式绑定优先级更高。

接下来比较 new 绑定和显式绑定谁的优先级更高。因为 new 和 call/apply 无法一起使用，所以我们用硬绑定来测试他们的优先级：

```javascript
function foo (something) {
  this.a = something
}
var obj1 = {}
var bar = foo.bind(obj1)
bar(2)
console.log(obj1.a) // 2
var baz = new bar(3)
console.log(obj1.a) // 2 
console.log(baz.a) // 3
```

从上面例子我们可以看出，虽然 bar 被硬绑定到 obj1 上了，但是 new bar(3) 并没有把 obj1.a 修改为 3，我们得到了一个名字为 baz 的新对象，并且 baz.a 的值是 3。所以说 new 绑定的优先级比显式绑定高。

#### 判断 this

现在我们可以根据优先级来判断函数在某个调用位置应用的是哪条规则。可以按照下面的顺序来进行判断:

1. 函数是否在new中调用(new绑定)？如果是的话this绑定的是新创建的对象。

   ```javascript
   var bar = new foo()
   ```

2. 函数是否通过call、apply(显式绑定)或者硬绑定调用？如果是的话，this绑定的是指定的对象。

   ```javascript
   var bar = foo.call(obj2)
   ```

3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this绑定的是那个上下文对象。

   ```javascript
   var bar = obj1.foo()
   ```

4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。

   ```javascript
   var bar = foo()
   ```

### ES6 箭头函数

ES6 中介绍了一种无法使用上面介绍的规则的特殊函数：箭头函数。

箭头函数并不是使用 function 关键字定义的，而是使用被称为“胖箭头”的操作符 => 定义的。箭头函数不使用 this 的四种标准规则，而是根据外层(函数或者全局)作用域来决定 this。

我们来看看箭头函数的词法作用域：

```javascript
function foo () {
  // 返回一个箭头函数 
  return (a) => {
    // this 继承自 foo()
    console.log(this.a);}
}
var obj1 = {
  a: 2
}
var obj2 = {
  a: 3
}
var bar = foo.call(obj1)
bar.call(obj2); // 2,不是3!
```

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。由于 foo() 的 this 绑定到 obj1，bar（引用箭头函数）的 this 也会绑定到 obj1，箭头函数的绑定无法被修改。（new 也不行！）

箭头函数可以像 bind(..) 一样确保函数的 this 被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的 this 机制。实际上，在 ES6 之前我们就已经在使用一种几乎和箭头函数完全一样的模式。

```javascript
function foo () {
  var self = this
  setTimeout(function () {
    console.log(self.a)
  }, 100)
}
var obj = {
  a: 2
}
foo.call(obj); // 2
```