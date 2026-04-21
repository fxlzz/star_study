# webpack 与 vite 的区别

## 构建速度

webpack 的构建速度也是阴得没边了，平均下来要 35s 左右（项目中），vite 1s 都没到。这是因为 webapck 和 vite 底层逻辑不一样。

**webpack**

webpack 构建时，会**递归解析整个项目的依赖树**，将所有模块（js、css 等）打包成一个或多个 bundle 文件后，才会启动开发服务器。这个过程需要遍历所有依赖，loader 转换，插件处理，项目越大依赖越多，打包就越久。

**vite**

vite 采用了“**原生 ES 模块的设计，按需编译**”，它是无论项目复杂程度的大小，都会先启动一个**开发服务器**。当浏览器请求某个模块时，vite 才会对改模块进行实时编译。

## 依赖处理

**webpack**

需要借助 label-loader、ts-loader、css-loader 等 loader 工具逐文件转换，将代码转化 js 代码在执行。

**vite**

node_modules 包下的第三方库，一般都变化少、可能是 commomjs 格式（非 esm，浏览器无法直接识别），那么 vite 基于这些特点，做了两点优化。

- 预构建依赖：vite 会用 esbuild 将第三方依赖转换成 esm 格式，合并成几个文件，减少浏览器请求次数
- 缓存机制：预处理的结果会被缓存到 node_modules/.vite 目录，后续直接服用，无需重复处理。只有当依赖版本变化或配置修改时，才会重新预处理

## 热更新

**webpack**

Webpack 的 HMR 依赖于 “打包后的 bundle”：**当某个模块修改后，Webpack 需要重新构建该模块及其依赖的整个子树，并生成新的 bundle 片段，再通过 HMR 客户端替换旧模块**。这个过程涉及复杂的依赖关系重新解析和打包，大型项目中可能有明显延迟。

**vite**

Vite 的 HMR 基于 “原生 ESM 模块图”：由于**浏览器直接请求单个模块，Vite 只需精确找到被修改的模块，重新编译该模块**，并通过 HMR 通知浏览器替换该模块即可。**无需重新处理整个依赖树**，更新速度极快（通常是毫秒级）。

# webpack
具体项目配置： [Webpack 工程化设计](../../project/improve_coding/improve_coding.md#Webpack%20工程化设计)

# 引言

也是进入前端工程化的学习了，虽然已经这个时候已经学习过了框架的知识，今天是 2025-5-19。加油，朋友！

之前都不知道，都没有关注过工程化这个东西，因为用框架的时候，都集成了这些构建工具。难怪不需要考虑那么多非技术问题（兼容性，模块化导入->ESM，CMJ）

## 浏览器端的模块化

问题：
- 效率问题：精细的模块划分带来了更多的JS文件，更多的JS文件带来了更多的请求，降低了页面访问效率
- 兼容性问题：浏览器目前仅支持ES6的模块化标准，并且还存在兼容性问题
- 工具问题：浏览器不支持npm下载的第三方包

这些仅仅是前端工程化的一个缩影

当开发一个具有规模的程序，你将遇到非常多的非业务问题，这些问题包括：执行效率、兼容性、代码的可维护性可扩展性、团队协作、测试等等等等，我们将这些问题称之为工程问题。工程问题与业务无关，但它深刻的影响到开发进度，如果没有一个好的工具解决这些问题，将使得开发进度变得极其缓慢，同时也让开发者陷入技术的泥潭。

## 根本原因

思考：上面提到的问题，为什么在node端没有那么明显，反而到了浏览器端变得如此严重呢？

答：在node端，运行的JS文件在本地，因此可以本地读取文件，它的效率比浏览器远程传输文件高的多

**根本原因**：在浏览器端，开发时态（devtime）和运行时态（runtime）的侧重点不一样

**开发时态，devtime：**
1. 模块划分越细越好
2. 支持多种模块化标准
3. 支持npm或其他包管理器下载的模块
4. 能够解决其他工程化的问题

**运行时态，runtime：**
1. 文件越少越好
2. 文件体积越小越好
3. 代码内容越乱越好
4. 所有浏览器都要兼容
5. 能够解决其他运行时的问题，主要是执行效率问题

这种差异在小项目中表现的并不明显，可是一旦项目形成规模，就越来越明显，如果不解决这些问题，前端项目形成规模只能是空谈

## 解决办法

既然开发时态和运行时态面临的局面有巨大的差异，因此，我们需要有一个工具，这个工具能够让开发者专心的在开发时态写代码，然后利用这个工具将开发时态编写的代码转换为运行时态需要的东西。

这样的工具，叫做**构建工具**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747621769430-3c5ee6a5-7b37-4111-9204-77d820174a51.png)

这样一来，开发者就可以专注于开发时态的代码结构，而不用担心运行时态遇到的问题了。

【补充】：打包后的结果，在哪个环境（无论是浏览器还是 node 环境下）下都能运行，这就是构建工具的牛逼之处~~如果有 ajax 的话，还要考虑后端的问题

# webpack的安装和使用

## webpack简介

webpack是基于模块化的打包（构建）工具，它把一切视为**模块**

它通过一个开发时态的入口模块为起点，分析出所有的依赖关系，然后经过一系列的过程（压缩、合并），最终生成运行时态的文件。

webpack的特点：

- **为前端工程化而生**：webpack致力于解决前端工程化，特别是浏览器端工程化中遇到的问题，让开发者集中注意力编写业务代码，而把工程化过程中的问题全部交给webpack来处理
- **简单易用**：支持零配置，可以不用写任何一行额外的代码就使用webpack
- **强大的生态**：webpack是非常灵活、可以扩展的，webpack本身的功能并不多，但它提供了一些可以扩展其功能的机制，使得一些第三方库可以融于到webpack中
- **基于nodejs**：由于webpack在构建的过程中需要读取文件，因此它是运行在node环境中的
- **基于模块化**：webpack在构建过程中要分析依赖关系，方式是通过模块化导入语句进行分析的，它支持各种模块化标准，包括但不限于CommonJS、ES6 Module

## webpack的安装

```bash
npm i -D wepack wepack-cli
```

## 使用

```bash
webpack
```

默认情况下，webpack会以`./src/index.js`作为入口文件分析依赖关系，打包到`./dist/main.js`文件中

通过--mode选项可以控制webpack的打包结果的运行环境

