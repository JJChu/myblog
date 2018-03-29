---
title: （上二）1-关于this
date: 2018-03-29 20:23:00
tags: [你不知道的JS]
categories: 读书笔记
description: this 基础
---
<!-- more -->

>`this` 是一个很特别的关键字，被自动定义在所有函数的作用域中，在没有足够认识的情况下，很难说清`this`到底指向什么？实际上，`JavaScript` 中`this`的机制并没有那么先进，但是开发者往往会把理解过程复杂化

## 为什么使用 this

以下面这段代码为例：

```
function identify() {
    return this.name.toUpperCase();
}
function speak() {
    var greeting = "Hello, I'm " + identify.call( this );
    console.log( greeting );
}
var me = {
    name: "Kyle"
};
var you = {
    name: "Reader"
};
identify.call( me ); // KYLE
identify.call( you ); // READER
speak.call( me ); // Hello, 我是 KYLE
speak.call( you ); // Hello, 我是 READER
```
我们发现：这段代码可以在不同的上下文对象（me 和 you）中重复使用函数 `identify()` 和 `speak()`。如果不适用`this`，代码将会变成下面这样：

```
function identify(context) {
    return context.name.toUpperCase();
}
function speak(context) {
    var greeting = "Hello, I'm " + identify( context );
    console.log( greeting );
}
identify( you );    // READER
speak( me );        // hello, 我是 KYLE
```
我们必须明确的传递一个参数到函数内部。因此，**`this` 提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将 API 设计得更加简洁并且易于复用**

## 误解

在深入理解`this`之前，我们首先要消除一些对`this`的错误认识：
- `this` 指向自身。`this`并非像其英语释义那样指向当前对象（函数）自身；
- `this` 指向函数作用域。因为在某种情况下它是正确的，但是在其他情况下它却是错误的；
    + 需要明确的是，`this`在任何情况下都不指向函数的词法作用域。在 JavaScript 内部，作用域确实和对象类似，可见的标识符都是它的属性。但是作用域“对象”无法通过 JavaScript 代码访问，它存在于 JavaScript 引擎内部。 

## this 到底是什么

`this` 是在运行时进行绑定的，并不是在编写时绑定。`this` 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。`this` 就是记录的其中一个属性，会在函数执行的过程中用到。

## 小结

#### 1.`this`到底是什么？

 `this` 既不指向函数自身也不指向函数的词法作用域。`this` 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。