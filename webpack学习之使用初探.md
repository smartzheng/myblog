+++
title= "webpack学习：使用初探"
date= "2019-08-05"
categories= [ "web前端" ]
tags=["webpack"]
+++

### 1.webpack是什么，解决了什么问题
模块打包工具,能简化复杂的模块引入,解决模块之间的相互依赖的问题.

### 2.模块是什么,如何使用模块  
module:JS借鉴其他面向对象语言实现的一种文件分组机制,将大程序拆分成互相依赖的小文件,能提高代码的可维护性(解耦和复用).  

**常见规范**：  

*  CommonJS(NodeJS):通用模块规范,主要由NodeJS具体实现,同步加载,不适用客户端  
*  AMD(RequireJS): Asynchronous Module Definition异步加载某模块的规范
*  CMD(SeaJS):Common Module Definition通用模块定义,由国内发展出来,SeaJS是其典型代表;模块内部的依赖在需要引入的时候再引入(懒加载)  
*  UMD:是在当前JS执行环境中对以上几种模块规范定义的define,module.exports等进行判断,根据不同场所返回不同结果
*  ES6模块(import,export):ES6在语言标准的层面上,实现了模块功能,而且实现得相当简单,完全可以取代CommonJS和AMD规范,是浏览器和服务器通用的模块解决方案。

### 3.webpack使用初探

新建空项目,在项目根目录依次执行以下命令初始化npm及webpack,最好选择非全局安装,因为有可能会有不同项目使用不同版本webpack的需求  

```
npm init  
npm install webpack  -D
npm install webpack-cli -D
```

新建src目录,在下面新建以下文件  

**index.html**  

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>webpack</title>
</head>
<body>
<div id="root">

</div>

<script src="./index.js"></script>
</body>
</html>

```

**index.js**

```javascript
import hello from "./module";

hello();
```

**module.js**

```javascript
const hello = () => {
    let div = document.createElement('div');
    div.innerHTML = 'Hello';
    document.getElementById('root').append(div)
};

export default hello;
```

我们希望在打开index.html时在浏览器显示'hello',但实际上运行时是会报Unexpected identifier错的,因为浏览器并不能识别index.js中的hello,也就是无法处理模块之间的依赖导入.这时我们需要使用webpack进行打包处理(也可以在引入index.js时加上type="module"使浏览器识别module,不过webpack的作用远不止于此,这里只是用处理模块依赖简单使用webpack)  

webpack会有自己的默认配置,可以在项目根目录新建webpack.config.js文件,webpack会自动更换为根据该配置文件进行打包.下面为简单配置:

```javascript
const path = require('path');

module.exports = {
    mode: 'development',//打包模式,默认为production,development不会对打包出的js进行压缩
    entry: {
        main: './src/index.js',//打包入口文件
    },
    output: {
        filename: "bundle.js",//输出文件
        path: path.resolve(__dirname, 'dist')//输出地址和文件名
    }
};

```

这里我们指定了webpack打包的模式,入口文件,输出文件地址和文件名;执行以下命令即可完成打包:

```
npx webpack
```
由于是非全局安装,所以需要加上npx,可以在package.json文件中配置scripts:  

```
"scripts": {
    "bundle": "webpack"
},
```

执行

```
npm run bundle
```
即可完成打包,当然也可以配置成其他命令名,由于配置的scripts会在node_modules去寻找命令,所以不再需要添加npx  

打包完成后,会在根目录生成dist文件夹,里面有一个bundle.js文件,将index.html拷贝到dist,修改引入的script为bundle.js,在浏览器打开,发现已经正常显示hello
