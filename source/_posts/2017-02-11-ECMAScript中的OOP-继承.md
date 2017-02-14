---
title: ECMAScript中的OOP-继承
date: 2017-02-08 11:29:04
tags:
 - JavaScript
 - ECMAScript
 - OOP
categories:
 - 学习小结
feature: /assets/images/blog_img/2017-02-11-ECMAScript中的OOP-继承/prototype_chain.png
toc: true
---
本文总结下ECMAScript中继承的几种实现模式。

## 继承模式
许多OO语言都支持两种继承模式：
1. 接口继承
2. 实现继承

C++中可以通过纯虚函数，虚函数来实现接口继承，同时如果不是虚函数则再派生中无法修改函数的实现，便是实现继承。而js中没有函数签名则无法仅继承接口，只支持实现继承，实现的方式便是**原型链**

<!-- more -->

## 原型链
原型链的本质就是**用父类的实例替换掉子类构造函数的原型对象**。

![](/assets/images/blog_img/2017-02-11-ECMAScript中的OOP-继承/prototype_0.png)

``` javascript
> function Father(){
... this.firstName = 'father';
... this.lastName = 'shao';
... }
> Father.prototype.getFirstName = function(){
... console.log(this.firstName);
... }
[Function]
> Father.prototype.getLastName = function(){
... console.log(this.lastName);
... }
[Function]
```
首先先创建一个`Father`类，然后我们通过原型链来实现`Son`类继承`Father`类。
``` javascript
> function Son(){
... this.firstName = 'son';
... }
undefined
> Son.prototype = new Father();
Father { firstName: 'father', lastName: 'shao' }
```
这个时候我用一个`Father`类的实例重写（替换）了`Son`类的原型对象，这个时候如果`Son`的实例中没有找到相应的属性便会去`Son`类的原型中去找，这个时候`Son`类的原型是`Father`的实例，也就是继承了`Father`类了。
``` javascript
> var father = new Father();
undefined
> var son = new Son();
undefined
> son.firstName
'son'
> father.firstName
'father'
> son.lastName 
'shao'  // 这个时候的这个属性是Father实例中的
> father.lastName
'shao'
> Object.getPrototypeOf(son).constructor
[Function: Father]  // son原型是一个Father实例
```
这样，若要新的类需要继承自`Son`，则需要用`Son`的实例替换掉子类的原型对象，这样便形成了一个链，多个构造函数通过这个链实现了继承的效果，这便是原型链了。

![](/assets/images/blog_img/2017-02-11-ECMAScript中的OOP-继承/prototype_chain.png)

### 缺点
1. 最主要的问题：原型中包含引用值，这样不同的实例引用同一个对象。
    ``` javascript
    > function Father(){
    ... this.items = ['car', 'house', 'money'];
    ... }
    undefined
    > function Son(){};
    undefined
    > Son.prototype = new Father()
    Father { items: [ 'car', 'house', 'money' ] }
    > var s1 = new Son()
    undefined
    > var s2 = new Son()
    undefined
    > s1.items
    [ 'car', 'house', 'money' ]
    > s2.items
    [ 'car', 'house', 'money' ]
    ```
    这个时候我们修改`s1`的`items`对象也会修改`s2`的`items`对象，因为items对象都是对原型对象中的`items`引用。
    ``` javascript
    > s2.items.push('watch')
    4
    > s1.items
    [ 'car', 'house', 'money', 'watch' ]
    > s2.items
    [ 'car', 'house', 'money', 'watch' ]
    ```
2. 在创建子类实例时，不能向父类的构造函数中传递参数

## 借用构造函数
为了解决在单独使用原型链中的问题，我们可以使用一种叫做借用构造函数(constructor stealing)的方法，也就是**在子类构造函数中调用父类的构造函数**，有点像我们在Python中使用`super(SonClass, self).__init__(*args, **kwargs)`，这样我们就可以在子类中创建属于子类实例的属性副本而不是共享对象的引用。

``` javascript
> function Father(name, age){
... this.name = name;
... this.age = age;
... }
```
我们在`Son`构造函数中调用`Father`的构造函数
``` javascript
> function Son(name, age){
... Father.call(this, name, age);
... this.job = 'student';
... }
> var s = new Son('shaozj', 25)
undefined
> s
Son { name: 'shaozj', age: 25, job: 'student' }
> var f = new Father('shaols', 51)
undefined
> f
Father { name: 'shaols', age: 51 }
```

### 缺点
如果仅仅使用构造函数而不使用原型链的话，那方法只能在构造函数中定义，这样每个实例都有自己的方法对象便无法实现方法的复用了。

