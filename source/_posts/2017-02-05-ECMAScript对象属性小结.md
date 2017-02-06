---
title: ECMAScript对象属性小结
date: 2017-02-05 10:10:21
tags:
 - JavaScript
 - ECMAScript
 - OOP
categories:
 - 学习小结
feature: /assets/images/features/js_logo.png
toc: true
---
最近两天过了一遍JS高程的前7章，第一次了解ECMAScript中基于原型(prototype-based)的面向对象的实现。思想看起来是那么的顺理成章并不难理解，但是由于属于新事物，所以打算对ECMAScript的OOP进行一下简单的小结，方便以后回来温习。

### 属性类型
ECMAScript对象中有两种属性：
- 数据属性
- 访问器属性

### 属性特征值(characteristic)
ECMAScript对象创建时都带有一些特征值，JS通过这些特征值来定义对象属性的行为。

<!-- more -->

先创建一个对象为后面提供例子：
``` javascript
var person = {
    name: 'shao',
    age: 24,
    job: 'student',
    sayName: function(){
        console.log(this.name);
    }
}
```

### 数据属性和访问器属性共有的属性特性

#### `[[Configurable]]`
表示能否通过delete删除属性从而重新定义属性，能否修改属性的特性，能否把属性修改为访问器属性。
直接在对象上定义的属性，默认为`true`。

也就是说如果对象的某个属性的`[[configurable]]`特性值改为`false`，则就不能再修改这个属性的特性了。

``` javascript
> Object.getOwnPropertyDescriptor(person, 'name')
{ value: 'shao',
  writable: true,
  enumerable: true,
  configurable: true }
> 
```
可见，默认情况下，`configurable`的值为`true`。这时我们修改`name`的值或者删除`name`属性都是可以的。
``` javascript
> person.name = 'zhao'
'zhao'
> delete person.name
true
> person
{ age: 24, job: 'student', sayName: [Function] }
>
```
但是，如果我将`configuratble`修改为`false`,这时再重复上述操作就被禁止了。

修改了`name`属性的`configurable`属性为`false`.

``` javascript
> Object.defineProperty(person, 'name', {configurable: false})
{ age: 24, job: 'student', sayName: [Function], name: 'shao' }
```
``` javascript
> person.name = 'zhao'
'zhao'
> person.name
'zhao'
> delete person.name
false
> person.name
'zhao'
```
可见，我们还是可以对该属性的值进行修改的，但是如果我们想删除这个属性的时候便会失败。

看看如果我们再想将`[[configurable]]`属性特性再修改回`true`会怎样：
``` javascript
> Object.defineProperty(person, 'name', {configurable: true})
TypeError: Cannot redefine property: name
    at Function.defineProperty (native)
    at repl:1:8
    at REPLServer.defaultEval (repl.js:252:27)
    at bound (domain.js:287:14)
    at REPLServer.runBound [as eval] (domain.js:300:12)
    at REPLServer.<anonymous> (repl.js:417:12)
    at emitOne (events.js:82:20)
    at REPLServer.emit (events.js:169:7)
    at REPLServer.Interface._onLine (readline.js:210:10)
    at REPLServer.Interface._line (readline.js:549:8)
> person
```

#### `[[Enumerable]]`
表示能否通过`for-in`循环返回该属性，例如：
``` javascript
> for (i in person) {
... console.log(i);
... }
name
age
job
sayName
>
```
可见这个对象的所有属性默认的`[[Enumerable]]`属性都为`true`。

但如果我将`name`的`[[Enumerable]]`改为`false`，输出中便不会出现`name`属性了。

``` javascript
> Object.defineProperty(person, 'name', {enumerable: false})
{ age: 24, job: 'student', sayName: [Function] }
> Object.getOwnPropertyDescriptor(person, 'name')
{ value: 'shao',
  writable: true,
  enumerable: false,
  configurable: true }
> for (i in person) {
... console.log(i);
... }
age
job
sayName
>
```

### 数据属性特性

#### `[[Writable]]`
该属性表示能否修改属性的值，默认值为`true`。

``` javascript
> Object.defineProperty(person, 'name', {writable: false})
{ name: 'shao', age: 24, job: 'student' }
> person.name = 'zhao'
'zhao'
> person.name
'shao'
```

可见我们无法修改`name`属性的值了。

#### `[[Value]]`
这个包含这个属性的数据值。读取属性的时候再这个位置读取;写入数据的时候把新值保存再这个位置。这个位置的值默认值为`undefined`。

### 访问器属性

#### `[[Get]]`
这个属性特性就是在读取属性的时候调用的函数，相当与Python描述符中的`__get__()`方法，也就是属性的`getter`方法。

#### `[[Set]]`
这个属性特性就是在写入属性的时候调用的函数，相当于Python描述符中的`__set__()`方法。

具体的来个例子：

``` javascript
> var person = {
... _name: 'shao',
... age: 24
... }

> Object.defineProperty(person, 'name', {
... get: function() {
..... return this._name;
..... },
... set: function(name) {
..... this._name = name;
..... }
... })
{ _name: 'shao', age: 24 }
> person.name
'shao'
> person.name = 'zhao'
'zhao'
> person.name
'zhao'
```

### 关于描述符
虽然都叫描述符(descriptor)，但是ECMAScript中和Python的完全是两个不同的概念。
- 在JS中，描述符就是个对象(object)，这个对象描述了对象的属性，也就是一个包含`configurable`, `writable`, `enumerable`, `value`, `get`, `set`的对象而已。
- 而在Python中描述符是一个实现描述符协议的对象，类似js中的访问器属性。

