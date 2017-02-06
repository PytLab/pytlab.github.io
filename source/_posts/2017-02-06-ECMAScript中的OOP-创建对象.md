---
title: ECMAScript中的OOP-对象创建
date: 2017-02-06 11:09:14
tags:
 - ECMAScript
 - JavaScript
 - OOP
categories:
 - 学习小结
feature: /assets/images/blog_img/2017-02-06-ECMAScript中的OOP-创建对象/prototype.jpg
toc: true
---
在ECMAScript中我们总不能使用`Object`构造函数或者字面量直接定义对象，这样将会产生大量的重复代码。

这里将ECMAScript中几种面向对象中的创建对象的方法进行总结, 其中包括：
- 工厂模式
- 构造函数模式


## 工厂模式
就是将对象的创建以及属性的添加等细节都封装在一个能够接受参数的函数中。

<!-- more -->

``` javascript
> function createPerson(name, age, job) {
... var person = new Object();
... person.name = name;
... person.age = age;
... person.job = job;
... person.sayName = function() {
..... console.log(this.name);
..... }
... return person;
... }
undefined
> p1 = createPerson('shao', 24, 'student')
{ name: 'shao', age: 24, job: 'student', sayName: [Function] }
> p2 = createPerson('zhao', 25, 'saler')
{ name: 'zhao', age: 25, job: 'saler', sayName: [Function] }
```

### 缺点
仅仅通过一个普通的工厂函数无法解决对象识别的问题，即无法知道一个对象的类型。

## 构造函数模式
像`Object`，`Array`这些都有原生的构造函数并且运行是会自动加载到执行环境中，我们可以用来创建相应的对象。
``` javascript
var o = new Object();
```
我们也可以自己创建自定义的构造函数，定义自定义的对象属性和方法：
``` javascript
function Person(name, age, job){
    this.name = name;
    this.age = age;
    this.job = job;
    this.sayName = function(){
        console.log(this.name);
    };
}
```
{% alert info %}
与普通工厂相比，构造函数有如下不同：
<li>没有显式的创建对象</li>
<li>直接将属性和方法赋给<code>this</code></li>
<li>没有<code>return</code>语句</li>
{% endalert %}

创建该对象使用`new`操作符:
``` javascript
var person = new Person('shao', 24, 'student')
```

{% alert warning %}
使用<code>new</code>操作符调用构造函数创建对象经历了如下的步骤：
<li>创建新<code>Object</code>对象</li>
<li>将构造函数的作用于赋给新的对象，也就是将<code>this</code>指向该对象</li>
<li>执行中括号中的函数体代码</li>
<li>返回该对象</li>
{% endalert %}

``` javascript
> typeof person
'object'
> person instanceof Person
true
> person instanceof Object
true
> person.constructor
[Function: Person]
```
可见`person`对象仍是`Object`实例，当然也是`Person`实例。

### 缺点
- 如果方法的绑定在构造函数中则不同的实例的同名函数是不同的函数对象，使其无法在不同实例中共享。
- 若将函数定义为全局函数，但如果对象需要定义很多方法，就要定义很多全局函数，便也没有什么**封装性**可言了。

## 原型模式

![](/assets/images/blog_img/2017-02-06-ECMAScript中的OOP-创建对象/prototype.jpg)

我先总结一下原型对象与构造函数以及实例之间的关系：

```javascript
> function Person(name, age, job){
... this.name = name;
... this.age = age;
... this.job = job;
... }
undefined
> Person.prototype.sayName = function(){
... console.log(this.name);
... }
[Function]
> var p1 = new Person('shao', 24, 'student')
undefined
> var p2 = new Person('zhao', 25, 'saler')
undefined
```

1. **原型对象**与**构造函数**
    - 无论什么时候创建了一个新函数，就会根据一定的规则为这个函数创建一个原型对象，这个函数对象中有一个属性`prototype`指向这个原型对象。
    - 原型对象中也有一个`constructor`属性指向构造函数

    这样原型对象与构造函数都可以互相访问。
    ``` javascript
    > Person.prototype
    Person { sayName: [Function] }
    > Person.prototype.constructor
    [Function: Person]
    ```

2. **原型对象**与**实例**
    实例内部将包含一个指针（内部属性）`[[prototype]]`指向原型对象
    虽然是内部属性我们无法直接访问，但是我们可以借助`Object.getPrototypeOf()`函数来获取实例的原型对象。
    ``` javascript
    > Object.getPrototypeOf(p1)
    Person { sayName: [Function] }
    > Object.getPrototypeOf(p1) === Object.getPrototypeOf(p2)
    true
    > Person.prototype.isPrototypeOf(p1)
    true
    ```

3. **构造函数**与**实例**
    实例访问`constructor`的时候会进行一次搜索，从实例 -> 原型对象然后将`constructor`返回，实例也就能够通过`constructor`访问构造函数本身了。
    ``` javascript
    > p1.constructor
    [Function: Person]
    > p1.constructor === p2.constructor
    true
    ```

### 原型与`in`操作符
只要是实例能够访问到相应的属性，无论该属性是原型对象中还是对象本身的，`in`操作符就会返回`true`。
``` javascript
> function Person(){}
undefined
> Person.prototype.name = 'shao'
'shao'
> Person.prototype.age = 24
24
> var person = new Person()
undefined
> person.job = 'student'
'student'
> 'job' in person
true
> 'name' in person
true
```

### `Object.keys()` 和 `Object.getOwnPropertyNames()`
这两个方法都是返回属于实例本身的属性，不向上到原型中继续查找。区别在于是否能返回不可枚举的属性。
``` javascript
> Object.getOwnPropertyNames(person)
[ 'job' ]
> Object.keys(person)
[ 'job' ]
> Object.getOwnPropertyNames(Person.prototype)
[ 'constructor', 'name', 'age' ]
> Object.keys(Person.prototype)
[ 'name', 'age' ]
```

### 更简单的原型模式写法
可以通过直接将构造函数的原型对象替换成新的对象来简化写法，不需要再写一堆`.prototype`了。
``` javascript
> function Person(){}
undefined
> Person.prototype = {
... name: 'shao',
... age: 24
... }
{ name: 'shao', age: 24 }
> Object.defineProperty(Person.prototype, 'constructor', {
... enumerable: false,
... value: Person})
Person { name: 'shao', age: 24 }
```

### 缺点
- 忽略了构造函数初始化参数的传递
- 所有属性都是共享的

## 组合使用构造函数和原型模式
这样使得实例都有自己的属性副本，共享方法。

``` javascript
> function Person(name, age){
... this.name = name;
... this.age = age;
... }
undefined
> Person.prototype.sayName = function(){
... console.log(this.name);
... }
[Function]
> var p1 = new Person('shao', 24)
undefined
> var p2 = new Person('zhao', 25)
undefined
> p1.sayName()
shao
undefined
> p2.sayName()
zhao
undefined
> p1.sayName === p2.sayName
true
```
这种混合模式是目前使用最广泛的一种创建自定义类型的方法。

### 缺点
构造函数和原型是相互独立的（算是缺点么？

### 动态原型模式
这种模式就将属性添加和原型的操作都封装到了构造函数中,之所以叫动态原型，我们在构造函数中需要动态的判断是否要给原型对象添加属性。

``` javascript
function Person(name, age){
    this.name = name;
    this.age = age;

    if (typeof Person.prototype.sayName != 'function'){
        Person.prototype.sayName = function(){
            console.log(this.name);
        };
    }
}
```
也就是说只有在第一次调用构造函数创建实例的时候条件才满足，才会给原型对象添加属性，这也就是多个实例共享一个方法。

