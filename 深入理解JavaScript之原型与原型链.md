+++
title= "深入理解JavaScript之原型与原型链"
date= "2019-09-29"
categories= [ "web前端" ]
tags= [ "JavaScript" ]
+++  

**JavaScript是一门面向对象的语言，继承是面向对象的一大特性，但是严格来讲JavaScript中却没有通常含义上的继承，只能模拟继承，即使ES6之后有了class，其实现和其他面向对象语言依然有本质不同。而JavaScript中模拟继承的方式则是通过原型链**

1、什么是原型和原型链？

JavaScript中，对象实例在创建的时候会关联一个[[Prototype]]属性，即对象的原型

ES6之前标准并没有定义获取原型对象的方法，但是基本所有浏览器都支持通过`__proto__`来获取，而ES6之后则提供了Object.getPrototypeOf()方法

当我们对对象进行“对象.属性名”操作时，如：

```javascript
var a = {
	name = 'smartzheng';
}  

console.log(a.name);
```
这里会调用a对象的[[Get]]属性，如果a对象含有name属性，那么a.name直接返回这个属性，如果a中没有该属性的话就会使用到原型了：会在`a.__proto__`得到的原型对象中去寻找name属性，如果还找不到，就会在`a.__proto__.__proto__`（如果有的话）中去找...当然这也有一个终点，也就是Object.prototype：这就形成了原型链

2、原型链和原型继承

继续详细分析一下JavaScript中的原型和原型链

我们知道，创建对象常用的有3种方式：字面量，new和Object.create()；上面说到，对象在创建的时候会有一个[[Prototype]]属性，它其实是对另一个对象的引用，我们称之为该实例的原型对象

分别来分析一下前面提到的三种方法创建对象之后默认关联的原型对象：

1）字面量形式

```javascript
var a = {};  
console.log(a.__proto__ === Object.prototype); //true  
console.log(a.__proto__);
```
下面是`a.__proto__`打印出的结果：

```json
{
	constructor: ƒ, 
	__defineGetter__: ƒ,
	 __defineSetter__: ƒ,   
	hasOwnProperty: ƒ, 
	__lookupGetter__: ƒ,
	…
}
```

展开下一级之后为：

```json
{
	constructor: ƒ Object()
	hasOwnProperty: ƒ hasOwnProperty()
	isPrototypeOf: ƒ isPrototypeOf()
	propertyIsEnumerable: ƒ propertyIsEnumerable()
	toLocaleString: ƒ toLocaleString()
	toString: ƒ toString()
	valueOf: ƒ valueOf()
	__defineGetter__: ƒ __defineGetter__()
	__defineSetter__: ƒ __defineSetter__()
	__lookupGetter__: ƒ __lookupGetter__()
	__lookupSetter__: ƒ __lookupSetter__()
	get __proto__: ƒ __proto__()
	set __proto__: ƒ __proto__()
}
```
这里可以发现几个熟悉的方法，如valueOf、toString、toLocaleString；其实这些都是Object的prototype中的方法，而我们使用字面量方式创建的对象实际上在创建时也默认将Object.prototype赋值给了它的`__proto__`，所以`a.__proto__ === Object.prototype`为true，这也是为什么所有对象都默认可以使用前面提到的几个方法的原因

2）new

```javascript
var a = new Object();  
console.log(a.__proto__ === Object.prototype); //true
```
显然，new和字面量创建对象一样，都是将Object.prototype赋值给了实例的原型对象

3）Object.create()

```javascript
var a = Object.create(Object.prototype);  
console.log(Object.getPrototypeOf(a) === Object.prototype); //true
```
可见，Object.create()是将传入对象直接赋值给了新实例的原型对象属性

通过上面的分析可以看出，三种方式创建对象都为实例默认关联了一个原型对象，通过\_\_proto__或者Object.getPrototypeOf()可以获取，如果是字面量模式，则新实例的原型对象即Object.prototype，但是new和Object.create()的话则可以自定义默认关联的原型对象：

