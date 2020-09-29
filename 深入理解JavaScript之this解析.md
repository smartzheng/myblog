+++
title= "深入理解JavaScript之this解析"
date= "2019-09-27"
categories= [ "web前端" ]
tags= [ "JavaScript" ]
+++
### 引言

在过去用Java或Kotlin进行Android开发的时候，this从来不是一个问题，即使是在内部类中this也存在指向问题，但是使用this@xxx的方式就可以很简单地解决，IDE也有很友好的提示  

与Java不同的是，JS中的this真是一大难题，我个人感觉而言，应该是我在学习JS的过程中最难理解的一点，虽然日常使用不会有太大问题，自己在项目代码中也因为以前的Java代码习惯很少使用this，但是要想深入全面地理解JS，this是一个避不开的问题

在《JavaScript高级程序设计》一书闭包一节中，直接是这么写的：

>我们知道，this对象是在运行时基于函数的执行环境绑定的:在全局函数中，this等于window，而当函数被作为某个对象的方法调用时，this等于那个对象。

这个结论虽然好用，但是并不全面，也并没有解释为什么会有这样的结果。《你不知道的JavaScript》中则将前因后果都介绍得比较清楚，另外，推荐一个不错的JS教程： [现代JavaScript教程](https://zh.javascript.info/) ，里面的 [对象方法与 "this"](https://zh.javascript.info/object-methods) 一节也很简明扼要地分析了this问题

### 词法作用域和动态作用域

要想理解this，最好先搞清楚词法作用域和动态作用域的区别，先看个例子

```javascript
var a = 0;
function foo() {
    console.log(a);
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo
bar()
bar.call(obj); 
```
这里会依次打印：0，0，0，2  

所谓词法作用域就是在代码执行之前就可以通过词法分析就可以确定的作用域，而动态作用域则是基于调用堆栈，需要在执行期间才能知道。JS中并没有动态作用域，但是this的规则却和动态作用域很类似

上面代码中console.log(a)两次都打印了0，也就是第一行声明的a的值；而console.log(this.a)两次打印的结果却不同。JS只有词法作用域，而this的类似于动态作用域的特点是通过其特有的绑定规则实现的

### this的四种绑定规则

1、默认绑定

```javascript
var a = 0;
function foo(){
	console.log(this.a);
}
```

上面的代码中this就是进行了默认绑定，foo函数没有被new，也没有通过对象调用，或者调用call、apply，this直接指向了全局对象

不过这里也有一个前提，就是不能使用严格模式，当使用严格模式时并不会进行默认绑定，而是会被绑定到undefined

2、隐式绑定

所谓隐式绑定就是对象中含有一个方法，通过这个对象来调用这个方法时，方法内部的this会被隐式绑定为该对象：

```javascript
var a = 0;
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
obj.foo(); //2 
```

这里需要注意一个隐式绑定丢失的问题，看个例子：

```javascript
var a = 0;
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2,
    foo: foo
};
var bar = obj.foo
bar()
```
这里会打印出0，当将obj.foo赋值给bar之后，再执行bar，会导致foo的隐式绑定丢失，最终会变为默认绑定。究其原因是当执行`var bar = obj.foo`时obj.foo只是一个方法的引用，最终调用取决于bar由谁调用

3、显式绑定

显式绑定是使用call或者apply来强制指定this的指向：

```javascript
var a = 0;
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
};
foo.call(obj); //2 
```

显示绑定并不能直接解决隐式丢失的问题，但是可以通过Function.prototype.bind(...)来进行硬绑定，该方法返回一个新函数，并且该函数中的this始终指向bind方法中传入的对象参数，并且可以重新绑定新的对象：

```javascript
var a = 0;
function foo() {
    console.log(this.a);
}
var obj1 = {
    a: 2
};
var obj2 = {
    a: 3
};
var bar = foo.bind(obj1); 
bar();//2
var another = foo.bind(obj2); 
another();//3
```

4、new绑定

当使用new来调用函数时，函数中的this指向的是new创建的实例：

```javascript
function Foo(a) {
    this.a = a;
}
var obj = new Foo();
console.log(a); //2 
```

### this绑定优先级

this绑定有上面4种方式，那假如当同时存在多种绑定时会以哪一个为准呢？

优先级的顺序是：new 绑定 > 显示绑定 > 隐式绑定 > 默认绑定

另外，还有几点需要注意：

1、箭头函数中没有this绑定，this的指向需要参考箭头函数的外层作用域，就好像以前常用的self = this捕获一样

2、当call、apply、bind中传入null时，该绑定会被忽略，直接采用默认绑定

3、除了上面提到的隐式丢失外，间接引用也会让this应用默认绑定：

```javascript
var a = 0;
function foo() {
    console.log(this.a);
}
var obj1 = {
    a: 2,
    foo: foo
};
var obj2 = {
    a: 3,
    foo: foo
};
(obj2.foo = obj1.foo)() //0
```
obj2.foo = obj1.foo的返回值也是单纯的foo函数引用





