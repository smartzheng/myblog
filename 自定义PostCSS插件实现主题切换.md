+++

title= "自定义PostCSS插件实现主题切换"
date= "2021-03-14"
categories= [ "前端" ]

+++

对于主题切换这一话题，社区上介绍的方案往往通过` CSS` 变量（`CSS` 自定义属性）来实现，但其自动化程度以及可维护性都较差。

`PostCSS` 可以接收一个` CSS` 文件，并提供了插件机制，提供给开发者分析、修改` CSS `的规则，具体实现方式也是基于 AST 技术，利用这一特点，我们可以实现一套更为自动化的主题切换功能。

假设我们这样编写`CSS`（定义一个`CSS`方法`cc`，下文会解释其作用）：

```css
a {
    color: cc(G01);
}
```

假设：

- 借助`PostCSS`的能力，根据不同的主题，最终输出不同的CSS样式，比如日间模式a标签的色值为`#eee`，而夜间模式色值为`#111`。

- 这里我们可以通过在`HTML`根节点加上属性选择器`data-theme='dark'`来动态改变当前页面主题是否为夜间主题。

所以需要做到的是，我们最终需要生成一份如下的`CSS`样式：

```css
a {
    color: #eee;
}

html[data-theme="dark"] a {
    color: #111;
}
```

实现步骤大致如下：

- 首先编写一个名为 `postcss-theme-colors` 的` PostCSS` 插件，实现上述编译过程。

- 维护一个色值，结合上例（这里以 `JSON `格式为例）就是：

```json
{
  C01: '#eee',
  C02: '#111'
}
```

`postcss-theme-colors` 需要：

- 识别`cc()`方法；

- 读取色值；

- 通过色值，对`cc()`方法求值，得到两种颜色，分别对应 dark 和 light 模式；

- 原地编译 `CSS` 中的颜色为日间模式色值；

- 同时将` dark `模式色值写到` HTML` 节点上。（通过`PostCSS Nested`或者`PostCSS Nesting`插件完成）

先简要介绍一下`PostCSS`的原理：

`PostCSS`自身只包括了`CSS分析器`，`CSS节点树API`，`source map生成器`，`CSS节点拼接器`，而基于`PostCSS`的插件都是使用了`CSS节点树API`来实现的。

我们都知道CSS的组成如下：

```
element {
  prop1 : rule1 rule2 ...;
  prop2 : rule1 rule2 ...;
  prop2 : rule1 rule2 ...;
  ...
}
```

也就是一条一条的样式规则组成，每一条样式规则包含一个或多个属性跟值。所以`PostCSS`的执行过程大致如下：

1. `Parser` 利用`CSS分析器`读取CSS字符内容，得到一个完整的`节点树`
2. `Plugin` 对上面拿到的`节点树`利用`CSS节点树API`进行一系列的转换操作
3. `Plugin` 利用`CSS节点拼接器`将上面转换之后的节点树重新组成CSS字符
4. `Stringifier` 在上面转换期间可利用`source map生成器`表明转换前后字符的对应关系

`PostCss`插件要做的就是拿到节点树上的CSS属性声明，通过转换拼接为新的CSS字符串；这里我们需要的功能的编写方式如下，可以参考 [PostCSS 8 插件](https://www.w3ctech.com/topic/2226)：

```javascript
 module.exports = (opts = {}) => {
   return {
     postcssPlugin: 'postcss-dark-theme-class', // 插件名
     Once (root, { result }) { // root为根节点树，Once方法会在该节点下的所有子元素被处理之前调用
        root.walkDecls(decl=>{...}) // 遍历CSS声明
     }
   }
 }
 module.exports.postcss = true // 声明导出为postcss插件
```

`post-theme-color`实现如下：

```javascript
const postcss = require('postcss')

const defaults = {
  function: 'cc', // 自定义CSS方法名
  groups: {}, // 存储色值分组
  colors: {}, // 存储所有色值
  useCustomProperties: false,// 是否使用自定义属性
  darkThemeSelector: 'html[data-theme="dark"]', // 夜间模式选择器
  nestingPlugin: null // 添加选择器的插件
}

/**
 * 计算最终色值
 * @param options
 * @param theme
 * @param group
 * @param defaultValue
 * @returns {string|*}
 */
const resolveColor = (options, theme, group, defaultValue) => {
  const [lightColor, darkColor] = options.groups[group] || []
  const color = theme === 'dark' ? darkColor : lightColor
  if (!color) {
    return defaultValue
  }
  if (options.useCustomProperties) {
    return color.startsWith('--') ? `var(${color})` : `var(--${color})`
  }
  return options.colors[color] || defaultValue
}

// 导出插件
module.exports = options => {
  options = Object.assign({}, defaults, options)
  // 获取色值函数（默认为 cc()）
  const reGroup = new RegExp(`\\b${options.function}\\(([^)]+)\\)`, 'g')
  return {
    postcssPlugin: 'postcss-theme-colors', // 定义插件名
    Once(root, { result }) {
      // 判断 PostCSS 工作流程中，是否使用了某些 plugins
      const hasPlugin = name =>
        name.replace(/^postcss-/, '') === options.nestingPlugin ||
        result.processor.plugins.some(p => p.postcssPlugin === name)
      // 获取最终 CSS 值
      const getValue = (value, theme) => {
        return value.replace(reGroup, (match, group) => {
          return resolveColor(options, theme, group, match)
        })
      }

      // 遍历 CSS 声明
      root.walkDecls(decl => {
        const value = decl.value
        // 如果不含有色值函数调用，则提前退出
        if (!value || !reGroup.test(value)) {
          return
        }
        const lightValue = getValue(value, 'light')
        const darkValue = getValue(value, 'dark')
        const darkDecl = decl.clone({ value: darkValue })
        let darkRule
        // 使用插件，生成 dark 样式
        if (hasPlugin('postcss-nesting')) {
          darkRule = postcss.atRule({
            name: 'nest',
            params: `${options.darkThemeSelector} &`,
          })
        } else if (hasPlugin('postcss-nested')) {
          darkRule = postcss.rule({
            selector: `${options.darkThemeSelector} &`,
          })
        } else {
          decl.warn(result, `Plugin(postcss-nesting or postcss-nested) not found`)
        }

        // 添加 dark 样式到目标 HTML 节点中
        if (darkRule) {
          darkRule.append(darkDecl)
          decl.after(darkRule)
        }
        const lightDecl = decl.clone({ value: lightValue })
        decl.replaceWith(lightDecl)
      })
    }
  }
}
module.exports.postcss = true
```

插件使用：

- 定义`CSS`源文件`source.css`：

```css
a {
    color: cc(G01);
}
```

- 定义色值：

```javascript
const colors = {
  C01: '#eee',
  C02: '#111',
  C03: '#fff',
  C04: '#222',
}
```

- 定义模式色值分组：

```javascript
const groups = {
  G01: ['C01', 'C02'], 
  G02: ['C03', 'C04'],
}
```

- 执行转换：

```javascript
const css = fs.readFileSync('source.css')
postcss([
  require('./postcss-theme-colors')({ colors, groups }),
  require('postcss-nested')
]).process(css).then(res => {
  fs.writeFileSync('index.css', res.css)
})
```

执行完成后即可在`index.css`生成如下代码：

```css
a {
    color: #eee;
}

html[data-theme="dark"] a {
    color: #111;
}
```

相关代码含义已在注释中详细注明。通过`post-theme-colors`插件，后续主题维护只需维护`colors`和`groups`两个对象即可，可以通过`JSON`或者`YML`进行维护。

源代码请查看：https://github.com/smartzheng/arch-demos/tree/master/post-theme-colors