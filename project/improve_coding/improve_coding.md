> 从这个名字即可看出，这是用于提升coding能力的，为我也为开发者

## 需求分析

## 系统设计

## 初始化项目
- `npm init`
- `.gitignore`
```js
	node_modules
	package-lock.json
```
- `eslintrc`
```json
{
  "extends": ["plugin:vue/base", "plugin:vue/recommended"],
  "plugins": ["vue"],
  "env": {
    "browser": true,
    "node": true,
  },
  "parser": "vue-eslint-parser",
  "parserOptions": {
    "parser": "babel-eslint",
    "ecmaVersion": 2017,
    "sourceType": "module",
  },

  "rules": {
    "no-unused-vars": [2, { "args": "none" }],
    "strict": "off",
    "valid-isdoc": "off",
    "jsdoc/require-param-description": "off",
    "jsdoc/require-param-type": "off",
    "jsdoc/check-param-names": "off",
    "jsdoc/require-param": "off",
    "jsdoc/check-tag-names": "off",
    "linebreak-style": "off",
    "array-bracket-spacing": "off",
    "prefer-promise-reject-errors": "off",
    "comma-dangle": "off",
    "newline-per-chained-call": "off",
    "no-loop-func": "off",
    "no-empty": "off",
    "no-else-return": "off",
    "no-unneeded-ternary": "off",
    "no-eval": "off",
    "prefer-destructuring": "off",
    "no-param-reassign": "off",
    "max-len": "off",
    "no-restricted-syntax": "off",
    "no-plusplus": "off",
    "no-useless-escape": "off",
    "no-nested-ternary": "off",
    "radix": "off",
    "arrow-body-style": "off",
    "arrow-parens": "off",
    "vue/multi-word-component-names": "off",
    "vue/valid-v-for": "off",
    "vue/no-multiple-template-root": "off",
  },
  "globals": {
    "$": true,
    "axios": true,
    "Vue": true,
  },
}
```
- `eslintignore`
```js
node_modules/
public/
```
配置`script`: `"lint": "eslint --quiet --ext js,vue ."` 检查当前目录下的 .js .vue 文件

- `validate-commit-msg` & `ghooks`
	规范 git 提交提示词，并且将其他规范工具集成到提交之前（eslint..)
```json
// 在 package.json 文件中写入
"config": {
    "ghooks": {
      "commit-msg": "validate-commit-msg",
      "pre-commit": "npm run lint"
    }
  }
```

# gitflow 规范
+ 主分支： main / master
+ 开发分支： develop 
	+ 功能分支： feature/xxx
具体可看： [git](../../general-tech/git.md)

# server-core 设计

## loader 设计
项目启动时，在 *运行时* 需要将所有的业务逻辑中，需要用到的配置或者说底层都加载到内存

业务代码： 大概目录结构
```text
app/
	- controller
	- extend
	- middleware 自定义中间件
	- public 静态资源目录
	- router
	- service 
	- config 
	  - config.local.js
	  - config.beta.js
	  - config.default.js
	  - config.prod.js
	- middleware.js 全局中间件
```

逻辑时这样的
请求 -> router 层 -> 调 controller 层 -> 调 service 层

**loader 的目的**：
将所有目录结构（除 public 外）都加载到内存中，挂载到 koa 实例上，可通过 `app.controller.xxx` 的方式进行访问

### router 兜底
注意： 写通配符 `*` ，koa 会优先匹配上，在匹配其他的路由，很有可能造成无限重定向

```js
 const indexPath = app?.options?.index ?? "/";
 // 这里不能写成 router.get("*", (xxx) => xxx)
  router.get(/(.*)/, async (ctx, next) => {
    if (ctx.path === indexPath || ctx.path === "/") {
      return next();
    }
    ctx.status = 302;
    ctx.redirect(indexPath);
  });
```

# 中间件设计
> 都是自定义的中间件设计

## 全局错误捕获
目的: 服务端的错误，不要直接暴露到前端。可能，数据库报错，将表结构信息泄露。

值得注意的是，写后端服务需要注意 日志 的采集