```js
webpack --mode=development # 生成开发环境代码
webpack --mode=prdouction  # 生成生产环境代码

# 一般在package.json中配置一些脚本命令
 "scripts": {
    "build": "webpack --mode=production",
    "dev": "webpack --mode=development --watch"
},
```


# 简单模拟一下打包结果

```js
(function () {
  // 将每一个模块当作一个函数来执行
  var modules = {
    "./src/a.js": function (module, exports) {
      // 这个地方wepack是使用的eval来执行js文件，
      // 因为eval开辟一个新的执行环境，并且可以告诉浏览器是哪里错了，方便调式
      console.log("a");
      module.exports = "a";
      // eval('console.log("a");\r\nmodule.exports = "a";\r\n\n\n//# sourceURL=webpack://webpack/./src/a.js?');
    },
    "./src/index.js": function (module, exports, require) {
      // 这个地方的路径会转成module中存在的路径，不然require查找的时候找不到
      require("./src/a.js"); 
      console.log("hello webpack");
    },
  };
  
  // 模块缓存 {"./src/a.js" : "a"}
  var modulesCache = {};
  function require(moduleId) {
    if (modulesCache[moduleId]) {
      // 如果缓存了模块，则直接返回模块结果即可
      return modulesCache[moduleId];
    }

    var func = modules[moduleId];
    var module = {
      exports: {},
    };
    func(module, module.exports, require);
    var res = module.exports;
    // 缓存模块
    modulesCache[moduleId] = res;
    return res;
  }

  require("./src/index.js");
})();
```

# 编译过程

大概分成三部分，

## 初始化

初始化就干一个事情，将**配置文件、命令行参数、默认配置**组合起来，形成一个最终的配置对象

对配置的处理过程是依托一个第三方库`yargs`完成的

## 编译

### 1、创建 chunk

chunk是webpack在内部构建过程中的一个概念，译为`块`，它表示通过某个入口找到的所有依赖的统称。

根据入口模块（默认为`./src/index.js`）创建一个chunk

每个chunk都有至少两个属性：
- name：默认为main
- id：唯一编号，**开发**环境和name相同，生产环境是一个数字，从0开始

### 2、构建所有依赖模块

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724588465-3ca9409b-049d-41ee-b8db-7f28108428a3.png)

AST在线测试工具：[https://astexplorer.net/](https://gitee.com/link?target=https%3A%2F%2Fastexplorer.net%2F)

简图

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724673453-dd17850d-b6be-4ded-9c9b-77cf55c86fd4.png)

例如：

```js
入口文件：./src/index.js

1、查找记录表 ---》没有

2、读取文件内容 ---》
require("./a");
require("./b");
console.log("hello webpack");

3、语法分析、转化成AST---》得知所有依赖的模块

4、将依赖记录到 dependencies ["./src/a.js", "./src/b.js"]

5、替换依赖函数 ---》将路径替换、require替换成__webpack_require_

6、转化后的代码
__webpack_require_("./src/a.js");
__webpack_require_("./src/b.js");
console.log("hello webpack");

7、记录到表中，类似一个这样的数据结构
[
	{
		moduleId: "./src/index.js",
		content: "__webpack_require_(\"./src/a.js\");
				__webpack_require_(\"./src/b.js\");
				console.log(\"hello webpack\");"
	}
]

继续将 dependencies 的模块继续分析，重复上面七步
```

### 3、产生 chunk assets

在第二步完成后，chunk中会产生一个模块列表，列表中包含了**模块id**和**模块转换后的代码**

接下来，webpack会根据配置为chunk生成一个资源列表，即`chunk assets`，资源列表可以理解为是生成到最终文件的文件名和文件内容

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724719354-8035e7db-863e-4010-9c9e-d64f16222cae.png)

chunk hash是根据所有chunk assets的内容生成的一个hash字符串

hash：一种算法，具体有很多分类，特点是将一个任意长度的字符串转换为一个固定长度的字符串，而且可以保证原始内容不变，产生的hash字符串就不变

简图

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724719358-aec000c8-e331-4535-b37b-b1e63baf7c3c.png)

### 4、合并 chunk assets

将多个chunk的assets合并到一起，并产生一个总的hash

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724762219-0cd4affb-4590-4b78-8256-bee0e69973a4.png)

## 输出

此步骤非常简单，webpack将利用node中的fs模块（文件处理模块），根据编译产生的总的assets，生成相应的文件。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724775899-622f6633-a82a-48f8-ba71-d355fedf79cd.png)

## 总过程

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747724798639-3afce7d3-f5e2-4d1e-a969-caa16cb969f8.png)

# 配置文件

在项目根目录下，添加`webpack.config.js`可以在这里面写一些配置项

但是，

这个文件中，只能写 **node 环境**中的代码

为什么？

开发环境中的代码（也就是 src 目录下面的源代码）是不参与运行的，真正运行的代码是通过构建工具打包后的代码。在开发环境中，使用什么环境下的导入导出语句都可以，因为反正不会参与运行。

但是，`webpack.config.js`这个文件在 **webpack 构建时，会参与运行**。因为需要读取文件，所以 webpack 构建时，是运行在 node 环境中的，所以这个配置文件只能使用 node 环境中支持的语法。

一些常见的配置项：

1. mode ：指定生成的打包文件是**开发**环境还是**生产**环境，一般在 package.json 中以脚本的方式配置
2. entry ：指定 webpack 运行的入口文件，其实真正配置的是 chunk（默认为 mian: "src/index.js"）
3. output ： 指定 webpack 打包后生成的打包文件目录，对象（默认为 dist/main.js）

```js
// 运行webpack是node环境
const path = require("path");

module.exports = {
  entry: {
    // chunk name : chunk的入口文件
    mian: "./src/index.js",
    a: "./src/a.js",
  },
  output: {
    path: path.resolve(__dirname, "target"),
    // filename: "bundle.js", // 静态配置文件名
    filename: "[name]-[chunkhash:5].js",
  },
};
```

output filename 中的规则，也就是 [] 中能写的东西：

- name：chunk name
- hash: 总的资源hash，通常用于解决缓存问题
- chunkhash: 使用chunkhash
- id: 使用chunkid，不推荐

## 源码地图(source map)

这个东西，就是方便开发的，方便**调式**

