# `<script>`标签
打包后，在 `index.html` 文件中，会引入`webpack`分包之后的`js`文件，有一些标签，在这里总结一下.
+ `defer`：控制脚本的加载和执行时机
	+ 在 HTML 文档解析完成之后，`DOMContentLoaded` 事件触发之前，按顺序执行
	+ 带有 `defer` 的脚本会在解析 html 文档的同时 **异步并行** 下载，不会阻塞 html 解析
	+ `defer` 对内联脚本（即没有 `src` 属性的`script`）无效
+ `async`：下载完立即执行，可能在 DOM 解析完成前
+ 普通的`<script>`标签： 下载完立即执行

# 优化策略
## 存在问题
### 构建性能问题
*本地开发需要达到 30-32s 左右*
- 解析 loader 工具不行
- 范围
- ...

*未配缓存策略*
- loader 转化后的结果未被缓存 - cache-loader
- webpack5 持久化缓存未启用 - cache 选项

其实这里逻辑，很简单 
- 转化工具转化效率慢 - babel-loader -> swc-loader
- 转化结果基本不变 - 缓存 -> cache-loader
- 人多力量大 - 多进程转化 -> thread-loader
- 只转化必要转化的 - 有些第三方库已经是兼容模式的（loadsh），不转化 -> include | exclude
---
这是 loader 的角度，那么从整体看 webpack 也是优化这些东西
+ 分包 -> 目的：首屏加载 index.js ，按需加载其他 chunk
+ 缓存
+ ...

### 首屏构建时间过长
*主 `bundle` 体积过大*
 - `index.js ~ 15.5MB`
 -  压缩后 `index.js.gz ~ 3.6MB`

**根因分析:** *没有分包*
- 所有代码打包到单个 index.js，包括 node_modules
- 任何代码变更都会导致整个 bundle 失效
- 浏览器缓存无法有效利用
---

**影响:**
- 首屏加载时间长（3.6 MB 解压后仍然很大）
- 缓存策略失效（任何代码变更都会导致整个 bundle 失效 -> 导致了 hmr 也会很慢，需要 1.2 秒 左右）
因为，webpack 的热更新，是重新构建 bundle 及其子模块，重新构建后，在通过 hmr 的客户端替换浏览器之前未被修改的包



## 实现策略

### 提升构建性能

**webpack 5 内置的 cache 配置项**
开启本地持久化缓存，可缓存编译后的结果，缓存在本地。

但是，首次构建还是很慢
+ ”发现好用工具“ -> `swc-loader`
+ “人多力量大” -> `thread-loader`
+ “减少不必要的解析” -> 确定 loader 转化范围 `include / exclude`

#### swc-loader 配置
`npm i -D @swc/core swc-loader`

```js
{
  test: /\.(ts|tsx|js|jsx)$/i,
  include: [sourcePath],
  exclude: /node_modules/,
  use: [
    {
      loader: 'swc-loader',
      options: {
        jsc: {
          parser: {
            syntax: 'typescript', // 使用 ts 解析器
            tsx: true, // 启用 tsx/jsx 支持
          },
          transform: {
            react: {
              runtime: 'automatic', // 使用react 17+ 自动 jsx 运行时
            },
          },
        },
      },
    },
  ],
},
```

#### cache 配置
*cache-loader 配置*

*webpack5 cache 配置*
```js
cache: {
  type: 'filesystem',
  cacheDirectory: resolve(dirname, '.webpack_cache'),
  // 缓存配置
  buildDependencies: {
    config: [__filename], // 配置文件变更时重建缓存
  },
  // 算法版本（依赖变更时更新）
  version: `${pkg.version}-${process.env.NODE_ENV}`,
  // 压缩缓存以减少磁盘占用
  compression: 'gzip',
  // 性能配置
  maxAge: 1000 * 60 * 60 * 24 * 7, // 7 天
  maxMemoryGenerations: 1, // 限制内存中的缓存代数
},
```

#### 多进程配置


### 首屏优化
> 那么这里采用最常见的处理方式，分包，懒加载

*分包优化*

