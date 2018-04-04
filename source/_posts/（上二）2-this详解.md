---
title: （上二）2-this详解
date: 2018-04-04 11:08:57
tags: [你不知道的JS]
categories: 读书笔记
description: 详解 JS 中的 this，判断 this 指向和妙用
---
<!-- more -->

>我们知道，this 在与函数声明的位置无关，而是在函数在调用时绑定的，完全取决于函数的调用位置（也就是函数的调用方法）

## 调用位置

- 调用位置 — 调用位置就是函数在代码中被调用的 位置（而不是声明的位置）
- 调用栈 — 为了到达当前执行位置所调用的所有函数。你可以把调用栈想象成一个函数调用链

通常来说，寻找调用位置就是寻找“函数被调用的位置”。而最重要的是要分析调用栈，我们关心的调用位置就在当前正在执行的函数的前一个调用中

```
function baz() {
    // 当前调用栈是:baz
    // 因此，当前调用位置是全局作用域
    console.log( "baz" );
    bar();              // <-- bar 的调用位置
}
function bar() { 
    // 当前调用栈是: baz -> bar
    // 因此，当前调用位置在 baz 中
    console.log( "bar" );
    foo();              // <-- foo 的调用位置
}
function foo() {
    // 当前调用栈是: baz -> bar -> foo
    // 因此，当前调用位置在 bar 中
    console.log( "foo" );
}
baz();                  // <-- baz 的调用位置
```

## 绑定规则

找到调用位置后，我们就可以依据下面的规则来决定 `this` 的绑定对象

### 默认绑定

首先要介绍的是最常用的函数调用类型：独立函数调用。可以把这条规则看作是无法应用其他规则时的默认规则

```
function foo() {
    console.log( this.a );
}

var a = 2;
foo();      // 2
```
在代码中，`foo()` 是直接使用不带任何修饰的函数引用进行调用的，因此只能使用
默认绑定，无法应用其他规则。因此 this 指向全局对象；如果使用严格模式（` strict mode` ），那么全局对象将无法使用默认绑定，因此 this 会绑定到 `undefined`

### 隐式绑定

另一条需要考虑的规则是调用位置是否有上下文对象，或者说是否被某个对象拥有或者包含，不过这种说法可能会造成一些误导

```
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo();      // 2
```
调用位置会使用 obj 上下文来引用函数，因此你可以说函数被调用时 obj 对象“拥 有”或者“包含”它。当函数引用有上下文对象时，隐式绑定规则会把函数调用中的 this 绑定到这个上下文对象。

>对象属性引用链中只有最顶层或者说最后一层会影响调用位置

#### 隐式丢失

```
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo;          // 函数别名!
var a = "oops, global";     // a 是全局对象的属性 
bar();                      // "oops, global"
```
虽然 `bar`是 `obj.foo` 的一个引用，但是实际上，它引用的是 foo 函数本身，因此此时的`bar()`其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

**还有一种更微妙**、更常见并且更出乎意料的情况发生在传入回调函数时。参数传递其实就是一种隐式赋值，因此我们传入函数时也会被隐式赋值，所以结果和上一 个例子一样
```
function foo() {
    console.log( this.a );
}
function doFoo(fn) {
    // fn 其实引用的是 foo 
    fn();               // <-- 调用位置!
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global"; // a 是全局对象的属性 
doFoo( obj.foo );       // "oops, global"

// 下面类似（把函数传入语言内置的函数而不是传入你自己声明的函数，结果一样）
function foo() {
    console.log( this.a );
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global";     // a 是全局对象的属性 
setTimeout( obj.foo, 100 ); // "oops, global"
```
**还有一种情况**， this 的行为会出乎我们意料:调用回调函数的函数可能会修改 this。在一些流行的 JavaScript 库中事件处理器常会把回调函数的 this 强制绑定到触发事件的 DOM 元素上。

### 显式绑定

JavaScript 提供的绝大多数函数以及你自己创建的所有函数都可以使用 `call(..)` 和 `apply(..)` 方法显式绑定某个对象到`this`。
- 它们的第一个参数是一个对象，它们会把这个对象绑定到`this`
- 如果你传入了一个原始值，这个原始值会被转换成它的对象形式

```
function foo() {
    console.log( this.a );
}
var obj = {
    a:2
};
foo.call( obj ); // 2
```
可惜，显式绑定仍然无法解决我们之前提出的丢失绑定问题

#### 1. 硬绑定

但显式绑定的一个变种（硬绑定）可以解决这个问题。

我们在函数 `bar()`内部手动调用了`foo.call(obj)`，强制把 `foo` 的 `this` 绑定到了 `obj`。无论之后如何调用函数 `bar`，它总会手动在`obj`上调用`foo`。这种绑定是一种显式的强制绑定，因此我们称之为**硬绑定**