可以在配置文件中，进行相关配置 [https://www.webpackjs.com/configuration/devtool/](https://www.webpackjs.com/configuration/devtool/)

`mode=production` -> 默认值 `devtool: "none"`不生成源码地图

`mode=development`-> 默认值 `devtool: "eval"`使用 eval 运行 js 代码

【建议】：**开发环境**下，使用 eval-source-map，会将源代码使用 base64 编码后加在原始代码后面，在浏览器中生成对应的源码地图。

```js
"./src/index.js":
((__unused_webpack_module, __unused_webpack_exports, __webpack_require__) => {
eval("__webpack_require__(/*! ./a */ \"./src/a.js\");\r\nconsole.log(\"hello webpack\");\r\n//# sourceURL=[module]\n//# sourceMappingURL=data:application/json;charset=utf-8;base64,eyJ2ZXJzaW9uIjozLCJmaWxlIjoiLi9zcmMvaW5kZXguanMiLCJtYXBwaW5ncyI6IkFBQUEsbUJBQU8sQ0FBQyx1QkFBSztBQUNiIiwic291cmNlcyI6WyJ3ZWJwYWNrOi8vd2VicGFjay8uL3NyYy9pbmRleC5qcz9iNjM1Il0sInNvdXJjZXNDb250ZW50IjpbInJlcXVpcmUoXCIuL2FcIik7XHJcbmNvbnNvbGUubG9nKFwiaGVsbG8gd2VicGFja1wiKTtcclxuIl0sIm5hbWVzIjpbXSwic291cmNlUm9vdCI6IiJ9\n//# sourceURL=webpack-internal:///./src/index.js\n");
})
```

**生产环境**下，推荐使用 none，

如果使用 source-map 就生成一个新的文件，告诉源码地图在哪

```js
(()=>{var o={967:o=>{console.log("a"),o.exports="a",null.abc()}},r={};!function e(l){var t=r[l];if(void 0!==t)return t.exports;var a=r[l]={exports:{}};return o[l](a,a.exports,e),a.exports}(967),console.log("hello webpack")})();
//# sourceMappingURL=main.js.map
```

如果使用 hidden-source-map 会在浏览器隐藏源码地图，也不会有上面的那行注释，一些 devtools 工具可以看到

# loader

webpack做的事情，仅仅是分析出各种模块的依赖关系，然后形成资源列表，最终打包生成到指定的文件中。 更多的功能需要借助webpack loaders和webpack plugins完成。

其实这个 loader 就是一个函数，可以将一串代码 以特定的变化变成 另一串代码。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747730336058-4ffd6bba-17b2-4ec8-879d-ff602a780855.png)

**chunk中解析模块的更详细流程：**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747730375159-187ec092-9131-4052-9e11-a72591d14d50.png)

**处理loaders流程：**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747730389974-406c5f6a-5b67-4c36-9c9c-bdb3100203b3.png)

**loader 配置：**

**【完整配置】：**

```js
module.exports = {
  module: {  // 针对模块的配置，目前版本只有两个配置，rules、noParse
    rules: [ // 模块匹配规则，可以存在多个规则
      { // 每个规则是一个对象
        test: /\.js$/, // 匹配的模块正则
        use: [ // 匹配到后应用的规则模块
          { 
            // 该字符串会被放置到require中，所以一定要./开头，不然node会去node_modules中查找
            loader: "模块路径",
            options: { "自定义名称": 值 } // 向对应loader传递的额外参数
            // 需要通过this获取，但一般用第三方库解决 loader-utils
          }
        ]
      }
    ]
  }
}
```

**【简化版】：**

```js
module.exports = {
  module: {
    rules: [ 
      { 
        test: /\.js$/,
        use: ["模块路径1", "模块路径2"]
      }
    ]
  }
}
```

**【loader 的运行机制】：**

webpack 运行时，扫描到 module，会从上往下扫描，将所有能匹配上的 use 中的模块路径加到一个新的数组中。在扫描完后，从后往前执行这个数组。

例如：

```js
// webpack.config.js
module.exports = {
  module: {
    rules: [
      {
        test: /index\.js$/,
        use: ["./src/loaders/loader1.js", "./src/loaders/loader2.js"],
      },
      {
        test: /\.js$/,
        use: ["./src/loaders/loader3.js", "./src/loaders/loader4.js"],
      },
    ],
  },
};

// index.js
const b = require("./b");
console.log(b);

// b.js
console.log("biashdoas");
module.exports = "b";

执行webpack命令后，初始化 -> 读取index.js文件 -> 执行能被匹配的loader（输出 4 3 2 1） -> 
                  转化成AST -> 将模块记录到 dependencies -> 转化代码
                  读取b.js文件 -> 执行loader（输出4 3）
```

# plugin

plugin 也是 webpack 中的一个重要组成部分，与 loader 不同，

plugin 主要干预 webpack 的编译过程（比如：在 webpack 编译完成后，输出一句话；多生成一个列表文件；等等）

如何干预呢？通过给 plugin 注册事件。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747810301817-99f17a03-46a3-4ca1-b0bd-7a1809d519ba.png)

**plugin 的本质，其实是带有 apply 方法的函数，一般写成 class 的形式**

```js
class MyPlugin() {
  apply(compiler) {
    
  }
}
// 使用时，var plugin = new MyPlugin()
```

要将插件应用到webpack，需要把插件对象配置到webpack的plugins数组中，如下：

```js
module.exports = {
  plugins:[
    new MyPlugin()
  ]
}
```

apply函数会在**初始化**阶段，创建好Compiler对象后运行。

compiler对象是在初始化阶段构建的，整个webpack打包期间**只有一个**compiler对象，后续完成打包工作的是compiler对象内部创建的compilation（可以有**多个**）

apply方法会在**创建好compiler对象后调用**，并向方法传入一个compiler对象

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747810506976-e578fd7a-e569-45b2-830a-5903be469e46.png)

compiler对象提供了大量的钩子函数（hooks，可以理解为事件），plugin的开发者可以注册这些钩子函数，参与webpack编译和生成。

你可以在apply方法中使用下面的代码注册钩子函数:

```js
class MyPlugin{
  apply(compiler){
    compiler.hooks.事件名称.事件类型(name, function(compilation){
      //事件处理函数
    })
  }
}
```

**事件名称**

