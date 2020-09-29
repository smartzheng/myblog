+++
title= "webpack学习： loader"
date= "2019-08-07"
categories= [ "web前端" ]
tags=["webpack"]
+++
### 1.什么是loader

> webpack enables use of loaders to preprocess files. This allows you to bundle any static resource way beyond JavaScript. You can easily write your own loaders using Node.js.

> Loaders are activated by using loadername! prefixes in require() statements, or are automatically applied via regex from your webpack configuration

**在webpack中,通过配置loader可以对js之外的任何静态资源进行预处理**

通过Node.js的require()语句可以使用loader,不同的文件可以通过正则表达式匹配使用不同的loader来处理  

常见的loader按处理内容不同来进行分类,如处理文件,JSON,转译,模板,样式,代码检测和测试,框架等的loader


### 2.loader的使用
使用loader首先需要安装  

```
npm install loaderName --save-dev
```

然后在webpack.config.js中进行loader配置,需要注意的是use数组中有多个loader对文件进行处理时,处理顺序为从数组最后一个loader依次往前,如下例中,某个.png图片会先被url-loader处理,再被file-loader处理,常见配置如下:

```javascript
module.exports = {
  module: {
    rules: [	//rules数组配置不同文件类型(test)的情况下使用(use)指定的loader
      {
        test: /\.(png|jpe?g|gif)$/,  //需要被该loader处理的文件正则表达式
        use: [  //对匹配到正则的文件进行处理的loader数组
          {
            loader: 'file-loader',  //loader名字
            options: {  //该loader的可选配置项,如输出路径,文件名等,可接收方法
            	 name: '[path][name].[ext]'
            },  
          },
          {
            loader: 'url-loader',  
            options: {}
          }
        ]
      },
      {
        test: /\.css$/,  
        use: [  
          {
            loader: 'css-loader',  
            options: {  
              
            },  
          }
        ]
      },
    ]
  },
};
```



**在webpack官网[webpack-loaders](https://webpack.docschina.org/loaders/)列出了常见的loader,如常见的打包静态资源的loader有:**

#### 文件
* file-loader: 将文件发送到输出文件夹(options中配置name)
* url-loader:  和file-loader类似，options可以配置limit值,如文件小于该大小,会被直接处理为base64数据写入到打包生成的js文件中,这样可以减少一次请求,但是会影响js脚本加载速度,所以limit值应该设置得不要太大

#### 样式

* style-loader 将模块的导出作为样式添加到DOM中 
* css-loader 解析CSS文件后，使用import加载，并且返回CSS代码  
* less-loader 加载和转译LESS文件
* sass-loader 加载和转译SASS/SCSS文件
* stylus-loader 加载和转译Stylus文件
* postcss-loader 使用PostCSS加载和转译CSS/SSS文件

**style-loader和css-loader往往配合使用,less,sass,stylus等loader是对非.css类型样式文件做加载转译的loader,由于loader的处理顺序是从后往前,所以一般css,less等loader写在后面;**

**下面是css-loader一些常见的options配置:**
- **importLoaders: number** 假如在use数组中配置了多个loader,某些情况下可能会出现部分文件不会被后面的loader处理的情况,比如:

```
{
  test: /\.scss$/,  
  use: [  
    {
      loader: 'css-loader',  
      options: {  
         //importLoaders:2     
      },  
    },
    'sass-loader',
    'postcss-loader'
  ]
},
```

在使用@import引入scss时,被引入的scss文件可能可能不会被sass-loader,postcss-loader处理,这时指定importLoaders:2可以强制所有.scss都会被sass-loader,postcss-loader处理(实际使用上基本不会出现这种情况)

- **modules: true/false** 在引入css时,如果采用类似`import './index.css'`的方式引入css文件,那么该css文件会作用于全局;也就是说假如在`index.js`中`import './index.css'`,那么在`index.js`中引入的其他模块也会使用到`index.css`,设置modules: true并且使用`import style from './index.css'`的方式引入(style为自定义名字),在使用css样式时使用style.className的方式使用则可以避免这个问题

**postcss-loader需要新建并配合postcss.config.js使用:**

```
module.exports = {
  parser: 'sugarss',
  plugins: {//配置插件,也可以写成数组形式,见下个例子
    'postcss-import': {},
    'postcss-preset-env': {},
    'autoprefixer': {}
  }
}
```
plugins中配置postcss插件,常用的插件有autoprefixer(解决css3的兼容性):

```
module.exports = {
    plugins: [
        require('autoprefixer')({
            "browsers": [  //配置兼容的浏览器范围
                "defaults",
                "not ie < 11",
                "last 2 versions",
                "> 1%",
                "iOS 7",
                "last 3 iOS versions"
            ]
        })
    ]
};
```