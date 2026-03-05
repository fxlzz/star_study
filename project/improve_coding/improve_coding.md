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

### Webpack. Base. Js

### Webpack. Prod. Js

### Webpack. Dev. Js

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