即要监听的事件名，即钩子名，所有的钩子：[https://www.webpackjs.com/api/compiler-hooks](https://gitee.com/link?target=https%3A%2F%2Fwww.webpackjs.com%2Fapi%2Fcompiler-hooks)

**事件类型**

这一部分使用的是 Tapable API，这个小型的库是一个专门用于钩子函数监听的库。

它提供了一些事件类型：

- tap：注册一个**同步**的钩子函数，函数运行完毕则表示事件处理结束
- tapAsync：注册一个基于**回调**的**异步**的钩子函数，函数通过调用一个回调表示事件处理结束
- tapPromise：注册一个基于**Promise**的**异步**的钩子函数，函数通过返回的Promise进入已决状态表示事件处理结束

# 其他配置对象

这些也是稍微有点印象就行，了解一下就行~

可以去这个官网上找，[https://www.webpackjs.com/configuration/](https://www.webpackjs.com/configuration/)

## context

```js
context: path.resolve(__dirname, "app")
```

该配置会影响入口和loaders的解析，入口和loaders的相对路径会以context的配置作为基准路径，这样，你的配置会独立于CWD（current working directory 当前执行路径）

## output

### publicPath

一些第三方库会使用这个目录，来作用相对目录，会加在生成 url 的前面

一般配置成`publicPath:"/"`将 url 变成**绝对路径**，与当前域名拼接

例如：

```js
const __WEBPACK_DEFAULT_EXPORT__ = (__webpack_require__.p + "imgs/img.d5033.png");

// 这个__webpack_require__.p 其实就是 publicPath 的值
```

### library

```js
library: "abc"
```

这样一来，打包后的结果中，会将自执行函数的执行结果暴露给abc

### libraryTarget

```js
libraryTarget: "var"
```

该配置可以更加精细的控制如何暴露入口包的导出结果

其他可用的值有：

- var：默认值，暴露给一个普通变量
- window：暴露给window对象的一个属性
- this：暴露给this的一个属性
- global：暴露给global的一个属性
- commonjs：暴露给exports的一个属性
- 其他：[https://www.webpackjs.com/configuration/output/#output-librarytarget](https://gitee.com/link?target=https%3A%2F%2Fwww.webpackjs.com%2Fconfiguration%2Foutput%2F%23output-librarytarget)

## target

```js
target:"web" //默认值
```

设置打包结果最终要运行的环境，常用值有

- web: 打包后的代码运行在web环境中
- node：打包后的代码运行在node环境中
- 其他：[https://www.webpackjs.com/configuration/target/](https://gitee.com/link?target=https%3A%2F%2Fwww.webpackjs.com%2Fconfiguration%2Ftarget%2F)

## module.noParse

```js
noParse: /jquery/
```

不解析正则表达式匹配的模块，通常用它来忽略那些大型的单模块库，以提高**构建性能**

## resolve

resolve的相关配置主要用于控制*模块解析*过程

### modules

```js
modules: ["node_modules"]  //默认值
```

当解析模块时，如果遇到导入语句，`require("test")`，webpack会从下面的位置寻找依赖的模块

1. 当前目录下的`node_modules`目录
2. 上级目录下的`node_modules`目录
3. ...

### extensions

```js
extensions: [".js", ".json"]  //默认值
```

当解析模块时，遇到无具体后缀的导入语句，例如`require("test")`，会依次测试它的后缀名

- test.js
- test.json

### alias

```js
alias: {
  "@": path.resolve(__dirname, 'src'),
  "_": __dirname
}
```

有了alias（别名）后，导入语句中可以加入配置的键名，例如`require("@/abc.js")`，webpack会将其看作是`require(src的绝对路径+"/abc.js")`。

在大型系统中，源码结构往往比较深和复杂，别名配置可以让我们更加方便的导入依赖

## externals

```js
externals: {
  jquery: "$",
  lodash: "_"
}
```

从最终的bundle中排除掉配置的配置的源码，例如，入口模块是

```js
//index.js
require("jquery")
require("lodash")
```

生成的bundle是：

```js
(function(){
  ...
})({
  "./src/index.js": function(module, exports, __webpack_require__){
    __webpack_require__("jquery")
    __webpack_require__("lodash")
  },
  "jquery": function(module, exports){
    //jquery的大量源码
  },
  "lodash": function(module, exports){
    //lodash的大量源码
  },
})
```

但有了上面的配置后，则变成了

```js
(function(){
  ...
})({
  "./src/index.js": function(module, exports, __webpack_require__){
    __webpack_require__("jquery")
    __webpack_require__("lodash")
  },
  "jquery": function(module, exports){
    module.exports = $;
  },
  "lodash": function(module, exports){
    module.exports = _;
  },
})
```

这比较适用于一些第三方库来自于外部CDN的情况，这样一来，即可以在页面中使用CDN，又让bundle的体积变得更小，还不影响源码的编写

# 常见 plugin 插件

## clean-webpack-plugin

可以自动删除文件变动后，打包的文件

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const path = require("path");

module.exports = {
  devtool: "source-map",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name]-[chunkhash:5].js",
  },
  plugins: [new CleanWebpackPlugin()],
};
```

需要注意的是，需要配置完整的 output，不然不起作用

## html-webpack-plugin

可以在打包目录下，自动生成一个 html 文件，并且自动引用打包好的 js 文件

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const HtmlWebpackPlugin = require("html-webpack-plugin");
const path = require("path");

module.exports = {
  devtool: "source-map",
  // 多个 chunk
  entry: {
    main: "./src/index.js",
    a: "./src/a.js",
  },
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name]-[chunkhash:5].js",
  },
  plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      template: "./public/index.html", // 参考该html文件当作模板在打包后目录生成html文件
      hash: true, 
      chunks: ["main"], // 解决多个chunk怎么在页面上引用js的问题
    }),
  ],
};
```

更多配置对象可以参考：[https://www.npmjs.com/package/html-webpack-plugin](https://www.npmjs.com/package/html-webpack-plugin)

## copy-webpack-plugin

这个插件就是用于复制一些**静态**资源（比方说，一些图片、logo 等在页面上写死了的）到打包目录

```js
const { CleanWebpackPlugin } = require("clean-webpack-plugin");
const path = require("path");

module.exports = {
  devtool: "source-map",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name]-[chunkhash:5].js",
  },
  plugins: [
    new CopyPlugin({
      // to 相对的目录就是 output 中指定的输出文件资源的目录
      patterns: [{ from: "./public/img.png", to: "./" }],
    }),
  ],
};
```

# 开发服务器

webpack-dev-server 这个是 webpack 官方写的一个库，用于**开发阶段**

后面学的 vue-cli 或者 react-create-app 应该都是根据 webpack 和 webpack-dev-server 开发的

不过，现在都看齐 vite 了，确实 vite 会比 webpack 的构建速度快的多

**一些常见的配置项**

可以参考：[https://www.webpackjs.com/configuration/dev-server/](https://gitee.com/link?target=https%3A%2F%2Fwww.webpackjs.com%2Fconfiguration%2Fdev-server%2F)

- port：配置监听端口
- open：自动在浏览器请求服务器地址
- proxy：配置代理，常用于跨域访问

```js
const path = require("path");

module.exports = {
  devtool: "source-map",
  output: {
    path: path.resolve(__dirname, "dist"),
    filename: "[name]-[chunkhash:5].js",
  },
  devServer: {
    port: 8000,
    proxy: [
      {
        context: ["/api"],
        target: "https://whyta.cn",
        changeOrigin: true,
        onProxyRes: function (proxyRes, req, res) {
          // 用于调试代理是否生效
          console.log("代理响应:", req.url);
        },
      },
    ],
  },
  stats: { modules: false },
};
```

# 常见 loader 加载器

## file-loader

将生成依赖的文件到输出目录，然后将模块文件设置为：导出一个路径

（就是会生成一个文件到输出目录）

```js
module: {
    rules: [
      {
        test: /\.(png)|(gif)|(jpg)$/,
        use: [
          {
            loader: "file-loader", // => require("file-loader")
            options: {
              name: "imgs/[name].[hash:5].[ext]",
            },
          },
        ],
      },
    ],
  },
```

生成的图片路径：

## url-loader

将文件以 base64 的形式导出

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.(png|jpg|gif)$/i,
        use: [
          {
            loader: 'url-loader',
            options: {
              limit: 8192,
            },
          },
        ],
      },
    ],
  },
};
```

这个库，在底层使用了 file-loader，如果超出了 limit 设置的字节数，那么就会调用 file-loader 创建文件到输出目录。

生成 base64 的编码

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1747916005354-78cb5640-bf02-42b1-94dc-771b9f90cc3d.png)

【注意】：这些都是动态加载文件，用 js 在页面上生成，而不是写死在页面上。如果写死的路径，一般用 copy-webpack-plugin 插件，原封不动复制到输出目录。

# webpack内置插件

这些稍微了解一下就可以了~

所有的webpack内置插件都作为webpack的静态属性存在的，使用下面的方式即可创建一个插件对象

```js
const webpack = require("webpack")

