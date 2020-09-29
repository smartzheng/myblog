+++
title= "深入理解JavaScript之作用域闭包"
date= "2019-08-23"
categories= [ "web前端" ]
tags= [ "JavaScript" ]
+++
***在Java中，由于1.8之前函数并不能被当做参数传递，而且Java中变量声明可以看做是类似于ES6中的let const，自动拥有块级作用域，所以闭包在Java中并不是一个很需要讨论的问题，虽然Java开发（尤其是Android开发）中内存泄漏的原因和JS中因为闭包导致的内存泄漏很类似。***  

***而JavaScript中，闭包无处不在，不过鲜有开发者能透彻地领悟它的意义。之前学习JS时，参考的是《JavaScript高级程序设计》，关于闭包，这本书讲的不可谓不深入，但总感觉理解的不是那么全面，似懂非懂，最近又仔细结合《你不知道的JavaScript》进一步加深了理解。***

**何为闭包**

先来看这个来自《JavaScript高级程序设计》中的例子：

``` javascript
function createComparisonFunction(propertyName) { //函数声明在全局环境中
    return function (object1, object2) {
        var value1 = object1[propertyName];
        var value2 = object2[propertyName];
        if (value1 < value2) {
            return -1;
        } else if (value1 > value2) {
            return 1;
        } else {
            return 0;
        }
    };
}
```
在上面代码中，方法createComparisonFunction返回了一个匿名方法，匿名方法中使用到了传入createComparisonFunction的参数propertyName，也就是等于说内部的匿名函数访问了外部函数中的变量。  
之所以能访问外部作用域中的变量，是因为该函数的作用域链中包含了外部函数的作用域，当自身作用域中找不到该变量名对应的变量时，会在其上一级作用域中查找，以此往复，这一点熟悉作用域链的人都知道，不过这里还是细讲一下：  

- 每个执行环境都会包含一个表示变量的对象，对于全局环境而言，也就是全局变量对象（window），而函数执行环境中同样也包含一个活动的变量对象（简称活动对象，之所以是活动的，是因为正常情况下它在函数执行完后会被销毁）用于表示该执行环境中的变量  
- 当在全局环境中创建一个函数时，会自动创建一个作用域链，并且其中预先包含全局变量对象，这个作用域链被保存在函数的[[Scope]]属性中
- 当调用该函数时，会为函数创建一个执行环境（同时创建了函数对应的活动对象，执行环境的范围也就是函数的作用域），并且复制[[Scope]]属性中的对象，作为该执行环境的作用域链；然后将函数对应的活动对象push到作用域链的前端

**有几个需要注意的概念：变量对象、作用域、作用域链、执行环境**  

看个例子：

```javascript
function compare(value1, value2){
    if (value1 < value2){
        return -1;
    } else if (value1 > value2){
        return 1;
    } else {
		 return 0; 
	}
}
var result = compare(5, 10);
```
根据以上规则，我们可以这么去解释上面的代码:  

- 在全局环境中有个全局变量对象window，它包含了value1、compare、result的引用；
- 在声明compare方法时，创建了一个作用域链，其中预先包含了全局变量对象，它被保存到了compare的[[Scope]]属性中
- 调用`compare(5, 10)`时，会为compare函数创建一个执行环境，并且创建一个活动对象用以保存此执行环境中的变量（即代码中的），复制compare的[[Scope]]属性对应的作用域链作为该执行环境的作用域链，并且将活动对象推入作用域链的前端
- 当函数执行时，当需要使用到某一个变量时，会在函数的作用域链中去查找该变量，首先会查找作用域链前端的活动对象中是否包含该变量的引用，如果没有在继续查找作用域链的下一层（这里即全局变量对象），以次类推


正常情况下，函数执行完毕后，其执行环境的作用域会被销毁，活动对象也应该被销毁。但是在闭包中，表现却有所差异；接下来，我们在全局环境中调用createComparisonFunction函数：

