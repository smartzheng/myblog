+++
title= "深入理解JavaScript之执行环境和作用域链"
date= "2019-08-28"
categories= [ "web前端" ]
tags=["JavaScript"]
+++

### 引言 

***在Java中，被花括号包起的代码具有独立的作用域，这一点与大部分语言都差不多，在理解和使用上都很简单，而JS中的作用域却相对较为复杂，例如***

```javascript
if (true) {
    a = 0
}
console.log(a)
```
***在非严格模式下，以上代码没有报错，打印出了数字0，即使在严格模式下，将`a = 0`改为`var a = 0`之后也能正确执行；而在类似的Java代码中肯定是编译不过的***

***上篇博客[深入理解JavaScript之作用域闭包](https://smartzheng.github.io/post/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3javascript%E4%B9%8B%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%97%AD%E5%8C%85/)已经详细分析了闭包，而闭包是基于JS作用域的一种特殊现象，博客中也介绍了作用域的核心原理，本文将更深入全面地分析作用域这一概念***

**先来看个简单的例子**

```javascript
var color = "blue"

function fun1() {
    var color = "red"
    console.log(color)
    return function fun2() {
        var color = "purple"
        console.log(color)
    }
}

console.log(color)
var foo = fun1()
foo()
```

这里会先后打印出blue、red、purple，对于稍有JS基础的人来讲这是显而易见的。本文的内容即是分析产生这样执行结果背后的原理

### 1. 执行环境

要理解作用域，需要先理解执行环境（execution context，上下文），只需要记住以下几点  

- 执行环境定义了变量或函数能访问的数据，每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和方法都保存在这个变量中
- 全局执行环境是所有执行环境中最外围的环境，在web浏览器中，它拥有全局变量window，所有的全局变量和函数都是window的属性  
- 每个函数都有自己的执行环境；当代码执行到一个函数时，其执行环境就会被压入环境栈，执行完毕后出栈销毁，保存在其中的变量和函数也随之销毁（[闭包](https://smartzheng.github.io/post/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3javascript%E4%B9%8B%E4%BD%9C%E7%94%A8%E5%9F%9F%E9%97%AD%E5%8C%85/)是个例外）

### 2.作用域

通常来说，一段程序代码中所用到的名字并不总是有效/可用的，而限定这个名字的可用性的代码范围就是这个名字的作用域  

也可以这么说：程序需要具备在变量中存储值和取出值的能力，而变量存取的范围拥有一套规则，我们可以称这组规则为作用域

### 3.作用域链

当创建一个函数时，会自动创建一个作用域链，并且其中预先包含所有外部的变量对象（链的最底层为全局变量），这个作用域链被保存在函数的[[Scope]]属性中

当调用该函数时，会为函数创建一个执行环境（同时创建了函数对应的活动对象，执行环境的范围也就是函数的作用域），并且复制[[Scope]]属性中的对象，作为该执行环境的作用域链；然后将函数对应的活动对象push到作用域链的前端

前面说了，作用域定义的是变量存取的规则或者可用性范围，下面通过例子来分析在代码执行时，是如何通过这个规则来查找变量的  

```javascript
var color = "blue"

function fun1() {
    var color = "red"
    console.log(color)
    return function fun2() {
        var color = "purple"
        console.log(color)
    }
}

console.log(color)
var foo = fun1()
foo()
```

是的，就是文章开始时的那个例子，来画一个图：

<img src = "https://raw.githubusercontent.com/smartzheng/images/master/blog/作用域链.png" width="60%"/>

**首先需要谨记，函数执行在它被定义所在的执行环境中，而非它自己的执行环境中**  

在上图中，作用域链1和作用域链2分别对应于fun1和fun2的作用域链，这里以fun2为例：
  
当fun2被创建时，会创建一个作用域链，它默认包含了全局变量window和外部fun1的执行环境中作用域链中的活动对象1（每一个活动对象或全局变量中都保存有自己环境中的所有变量）  

当执行fun2时，又创建了一个活动对象2，其中保存了fun2中定义的color（“red”）和默认参数arguments

在执行fun2的`console.log(color)`时，会在对应的作用域链2的活动对象2中去寻找color变量，如果找不到，则去外层环境的活动对象1中找...直到window

这里还有一个疑问，当在window中也找不到需要的变量时，会怎么样呢？看两个例子：

```javascript
function foo() {
	 a = 1
    console.log(a)
}
foo()//非严格模式下打印出 1
```

```javascript
function foo() {
    console.log(a)
}
foo()//ReferenceError
```

在上面的两段代码中，都需要使用到变量a，但是运行的结果却不一致。这里需要理解一下RHS查询和LHS查询： 

简单来讲RHS查询与简单地查找某个变量的值别无二致，而LHS查询则是试图找到变量的容器本身，从而可以对其赋值，当RHS查询不到变量时，会直接报错，而RHS查询不到值时，非严格模式的情况下，会自动在全局变量中创建一个（严格模式下也会报ReferenceError），很明显在上面的代码中，前者属于LHS，后者属于RHS。关于自动在全局变量中创建变量，这点可以通过下面几段代码做个实验：

```javascript
function foo() {
	 a = 1
}
console.log(a)//ReferenceError
```

```javascript
function foo() {
	 a = 1
}
foo()
console.log(a)//非严格模式下打印出 1
```

```javascript
function foo() {
	var a = 1
}
foo()
console.log(a)//ReferenceError
```

### 4.eval、with、try catch

除了函数，这里还有几个能够改变作用域的方法（块级作用域后面专门分析），虽然eval、with并不被推荐使用，但是最好还是有所了解

```javascript
function foo(str, a) {
    eval(str); // 欺骗!
    console.log( a, b );
}
var b = 2;
foo("var b = 3;", 1); // 1, 3
```
以上代码通过eval传入`"var b = 3;"`，使得b成为foo作用域内的变量

```javascript
function foo(obj) {
    with (obj) {
        a = 1
        var b = 0
        c = 3
    }
    console.log(b)//"0"
}
var obj = { a: 2 }
foo(obj)
console.log(obj.a)//"1"
console.log(c)//"3"
console.log(a)//"ReferenceError"
```
with新建了一个作用域，似乎可以说它的参数成为该作用域的活动变量；  

值得注意的是，在with语句中通过var声明的变量对应的作用域其实是外部方法作用域，所以例子中在with外部打印b时为3；  

其次在上述代码中，传入with的obj没有c属性，所以在进行LHS查询时，直到全局变量都没有找到c，导致在全局变量中声明了一个c，所以`console.log(c)`的结果为3

**with和eval虽然使用简洁，但是对性能的影响却很大：在编译器进行词法分析时，会对每个变量和方法进行静态处理优化，即通过设置一个标识符预先确定它们的位置，以便在运行时能快速找到它们的位置；但是一旦使用到with和eval，由于它们会在运行期改变作用域，所以编译器无法通过静态分析来预先确定作用域，无法进行这种静态处理优化，“引擎多聪明，试图将这些悲观情况的副作用限制在最小范围内，也无法避免如果没有这些优化，代码会运行得更慢这个事实。”**

catch语句则比较简单，它会创建一个新的变量对象，其中包含的是被抛出的错误对象的声明

### 5.块级作用域

在ES6之前，JS中没有块级作用域；而在其他类C语言中，由花括号封闭的代码块都有自己的作用域(如果用ECMAScript的话来讲，就是它们自己的执行环境)，因而支持根据条件来定义变量。例如，下面的代码在 JavaScript中并不会得到想象中的结果:

```javascript
if (true) {
    var color = "blue";
}
alert(color);    //"blue"
```

同样的现象还存在于for循环中：

```javascript
for (var i = 0; i < 10; i++){
	doSomething(i);
}
alert(i);      //10
```
在其他类C语言中，if语句和for循环中的i只作用于于花括号范围内，而在JS中括号内的语句则是在其外部环境中执行，定义的变量会被绑定在外部作用域(函数或全局)中，这点在我刚开始接触JS时也产生了很大的困惑，很难理解为什么要这样设计   

不过ES6新增了let和const，解决了这个问题，可能也说明设计者也认为这算是一个缺陷吧：

```javascript
for (let i=0; i < 10; i++){
	doSomething(i);
}
alert(i); //ReferenceError
```
let关键字可以将变量绑定到所在的任意作用域中，通常是在{ }中；任意位置都可以使用 {  } ，这类似Java中的代码块，并且其中的let（或const）声明的变量都会只作用在这段括号中：

```javascript
{
    let bar = 2;
}
console.log(bar); // ReferenceError! 
```

关于let、const还有一个很有趣的现象，正常情况下，typeof使用在一个未声明的变量会返回undefinde：

```javascript
console.log(typeof a)//undefinde
```
但是修改以上代码为：

```javascript
console.log(typeof a)//ReferenceError
let a = 1
```
这里却打印了ReferenceError，而使用var则没有问题，原因是在上面这种情况下，当使用let声明a时，在声明代码之前，a被放入了一个被称为暂时性死区的地方从而无法被应用到


> 参考：  
> 《JavaScript高级程序设计》  
> 《你不知道的JavaScript》上卷  
> 《深入理解ES6》  