new webpack.插件名(options)
```

## DefinePlugin

全局常量定义插件，使用该插件通常定义一些常量值，例如：

```js
new webpack.DefinePlugin({
    PI: `Math.PI`, // PI = Math.PI
    VERSION: `"1.0.0"`, // VERSION = "1.0.0"
    DOMAIN: JSON.stringify("duyi.com")
})
```

这样一来，在源码中，我们可以直接使用插件中提供的常量，当webpack编译完成后，会自动替换为常量的值

## BannerPlugin

它可以为每个chunk生成的文件头部添加一行注释，一般用于添加作者、公司、版权等信息

```js
new webpack.BannerPlugin({
  banner: `
  hash:[hash]
  chunkhash:[chunkhash]
  name:[name]
  author:yuanjin
  corporation:duyi
  `
})
```

## ProvidePlugin

自动加载模块，而不必到处 import 或 require

```js
new webpack.ProvidePlugin({
  $: 'jquery',
  _: 'lodash'
})
```

然后在我们任意源码中：

```js
$('#item'); // <= 起作用
_.drop([1, 2, 3], 2); // <= 起作用
```

# css 工程化

css 问题？

1. css 类名冲突
2. 代码不能复用
3. css 文件不好细分

如何解决？

## 解决类名冲突

1. css module ---> css 模块化
2. css in js ---> 将 css 写成 js 对象使用
3. 使用一些命名规范 ---> BEM 等

### BEM

其实也很简单，就是一个命名规范而已，遵守这个命名规范就能保证 css 类名不会冲突

其他命名方法还有：OOCSS、AMCSS、SMACSS等等

**【注意】：需要注意的呢，不要嵌套类名，写顶级类名**

一个完整的BEM类名：block__element_modifier，例如：`banner__dot_selected`，可以表示：轮播图中，处于选中状态的小圆点

【下面的了解一下就行】

三个部分的具体含义为：

- **Block**：页面中的大区域，表示最顶级的划分，例如：轮播图(`banner`)、布局(`layout`)、文章(`article`)等等
- **element**：区域中的组成部分，例如：轮播图中的横幅图片(`banner__img`)、轮播图中的容器（`banner__container`）、布局中的头部(`layout__header`)、文章中的标题(`article_title`)
- **modifier**：可选。通常表示状态，例如：处于展开状态的布局左边栏（`layout__left_expand`）、处于选中状态的轮播图小圆点(`banner__dot_selected`)

在某些大型工程中，如果使用BEM命名法，还可能会增加一个前缀，来表示类名的用途，常见的前缀有：

- **l**: layout，表示这个样式是用于布局的
- **c**: component，表示这个样式是一个组件，即一个功能区域
- **u**: util，表示这个样式是一个通用的、工具性质的样式
- **j**: javascript，表示这个样式没有实际意义，是专门提供给js获取元素使用的

### css in js

在一些不支持 css 的设备上有很好的表现，一些移动端，手机 App 等

vue、react 都支持，这种写法

```js
const styles = {
    backgroundColor: "#f40",
    color: "#fff",
    width: "400px",
    height: "500px",
    margin: "0 auto"
}
```

就是将 css 当作 js 对象来用，具体应用到每个元素上，可以写个函数等等

在 react 中，可以直接用`{}`中应用到 style 中

```js
<div style={styles}>css in js<div/>
```

### css module

css 模块化，这个应用的也很多，借鉴了 js 模块化，将 css 分离每一个模块对应一个功能。需要借助构建工具的帮忙，不过使用了 vue、react 这种框架，里面的脚手架一般都配置好了。只要写代码即可。

扯远了，在 webpack 中开启 css module，直接用 css-loader 开启 module:true 即可

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", {
          loader: "css-loader",
          options: {
            modules: true
         // modules: {
         //     localIdentName: "[local]-[hash:5]"
         // }
          }
        }]
      }
    ]
  }
}
```

