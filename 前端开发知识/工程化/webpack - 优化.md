# `<script>`标签
打包后，在 `index.html` 文件中，会引入`webpack`分包之后的`js`文件，有一些标签，在这里总结一下.
+ `defer`：控制脚本的加载和执行时机
	+ 在 HTML 文档解析完成之后，`DOMContentLoaded` 事件触发之前，按顺序执行
	+ 带有 `defer` 的脚本会在解析 html 文档的同时 **异步并行** 下载，不会阻塞 html 解析
	+ `defer` 对内联脚本（即没有 `src` 属性的`script`）无效
+ `async`：下载完立即执行，可能在 DOM 解析完成前
+ 普通的`<script>`标签： 下载完立即执行