---
title: bind 的实现细节
date: 2021-03-04 15:19:35
tags: [手写代码]
categories: [你不知道的JS]
description: bind apply call 的实现细节
---
<!-- more -->

在讲解 bind 之前，先说说我们为什么要去了解一些低层 API 的实现，甚至手撸代码，这些很多在业务中并不会用到。在这里我以自己的经验说明三点：

1. 阅读优秀代码是提高自己的捷径；

毋庸置疑，很多事情我们都是先从抄别人开始的，尤其在程序员这个群体，但抄并不是目的，如果我们想要积累更多的创造思维就需要庖丁解牛一样去了解它的低层实现。优秀的源码中包含了很多设计美学和创造性思维，这些才是值得我们积累的东西，而且越到低层，我们越发现相似的东西越多，很多创造性东西都是从这些基础知识演进而来。

2. 了解并不等于学会；

程序员有一个很大的谎言，就是“我认为我懂了”。我一直认为代码不只是一个需要学习的东西，它更多的是一个需要重复练习的东西，很多人懂了的人却写不出来，是非常典型的眼高手低的表现。能把学会的东西用起来，甚至融会贯通创造新的东西才是最终目的。

3. 能够大大提高你的 debug 能力

我们经常看到一些厉害的人定位 BUG 一眼便能定乾坤，这个一方面处理需要极多的 case 积累，更重要的是对于技术实现上的运筹帷幄，看现象便能猜出个八九不离十，剩下的只是去验证自己的想法。

## 代码实现

我们都知道，bind 方法返回一个新函数，并将新函数的 this 绑定到指定值。我们来看看 bind 函数的官方描述，[摘自 MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)：

- bind() 方法创建一个新的函数；
- bind() 被调用时，这个新函数的 this 被指定为 bind() 的第一个参数；
- 其余参数将作为新函数的参数，供调用时使用；
  
bind 函数的模拟实现其实相对简单，这里直接贴出代码：

```js
Function.prototype.myBind = function() {
  var thatFunc = this, thatArg = arguments[0];
  var args = Array.prototype.slice.call(arguments, 1);
  if (typeof thatFunc !== 'function') {
    throw new TypeError('Function.prototype.bind - what is trying to be bound is not callable');
  }
  return function(){
    var funcArgs = args.concat(slice.call(arguments))
    return thatFunc.apply(thatArg, funcArgs);
  };
};
```

上面代码摘自 MDN 上的 polyfill，是用纯 es5 模拟的，这里还有个问题就是它借用了 apply 用法，我们后面再手动实现 apply。我们先来解释上面代码中涉及的问题：

1. 获取原函数

bind 方法改变了函数执行时候的 this，其实只是在新函数里通过 apply 调用了原函数来达到目的。我们怎么去拿到原函数，这里涉及到 JS 里的一条规则：`函数里的 this 指向它的调用者`。bind 的使用语法是 `fn.bind(thisArg[, arg1[, arg2[, ...]]])`，所以函数内的 this 就指向了原函数 fn。

其实这里还用到**访问者模式**，我们应该对 `Array.prototype.slice.call(arguments)` 不陌生，我们可以在不改变原对象（代指上面的 arguments ，它本身并没有 slice 方法）的前提下，访问其他对象的方法，这种设计方式可以重用代码或者产生一些其他妙用，还有类似用 `Object.prototype.toString.call(fn)` 判断类型等。

2. 返回新函数并拼接参数

我们使用 bind 可以实现类似于下面这种：

```js
const fn = (a, b) => {
  console.log(a, b);
}
// 先传入一个 1
const newFn = fn.bind(null, 1);
// 调用的时候传入 2 和 3
newFn(2); // 1 2
newFn(3); // 1 3
```
我们发现，使用 bind 我们可以让我们预埋一些参数，因为它会将调用 bind 传入的参数和新函数的参数做拼接，这种有点接近于柯里化的思想（虽然不是真正的柯里化），对于之前使用过 papp 的同学这个并不陌生。

3. 实现 apply 方法

上面我们遗留了一个问题，实现 bind 的时候使用了 apply，现在我们来实现它。其实 apply 重点在于如何改变函数中的 this，这个又用到了我们上面那个原则：函数里的 this 指向它的调用者。我们先来看 apply 函数的功能：

- apply 方法接受两个值，第一个参数函数运行时使用的 this 值，第二个参数是一个数组，其中的数组元素将作为单独的参数传给 func 函数；
- 这个函数处于非严格模式下，第一个参数指定为 null 或 undefined 时会自动替换为指向全局对象，原始值会被包装；
- 第二个参数的值为 null 或 undefined，则表示不需要传入任何参数。

直接来看代码：

```js
Function.prototype.myApply = function(otherThis, args) {
  // 判断是否是 null undefined
  function isNil(val) {
    return [null, undefined].includes(val);
  }
  // 原始值会被包装成对象形式，1 会被包装成 new Number(1);
  function toObject(val) {
    const type = typeof(val);
    const fn = {
      string: () => new String(val),
      number: () => new Number(val),
      boolean: () => new Boolean(val),
    }[type];
    return fn ? fn() : val;
  }
  // 如果传入的 this 为 null, undefined，则为 window；否则要包装成对象
  if (isNil(otherThis)) {
    otherThis = window;
  } else {
    otherThis = toObject(otherThis);
  }
  const uniqueID = Symbol("fn");
  // 我们将原函数挂在指定的 this 对象上进行调用，就能达到改变 this 的目的
  otherThis[uniqueID] = this;

  var result = null;
  // 第二个参数的值为 null 或 undefined，则表示不需要传入任何参数
  if (isNil(args)) {
    result = otherThis[uniqueID]();
  } else {
    result = otherThis[uniqueID](...args);
  }
  delete otherThis[uniqueID];
  return result;
};
```

## 相关问题

1. 下面的方法输出什么？

```js
const fn = function() {
  console.log(this);
}
const newFn = fn.bind(1).bind(2).bind(3);
newFn();
```
明白了 bind 的实现原理，我们就很好得出结论，答案是`new Number(1)`。bind 方法每次都会返回一个新函数，并不是原函数 fn 了，所以我们之后再次进行 bind 操作，是对这个新函数的操作，原函数的 this 在第一次的时候就已经确认了。