## 解决重复样式

1. css in js ---> 转成 js 当然能重复使用咯
2. 预编译器 ---> less、sass

### less

用于增强 css 语言的一种新语言，让开发者更好的开发。

预编译器的原理很简单，即使用一种更加优雅的方式来书写样式代码，通过一个编译器，将其转换为可被浏览器识别的传统css代码

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748050044219-bbf40067-b20b-43a5-a36e-507557ee5484.png)

可参考文档：

[https://www.w3cschool.cn/less/less_installation.html](https://www.w3cschool.cn/less/less_installation.html)

常见的一些操作：

- 变量
- 混合
- 嵌套
- 运算
- 函数
- 作用域
- 注释
- 导入

## 解决 css 文件细分

这一部分，就要依靠构建工具，例如webpack来解决了

利用一些loader或plugin来打包、合并、压缩css文件

比如：css-loader (兼容性)、style-loader（将 css 文件通过 `<style>` 注入到 html 文件）

```js
module: {
    rules: [
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
    ],
  },
// 这里需要注意的是，loader是从后往前执行的~
```

# js 工程化

## bebal

babel的出现，就是用于解决这样的问题，它是一个编译器，可以把不同标准书写的语言，编译为统一的、能被各种浏览器识别的语言

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748053485382-94a2c83b-5c45-476f-b764-3603221f4fef.png)

由于语言的转换工作灵活多样，babel的做法和postcss、webpack差不多，它本身仅提供一些分析功能，真正的转换需要依托于插件完成

不过，如果使用 _vue、react_ 等框架，这些构建工具都配置好了这些东西，所以这些大概了解一下就可以了。

## babel的配置

可以看到，babel本身没有做任何事情，真正的编译要依托于**babel插件**和**babel预设**来完成

babel预设和postcss预设含义一样，是多个插件的集合体，用于解决一系列常见的兼容问题

如何告诉babel要使用哪些插件或预设呢？需要通过一个配置文件`.babelrc`

```
{
  "presets": [],
  "plugins": []
}
```

## babel预设

babel有多种预设，最常见的预设是`@babel/preset-env`

`@babel/preset-env`可以让你使用最新的JS语法，而无需针对每种语法转换设置具体的插件

**配置**

```
{
    "presets": [
        "@babel/preset-env"
    ]
}
```

**兼容的浏览器**

`@babel/preset-env`需要根据兼容的浏览器范围来确定如何编译，和postcss一样，可以使用文件`.browserslistrc`来描述浏览器的兼容范围

```
last 3 version
> 1%
not ie <= 8
```

**自身的配置**

和`postcss-preset-env`一样，`@babel/preset-env`自身也有一些配置

配置方式是：

```
{
    "presets": [
        ["@babel/preset-env", {
            "配置项1": "配置值",
            "配置项2": "配置值",
            "配置项3": "配置值"
        }]
    ]
}
```

其中一个比较常见的配置项是`usebuiltins`，该配置的默认值是false

配置`usebuiltins`可以在编译结果中注入这些新的API（例如：Promise，会从 core-js 库中导入），它的值默认为`false`，表示不注入任何新的API，可以将其设置为`usage`，表示根据API的使用情况，按需导入API

```
{
    "presets": [
        ["@babel/preset-env", {
            "useBuiltIns": "usage",
            "corejs": 3
        }]
    ]
}
```

## bebal 插件

预设和插件运行的顺序：

- 插件在 Presets 前运行。
- 插件顺序从前往后排列。
- Preset 顺序是颠倒的（从后往前）。

例如：

```
{
 "presets": ["a", "b"],
 "plugin": ["c", "d"]
}
那么运行顺序是： c -> d -> d -> a
```

其实就记得这个就行了，具体的插件，在框架中都预设好了~

# 性能优化

性能优化大概分成三个部分：

1. **构建**性能：开发阶段 -> 提升打包速度
2. **传输**性能：生成的 js 代码尽可能少
3. **运行**性能：代码怎么写的优雅

## 构建性能

### 减少模块解析

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748137664866-1b7400be-a0ed-4821-8c07-1fc95bceb9a3.png)

模块解析包括：抽象语法树分析、依赖分析、模块语法替换

#### 不做模块解析会怎样？

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748137665039-fee0b4d5-40cc-4c48-a670-cd1893a110d1.png)

如果某个模块不做解析，该模块经过loader处理后的代码就是最终代码。

如果没有loader对该模块进行处理，该模块的源码就是最终打包结果的代码。

**如果不对某个模块进行解析，可以缩短构建时间**

比如一些第三方库（jquery）向外提供的模块，已经是使用过构建工具打包后的结果，没有用到任何依赖。那么在构建的时候，在 webpack 中就可以不解析这个模块。

在 webpack.config,js 中配置

```js
module.exports = {
  output: {
    filneme: "[name]-[chunkhash:5].js",
  },
  module: {
    rules: [...],
    noParse: /jquery/
  }
}
```

### 优化 loader 性能

#### 限制 loader 应用范围

对于某些库，不使用 loader 去处理

比如说，lodash 本来就是兼容的，是用 es3 写的，那么就不需要 babel-loader 进行兼容性处理

通过`module.rule.exclude`或`module.rule.include`，排除或仅包含需要应用loader的场景

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /lodash/, // label-loader不需要对lodash这个库进行解析
        use: "babel-loader"
      }
    ]
  }
}
```

如果暴力一点，甚至可以排除掉`node_modules`目录中的模块，或仅转换`src`目录的模块

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/, // 一般第三方库都进行了兼容性处理
        //或
        // include: /src/,  // 只有src中的文件进行解析，其他不解析
        use: "babel-loader"
      }
    ]
  }
}
```

#### 缓存 loader 处理后的结果

只要文件内容不变，那么 loader 处理后的结果应该跟之前处理完的结果是一样的，那么使用缓存的话，就不用在一次进行处理了。

使用一个库：`cache-loader`

```js
module.exports = {
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /loadsh/,
        use: [
          "cache-loader",
          {
            loader: "babel-loader",
            options: {
              presets: [["@babel/preset-env"]],
            },
          },
        ],
      },
    ],
};
```

有趣的是，`cache-loader`放到最前面，却能够决定后续的loader是否运行

实际上，loader的运行过程中，还包含一个过程，即`pitch`

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748139102256-871c0a55-f08e-4827-9058-bb83c31342f1.png)