```js
module.exports = (app) => {
  return async (ctx, next) => {
    try {
      await next();
    } catch (error) {
      const { message } = error;
      
      app.logger.info(error);
      app.logger.error("-- [exception] --: ", error);
      app.logger.error("-- [exception] --: ", message);

      if (message && message.includes("template not found")) {
        ctx.status = 404;
        ctx.body = "页面不存在";
        return;
      }

      const body = {
        success: false,
        code: 50000,
        message: "服务器异常",
      };
      
      ctx.status = 200; // http 请求没问题，服务器逻辑有问题
      ctx.body = body;
    }
  };
};
```

## API 请求签名校验
目的： 提升相对安全，为每个 API 接口都添加唯一签名，以及判断接口的时效性。

采用: 对称加密、加盐哈希的方式
公式： `sign = md5(密钥 + 时间戳 + 随机值)`

注意： 密钥读取环境变量中的值，（开发时） git 忽略 .env 文件；（部署时） 通过docker等工具，配置化将密钥添加到宿主机。

```js
const path = require("path");
const md5 = require("md5");
const { SIGN_TIMEOUT } = require("../constant/sgin");

require("dotenv").config({ path: path.resolve(__dirname, "..", ".env") });

/**
 * 验证 API 请求签名
 */
module.exports = (app) => {
  return async (ctx, next) => {
    const { path, method } = ctx;
    const { headers } = ctx.request;
    const { s_t: st, s_sign: sSign, s_rand: sRand } = headers;

    if (!path.startsWith("/api/")) {
      await next();
      return;
    }
    
    // 使用 md5 对称加盐哈希
    const signKey = process.env.SIGN_KEY;
    const signature = md5(`${signKey}_${st}_${sRand}`);
    app.logger.info(`[${method} ${path}] - signature: ${signature}, sSign: ${sSign}`);
  
    if (!st || !sSign || signature !== sSign || Date.now() - Number(st) > SIGN_TIMEOUT) {
      ctx.status = 200;
      ctx.body = {
        success: false,
        code: 445,
        message: "签名验证失败或API超时",
      };
      return;
    }

    await next();
  };
};
```

## API 参数校验
还有一个问题，暂时不支持 params 参数的校验
为什么？
	因为设计原因，params 参数的读取（全局中间件中引入）需要等 router-loader 加载完后，koa-router 才会将 params 参数加入到 koa 的 ctx 上下文中... 而 router-loader 需要等其他的 loader 加载完后才能加载（需要依赖controller、service 等 loader ）
	
	因此，params 校验在 router-loader 之前，所以校验时值为 undefined

>  这里的设计，是结合了 `json-schema` 和 `ajv` 进行设计的

```js
const { BizError } = require("../utils/biz-error");
const Ajv = require("ajv");
const ajv = new Ajv();

const $schema = "http://json-schema.org/draft-07/schema#";

module.exports = (app) => {
  return async (ctx, next) => {
    const { path, method, params } = ctx;
    const { headers, body, query } = ctx.request;

    if (!path.startsWith("/api/")) {
      return await next();
    }

    app.logger.info(
      `[${method} ${path}] - params: ${JSON.stringify(params)}, query: ${JSON.stringify(query)}, body: ${JSON.stringify(body)}`,
    );

    const schema = app.routerSchema[path]?.[method.toLowerCase()];

    if (!schema) {
      return await next();
    }

    let valid = true;
    let validate = null;

    // 校验 headers
    if (valid && headers && schema.headers) {
      schema.headers.$schema = $schema;
      validate = ajv.compile(schema.headers);
      valid = validate(headers);
    }
    
    // 校验 query
    if (valid && query && schema.query) {
      schema.query.$schema = $schema;
      validate = ajv.compile(schema.query);
      valid = validate(query);
    }

    // 校验 body
    if (valid && body && schema.body) {
      schema.body.$schema = $schema;
      validate = ajv.compile(schema.body);
      valid = validate(body);
    }

    // 校验 params
    if (valid && params && schema.params) {
      schema.params.$schema = $schema;
      validate = ajv.compile(schema.params);
      valid = validate(params);
    }

    if (!valid) {
      app.logger.error(
        `[${method} ${path}] - API 参数校验失败: ${ajv.errorsText(validate.errors)}`,
      );
      throw BizError(442, `API 参数校验失败: ${ajv.errorsText(validate.errors)}`);
    }

    await next();
  };
};
```

