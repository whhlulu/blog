> 作者：xiaopang
> [原文](https://github.com/whhlulu/blog)

# Babel 
# 构建/webpack #babel

#### 目录

- 定义
- 原理
- 配置

### 参考资料

- [https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md) 
- [GitHub - jamiebuilds/the-super-tiny-compiler: Possibly the smallest compiler ever](https://github.com/jamiebuilds/the-super-tiny-compiler)
- [https://yq.aliyun.com/articles/62671](https://yq.aliyun.com/articles/62671) 
- [GitHub - estree/estree: The ESTree Spec](https://github.com/estree/estree)
- [GitHub - acornjs/acorn: A small, fast, JavaScript-based JavaScript parser](https://github.com/acornjs/acorn)
- [AST explorer](https://astexplorer.net/)
- [GitHub - ant-design/babel-plugin-import: Modularly import plugin for babel.](https://github.com/ant-design/babel-plugin-import)
- [一口(很长的)气了解 babel - 掘金](https://juejin.im/post/5c19c5e0e51d4502a232c1c6)
- [深入Babel，这一篇就够了 - 掘金](https://juejin.im/post/5c21b584e51d4548ac6f6c99)

# 定义
**Babel 是一个 JavaScript 编译器**

可以通过众多模块可以用于不同形式的静态分析

> 静态分析是在不需要执行代码的前提下对代码进行分析的处理过程 （执行代码的同时进行代码分析即是动态分析）。
> 静态分析的目的是多种多样的， 它可用于语法检查，编译，代码高亮，代码转换，优化，压缩等等场景。

Babel 是一个工具链，主要用于将 ECMAScript 2015+ 版本的代码转换为向后兼容的 JavaScript 语法，以便能够运行在当前和旧版本的浏览器或其他环境中。下面是 Babel 能为你做的事情：

* 语法转换
* 通过 Polyfill 方式在目标环境中添加缺失的特性 (通过 [@babel/polyfill](https://www.babeljs.cn/docs/babel-polyfill) 模块)
* 源码转换 (codemods)

# 基础
## 主要流程

Babel 的三个主要处理步骤分别是：**解析（parse）**，**转换（transform）**，**生成（generate）**

如下图所示：

[image:E596FA6C-C73E-45A8-90E8-904AC088810B-533-000008170FA52C26/babel_process.png]

### 解析

**解析**步骤接收代码并输出 [AST](https://zh.wikipedia.org/zh-hans/%E6%8A%BD%E8%B1%A1%E8%AA%9E%E6%B3%95%E6%A8%B9 )。 这个步骤分为两个阶段： [**词法分析（Lexical Analysis） **](https://en.wikipedia.org/wiki/Lexical_analysis) 和 [语法分析（Syntactic Analysis）](https://en.wikipedia.org/wiki/Parsing) 。词法分析阶段把字符串形式的代码转换为**令牌**（tokens）流，令牌类似于AST中节点；而语法分析阶段则会把一个令牌流转换成 AST的形式，同时这个阶段会把令牌中的信息转换成AST的表述结构。

Babel 是借助 Babylon 实现解析流程的。现在 Babel 已经将 Babylon 重命名为 `babel/parser` [https://github.com/babel/babel/tree/master/packages/babel-parser](https://github.com/babel/babel/tree/master/packages/babel-parser) 

这里有个将源代码装换成抽象语法树的工具： [https://astexplorer.net/](https://astexplorer.net/) 

### 转换

[转换](https://en.wikipedia.org/wiki/Program_transformation) 步骤接收 AST 并对其进行遍历，在此过程中对节点进行添加、更新及移除等操作。 这是 Babel 或是其他编译器中最复杂的过程 同时也是插件将要介入工作的部分

Babel 主要是借助 `babel-traverse` [babel/packages/babel-traverse at master · babel/babel · GitHub](https://github.com/babel/babel/tree/master/packages/babel-traverse)来完成装换的过程 

最风骚的是 这个没有文档… [babel-traverse: Where we can find `path` API documentation? · Issue #5868 · babel/babel · GitHub](https://github.com/babel/babel/issues/5868#issuecomment-309337703)

### 生成

[代码生成](https://en.wikipedia.org/wiki/Code_generation_(compiler)) 步骤把最终（经过一系列转换之后）的 AST 转换成字符串形式的代码，同时还会创建 [源码映射（source maps）](http://www.html5rocks.com/en/tutorials/developertools/sourcemaps/) 
代码生成其实很简单：深度优先遍历整个 AST，然后生成代码转换后的字符串

Babel 主要是借助 `babel-generator` 来生成目标代码

## 转换流程
转换流程主要是以下的步骤：遍历、变动

### 遍历

使用深度优先遍历来遍历 AST

// @TODOS 为什么选择深度优先遍历？
[深度优先搜索和广度优先搜索的区别？ - 知乎](https://www.zhihu.com/question/28549888)

这边我猜测是 深度优先遍历的栈结构 更加合适模拟 JavaScript 词法作用域的实现

### 访问者模式

[访问者模式 - 维基百科，自由的百科全书](https://zh.wikipedia.org/wiki/%E8%AE%BF%E9%97%AE%E8%80%85%E6%A8%A1%E5%BC%8F)

当 Babel 需要处理一个节点的时候，是以访问者的模式获取节点信息，并进行相关操作。
这种方式是通过一个 visitor 对象来完成的，在 visitor 对象中定义了对于各种节点的访问函数，这样就可以针对不同的节点做出不同的处理。

我们编写的 Babel 插件其实也是通过定义一个实例化 visitor 对象，然后处理一系列的 AST 节点来完成我们对代码的修改操作。

``` javascript
const MyVisitor = {
  Identifier(path, state) {
    console.log("Called!");
  }
};

// 你也可以先创建一个访问者对象，并在稍后给它添加方法。
let visitor = {};
visitor.MemberExpression = {
	enter(path, state) {},
	exit(path, state) {}
};
visitor.FunctionDeclaration = function() {}
```

所有的 Babel 所用的 AST 节点定义：[babel/spec.md at master · babel/babel · GitHub](https://github.com/babel/babel/blob/master/packages/babel-parser/ast/spec.md)

这里可以使用 或 或者 别名[babel/packages/babel-types/src/definitions at master · babel/babel · GitHub](https://github.com/babel/babel/tree/master/packages/babel-types/src/definitions)的形式来简化遍历的过程

``` javascript
const MyVisitor = {
  ["ExportNamedDeclaration|Identifier"](path) {}
}
// 或者 Function is an alias for FunctionDeclaration, FunctionExpression, ArrowFunctionExpression, ObjectMethod and ClassMethod.
const MyVisitor = {
  Function(path) {}
};
```

#### 实践

- [GitHub - ant-design/babel-plugin-import: Modularly import plugin for babel.](https://github.com/ant-design/babel-plugin-import)
- [GitHub - Riokai/babel-plugin-transform-remove-console: Add extra options](https://github.com/Riokai/babel-plugin-transform-remove-console)

``` javascript
// babel plugin 的实例
export default function({ types }) {
	return {
	  Identifier(path, state) {}
	}
}
```

参考  [https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-writing-your-first-babel-plugin](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/plugin-handbook.md#toc-writing-your-first-babel-plugin) 

# 配置
目前前端项目中的 Babel 配置一般是出现在：

- 独立的配置文件 `.babelrc` `babel.config.js` `.babelrc.js`
- package.json 中的 `babel` 属性

主要需要配置的就是 `presets` & `plugins` 这两个属性字段

## Babel 插件

Bable 本质上是一个编译器。总体来说运行过程分为三个阶段：**解析、转换以及生成**

其中插件主要是作用于 解析、转换 这个两个阶段

插件主要分为两种表现形式：

- presets
- plugins

#### presets

一般 presets 表示是 plugins 的合集 分为以下的种类

- 官方合集：env react flow 等
- stage-x
- es201x

`Stage num`: 随着 `num` 的越来越小，表示支持得越来越宽泛 （0 ~ 4）

``` json
"presets": [
	"stage-0"
]
```

##### presets 的执行顺序

preset 的执行顺序是从后往前，主要是为了保证**向后兼容**

#### plugins

- syntax syntax-async-functions
- transform transform-runtime

插件的执行顺序是从前到后

### 详细的配置方法

插件会在 presets 之前执行

#### 基础

安装插件无论是 presets 还是 插件。相关的依赖需要安装全名，而使用可以省略前缀

``` bash
npm i --save-dev babel-preset-env
```

``` javascript
{
  presets: ['env', 'react-app', 'react'],
	plugins: ['import', '']
}
```

#### 传参

简略情况下，插件和 preset 只要列出字符串格式的名字即可。

但如果某个 preset 或者插件需要一些配置项(或者说参数)，就需要把自己先变成数组。

- 第一个元素依然是字符串，表示自己的名字
- 第二个元素是一个对象，即配置对象

``` json
{
	"plugins": [
    [
      "transform-runtime",
      {
        "polyfill": false
      }
    ]
  ],
  "presets": [
    [
      "env",
      {
        "targets": {
          "browsers": ["ie >= 8"]
        }
      }
    ],
    "stage-0"
  ]
}
```

### preset-env

env 的核心目的是通过配置得知目标环境的特点，然后只做必要的转换。例如目标浏览器支持 es2015，那么 es2015 这个 preset 其实是不需要的，于是代码就可以小一点(一般转化后的代码总是更长)，构建时间也可以缩短一些。

之前都是使用 `babel-preset-es2015` 类似这种形式，现在这种形式已经不被官方推荐

譬如

``` javascript
"presets": [
	[
		"env",
		{
			"targets": {
				"browsers": ["ie >= 8"], // 参考 browserlist
				"node": "6.0.0"
				// "module": false
			}
		}
	]
]
```

浏览器环境的定义使用的是：[GitHub - browserslist/browserslist: 🦔 Share target browsers between different front-end tools, like Autoprefixer, Stylelint and babel-preset-env](https://github.com/browserslist/browserslist)

## 其他工具
### babel-core

如同包的名称一样，实现 `Babel` 相关的核心功能

其实主要的内容就是 `transform` & `parse`

### babel-node

babel-node 是 babel-cli 的一部分，它不需要单独安装。

它的作用是在 node 环境中，直接运行 es2015 的代码，而不需要额外进行转码。例如我们有一个 js 文件以 es2015 的语法进行编写(如使用了箭头函数)。我们可以直接使用 babel-node es2015.js 进行执行，而不用再进行转码了。

可以说：babel-node = babel-polyfill + babel-register。那这两位又是谁呢？

### babel-register

babel-register 模块改写 require 命令，为它加上一个钩子。此后，每当使用 require 加载 .js、.jsx、.es 和 .es6 后缀名的文件，就会先用 babel 进行转码。

使用时，必须首先加载 require(‘babel-register’)

需要注意的是，babel-register 只会对 require 命令加载的文件转码，而 **不会对当前文件转码**

另外，由于它是实时转码，所以 **只适合在开发环境使用**

### babel-polyfill

会提供完整的 `ES6` 以上的执行环境

在使用 `babel-node` 的时候自动加载

因为本质上是环境的实现，所以在包管理中是执行时的依赖（`dependency`），而不是打包过程的依赖（`devDependency`）

通过下面的方式来使用：

``` javascript
// 在源代码中使用
require('babel-polyfill')
import 'babel-polyfill'

// 在 Webpack 中使用
module.exports = {
	entry: ['babel-polyfill', './app/js]
}
```

甚至可以在 HTML 中直接引入

``` html
<script src="dist/polyfill.js"></script>
```

存在的问题就是：

- 这是个大而全的环境补丁
- 实现的过程污染了全局的环境

通常生产环境中采用  `transform-runtime` 插件来规避上面的问题

### 运行时编译

为了避免前面提到的 `babel-polyfill` 的问题，生产环境中可以使用

`babel-plugin-transform-runtime` 配合 `babel-runtime`

``` bash
npm i --save-dev babel-plugin-transform-runtime
npm i --save babel-runtime
```

默认情况下，`Babel` 会给每个需要转化的函数进行转化，常常会出现大量无意义的重复代码，这个插件使用 `babel-runtime` 来避免对每个独立内容进行转化，最终将依赖收集完成的 `runtime` 打包到生成代码中。与此同时，这个转化过程会给代码创建沙箱环境，以避免污染全局环境

```javascript
// 之前的转化代码
function _extent(target) {
  for (var i = 1; i < arguments.length; i++) {
    var source = arguments[i];
    for (var key in source) {
      if (Object.prototype.hasOwnProperty.call(source, key)) {
        target[key] = source[key];
      }
    }
  }
  return target;
}

// 使用 transform-runtime
var _extent = require('babel-runtime/helpers/_extent');
```

有个问题是，runtime 不实现 实例方法，例如：`[].includes(1)` 这种还是需要独立的插件来支持