loader 的执行确实是从前往后的，如果 loader1 没有返回的话，就交给 loader2，如果有返回的话，那么直接返回结果。

所以，这就是 cache-loade 可以控制后面 loader 运行的原因。第一次执行时，cache-loader 看有没有缓存，如果没有就不返回值，那么就会继续交给后面的 loader 运行。下次执行 loader 时，此时有缓存了，那么 cache-loader 就直接用缓存值返回结果了。

#### 使用多线程来解析文件

在大项目中，可以使用这个，但是不能乱用。

库：`thread-loader`

### 热更新（HMR）

全称：hot module replacement，简单解释一下，热更新的概念：如果不开启热更新，那么更新代码会进行重新打包，重新刷新页面。开启热更新后，更新代码后，浏览器只会请求更新的那块代码，不会全部更新。

热替换并不能降低构建时间（可能还会稍微增加），但可以降低代码改动到效果呈现的时间

当使用`webpack-dev-server`时，考虑代码改动到效果呈现的过程

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748141933372-ab66aa94-392f-481d-85be-0a1bad18c593.png)

而使用了热替换后，流程发生了变化

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748141933723-af88f97b-9956-4045-8104-a971314ef6b8.png)

**【使用】**

在 webpack 中使用热更新，其实很简单，在 webpack v4 版本后，dev-server 默认是开启热更新的

```js
module.exports = {
  devServer: {
    open: true,
    hot: true // 手动开启，其实不写这句话也是开启的
  }
}

// index.js
import a from "./a";
import "./assets/index.css";
console.log(a);

// 开启热更新
if (module.hot) {
  module.hot.accept(); // 接受热更新
}
```

热更新原理 -> 了解即可

热更新怎么实现的呢？是通过`webSokect`来与服务器端通信的，改动代码后，服务器会知道，此时会发送一个信息给浏览器告诉它有东西改动了，然后浏览器会重新发一个请求给服务器，请求改动后的代码。

## 传输性能

### 手动分包

为什么需要分包呢？

是这样的，因为在开发过程中，可能几个文件都需要引用同一个公共库（也有可能是第三方库），此时，如果不做任何动作的话，每个 js 文件打包出来后，公共库代码也会在打包后的结果中。也就是说，**公共库在打包结果中重复了**。

这样，就会浪费很多存储空间，并且会增加很多传输时间。

所以我们需要分包，将公共库在打包结果后提取出来。

1. 先将公共库先打包 **【关键：需生成对应的描述文件，并且暴露全局变量】**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748226062194-3641b442-cd73-4b53-bc84-742e73c26a38.png)

1.1 创建公共模块的配置文件 -> 用于打包

```js
// 例如：
module.exports = {
  mode: "production",
  entry: {
    jquery: ["jquery"],
    lodash: ["lodash"],
  },
  output: {
    filename: "dll/[name].js",
    library: "[name]", // 暴露一个全局变量
  },
};
```

1.2 利用`DllPlugin`生成资源清单 -> 分析模块前会查看资源清单，如果有就不需要加载

```js
// webpack.dll.config.js
module.exports = {
  plugins: [
    new webpack.DllPlugin({
      path: path.resolve(__dirname, "dll", "[name].manifest.json"), //资源清单的保存位置
      name: "[name]"//资源清单中，暴露的变量名
    })
  ]
};
```

1.3 重新设置`CleanWebpackPlugin` -> 为了避免它把公共模块清除

```js
new CleanWebpackPlugin({
  // 要清除的文件或目录
  // 排除掉dll目录本身和它里面的文件
  cleanOnceBeforeBuildPatterns: ["**/*", '!dll', '!dll/*']
})
```

1.4 利用 `webpack.DllReferencePlugin` 将资源清单和打包好公共库关联起来

```js
module.exports = {
  plugins: [
    new webpack.DllReferencePlugin({
      manifest: require("./dll/jquery.manifest.json"),
    }),
    new webpack.DllReferencePlugin({
      manifest: require("./dll/lodash.manifest.json"),
    }),
  ],
}
```

2. 根据入口模块，正常打包

```bash
npx webpack --mode development
```

3. 在页面中引入公共库

```js
  <script src="./dll/jquery.js"></script>
  <script src="./dll/lodash.js"></script>
```

【使用 npm run build 工程目录】：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1748228351145-37eb4644-f18d-43b7-95a3-9fdd52920bdc.png)

### 自动分包

webpack 有默认的分包策略

webpack在内部是使用`SplitChunksPlugin`进行分包的

**分包基本配置**

webpack提供了`optimization`配置项，用于配置一些优化信息

其中`splitChunks`是分包策略的配置

```js
module.exports = {
  optimization: {
    splitChunks: {
      // 分包策略
    }
  }
}
```

1. chunks

我们知道，分包是从已有的chunk中分离出新的chunk，那么哪些chunk需要分离呢

chunks有三个取值，分别是：

- all：对于所有的chunk都要应用分包策略
- async：【默认】仅针对异步chunk应用分包策略 -> 【懒加载】
- initial：仅针对普通chunk应用分包策略

所以，**你只需要配置**`**chunks**`**为**`**all**`**即可**

2. maxSize

该配置可以控制包的最大字节数，一般不配置

如果要减少公共库的体积，只能使用压缩和 tree shaking

**其他配置**

如果不想使用其他配置的默认值，可以手动进行配置：

- automaticNameDelimiter：新chunk名称的**分隔符**，默认值~
- minChunks：一个模块被多少个chunk使用时，才会进行分包，默认值1
- minSize：当分包达到多少字节后才允许被真正的拆分，默认值30000

一般不需要修改默认配置

### 代码压缩

目标：减少打包出来后的文件体积 -> 用于**生产**环境

代码压缩工具：Terser

`webpack`安装后会内置`Terser`，当启用生产环境后即可用其进行代码压缩。

webpack自动集成了Terser

如果你想更改、添加压缩工具，又或者是想对Terser进行配置，使用下面的webpack配置即可

```js
const TerserPlugin = require('terser-webpack-plugin');
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin');
module.exports = {
  optimization: {
    // 是否要启用压缩，默认情况下，生产环境会自动开启
    minimize: true, 
    minimizer: [ // 压缩时使用的插件，可以有多个
      new TerserPlugin(), 
      new OptimizeCSSAssetsPlugin() // 开启css文件的压缩
    ],
  },
};
```