# Webpack 工程化设计
这里就简单介绍一下，
分成 *开发* 和 *生产* 两个环境，对应不同的 webpack 配置文件，以及对应着不同的配置

还是，面向对象的思想，从基类中派生子类
```text
webpack.base.js
   	   |
	   | -- production -> merge.smart(webpackBaseConfig, {...})
	   |
	   | -- development -> merge.smart(webpackBaseConfig, {...})
```

### `webpack.base.js`
基础配置：
- 入口 entry
- 缓存 cache
- 基本解析模块 loader
- 插件 plugin
- 分包优化 optimization

```js
const webpack = require("webpack");
const path = require("path");
const glob = require("glob");

const { businessPath, publicPath, templatePath } = require("./path");
const { VueLoaderPlugin } = require("vue-loader");
const HtmlWebpackPlugin = require("html-webpack-plugin");

const entryList = {}; // 配置多页入口
const htmlWebpackPluginList = [];
const entryPath = path.join(businessPath, "/**/entry.*.js");
glob.sync(entryPath).forEach((file) => {
  // file 是路绝对路径
  const entryName = path.basename(file, ".js");
  entryList[entryName] = file;
  
  htmlWebpackPluginList.push(
    new HtmlWebpackPlugin({
      // 产物的输出目录
      filename: path.resolve(publicPath, "./dist", `./${entryName}.html`),
      // 指定要使用的模板
      template: path.resolve(templatePath, "./index.html"),
      // 要注入的代码块
      chunks: [entryName],
    }),
  );
});

// webpack 配置
module.exports = {
  // 入口配置
  entry: entryList,
  // 缓存配置
  cache: {
    type: "filesystem", // 持久化缓存 -> 默认存储在 node_modules/.cache/webpack
    buildDependencies: {
      config: [__filename], //当配置文件修改时，整个缓存失效
    }
  },
  // 模块解析配置
  module: {
    rules: [
      {
        test: /\.vue$/,
        use: "vue-loader",
      },
      {
        test: /\.js$/,
        include: businessPath,
        use: "swc-loader",
      },
      {
        test: /\.css$/,
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        use: ["style-loader", "css-loader", "less-loader"],
      },
      {
        test: /\.(png|jpeg|gif)(\?.+)?$/,
        use: {
          loader: "url-loader",
          options: {
            limit: 300,
            esModule: false,
          },
        },
      },
      {
        test: /\.(eot|svg|ttf|woff|woff2)(\?\S*)?$/,
        use: ["file-loader"],
      },
    ],
  },
  // 配置解析模块的具体行为
  resolve: {
    extensions: [".vue", ".js", ".less", ".json"],
    alias: {
      "@pages": path.resolve(businessPath),
      "@common": path.resolve(businessPath, "./common"),
      "@components": path.resolve(businessPath, "./components"),
      "@stores": path.resolve(businessPath, "./stores"),
      "@utils": path.resolve(businessPath, "./utils"),
      "@api": path.resolve(businessPath, "./api"),
    },
  },
  // 插件配置（在合适的时机干预 webpack 解析）
  plugins: [
    // 处理 .vue 文件，与 vue-loader 配合起来用
    new VueLoaderPlugin(),
    // 把第三方库暴露到 window context 下（提前下载这些模块，具体来说，这些变量的使用在模块中就不需要显示引入了）
    new webpack.ProvidePlugin({
      Vue: "vue",
      axios: "axios",
      _: "lodash",
    }),

    // 定义全局变量
    new webpack.DefinePlugin({
      __VUE_OPTIONS_API__: "true", // 支持 vue 解析 optionsAPI
      __VUE_PROD_DEVTOOLS__: "false", // 解析过程中禁用调式工具
      __VUE_PROD_HYDRTION_MISMATCH_DETAYILS__: "false",
    }),

    // 构建最终产物
    ...htmlWebpackPluginList,
  ],
  // 优化打包/构建
  optimization: {
    /**
     * 代码分割
     * 1. vendor 改动很少的一些第三方库
     * 2. commons 业务中公共的代码
     * 3. entry 业务中每个页面的入口代码
     */
    splitChunks: {
      chunks: "all", // 同步和异步的模块都进行分割
      maxAsyncRequests: 10, // 最大异步加载请求并行数
      maxInitialRequests: 10, // 最大初始加载请求并行数
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: "vendor",
          chunks: "all",
          priority: 20, // 优先级越高，越先被分割
          enforce: true, // 强制分割
          reuseExistingChunk: true, // 复用已有的 chunk
        },

        commons: {
          test: /[\\/]pages[\\/]/,
          name: "commons",
          chunks: "all",
          minChunks: 2, // 最少引用次数
          priority: 10,
          reuseExistingChunk: true, // 复用已有的 chunk
        },
      }
    },
  }
};
```