```javascript
function Foo(){
	//...
}  
var f1 = new Foo();  
var f2 = Object.create(Foo.prototype);
```

3、“类”、“构造函数”和“继承”

最后再看一个全面的例子:

```javascript
function Foo(){
	//...
}
var f1 = new Foo();   
var f2 = Object.create(Foo.prototype);  
console.log(Foo.prototype) //{constructor: ƒ Foo(), __proto__: Object}  
console.log(Foo.prototype.__proto__ === Object.prototype); //true  
console.log(Object.getPrototypeOf(f1) === Foo.prototype); //true  
console.log(Object.getPrototypeOf(f2) === Foo.prototype); //true  

```
在JavaScript中经常把首字母大写的function称之为“类”，例如上面的Foo，这里Foo.prototype包含两个属性，constructor属性指向Foo()自身，当我们使用new调用Foo()时，实际上执行了以下几步：  
1）创建一个新对象：  

```javascript
const f1 = {}  
```  
2）设置新对象的constructor属性为构造函数的名称，设置新对象的\_\_proto\_\_属性指向构造函数的prototype对象：

```javascript
f1.constructor = Foo;     
f1.__proto__ = Foo.prototype  
```
 
3）使用新对象调用函数，函数中的this被指向新实例对象：

```javascript
Foo.call(f1) 
```  
4）将初始化完毕的新对象地址，保存到等号左边的变量中

很多人习惯称Foo为一个类，它的实例f1的原型对象对Foo的prototype，Foo也有一个\_\_proto\_\_属性，指向的是Object.prototype，根据前面的逻辑，在f1若找不到某个属性，则会在f1.\_\_proto\_\_，也就是Foo.prototype中寻找该属性，如果从f1.\_\_proto\_\_中取不到对应的属性，则会在f1.\_\_proto\_\_.\_\_proto\_\_即Object.prototype中寻找该属性，若还找不到，则返回undefined

这一表现确实和继承很像：有一个顶级的父类Object，所有对象都是它的实例或者它子类的实例，所有这些实例都可以使用这个顶级父类中定义的方法，比如上面的Foo可以理解成继承自Object，其实例f1可以使用Foo.prototype中定义的属性，如果找不到则使用父类Object.prototype中的属性；这虽然看起来很像面向对象编程中的继承，但是JavaScript中实际上没有继承（实际上从new的工作原理看来JavaScript也算不上有类，并没有类实例的拷贝操作），只是通过原型链的方式实现了类似继承的表现

在Java中，所有类都继承自Object，所有的类的实例对象都可以使用Object中的方法，比如hashCode，equals等，但是与JavaScript不同的是，在Java中，继承的含义是指每一个子类中都有父类方法和属性的一份拷贝，在父类中定义了一个方法，子类可以直接使用也可以重写覆盖，但是子类都是调用的自己的属性和方法

而在JavaScript中其实只是建立了对象之间的关联，当在对象自己的属性中找不到想要获取的属性时，就会去关联在它的`__proto__`属性上的对象中去找，更像是一种委托

4、总结

下面这张图很形象的说明了文中各个角色的关系：

<img src = "https://raw.githubusercontent.com/smartzheng/images/master/%E5%8E%9F%E5%9E%8B%E5%92%8C%E5%8E%9F%E5%9E%8B%E9%93%BE.png" width="80%"/>


1）每一个函数通过new会创建一个以该函数的prototype为原型对象的实例，可以通过对象的\_\_proto__属性获取该原型对象  
2）函数的prototype中有两个属性，constructor指向自身，具体调用时机和方式参见上文关于new的分析，另一个属性\_\_proto__指向上一级关联的原型对象（最高为Object.prototype）
3）当实例触发[[Get]]操作时首先从自己内部查找属性，若找不到则依次通过\_\_proto__往上找，从而形成了原型链





