+++
title= "webpack学习：进阶知识点"
date= "2019-08-08"
categories= [ "web前端" ]
tags=["webpack"]
+++


### 1.Tree Shaking
移除JavaScript上下文中的未引用代码(dead-code)，需要将mode选项设置为production开启，且只支持ES Module  
配置：  

- package.json->useExports:true/false true为开启  
- sideEffects:false/[]，false为对所有文件都开启Tree Shaking，但在导入时会执行特殊行为的代码，而不是仅仅暴露一个export或多个export。例如`import '@bable/poly-fill'`，它影响全局作用域，并且通常不提供export。这时需要手动加到sideEffects数组中防止被删除。

### 2.codeSplitting   

#### js代码分割
概念：代码分割，与webpack无关，但webpack可以帮助开发者自动进行代码分割  
两种方式：  
1.optimization配置   
2.异步import会自动被代码分割   
webpack中结合SplitChunksPlugin实现代码分割  

**为什么webpack中splitChunks默认只针对async引入的代码？**   
js优化两个方向：缓存和代码使用率  
缓存: splitChunks:'all' 相同chunk代码不重复加载；提升较小，因为只能提高第二次访问时的速度  
代码使用（覆盖）率（coverage）: preLoading（初始页面同时预加载）、preFetching（网络空闲预加载） 提升较大 

#### CSS代码优化
分割：MiniCssExtractPlugin(不支持HMR，适合使用在production)  
压缩：OptimizeCSSAssetsPlugin

### 3.打包分析常用工具  
webpack/analyse，bundle analysis


### 4.webpack与浏览器缓存
由于浏览器缓存的原因，若每次打包生成的文件名都一样，那有可能新代码上线后浏览器还是访问的本地缓存，可以output配置name和chunkFilename时加上contenthash标识：

```
output: {
	filename: '[name].[contenthash].js',
	chunkFilename: '[name].[contenthash].js'
}
```

### 5.Shimming
更改部分代码的默认行为  
如：modules为true的情况下，每个模块都有自己的运行环境，被引入的module无法使用引入该模块的module中导入的变量，在plugins中配置webpack.ProviderPlugin可以实现插入变量的功能。  
举例说明：  
a.js引入b.js，a.js中import了jquery，b中需要使用到jquery，正常模式下必须在b中再次导入jquery，但是针对于某些第三方模块，我们无法去改变它的默认代码，这个时候可以在webpack.config.js中配置

```
plugins:[
	new webpack.ProvidePlugin({
		$: 'jquery'
	}),
]
```
这样，被引入的模块中的$都能在不用添加导入jquery代码而被正确使用。

### 6.Library基本配置


```javascript

...
externals:'lodash',
output:{
	...
	libraryTarget：'umd',//window、this、global
	library:'libraryName'
}
```
externals代表被引入使用时定义的名字，libraryTarget中，umd代表打包生成的库可以通过各种方式引入（import、require等），配置为window、this、global则将该库的导出变量作为libraryTarget的一个变量



