### `webpack.prod.js`
生产配置：
- 模式 mode
- 输出目录 output
- 打包压缩 optimization
	- Css -> `CssMinimizerPlugin` & `MiniCssExtractPlugin.loader` & `MiniCssExtractPlugin`
	- Js -> `TerserPlugin`

```js
const path = require("path");
const merge = require("webpack-merge");
const webpackBaseConfig = require("./webpack.base");
  
const { publicPath } = require("./path");
  
const TerserPlugin = require("terser-webpack-plugin");
const CssMinimizerPlugin = require("css-minimizer-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = merge.smart(webpackBaseConfig, {
    mode: "production",
    output: {
        clean: true, // 相当于 CleanWebpackPlugin
        // contenthash 使得浏览器缓存失效
        filename: "js/[name]_[contenthash:8].bundle.js",
        path: path.resolve(publicPath, "./dist/prod"),
        publicPath: "/dist/prod/",
        crossOriginLoading: "anonymous",
    },

    module: {
        rules: [
            {
                test: /\.css$/,
                use: [MiniCssExtractPlugin.loader, "css-loader"],
            },
            {
                test: /\.less$/,
                use: [MiniCssExtractPlugin.loader, "css-loader", "less-loader"],
            },
        ]
    },

    plugins: [
        // 提取 CSS 代码
        new MiniCssExtractPlugin({
            filename: "css/[name]_[contenthash:8].bundle.css",
        }),
    ],

    optimization: {
        splitChunks: { chunks: "all" },
        runtimeChunk: "single", // 提取 runtime 代码
        minimizer: [
            new CssMinimizerPlugin(), // 压缩 CSS 代码
            new TerserPlugin({
                minify: TerserPlugin.swcMinify, // 压缩 JS 代码，压缩使用 swc 
                terserOptions: {
                    compress: {
                        drop_console: true, // 删除 console.log
                    },
                },
            }),
        ],
    },
});
```

### `webpack.dev.js`
开发配置：
- 模式 mode
- 输出目录 output
- 源码地图 devtool
- 配置开发服务器 devServer -- 多页应用 -- 自定义开发服务器
	- Webpack 作为客户端
		- 需要给每个入口配置 hmr 的客户端
		- 配置插件支持在应用程序运行时替换模块 `HotModuleReplacementPlugin`
	- 手写 hmr 服务端 `express.js` -- 在这个服务中需要满足两个功能
		- 监听代码变动 - `webpack-dev-middleware`
		- 告诉浏览器变动 - `webpack-hot-middleware`

*webpack 配置 --- hmr 客户端*

```js
const webpack = require("webpack");
const merge = require("webpack-merge");
const path = require("path");
const webpackBaseConfig = require("./webpack.base.js");

const { publicPath } = require("./path");

// 开发服务器
const DEV_SERVER_CONFIG = {
    HOST: "127.0.0.1",
    PORT: 9002,
    HMR_PATH: "__webpack_hmr", // 官方规定，hmr客户端路径
    TIMEOUT: 20000,
    RELOAD: true,
}

// 注入hmr客户端
const { HOST, PORT, HMR_PATH, TIMEOUT, RELOAD } = DEV_SERVER_CONFIG;
Object.keys(webpackBaseConfig.entry).forEach(key => {
    // 第三方库改动较少，不需要注入hmr客户端
    if (key !== "vendor") {
        webpackBaseConfig.entry[key] = [
            webpackBaseConfig.entry[key],
            `webpack-hot-middleware/client?path=http://${HOST}:${PORT}/${HMR_PATH}&timeout=${TIMEOUT}&reload=${RELOAD}`,
        ]
    }
})