> 这里有发现一些其他的问题，
> 问题一：引入的其他自定义组件，体积特别大。基本上都能占到 1 MB 左右（pkg-icon、q-alarm、antd）
> 问题二：有些库在项目根目录的依赖中已经有了，在fusion项目再一次引入了一遍(echarts、moment...)

```js
optimization: {
  runtimeChunk: 'single', // 值 "single" 会创建一个在所有生成 chunk 之间共享的运行时文件
  splitChunks: {
	  chunks: 'all',
	  maxSize: 1024 * 1024, // 1MB
	  cacheGroups: { // 缓存组
    // ============ 核心依赖 ==============
    react: { 
      test: /[\\/]node_modules[\\/](react|react-dom|react-router-dom)[\\/]
      name: 'react-core',
      priority: 50,
      reuseExistingChunk: true,
    },
    // ============ 状态管理 ==============
    state: {
      test: /[\\/]node_modules[\\/](mobx|mobx-react|concent)[\\/]/,
      name: 'state-management',
      priority: 40,
      reuseExistingChunk: true,
    },
    // ============ ui 组件库 ==============
    antd: {
      test: /[\\/]node_modules[\\/](@ant-design|antd|rc-)[\\/]/,
      name: 'antd-ui',
      priority: 35,
      reuseExistingChunk: true,
    },
    // ============ 自定义组件 ==============
    'woqutech-pkg': {
      test: /[\\/]node_modules[\\/](@woqutech|q-theme-default)[\\/]/,
      name: 'woqutech-pkg',
      priority: 30,
      reuseExistingChunk: true,
    },
    // ============ 图表库 ==============
    echarts: {
      test: /[\\/]node_modules[\\/](echarts|echarts-for-react)[\\/]/,
      name: 'echarts',
      priority: 25,
      reuseExistingChunk: true,
    },
    // ============ 工具库 ==============
    utils: {
      test: /[\\/]node_modules[\\/](lodash|dayjs|moment|qs|classnames)[\\/
      name: 'utils',
      priority: 20,
      reuseExistingChunk: true,
    },
    // ============ 微前端 ==============
    qiankun: {
      test: /[\\/]node_modules[\\/](qiankun|import-html-entry)[\\/]/,
      name: 'qiankun',
      priority: 15,
      reuseExistingChunk: true,
    },
    // ============ 其他第三方依赖 ==============
    vendors: {
      test: /[\\/]node_modules[\\/]/,
      name: 'vendors',
      priority: 10,
      reuseExistingChunk: true,
      minChunks: 2, 
    },
    // ============ 公共模块 ==============
    common: {
      test: /[\\/]src[\\/](components|hooks|utils|services)[\\/]/,
      name: 'common',
      minChunks: 2,
      priority: 5,
      reuseExistingChunk: true,
    },
    default: {
      minChunks: 2,
      priority: -10,
      reuseExistingChunk: true,
    },
	  },
  },
  minimize: envConf.isMini,
  minimizer: [
    '...',
    !envConf.isDev && new CssMinimizerPlugin(),
    !envConf.isDev &&
      new TerserPlugin({
        terserOptions: {
          compress: {
            drop_console: true,
            pure_funcs: ['console.log', 'console.info'],
          },
        },
        extractComments: false,
        parallel: true,
      }),
  ].filter(Boolean),
}
```

那么，分包之后出现了一个问题
分包数量过多，能到达 124个 js 文件，太多了...

![](assets/webpack%20-%20优化/file-20260129102154165.png)

处理的方法 --> 最好能达到首屏只加载 20-30 个文件
- 非首屏代码，按需加载
- webpack 预加载

---
 首屏必需的核心代码必须同步加载：
  - React、React DOM
  - 首屏路由对应的组件
  - 核心状态管理（mobx、concent）
  - 基础 UI 库（antd）

*先挑软柿子*
其实，可以在 index.html 看到，echarts 引入了很多文件。但实际上，qfusion 项目中使用 echarts 库的频率不高。
![500](assets/webpack%20-%20优化/file-20260129103933039.png)