```
function foo() {
    console.log( this.a );
}
var obj = {
    a:2
};
var bar = function() {
    foo.call( obj );
};
bar();                      // 2
setTimeout( bar, 100 );     // 2
// 硬绑定的 bar 不可能再修改它的 this 
bar.call( window );         // 2
```
硬绑定的应用场景：
- 创建一个包裹函数，传入所有的参数并返回接收到的所有值；
- 另一种使用方法是创建一个可以重复使用的辅助函数；

```
// 1. 包裹函数
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
var obj = {
    a:2
};
var bar = function() {
    return foo.apply( obj, arguments );
};
var b = bar( 3 ); // 2 3 
console.log( b ); // 5

// 2. 辅助函数
function foo(something) {
    console.log( this.a, something ); 
    return this.a + something;
}
// 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    };
}
var obj = {
    a:2
};
var bar = bind( foo, obj );
var b = bar( 3 ); // 2 3
console.log( b ); // 5
```
由于硬绑定是一种非常常用的模式，所以在 `ES5` 中提供了内置的方法 `Function.prototype. bind`，`bind(..)` 会返回一个硬编码的新函数，它会把参数设置为 `this` 的上下文并调用原始函数。

#### 2. API 调用的“上下文”

第三方库的许多函数，以及 `JavaScript` 语言和宿主环境中许多新的内置函数，都提供了一 个可选的参数，通常被称为“上下文”(`context`)，其作用和 `bind(..)` 一样，确保你的回调函数使用指定的 `this`。比如：`forEach`
```
var obj = {
    id: "awesome"
};
// 调用 foo(..) 时把 this 绑定到 obj 
[1, 2, 3].forEach(function(id) {
    console.log(this.id)    // awesome awesome awesome
}, obj );
```
这些函数实际上就是通过 `call(..)` 或者 `apply(..)` 实现了显式绑定，这样你可以少写一些代码

### new 绑定

>在传统的面向类的语言中，“构造函数”是类中的一些特殊方法，使用 new 初始化类时会调用类中的构造函数。通常的形式是这样的：`something = new MyClass(..);`

在 `JavaScript` 也有一个 `new` 操作符，使用方法看起来也和那些面向类的语言一样。然而，`JavaScript` 中 `new` 的机制实际上和面向类的语言完全不同。

说到这里，我们要说一下`JS`中的类，在`JS`中没有类的概念，虽然在`ES6`中提出了类，但也只是一种语法糖。

`JS`中的构造函数也只是一些使用`new`操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被`new`操作符调用的普通函数而已。所以，包括内置对象函数（比如`Number(..)`）在内的所有函数都可以用`new`来调用，这种函数调用被称为构造函数调用。

>注意：这里有一个重要但是非常细微的区别：实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作：
1. 创建(或者说构造)一个全新的对象。
2. 这个新对象会被执行[[原型]]连接。
3. 这个新对象会绑定到函数调用的this。
4. 如果函数没有返回其他对象，那么new表达式中的函数调用会自动返回这个新对象。

使用 `new` 来调用 `foo(..)` 时，我们会构造一个新对象并把它绑定到 `foo(..)` 调用中的 `this` 上。`new` 是最后一种可以影响函数调用时 `this` 绑定行为的方法，我们称之为**new 绑定**。

## 优先级

经测试，我们发现下面的优先级：