## 组合继承
组合继承就是将**原型链**和**借用构造函数**的技术组合到一起来实现继承，也就是我在子类构造函数中调用父类构造函数的同时，也将原型对象替换成父类的一个实例，然后**重新指定子类原型的`constructor`**。
``` javascript
// 父类
> function Father(name, age){
... this.name = name;
... this.age = age;
... }
> Father.prototype.sayName = function(){
... console.log(this.name);
... }

// 子类
> function Son(name, age){
... Father.call(this, name, age);
... this.job = 'student';
... }
// 原型链
> Son.prototype = new Father('shao', 0)
Father { name: 'shao', age: 0 }
// 修改constructor
> Son.prototype.constructor
[Function: Father]
> Son.prototype.constructor = Son
[Function: Son]

> var s = new Son('shaozj', 25)
undefined
> s
Son { name: 'shaozj', age: 25, job: 'student' }
> var f = new Father('shaols', 51)
undefined
> f
Father { name: 'shaols', age: 51 }
```
`instanceof`和`isPrototypeOf()`可以识别基于组合继承创建的对象，
``` javascript
> s
Son { name: 'shaozj', age: 25, job: 'student' }
> s instanceof Son
true
> s instanceof Father
true
> Object.getPrototypeOf(s)
Son { name: 'shao', age: 0, constructor: [Function: Son] }
```

{% alert info %}
组合继承避免了原型链和借用构造函数的缺陷，融合了他们的有点，是js中最常用的继承模式
{% endalert %}

### 缺点
调用了两次父类的构造函数：一次是在子类构造函数中，一次是在创建子类原型对象的时候。

## 原型式继承
原型式继承一句话总结可以是：对父类对象进行一次浅复制而得到子类对象。

实现的方式是通过将父类实例赋给子类（临时）构造函数的原型。

``` javascript
function object(o) {
    function F(){};
    F.prototype = o;
    return new F();
}
```
ECMAScript5提供了`Object.create()`方法来规范化原型式继承。

``` javascript
> var father = {
... lastName: 'shao',
... }
undefined
> var s1 = Object.create(father)
undefined
> var s2 = Object.create(father)
undefined
> s1.name === s2.name
true
```

### 缺点:
不同实例的属性都是对父类对象（传入）的属性的引用，因此折合使用[**原型模式**](http://pytlab.org/2017/02/06/ECMAScript中的OOP-创建对象/#原型模式)创建对象的效果是一样的。
因此在完全没有必要兴师动众的创建构造函数，而只是想让一个对象与另一个对象保持类似（浅复制）的情况下，我们可以使用原型继承模式。

## 寄生式继承
就是再原型式的基础上将子类对象的属性添加都封装进函数中。
``` javascript
function createOne(o){
    var clone = object(o);  // 使用原型式浅复制创建副本
    clone.saySomething = function() {
        alert("hi");
    };
    return clone;
```

### 缺点
使用寄生式继承来为对象添加函数，会造成无法复用而降低效率，类似构造模式创建对象一样。

## 寄生组合式继承
由于组合式继承有调用两次父类构造函数的缺陷，寄生组合是继承便通过寄生继承用于子类原型的创建，也就是说我们通过寄生函数创建父类原型对象的一个副本，然后**用这个副本去当子类的原型**，这样就避免了第一次的父类构造函数的调用。

完整的实现：

1. 先来个返回浅复制副本的函数，此函数不需要调用父类构造函数便可返回父类原型对象的副本
    ``` javascript
    > function clone(o){
    ... function F(){};
    ... F.prototype = o;
    ... return new F();
    ... }
    ```

2. 再来个辅助函数将父类和子类通过原型链联系起来
    ``` javascript
    > function inheritPrototype(subType, superType){
    ... var prototype = clone(superType.prototype);  // 获取父类原型副本
    ... prototype.constructor = subType;             // 使constructor指向子类构造函数
    ... subType.prototype = prototype;               // 替换子类构造函数原型对象
    ... }
    ```

3. 借用构造函数继承
    ``` javascript
    > function Father(name){
    ... this.name = name;
    ... }
    undefined
    > Father.prototype.sayName = function(){
    ... console.log(this.name);
    ... }
    [Function]
    > function Son(name, age){
    ... Father.call(this, name);
    ... this.age = age;
    ... }
    undefined
    // 关联原型
    > inheritPrototype(Son, Father)
    undefined
    > Son.prototype.sayAge = function(){
    ... console.log(this.age);
    ... }
    [Function]
    > var f = new Father('ls')
    undefined
    > var s = new Son('zj', 25)
    undefined
    > f
    Father { name: 'ls' }
    > s
    Son { name: 'zj', age: 25 }
    > f.sayName()
    ls
    undefined
    > s.sayName()
    zj
    undefined
    > s.sayAge()
    25
    undefined
    ```
这样我们充分利用了构造函数和原型链的优点有避免了重复的调用和重复的属性赋值，构造函数负责属性，原型链负责共享方法。