qfusion 中，只有 Chart 这个组件用到了 echarts 库
![500](assets/webpack%20-%20优化/file-20260129104047870.png)

两种方案：
- 懒加载
- echarts 按需导入
--> 最终采用 懒加载 + 按需导入 100kb左右。事实上，改起来不太容易。因为，echarts 4.x 版本不支持 esm 的按需导入。
查阅资料可知，echarts 4.x 已于2022年停止更新，并且存在 xss [CVE-2021-37127](https://nvd.nist.gov/vuln/detail/CVE-2021-37127?spm=a2ty_o01.29997173.0.0.4fff5171sxywXn)缺陷。建议升级至 echarts 5.5.1 版本，90%兼容 4.x，支持按需导入，支持tree shaking。兼容 node 16 + react 17。

| 类型  | 位置                                |
| --- | --------------------------------- |
| 折线图 | SQLDiagnosisDetails/index.jsx:494 |
| 饼图  | ResourceDetails.jsx:250           |

升级 echarts 版本为 5.5.1，改写为函数组件的写法
```js
import React, { useRef, useEffect, useState } from 'react'
import classNames from 'classnames'
import ResizeObserver from 'resize-observer-polyfill'
import isEqual from 'lodash/isEqual'
import throttle from 'lodash/throttle'

import './index.scss'

const Chart = ({ className = '', option }) => {
  const chartRef = useRef(null)
  const chartInstance = useRef(null)
  const resizeObserver = useRef(null)
  const [isReady, setIsReady] = useState(false)

  useEffect(() => {
    let active = true

    const initChart = async () => {
      // 懒加载，按需加载 ECharts 模块
      const [
        echarts,
        { LineChart, PieChart },
        { TitleComponent, TooltipComponent, GridComponent, LegendComponent },
        { CanvasRenderer },
      ] = await Promise.all([
        import('echarts/core'),
        import('echarts/charts'),
        import('echarts/components'),
        import('echarts/renderers'),
      ])

      if (!active) return

      // 注册组件
      echarts.use([
        LineChart,
        PieChart,
        TitleComponent,
        TooltipComponent,
        GridComponent,
        LegendComponent,
        CanvasRenderer,
      ])

      if (chartRef.current) {
        // 初始化
        const instance = echarts.init(chartRef.current)
        chartInstance.current = instance
        instance.setOption(option)
        setIsReady(true)

        const ro = new ResizeObserver(
          throttle(() => {
            instance.resize({ animation: false })
          }, 150)
        )
        ro.observe(chartRef.current)
        resizeObserver.current = ro
      }
    }

    initChart()

    return () => {
      active = false
      if (resizeObserver.current) {
        resizeObserver.current.disconnect()
      }
      if (chartInstance.current) {
        chartInstance.current.dispose()
        chartInstance.current = null
      }
    }
  }, [])

  useEffect(() => {
    if (isReady && chartInstance.current) {
      const currentOption = chartInstance.current.getOption()
      if (!isEqual(option, currentOption)) {
        chartInstance.current.setOption(option)
      }
    }
  }, [option, isReady])

  return <div ref={chartRef} className={classNames('chart', className)} />
}

export default Chart

```

就在你以为万事大吉的时候，骗你的，其实 index.html 还是加载了 echarts 相关的 js 文件
那怎咋办呢？

原因
- echarts 相关的 node_modules 会被强制打包到独立的 echarts.js chunk 中
- 当 webpack 分析依赖时，如果发现任何模块（包括动态 import）使用了 echarts，就会提前生成这个 chunk

方法：
删掉相关配置 -> 但是有个问题，会被打入 node_moudles 这个包中
- 为什么？
	- 最根本的原因是存在 *隐式调用*，虽然在 qfusion 这个项目中处理了 echarts 的引入，但是在 q-bus-monitoring-alarm 项目中依然存在 *同步* 引入 echarts 库
	- 同时，所有的数据库模块都是 *同步* 引入的`ClusterDetails` -> `MonitorAddAlarm` -> `q-bus-monitoring-alarm`