![](http://picture-market.oss-cn-beijing.aliyuncs.com/18-4-4/78022073.jpg)

硬绑定和 new 绑定的优先级，看下面例子：

```
function foo(something) {
    this.a = something;
}
var obj = {};
var bar = foo.bind( obj ); 
bar( 2 );
console.log( obj.a );  // 2

var baz = new bar(3);
console.log( obj.a );  // 2 
console.log( baz.a );  // 3
```
我们发现，给`foo`函数硬绑定`obj`之后。普通调用`foo`后为`obj`添加上了属性（正常）；通过`new`操作符调用之后，`obj`没变化，但生成的新对象`baz`变化了。**其实是因为硬绑定函数在被`new`调用时，会使用新创建的 `this` 替换硬绑定的 `this`**

判断`this`的步骤：

1. 函数是否在`new`中调用（`new`绑定）？如果是的话`this`绑定的是新创建的对象。`var bar = new foo()`
2. 函数是否通过`call`、`apply`（显式绑定）或者硬绑定调用？如果是的话，`this`绑定的是指定的对象。`var bar = foo.call(obj2)`
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，`this` 绑定的是那个上下文对象。`var bar = obj1.foo()`
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到`undefined`，否则绑定到全局对象。

## 绑定例外

规则总有例外，这里也一样；

### 被忽略的`this`

如果你把 `null` 或者 `undefined` 作为 `this` 的绑定对象传入 `call`、`apply` 或者 `bind`，这些值
在调用时会被忽略，实际应用的是默认绑定规则：
```
function foo() {
    console.log( this.a );
}
var a = 2;
foo.call( null ); // 2
```

什么情况下你会传入 `null` 呢？
- 使用 `apply(..)` 来“展开”一个数组，并当作参数传入一个函数（在 ES6 中，可以用 `...` 操作符代替）
- `bind(..)` 可以对参数进行柯里化（预先设置一些参数）

```
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}
// 1. 把数组“展开”成参数
foo.apply( null, [2, 3] );      // a:2, b:3
// 2. 使用 bind(..) 进行柯里化
var bar = foo.bind( null, 2 );
bar( 3 );                       // a:2, b:3
```
这两种方法都需要传入一个参数当作`this`的绑定对象。如果函数并不关心`this`的话，你仍然需要传入一个占位值，这时`null`可能是一个不错的选择。

**但是**，总是使用 `null` 来忽略 `this` 绑定可能产生一些副作用。如果某个函数（第三方库）确实使用了`this`，使用`null`会把`this`绑定到全局对象，导致问题。

**所以，一个更安全的`this`**

一种“更安全”的做法是传入一个特殊的对象，把`this`绑定到这个对象不会对你的程序产生任何副作用。我们可以创建一个“DMZ”（demilitarized zone，非军事区）对象 — 它就是一个空的非委托的对象。
```
function foo(a,b) {
    console.log( "a:" + a + ", b:" + b );
}
// 我们的 DMZ 空对象
var ø = Object.create( null ); 
// 1. 把数组展开成参数
foo.apply( ø, [2, 3] );     // a:2, b:3
// 2. 使用 bind(..) 进行柯里化
var bar = foo.bind( ø, 2 ); 
bar( 3 );                   // a:2, b:3
```
>`Object.create(null)` 和 `{}` 很像，但是并不会创建 `Object.prototype` 这个委托，所以它比 `{}`“更空”

### 间接引用

你有可能（有意或者无意地）创建一个函数的“间接引用”，在这 种情况下，调用这个函数会应用默认绑定规则：
```
function foo() {
    console.log( this.a );
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo();                // 3
(p.foo = o.foo)();      // 2
```
赋值表达式 `p.foo = o.foo` 的返回值是目标函数的引用，因此调用位置是 `foo()` 而不是 `p.foo()` 或者 `o.foo()`。根据我们之前说过的，这里会应用默认绑定。

### 软绑定

硬绑定这种方式可以把 this 强制绑定到指定的对象（除了使用 new 时），问题在于，硬绑定会大大降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改 `this`。

可以通过一种被称为软绑定的方法来实现我们想要的效果：
```
if (!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有 curried 参数
        var curried = [].slice.call( arguments, 1 );
        var bound = function() {
            return fn.apply(
            (
                !this || this === (window || global)) ? obj : this,
                curried.concat.apply( curried, arguments )
            );
        };
        bound.prototype = Object.create( fn.prototype );
        return bound;
    };
}

```
试试效果
```
function foo() {
    console.log("name: " + this.name);
}
var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
    
var fooOBJ = foo.softBind( obj );
fooOBJ();               // name: obj

obj2.foo = foo.softBind(obj);
obj2.foo();             // name: obj2 <---- 看!!!

fooOBJ.call( obj3 );    // name: obj3 <---- 看! 

setTimeout( obj2.foo, 10 );
// name: obj <---- 应用了软绑定
```

## `this`词法

ES6 中的箭头函数不使用 `this` 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 `this`。箭头函数可以像 `bind(..)` 一样确保函数的 `this` 被绑定到指定对象，此外，其重要性还体现在它用更常见的词法作用域取代了传统的 `this` 机制
- 箭头函数的绑定无法被修改。(new 也不 行!)
- 箭头函数最常用于回调函数中，例如事件处理器或者定时器

## 小结

#### 1. 如何判断 `this`？

`this` 与函数声明的位置无关，而是在函数在调用时绑定的，取决于函数的调用位置。判断`this`可以使用下面规则：

1. 箭头函数中的`this`由外层作用域中的`this`决定；
2. 由`new`调用？绑定到新创建的对象。
3. 由`call`或者`apply`（或者`bind`）调用？绑定到指定的对象。
4. 由上下文对象调用？绑定到那个上下文对象。（谁调用指向谁）
5. 默认：在严格模式下绑定到`undefined`，否则绑定到全局对象。

[彻底理解js中this的指向](http://web.jobbole.com/85198/)
