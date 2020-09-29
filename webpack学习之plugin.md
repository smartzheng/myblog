
+++
title= "Webpack学习：plugin"
date= "2019-08-07"
categories= [ "web前端" ]
tags=["webpack"]
+++


### 1.plugin和loader的区别

#### loader:
webpack只能理解JavaScript和JSON文件。loader让webpack能够去处理其他类型的文件，并将它们转换为有效 模块，以供应用程序使用，以及被添加到依赖图中。

思考如下webpack.config.js配置:

```JavaScript
const path = require('path');

module.exports = {
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

```
它告诉webpack编译器在打包过程中，碰到「在 require()/import语句中被解析为'.txt'的路径」时，在对它打包之前，先使用raw-loader转换一下。”

#### plugin
loader用于转换某些类型的模块，而插件则可以用于执行范围更广的任务。包括：打包优化，资源管理，注入环境变量。

思考如下webpack.config.js配置:

```
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  module: {
    rules: []
  },
  plugins: [
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};
```
它告诉webpack编译器在打包过程中需要使用到plugin，而什么时候使用什么plugin，则根据plugin的具体实现而定  

**总结：
loader用于对模块源码的转换，定义webpack如何处理非javascript模块，并且在buld中引入这些依赖。loader可以将文件从不同的语言（如TypeScript）转换为JavaScript，或者将内联图像转换为data URL。而plugin可以解决loader无法实现的其他事，从打包优化和压缩，到重新定义环境变量，功能强大到可以用来处理各种各样的任务。**

### 2.使用
以HtmlWebpackPlugin为例，当我们只使用loader打包时，打包出的文件夹（假设为输出路径名为dist）里并不含有html文件，必须手动复制到dist文件夹下，并且手动更改引入的js文件为打包生成的js文件。而HtmlWebpackPlugin可以自动处理这个问题：

```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); // 通过 npm 安装
const webpack = require('webpack'); // 用于访问内置插件

module.exports = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js'
  },
  plugins: [
    new HtmlWebpackPlugin()
  ]
};
```
执行打包后就会在dist目录下生成index.html：

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <title>webpack App</title>
  </head>
  <body>
    <script src="bundle.js"></script>
  </body>
</html>
```
如果希望使用src下自己的index.html，可以配置：

```html
module.exports = {
  entry: 'index.js',
  output: {
    path: path.resolve(__dirname, './dist'),
    filename: 'bundle.js'
  },
  plugins: [new HtmlWebpackPlugin({  
      filename: 'bundle.html',//自定义输出html的名字
      template: 'src/index.html'//以自己的html文件为模板    
  })]
};
```
该配置代表，生成之后的文件名为bundle.html，它的内容等于在src/index.html文件中自动插入<script src="bundle.js"></script>的结果
往往一个插件的配置项有很多，可以在其官方地址查看，HtmlWebpackPlugin的options文档地址为：https://github.com/jantimon/html-webpack-plugin#options  

[这里](https://webpack.docschina.org/plugins/)列举了很多webpack官方推荐的plugin，常用的除了上面的html-webpack-plugin，还有：

- clean-webpack-plugin：每次构建前清理/dist文件夹，防止新旧打包生成的代码混合
- webpack.HotModuleReplacementPlugin：webpack自带，热替换插件  
- uglifyjs-webpack-plugin：js代码压缩  
- optimize-css-assets-webpack-plugin：CSS压缩优化  
- mini-css-extract-plugin：将CSS提取为独立的文件的插件，对每个包含css的js文件都会创建一个CSS文件  