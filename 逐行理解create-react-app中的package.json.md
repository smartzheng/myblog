+++
title= "逐行理解create-react-app中的package.json"
date= "2019-09-30"
categories= [ "web前端" ]
tags= [ "React" ]
+++

下面是自己react项目初始化之后的package.json文件，除了个别自己新增依赖以外，基本是create-react-app生成的默认配置，下面是对package.json中每一行（除jest之外）的解释：

```json
{
  "name": "react-wanandroid", //项目名
  "version": "0.1.0", //版本号
  "private": true, //私有项目,禁止意外发布私有存储库的方法
  "dependencies": { //依赖包,在开发和线上环境均需要使用
    "axios": "^0.19.0", //网络请求
    "dotenv": "8.1.0", //管理环境变量的模块,通过配置.env文件可以控制环境变量
    "dotenv-expand": "5.1.0", //扩展环境变量
    "react": "^16.10.1",  //react核心库
    "react-app-polyfill": "^1.0.3", //官方提供的兼容库,主要用于处理IE兼容问题
    "react-dev-utils": "^9.0.4",
    "react-dom": "^16.10.1", //react核心库,处理虚拟DOM渲染等功能
    "react-loadable": "^5.5.0", //组件按需加载,可以提高首次加载速度
    "react-router-dom": "^5.1.1", //路由
  },
  "devDependencies": { //只在开发环境存在的依赖
    "@babel/core": "7.6.2", //babel核心;babel是一个主要(不仅仅只是)用于将ECMAScript2015+版本的代码转换为向后兼容的JavaScript语法的工具链
    "@hot-loader/react-dom": "^16.9.0", //热更新,提高开发效率
    "@svgr/webpack": "4.3.3", //处理svg格式文件的webpack loader
    "@typescript-eslint/eslint-plugin": "^2.3.1",//ts语法审查
    "@typescript-eslint/parser": "^2.3.1", //ts解析器
    "acorn": "^7.1.0",
    "babel-eslint": "10.0.3", //对所有有效的babel代码进行lint处理
    "babel-jest": "^24.9.0", //转换jest代码
    "babel-loader": "8.0.6", //对js|mjs|jsx|ts|tsx处理的webpack loader
    "babel-plugin-named-asset-import": "^0.3.4", //babel-loader中的文件导入处理plugin
    "babel-preset-react-app": "^9.0.2", //按照react-app的模式配置babel,下面的babel项中可以配置详细参数(指定typescript语法等)
    "camelcase": "^5.3.1", //驼峰命名检查
    "case-sensitive-paths-webpack-plugin": "2.2.0", //处理导入路径语法时区分大小写检查
    "css-loader": "3.2.0", //.css文件loader
    "eslint": "^6.5.0", //静态检查代码错误核心
    "eslint-config-react-app": "^5.0.2", //配置eslint为react-app模式
    "eslint-loader": "3.0.2", //静态检查代码错误loader
    "eslint-plugin-flowtype": "^3.13.0", //eslint检查的plugin
    "eslint-plugin-import": "2.18.2", //eslint检查导入语句的plugin
    "eslint-plugin-jsx-a11y": "6.2.3", //eslint检查jsx语句的plugin
    "eslint-plugin-react": "7.14.3", //eslint检查react语句的plugin
    "eslint-plugin-react-hooks": "^1.7.0", //eslint检查react-hooks的plugin
    "file-loader": "4.2.0", //文件处理默认loader,webpack中使用了oneOf,并且file-loader放最后,所以只有其他loader未处理情况下才会交由其处理 
    "fs-extra": "8.1.0", //扩展自带fs的功能
    "html-webpack-plugin": "4.0.0-beta.8", //打包html文件的webpack插件
    "http-proxy-middleware": "^0.20.0", //HTTP请求代理中间件,一般主要用于处理代理和跨域问题
    "identity-obj-proxy": "3.0.0",
    "is-wsl": "^2.1.1",
    //下面几项是关于测试的依赖
    "jest": "24.9.0",
    "jest-environment-jsdom-fourteen": "0.1.0",
    "jest-resolve": "24.9.0",
    "jest-watch-typeahead": "0.4.0",
    "mini-css-extract-plugin": "0.8.0", //压缩css文件插件
    "optimize-css-assets-webpack-plugin": "5.0.3", //优化css和资源文件插件
    "pnp-webpack-plugin": "1.5.0", //维护依赖路径,加快node_modules的查找速度
    "postcss-flexbugs-fixes": "4.1.0", //预处理css,处理flex兼容问题
    "postcss-loader": "3.0.0", //css预处理loader
    "postcss-normalize": "8.0.1", //引入normalize.css或者sanitize.css 
    "postcss-preset-env": "6.7.0", //css兼容处理
    "postcss-safe-parser": "4.0.1", //修复css语法错误
    "react-hot-loader": "^4.12.14", //热加载处理loader
    "resolve": "1.12.0", //引入node中require.resolve()方法
    "resolve-url-loader": "3.1.0", //处理url()语句的loader
    "sass-loader": "8.0.0", //处理sass文件的loader
    "semver": "6.3.0", //格式化版本号字符串
    "style-loader": "1.0.0", //处理style文件的loader
    "terser-webpack-plugin": "2.1.2", //压缩JavaScript
    "ts-pnp": "1.1.4", //和pnp-webpack-plugin类似
    "typescript": "^3.6.3", //ts
    "url-loader": "2.1.0", //将文件转换为base64格式(可以配置应用在小文件上)
    "webpack": "4.41.0", //webpack核心
    "webpack-dev-server": "3.8.1", //开发环境服务器,具有热加载功能
    "webpack-manifest-plugin": "2.1.2", //帮助打包时生成资源清单文件的插件
    "workbox-webpack-plugin": "4.3.1" //提供完善的ServiceWorker和预缓存文件清单的插件
  },
  "scripts": { //配置命令,执行npm run xxx即可运行scripts文件下对应的js文件
    "start": "node scripts/start.js",
    "build": "node scripts/build.js",
    "test": "node scripts/test.js"
  },
  "babel": { //babel配置,也可以配置在.babelrc文件
    "presets": [ 
      "react-app" //指定按照react-app的模式配置babel
    ],
    "plugins": [ 
      "react-hot-loader/babel" //热更新的babel插件
    ]
  },
  "eslintConfig": {
    "extends": "react-app" //eslint规则
  },
  "browserslist": { //浏览器兼容范围,也可以配置在.browserslistrc文件,会被Autoprefixer Babel postcss-preset-env等使用
    "production": [
      ">0.2%", //兼容市场份额在0.2%以上的浏览器
      "not dead", //在维护中
      "not op_mini all" //忽略OperaMini浏览器
    ],
    "development": [ //开发环境只需兼容以下三种浏览器的最新版本
      "last 1 chrome version",
      "last 1 firefox version",
      "last 1 safari version"
    ]
  },
  "jest": { //测试
    "roots": [
      "<rootDir>/src"
    ],
    "collectCoverageFrom": [
      "src/**/*.{js,jsx,ts,tsx}",
      "!src/**/*.d.ts"
    ],
    "setupFiles": [
      "react-app-polyfill/jsdom"
    ],
    "setupFilesAfterEnv": [],
    "testMatch": [
      "<rootDir>/src/**/__tests__/**/*.{js,jsx,ts,tsx}",
      "<rootDir>/src/**/*.{spec,test}.{js,jsx,ts,tsx}"
    ],
    "testEnvironment": "jest-environment-jsdom-fourteen",
    "transform": {
      "^.+\\.(js|jsx|ts|tsx)$": "<rootDir>/node_modules/babel-jest",
      "^.+\\.css$": "<rootDir>/config/jest/cssTransform.js",
      "^(?!.*\\.(js|jsx|ts|tsx|css|json)$)": "<rootDir>/config/jest/fileTransform.js"
    },
    "transformIgnorePatterns": [
      "[/\\\\]node_modules[/\\\\].+\\.(js|jsx|ts|tsx)$",
      "^.+\\.module\\.(css|sass|scss)$"
    ],
    "modulePaths": [],
    "moduleNameMapper": {
      "^react-native$": "react-native-web",
      "^.+\\.module\\.(css|sass|scss)$": "identity-obj-proxy"
    },
    "moduleFileExtensions": [
      "web.js",
      "js",
      "web.ts",
      "ts",
      "web.tsx",
      "tsx",
      "json",
      "web.jsx",
      "jsx",
      "node"
    ],
    "watchPlugins": [
      "jest-watch-typeahead/filename",
      "jest-watch-typeahead/testname"
    ]
  },
  
  "description": "This project was bootstrapped with Create React App.", //项目描述
  "main": "index.js", //入口文件
  "repository": { //Git仓库地址
    "type": "git",
    "url": "git+https://github.com/smartzheng/react-wanandroid.git"
  },
  "keywords": [ //密码
    "root"
  ],
  "author": "smartzheng", //作者
  "license": "ISC", //协议
  "bugs": {
    "url": "https://github.com/smartzheng/react-wanandroid/issues" //bug是上报地址
  },
  "homepage": "https://github.com/smartzheng/react-wanandroid#readme" //git项目主页
}
```