const webpackDevConfig = merge.smart(webpackBaseConfig, {
    mode: "development",
    devtool: "eval-cheap-module-source-map",
    output: {
        clean: true,
        filename: "js/[name].bundle.js",
        path: path.resolve(publicPath, "./dist/dev"),
        publicPath: `http://${HOST}:${PORT}/public/dist/dev/`,
    },
    // 开发环境插件
    plugins: [
        // 该插件允许在应用程序运行时替换模块
        new webpack.HotModuleReplacementPlugin({
            multiStep: true,
        }),
    ]
})
  
module.exports = {
    // webpack 开发环境配置
    webpackDevConfig,
    // 开发服务器配置
    DEV_SERVER_CONFIG
}
```

*express 服务端 --- hmr 服务端*

```js
// 使用 express 启动开发服务器
const express = require("express");
const path = require("path");
const webpack = require("webpack");
const consoler = require("consoler");
  
// hmr 中间件
const devMiddleware = require("webpack-dev-middleware");
const hotMiddleware = require("webpack-hot-middleware");

// 获取 webpack 开发环境配置和开发服务器配置
const { webpackDevConfig, DEV_SERVER_CONFIG } = require("./config/webpack.dev");

consoler.info("webpack 开发服务器正在启动...");

const app = express();
const complier = webpack(webpackDevConfig);

// 指定静态文件目录
app.use(express.static(path.resolve(__dirname, "../public/dist/")));

// 配置监听中间件（dev）
app.use(devMiddleware(complier, {
    // .html 文件 不加入内存的文件 --- 引入的资源在内存中 改动更频繁是引用的资源，修改后，hmr服务端将修改后的代码片段放入内容中，浏览器从内容中取更新后的代码
    writeToDisk: filename => filename.endsWith(".html"),
    // 资源目录
    publicPath: webpackDevConfig.output.publicPath,
    // 跨域
    headers: {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, PATCH, OPTIONS",
        "Access-Control-Allow-Headers": "X-Requested-With, Content-Type, Authorization",
    },
}))

// 配置通知中间件（hot）
app.use(hotMiddleware(complier, {
    path: `/${DEV_SERVER_CONFIG.HMR_PATH}`,
    log: () => { }
}))

const port = DEV_SERVER_CONFIG.PORT;
app.listen(port, () => {
    console.log(`开发服务器正在运行，端口：${port}`);
});
```

#### 开发服务器
问题：为什么不直接用 Proxy 代理呢？
	 直接启动一个 `webpack-dev-server` 端口 (8080) 和一个 Koa 端口（3003），然后互相代理？
+ 如果 Koa 只提供 API： 那么直接在 webpack 中配置 devServer 
+ 但是如果 Koa 要渲染 HTML：这么干的话，就比较麻烦了，可能会产生复杂的路径问题和模板注入问题
**因此**：使用*middleware* 模式，让 Koa 和 webpack 共用同一个上下文和内存文件系统（手动编写一个开发服务器）

---
由于，与热更新相关的两个中间件，都更加支持 express，使用 koa 需要兼容，所以将采用 express 启动 hmr 的服务器

+ **webpack-dev-middleware**: 将 Webpack 编译后的文件（存在内存里）映射到 Koa 的路由上
+ **webpack-hot-middleware**: 实现浏览器的热更新（HMR）

# DSL 领域模型设计
对于 ToB 的项目而言，开发者往往不仅仅只会维护一套系统，还有其他客户的定制化需求
但，这就造成了比较尴尬的场景
就我实习的公司而言，是在 master 的基础上拉取新的分支，莫条独立的分支用于维护某个特定的客户需求。

这样做的缺点，当客户到达一定程度，那么熵增就会越大。导致，管理混乱，分支杂乱。不同客户可能还是不同的版本。

改进作法：
将每一位客户的系统作为一个单页应用，这个应用中用到的组件，都是模板经过解析生成的。
[DSL领域模型](DSL领域模型.md)

-- 疑问： 不同版本之间，如果UI 不同，如果是复用同一套代码，好像也不对？



# 组件
通用组件 --> 记得写 initData 初始化数据 -- 保证数据渲染的正确性