```javascript
 var compare = createComparisonFunction("name");
 var result = compare({ name: "Nicholas" }, { name: "Greg" });
```
这里的compare被赋值为createComparisonFunction方法中的匿名函数，在执行`compare({ name: "Nicholas" }, { name: "Greg" })`时，compare函数执行在全局作用域中，但compare函数内部访问了createComparisonFunction函数作用域下的变量propertyName（即第一行传入的"name"）；同时，compare保存了createComparisonFunction函数作用域中的变量；这就出现了闭包。  

一个完整的定义：**当函数可以记住并访问所在的词法作用域，即使函数实在当前词法作用域之外执行，这时就产生了闭包**（《你不知道的JavaScript上卷》）  

在上面的例子中，在全局环境下调用compare方法，实际上是把createComparisonFunction内部的匿名函数对象的引用赋值给了window，window不会被销毁，因此它持有的compare不会被销毁，所以createComparisonFunction执行完毕后，compare中使用到的propertyName同样不会被销毁。可以手动将compare置为null来让垃圾回收器回收

**以上的解释主要来自《JavaScript高级程序设计》，直观的看，闭包似乎就是在函数中返回函数，然后在别的地方调用该函数，这个说法不可谓不对，但并不全面。下面结合《你不知道的JavaScript》来做进一步解释，可能会更理解为什么说闭包无处不在**  
其次，个人感觉《JavaScript高级程序设计》中的闭包例子有点绕，这里附一段更简单闭包代码：

```javascript
function foo() {
  var a = 2;
  function bar() {
    console.log(a);
  }
  return bar;
}
var baz = foo();
baz(); // 2 —— 这就是闭包的效果,foo中的a被保存了
```

好了，下面再看个IIFE的例子

```javascript
var a = 2;
(function IIFE() {
  console.log(a);
})();
```
这是闭包吗？ 
 
从技术层面讲，它确实创建了闭包，因为**函数IIFE可以记住并访问所在的词法作用域**，但是IIFE并不是在自身的作用域外执行，所以，严格来讲，这并不是闭包  

再看个更熟悉的例子：  

```javascript
function wait(message) {
  setTimeout(function timer() {
    console.log(message);
  }, 1000);
}
wait("Hello World");
```
在wait函数内部，将timer函数作为参数传递给setTimeout方法，timer中保存了外部作用域中的message，并且timer函数最终是被引擎调用，它执行所在的作用域并不是wait下的作用域，所以显然这里产生了闭包。

**如果将函数(函数访问了它们各自的词法作用域)当做第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、Ajax 请求、跨窗口通信、Web Workers或者任何其他的异步(或者同步)任务中，只要使用了回调函数，实际上就是在使用闭包!**（《你不知道的JavaScript上卷》）

***理解了闭包之后，接下来再看一下闭包与循环的问题***

```javascript
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i);
  }, i * 1000);
}
```

类似的代码在Java或其他类C语言中，都会每隔一秒顺序打印出1到5，但在这里却输出了5个6，这也是闭包所致  

在5次for循环中，其实依次创建了5个timer，由于闭包的存在，每个timer中都保存了所在作用域中的i变量，并且它们都是同一个变量，for循环结束后i为6，此时每个timer中保存的值也都变为了6  

那为什么在Java中不会出现这种情况呢？原因是Java中的变量默认带有块级作用域，等于是说循环中的i与timer中的i其实是两个变量，JS中可以这样实现同样的效果：

```javascript
for (var i = 1; i <= 5; i++) {
  (function() {	
    var i = i;
    setTimeout(function timer() {
      console.log(i);
    }, i * 1000);
  })();
}
```
在for循环中通过一个IIFE创建一个新的作用域，那么该作用域中的i也就与外部作用域中的i无关了，便于理解，可以把内部的i换为j：

```javascript
for (var i = 1; i <= 5; i++) {
  (function() {
    var j = i;
    setTimeout(function timer() {
      console.log(j);
    }, j * 1000);
  })();
}
```
i和j是两个无关变量，timer中保存的j只是被赋值为每一次for循环对应的i的值  

上面的写法是对块级作用域的模仿，ES6中的let也提供块级作用域：

```javascript
for (var i = 1; i <= 5; i++) {
  let j = i; // 闭包的块作用域!
  setTimeout(function timer() {
    console.log(j);
  }, j * 1000);
}
```
或

```javascript
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(j);
  }, j * 1000);
}
```






