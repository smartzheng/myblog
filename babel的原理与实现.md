title= "babel的原理与实现"
date= "2022-03-05"
categories= [ "前端" ]

### 一、babel是什么？

> 新一代JavaScript编译器（转译器）

babel原名6to5，顾名思义，是将es6转为es5，但随着es7、8、9...诞生，不再适用，改名babel

babel是巴别塔的意思，来自圣经中的典故：当时人类联合起来兴建希望能通往天堂的高塔，为了阻止人类的计划，上帝让人类说不同的语言，使人类相互之间不能沟通，计划因此失败，人类自此各散东西。此事件，为世上出现不同语言和种族提供解释。这座塔就是巴别塔。

### 二、babel有什么用？

> 为什么要用babel？

- 语法转译：将 esnext、typescript、flow 等转译到目标环境支持的 js
- 代码静态分享：linter、文档生成、压缩混淆
- 函数插桩（函数中自动插入一些代码，例如埋点代码）、自动国际化等特定用途代码的转化；小程序转译工具 taro，就是基于 babel 的 api 来实现的

### 三、babel如何工作？

#### 编译原理回顾

熟悉编译原理的同学知道，编译的流程大致可以分为以下的步骤：

![img](https://static001.geekbang.org/resource/image/06/93/06b80f8484f4d88c6510213eb27f2093.jpg style="zoom:60%;")

词法分析，就是指将代码进行分词，比如 `let name = 'smart';` 这样一段源码，我们要先把它分成一个个不能细分的单词（token），也就是 `let`, `name`, `=`, `'smart'`，一个简易的词法分析器的原理大概是这样的（有限自动机）：

<img src="https://static001.geekbang.org/resource/image/6d/7e/6d78396e6426d0ad5c5230203d17da7e.jpg" alt="img" style="zoom:60%;" />

词法分析是识别一个个的单词，而语法分析就是在词法分析的基础上识别出程序的语法结构，举个🌰，“我喜欢又聪明又勇敢的你”，可以这样表示：

<img src="https://static001.geekbang.org/resource/image/93/fb/9380037e2d2c2c2a8ff50f1367ff37fb.jpg" alt="img" style="zoom:25%;" />

我们可以通过遍历词法分析生成的token，当遇到不同的关键字时，依次生成一个树状的语法树，这样的算法叫做递归下降算法，V8编译器也是使用的这个算法。

<img src="https://static001.geekbang.org/resource/image/cb/16/cbf2b953cb84ef30b154470804262c16.jpg" alt="img" style="zoom:70%;" />

语义分析，则是分析整个句子在上下文中的含义，让计算机真正理解这段代码的用途，比如：在语法分析阶段，对于`int b = a + 3`这样一条语句，无论 a 是否提前声明过，在语法上都是正确的。而在实际的计算机语言中，如果引用某个变量，这个变量就必须是已经声明过的。同时，当前这行代码，要处于变量 a 的**作用域**中才行，这就是语义分析中的引用消解，除此之外还有闭包分析、控制流检查也都是在语义分析阶段。

<img src="https://static001.geekbang.org/resource/image/50/03/5092c6f4103a967ea0609dceea873d03.jpg" alt="img" style="zoom:27%;" />

#### AST

在梳理babel工作流程之前，我们先再了解一下AST（abstract syntax tree），即抽象语法树。

不同的语法分析器有不同的标准，JS parser 的 AST 大多是 [estree 标准](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Festree%2Festree)。我们可以通过[AST explorer](https://astexplorer.net/)来实际体验一下。

babel生成的语法树节点主要有这些分类：标识符 Identifer、各种字面量 xxLiteral、各种语句 xxStatement，各种声明语句 xxDeclaration，各种表达式 xxExpression，以及 Class、Modules、File、Program、Directive、Comment。具体可以参考[estree 标准](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Festree%2Festree) 标准。这里对常用的Literal和Declaration做一个示意：

<img src="https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29185815036a4ea1878484ba773a3b6e~tplv-k3u1fbpfcp-watermark.awebp" alt="img" style="zoom:20%;" /> <img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5303fa5530944a638d6b3d1af93f0e3f~tplv-k3u1fbpfcp-watermark.awebp" alt="img" style="zoom:30%;" />



#### babel工作流程

同样的，babel的工作原理也是类似的，主要包含了代码的解析（parse）、转换（transform）、生成（generate）三个阶段，如下图：

![img](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee9eaa1f265c4c49ad156f2c691748d9~tplv-k3u1fbpfcp-watermark.awebp)

> 相比于一门语言的编译器而言，babel应该只能算是一个转译器，因为编译器做的事往往是把高级语言编译成字节码、机器码等更低级的语言，而且最难的点在于进行语义分析，这点babel基本没有涉及；而babel更多做的是一个转译工作，往往只需要对语法分析的产物进行修改，再重新生成js代码即可，比如将ts、flow转换为js（或者说，我们通过对babel生成的AST进行分析，算是做了一点语义分析的工作？）。并且babel的语法分析功能也是依赖于开源项目acorn实现，相对于实现一个编译器而言要简单许多。

这三步主要做了以下事情：

- parse：通过 parser 把源码转成抽象语法树（**AST**）

<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/03bdbe8096944a0fa09c86ac2ff09e56~tplv-k3u1fbpfcp-watermark.awebp" alt="img" style="zoom:27%;" />

- transform：遍历 AST，调用各种 transform 插件对 AST 进行增删改

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/494b0bc006f64c71a92947f560e97e8c~tplv-k3u1fbpfcp-watermark.awebp" alt="img" style="zoom:33%;" />

- generate：把转换后的 AST 打印成目标代码，并生成 sourcemap

<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/84530b477a7540ee87e5bb12e9375569~tplv-k3u1fbpfcp-watermark.awebp" alt="img" style="zoom:33%;" />

#### API



#### 插件

babel提供了基础的解析、遍历、转换，以及大量的工具API，但是我们要实现对代码的修改，需要依赖于插件，这是babel应用的核心要素。我们可以通过一个例子来简单看一下

#### 其他

- Vue的编译

### 四、babel如何实现？

#### parser

#### tranverse

- path

- scope

#### generater

#### core

#### cli