### tree shaking
>  只能在 *ESM* 的环境下使用（因为 CJM 是动态导入的，tree shaking 不能解析）

减少打包后文件的体积，用于生产环境下的优化 —— 将一些第三方库中没有用到的代码都删除掉
只要是生产环境，`tree shaking`自动开启

比如：
```js
// index.js
export function add(a,b) {
  return a+b;
}

export function jian(a,b) {
  return a-b;
}

// page.js
import {add} from "./index.js";
console.log(add(1,2));

在page.js中没有使用index.js中的方法，
所以tree shaking就会自动将jian这个方法在打包结果中删除掉
```

执行`npm run build` 打包后的结果

```
(()=>{"use strict";console.log(3)})();
```

当然这个结果也是自动被压缩过的 -> `terser`

---

【简要原理】

**tree shaking 是根据 es6 的导入导出语法进行的模块分析**

当解析一个模块时，webpack会根据ES6的模块导入语句来判断，该模块依赖了另一个模块的哪个导出

webpack之所以选择ES6的模块导入语句，是因为ES6模块有以下特点：

1. 导入导出语句只能是顶层语句
2. import的模块名只能是字符串常量
3. import绑定的变量是不可变的

因此，我们在编写代码的时候，**尽量**：

- 使用`export xxx`导出，而不使用`export default {xxx}`导出
- 使用`import {xxx} from "xxx"`导入，而不使用`import xxx from "xxx"`导入

依赖分析完毕后，`webpack`会根据每个模块每个导出是否被使用，标记其他导出为`dead code`，然后交给代码压缩工具处理

代码压缩工具最终移除掉那些`dead code`代码

#### css tree shaking

`webpack`无法对`css`完成`tree shaking`，因为`css`跟`es6`没有半毛钱关系

因此对`css`的`tree shaking`需要其他插件完成

例如：`purgecss-webpack-plugin`

注意：`purgecss-webpack-plugin`对`css module`无能为力

### 懒加载

就是一开始不加载那么多的 js 文件，当在进行完用户交互（比如点击之后）在加载相应的 js 文件

使用`import()` 动态导入

```js
const btn = document.querySelector(".btn");
btn.addEventListener("click", async () => {
  // 动态加载 -> 懒加载
  // 返回一个 promise 类似于 （* as obj）
  const obj = await import("./a.js");
  console.log(obj.add(1, 2));
});
```

## 开发实践： 减少构建速度

1. source map 配置，devtool

2. 生产：`source-map`
3. 开发：`cheap-module-source-map`

4. 模块解析，resolve

5. 配置别名： alias
6. 扩展名： extensions
7. 模块从哪里查找：modules

8. 限制loader解析范围：include / exclude、开启缓存

9. babel-loader

```js
options: {  
  cacheDirectory: true, // 优化点：启用babel缓存
  cacheCompression: false, // 优化点：禁用缓存压缩，提高构建速度
}
```

2. 使用 type: “asset” 代替file-loader

```js
{
  test: /\.(png|jpe?g|gif|webp)$/i,
    type: "asset", // 优化点：使用新的asset模块替代file-loader/url-loader
    parser: {
    dataUrlCondition: {
      maxSize: 8 * 1024, // 8kb以下的文件转为base64
     },
    },
    generator: {
      filename: "images/[name].[contenthash:8][ext]",
      },
    },
}
```

3. 使用 cache-loader 、thread-loader 、swc-laoder

```js
{
  test: /\.(ts|tsx|js|jsx)$/i,
    include: [sourcePath],
    exclude: /node_modules/,
    use: [
    'cache-loader',
    'thread-loader',
    {
      loader: 'swc-loader',
      options: {
        jsc: {
          parser: {
            syntax: 'typescript',
            tsx: true,
            jsx: true,
            dynamicImport: true,
            decorators: true,
          },
          transform: {
            react: {
              runtime: 'automatic',
              pragma: 'React.createElement',
              pragmaFrag: 'React.Fragment',
              throwIfNamespace: true,
            },
          },
          target: 'es2020',
        },
        cacheDirectory: true,
        cacheCompression: false,
      },
    },
  ],
},
```

还有其他的方法：

webpack5.0+ 可以开缓存

```js
cache: {
  type: 'filesystem', // type: 'memory'
    buildDependencies: {
    // 确保配置变动时失效缓存
    config: [__filename],
  },
},
```

最终结果：

原来构建一次 30-33s 左右，配置完后可以到达 10s 左右

# ESLint

检查代码风格的工具

eslint会识别工程中的`.eslintrc.*`文件，也能够识别`package.json`中的`eslintConfig`字段

# 配置

## env

配置代码的运行环境

- browser：代码是否在浏览器环境中运行
- es6：是否启用ES6的全局API，例如`Promise`等

## parserOptions

该配置指定`eslint`对哪些语法的支持

- ecmaVersion: 支持的ES语法版本
- sourceType

- script：传统脚本
- module：模块化脚本

## parser

`eslint`的工作原理是先将代码进行解析，然后按照规则进行分析

`eslint` 默认使用`Espree`作为其解析器，你可以在配置文件中指定一个不同的解析器。

## globals

配置可以使用的额外的全局变量

```
{
  "globals": {
    "var1": "readonly",
      "var2": "writable"
  }
}
```

`eslint`支持注释形式的配置，在代码中使用下面的注释也可以完成配置

```
/* global var1, var2 */
/* global var3:writable, var4:writable */
```

## extends

该配置继承自哪里

它的值可以是字符串或者数组

比如：

```
{
  "extends": "eslint:recommended"
}
```

表示，该配置缺失的位置，使用`eslint`推荐的规则

## ignoreFiles

排除掉某些不需要验证的文件

`.eslintignore`

```
dist/**/*.js
node_modules
```

## rules

`eslint`规则集

每条规则影响某个方面的代码风格

每条规则都有下面几个取值：

- off 或 0 或 false: 关闭该规则的检查
- warn 或 1 或 true：警告，不会导致程序退出
- error 或 2：错误，当被触发的时候，程序会退出

除了在配置文件中使用规则外，还可以在注释中使用：

```
/* eslint eqeqeq: "off", curly: "error" */
```

[https://zh-hans.eslint.org/docs/latest/rules/](https://zh-hans.eslint.org/docs/latest/rules/)