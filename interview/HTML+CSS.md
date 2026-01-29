# 一、css 选择器有哪些？优先级分别是什么？哪些属性可以继承?

1. **基础**选择器

```css
.box {}  类选择器
#box {}  id 选择器
div {}   元素选择器
* {}     通配符选择器
```

2. **组合**选择器

```CSS
ul li {}    后代选择器
div > p {}  子选择器（>） 仅div的直接子元素 p
h2 + p {}   相邻兄弟选择器（+） 紧跟在h2后面的 p
h2 ~ p {}   通用兄弟选择器 (~)  h2 后面的所有 p
```

3. **伪类**选择器

```css
状态伪类：
a:hover {} input:focus {} input:disabled {}
位置伪类：li后面的第几个元素
li:first-child {} last-child {} nth-child(odd) {} 
结构伪类：同类型的第几个元素
p:first-of-type {} last-of-type nth-of-type(odd) {}
表单相关伪类：
input:required {} input:valid {}
```

4. **属性**选择器

```css
[disabled] {}       存在选择器  选择具有disabled的元素
[type="text"] {}    值匹配选择器  精准匹配到属性值
[class~="error"] {}              属性值包含单词 class包含 error的元素
a[href^="https"] {} 部分匹配选择器 属性以什么(https)开头
[src$=".jpg"] {}                  属性以什么(.jpg)结尾
```

5. **伪元素**选择器

```css
p::before { content: "→ "; } 					/* 在元素前插入内容 */
p::after { content: " ←"; }  					/* 在元素后插入内容 */
p::first-letter { font-size: 2em; }	  /* 首字母 */
p::first-line { font-weight: bold; }  /* 首行 */
input::placeholder { color: #999; }   /* 占位文本 */
p::selection {}                       /* 选中文字样式 */
```

6. **否定**选择器

```css
li:not(.active) { opacity: 0.5; } /* 非 active 类的 <li> */
input:not([type="submit"]) { margin-bottom: 10px; } /* 非提交按钮 */
```

## **优先级**

1. 元素选择器、伪元素选择器
2. 类选择器、属性选择器、伪类选择器
3. ID 选择器
4. 内联样式（如 `<div style="color: red">`）
5. `!important`（最高，但不推荐滥用）

## [CSS中哪些属性可以继承](https://blog.csdn.net/karlaofsky/article/details/135990234)

**可继承的属性**

- 字体属性：包括_font_、_font-family_、_font-weight_、_font-size_、_font-style_。
- 文本属性： 内联元素：_color、line-height、word-spacing、letter-spacing、text-transform。_

_块级元素：text-indent、text-align。_

- 元素可见性：_visibility_。
- 表格布局属性：_caption-side_、_border-collapse_、_border-spacing_、_empty-cells_、_table-layout_。
- 列表布局属性：_list-style_。

**不可继承的属性**

- 边框属性：_border_。
- 内边距属性：_padding_。
- 外边距属性：_margin_。
- 定位属性：_position_。
- 大小属性：_width_、_height_。
- 背景属性：_background_。
- 盒模型属性：_box-sizing_。
- 显示属性：_display_。
- 文本属性：_vertical-align_、_text-decoration_。
- 定位属性：_float_、_clear_、_top_、_right_、_bottom_、_left_、_min-width_、_min-height_、_max-width_、_max-height_、_overflow_、_clip_。

需要注意的是，尽管某些属性可以继承，但子元素仍可以通过显式设置自己的值来覆盖继承的值。此外，有些属性可以通过使用 **_inherit_** 关键字来强制继承父元素的值。

# 二、css 怎么画一个大小为父元素面积一半的正方形

【方法一】： 用 `calc()`计算出子元素的长宽

```css
.parent {
  width: 200px; 
  height: 200px;
  background: #f1f1f1;
  position: relative;
}

.square {
  position: absolute;
  width: calc(100% * (sqrt(2)/2));   /* 设置定位 子元素的100%就可以继承到父元素的宽度了 */
  height: calc(100% * (sqrt(2)/2));
  background: #4a90e2;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%); /* 居中 */
}
```

【方法二】：使用`padding-top`

```css
.square {
  width: 400px;
  height: 400px;
  background-color: lightblue;
  position: relative;
}
.content {
  position: absolute;
  top: 0;
  left: 0;
  width: 50%;
  padding-top: 50%;  /* 不设置高度，用padding-top挤开 */
  display: flex;
  align-items: center;
  justify-content: center;
  outline: 1px solid red;
}
```

【方法三】：使用 css 一个属性`**aspect-ratio**`

这个属性，可以直接设置元素的宽高比

```css
.square {
  width: 50%;             /* 宽度为父元素的一半 */
  aspect-ratio: 1 / 1;    /* 宽高比为 1:1 */
  background-color: lightblue; 
  display: flex;
  align-items: center;
  justify-content: center;
}
```

# 三、flex 实现 8 个元素分两行排列，每行 4 个平均分布？

这个简单，

```css
.box {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-around;
  gap: 20px;
}
.item {
  width: calc(25% - 20px);  /* 主要是这个宽度的计算 */
  height: 50px;
  background-color: antiquewhite;
}
```

## 【加一问】：怎么实现三栏布局？

```css
.box {
  display: flex;
  justify-content: space-between;
  height: 100vh;
}
.left {
  width: 200px;
  background-color: aliceblue;
}
.right {
  width: 200px;
  background-color: antiquewhite;
}
.center {
  /* flex: flex-shrink: 1 flex-grow: 1 flex-basis: 0 */
  flex: 1;
}
```

## 【加一问】：怎么实现左侧固定，右侧自适应？

```css
/* flex布局 */
display: flex;
.left {
  width: 200px;
  height: 100%;
}
.right {
  flex: 1;
}

/* BFC实现 */
.left {
  width: 200px;
  float: left;
}
.right {
  overflow: hidden;
}
```

# 四、怎么实现样式隔离？

【方法一】： 可以使用 CSS 模块化

命名为：xxx.module.css

使用：`import styles from "./xxx.module.css"`

【方法二】：使用 CSS-in-JS

在 JavaScript 中编写 CSS，通过生成唯一类名实现隔离

```css
import styled from 'styled-components';

const Container = styled.div`
  display: flex;
  background: white;
`;

const Title = styled.h1`
  color: blue;
`;

const MyComponent = () => (
  <Container>
  <Title>Hello World</Title>
  </Container>
);
```

# 五、怎么用 css 水平垂直居中？

## 1、行内元素或文本元素

```css
/* 单行文本垂直居中 */
.parent {
  text-align: center;    /* 水平居中 */
  line-height: 100px;    /* 与容器高度相同 */
  height: 100px;
}

/* 多行文本垂直居中 */
.parent {
  display: table-cell;
  text-align: center;    /* 水平居中 */
  vertical-align: middle; /* 垂直居中 */
  height: 100px;
}
```

## 2、块级元素

适用于**已知或未知宽高**的块级元素

```css
/* flex布局 */
.parent {
  display: flex;
  justify-content: center; /* 水平居中 */
  align-items: center;     /* 垂直居中 */
}

/* grid布局 */
.parent {
  display: grid;
  place-items: center; /* 同时实现水平和垂直居中 */
}
```

## 3、定位实现居中

适用于需要**脱离文档流**的元素

```css
/* 已知宽高 */
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 50%;  // calc(50% - 50px);
  left: 50%; // calc(50% - 50px);
  width: 100px;
  height: 100px;
  margin-top: -50px;  /* 高度的一半的负值 */
  margin-left: -50px; /* 宽度的一半的负值 */
}

/* 未知宽高 */
.parent {
  position: relative;
}

.child {
  position: absolute;
  top: 50%;	
  left: 50%;
  transform: translate(-50%, -50%); /* 向上向左移动自身宽高的50% */
}
```

## 4、flex 加 margin

```css
.parent {
  display: flex; /* 或者display: grid */
}

.child {
  margin: auto; /* 上下左右自动分配空间 */
}
```

## 5、verticle-align:middle

要生效，需要是 display: inline-block 并且在使用verticle-align:middle 时需要有一个兄弟元素做参照物，让他垂直于兄弟元素的中心点。

```css
<div class="container">
  <div class="item"></div>
</div>

<style>
.container{
  width: 500px;
  height: 300px;
  background-color: pink;
  text-align: center;
}
/* 这个是为了 vertical-align: middle; 做参考 */
.container::before{ 
  content : '';
  display: inline-block;
  vertical-align: middle;
  height: 100%;
}
.item{
  width: 100px;
  height: 100px;
  background-color: skyblue;
  vertical-align: middle;
  margin: 0 auto;
  display: inline-block;
}
</style>
```

# 六、css 隐藏元素方式

分为三大类，

- 完全隐藏：元素不在布局树（渲染树）上，不占空间。
- 视觉上隐藏：屏幕中不可见，占据空间。
- 语义上隐藏：读屏软件不可读，但正常占据空间。

## 完全隐藏

**display: none**

- 元素从文档流中移除，**不****占据空间**。
- **不响应**任何事件（如点击）。
- 不会被屏幕阅读器等辅助设备读取。
- 子元素也会被完全隐藏。

**hidden** 属性，相当于 display: none，直接写在元素上面，例如：

```
<div hidden></div>
```

## 视觉上隐藏

**1、visibility: hidden**

- 元素隐藏，但仍**占据****原****空间**（保留布局）。
- **不响应**任何事件。
- 不会被屏幕阅读器读取。
- 子元素可以通过 `visibility: visible` 单独显示。

**2、opacity: 0**

- 元素完全透明，但仍**占据空间**且可点击。
- **响应**所有事件（如点击、悬停）。
- 会被屏幕阅读器读取。

**3、transform: scale(0)**

- 元素缩放到不可见，但仍**占据****原****空间**。
- **不响应**事件（缩放后元素实际不存在于视觉区域）。
- 可通过过渡动画实现缩放效果。

```css
.hidden {
  transform: scale(0);
  transform-origin: top left; /* 可选：设置缩放原点 */
}
```

**4、clip-path**

- 元素被裁剪为不可见，但仍**占据空间**。
- **不响应**裁剪区域外的事件。

```css
.hidden {
  clip-path: polygon(0 0, 0 0, 0 0, 0 0); /* 裁剪为无面积的点 */
}
```

## 语义上隐藏

_**aria-hidden**_ **属性**

通过设置 _aria-hidden_ 属性为 _true_ 使读屏软件不可读，但是元素仍然占据空间并且可见。

```
<div aria-hidden="true"></div>
```

# 七、定位

目前在 _CSS_ 中，有 _5_ 种定位方案，分别是：

- _static_ 静态定位
- _relative_ 相对定位
- _absolute_ 绝对定位
- _fixed_ 固定定位
- _sticky_ 粘性定位

## _static_ 静态定位

所谓静态定位，就是我们的标准流。

在标准流里面，块盒元素独占一行，行盒元素共享一行。静态定位是 _HTML_ 元素的默认值，静态定位的元素不会受到 _top，bottom，left，right_ 的影响。

## _relative_ 相对定位

相对定位的参照点是，它自己，并且相对定位是**不脱离标准流的**

```css

<style>
  * {
    padding: 0;
    margin: 0;
  }

  div {
    width: 100px;
    height: 100px;
    outline: 1px solid;
    line-height: 100px;
    text-align: center;
  }

  .two {
    position: relative;
    top: 50px; // 相对于之前的位置 top、left
    left: 50px; 
  } 
</style>

<div class="one">one</div>
<div class="two">two</div>
<div class="three">three</div>
```

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751366245105-72847010-9bd1-45aa-92c4-55acfe0a6317.png)![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751366319781-48951f05-674d-479c-9968-5f50791f200e.png)

## absolute 绝对定位

绝对定位呢，要搞清楚，它的参考点是它的祖先元素的第一个有定位的元素。同时，如果使用 top 来描述，那么该元素的参考点是**页面的左上角，**如果是 bottom 来描述，参考点是**最开始视口的左下角**

**top 描述：**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751423459617-e96dffa3-c4e2-404c-8206-404cb2635323.png)

**bottom 描述：**

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751423489296-4c926183-682a-4344-b327-8803dc1f6b18.png)

但是上面两种情况，用的都比较少，常用的是给父元素加一个相对定位

这样的好处在于该元素的**父级元素没有脱离标准流**，该元素将会以这个相对定位了的父元素作为参考点，在父元素的范围内进行移动，方便我们对元素的位置进行掌控。如下图：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751423979129-ccede6cf-6c2f-4d35-9eed-e665e0a0f0d7.png)

还有一个，设置了 absolute 的元素会变成块盒元素。

比如，

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751423188361-f8d498fd-c813-4102-8916-38ba9b64e9a6.png)这是用没加定位之前的 span 元素`<span>i am a inline-block</span>`

加了定位只后，

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751423242472-d7710060-7141-427e-9a44-4f6c0637c163.png)这个 span 元素变成了块盒元素，宽高都能设置了

```css
span {
  position: absolute;
  top: 50px;
  left: 50px;
  width: 100px;
  height: 100px;
  outline: 1px solid;
}
```

## fixed 固定定位

这个类似于绝对定位的变种，这个是随着浏览器当前视口走的。

即使用了滚动条，设置了这个定位的元素，不会动一点（但是，绝对定位会动，因为它是跟者最开始的浏览器定位的）

```css
<style>
  * {
    margin: 0;
    padding: 0;
  }
  body {
    height: 5000px;
  }
  div {
    width: 100px;
    height: 100px;
    background-color: red;
    position: fixed;
    right: 100px;
    bottom: 200px;
  }
</style>
</head>
<body>
  <div></div>
</body>
```

## sticky 粘性定位

这是 CSS3 中新出的定位，这个的效果就是导航栏随着滑动走的那种效果。

```css
<style>
  * {
    margin: 0;
    padding: 0;
  }
  body {
    height: 5000px;
  }
  header h2 {
    text-align: center;
    margin: 10px 0;
  }
  nav {
    position: sticky;
    /* 当nav这个元素离浏览器还有20px时，让它随着浏览器一起滚动 */
    top: 20px;
  }
  nav ul {
    display: flex;
    justify-content: space-evenly;
    background-color: rgba(0, 0, 0, 0.5);
  }
  nav ul li {
    cursor: pointer;
    width: 100%;
    height: 30px;
    line-height: 30px;
    text-align: center;
    list-style: none;
    color: #fff;
  }
  nav ul li:hover {
    background-color: #333;
  }
</style>
<body>
  <header>
    <h2>这是一个随便写的网站</h2>
  </header>
  <nav>
    <ul>
      <li>导航1</li>
      <li>导航2</li>
      <li>导航3</li>
      <li>导航4</li>
      <li>导航5</li>
    </ul>
  </nav>
</body>
```

## 相对定位和绝对定位的区别

|   |   |   |
|---|---|---|
||绝对定位|相对定位|
|定位基准|根据已定位的祖先元素偏移，若没有已被定位的元素，则会根据浏览器窗口定位|相对于元素本身在文档流的初始位置进行偏移|
|对于文档流的影响|脱离文档流，影响布局，可以用<br><br>z-index 来维护层级|保留文档流的位置|

# 八、BFC 是什么？

BFC：是块级格式化上下文；

**意思就是，有独立的布局环境，内容元素布局与外部无关**

开启 BFC：

- float : left / right
- position: absolute / fixed
- display: inline-block / flow-root / flex / grid
- overflow: auto / hidden / scroll
- contain: layout / content

作用：

1. 让非浮动元素不要被浮动元素盖住 -> 两栏布局

这个也是浮动元素会脱标造成的，只要给非浮动元素设置成 BFC 盒子即可

2. 解决父元素高度塌陷问题，

子元素撑开了父元素的高度，如果子元素设置了 float（脱离标准流）那么父元素就会高度坍塌。这时候给父元素设置一个 BFC 就可以解决

3. 防止 margin 重叠

如果两个块盒元素，同时在同一方向设置了 margin，那么浏览器会选择最大的那个 margin

解决的话，在中间加一个盒子，设置成 BFC

【扩展】：

IFC、GFC、FFC

- _IFC_（_Inline formatting context_）：翻译成中文就是“行内格式化上下文”，也就是一块区域以行内元素的形式来格式化
- _GFC_（_GrideLayout formatting contexts_）：翻译成中文就是“网格布局格式化上下文”，将一块区域以 _grid_ 网格的形式来格式化
- _FFC_（_Flex formatting contexts_）：翻译成中文就是“弹性格式化上下文“，将一块区域以弹性盒的形式来格式化

# 九、盒模型是什么？

CSS 中有两种盒模型，一种是标准盒模型，一种是 IE 盒模型

标准盒模型：`box-sizing: content-box`

width/height：仅指的是 content 区域

总宽度：width+padding*2+border*2+margin*2

IE 盒模型：`box-sizing: border-box`

width/height 包含 padding 和 border

总宽度：width+margin*2

【注意】：margin 两个盒子都**不算**到宽度或高度中

# 十、CSS 中的单位

在 CSS 中，单位大致分为**绝对单位**和**相对单位**

**【绝对长度单位】**

常见的绝对单位长度如下：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751341804355-9e337bc5-9303-4b8f-9bb0-0abc497e0d3f.png)

唯一一个经常使用的值，估计就是 _px_(像素)

**【相对长度单位】**

下表列出了 _web_ 开发中一些最有用的单位

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751341889638-0a2a1efe-fc13-435a-bf64-308235ac79e1.png)

上面的单位中，常用的有 _em、rem、vw、vh_，其中 _vw_ 和 _vh_ 代表的是视口的宽度和高度，_em_ 在 font-size 中使用，是相对于**父元素的字体大小**，其他属性是相对于**自身的字体大小**，如 width，_rem_ 是相对**根元素**的**字体大小**。

例如：

```css
<div>
  我是父元素div
  <p>
    我是子元素p
    <span>我是孙元素span</span>
  </p>
</div>
```

```css
* {
  margin: 0;
  padding: 0;
}

html {
  font-size: 10px;
}

div {
  font-size: 4rem; 
  /* 40px  html的字体大小为 10px rem是相对这个数的，4*10px*/
  width: 30rem;
  /* 300px */
  height: 30rem;
  /* 300px */
  outline: solid 1px black;
  margin: 10px;
}

p {
  font-size: 2rem;
  /* 20px */
  width: 15rem;
  /* 150px */
  height: 15rem;
  /* 150px */
  outline: solid 1px red;
}

span {
  font-size: 1.5rem;
  width: 10rem;
  height: 10rem;
  outline: solid 1px blue;
  display: block;
}
```

# 十一、浮动

浮动现在一般用于做文字环绕的效果，布局已经不用了，有 flex 和 grid 专门用来布局。我感觉面试应该也不会考这种东西，可能会随着 BFC 问一下吧。了解一下即可。

## 浮动的特性

1. 脱离标准流，就跟绝对定位一样
2. 浮动的元素会互相贴靠
3. 宽度收缩

在没有设置宽度的情况下，块级元素在标准流时很多时独占一行，宽度也会占满整个容器，但是一旦被设置为浮动后，宽度就会收缩。

例如：

```
<div>this is a test</div>
```

```
div{
  background-color: red;
  float: left;
} 
这样设置过后，div的width会收缩到与文字的width一样
```

本来 _div_ 是占满整行的，但是当我们设置了浮动后，由于 _div_ 又没有设置宽度，所以宽度就收缩了。

## 清除浮动

有下面这个场景，

```css
 <style>
  li {
    float: left;
    width: 100px;
    height: 20px;
    background-color: pink;
  }
</style>

<ul>
  <li>导航1</li>
  <li>导航2</li>
  <li>导航3</li>
</ul>
<ul>
  <li>游戏</li>
  <li>动漫</li>
  <li>音乐</li>
</ul>
```

结果：应该是导航 1 一行，游戏一行，但是因为给 li 设置了 float（浮动的元素会贴靠在一起）这显然不是我们想要的，所有要解决这个浮动带来的副作用。

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751362268509-dc403098-2c45-48ef-86df-275c1ed402c8.png)

清除浮动后的结果：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751362491157-b7530e7f-f7e3-473c-bd11-e491ad7a2f67.png)

**方法一：给父元素设置高度**

注意，设置的父元素高度一定要超过子元素的高度

```css
ul {
  height: 50px;
}
```

**方法二：使用 clear 属性**

- left： 左侧不允许浮动
- right：右侧不允许浮动
- both：两端都不许
- none：默认，允许
- inherit：继承至父元素的 clear 属性的值

但是，直接使用这个元素，虽然没有浮动，但是 margin 也不会生效

所以，会创建一个空白的元素，它来设置这个属性

```css
.clearfix {
  clear: both;
}

<ul>
  <li>导航1</li>
  <li>导航2</li>
  <li>导航3</li>
  /* <div class="clearfix"></div>  放到父元素里面也可以*/ 
</ul>
<div class="clearfix"></div>
<ul>
  <li>游戏</li>
  <li>动漫</li>
  <li>音乐</li>
</ul>
```

**方法三：overflow 属性**

其实就是创建了一个 BFC，隔离开了。给想隔离的元素，加上 overflow: hidden

这里，就给 ul 加就可以了

```css
ul {
  overflow: hidden;
}
```

**方法四：伪类清除法**

该方法的核心思想就是为**父元素设置一个伪元素**，其实就是无形的添加了一堵墙，然后在伪元素中设置一系列的属性。

```css
ul::after {
  content: "";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
```

# 十二、css 属性计算过程

这个也是一个很重要的部分，在浏览器渲染原理里面，第二步求计算样式的时候，就需要用到这个部分的知识。

有个前提，每一个元素所有的属性都是有值的

大致分为四个步骤：

1. 确定声明值
2. 层叠冲突
3. 使用继承值
4. 使用默认值

## 确定声明值

比如，作者样式表中写了（如果没有层叠冲突的话），会覆盖掉浏览器的默认样式表

```css
p {
  color: red
}

<p>这是文字</p>
```

那么呈现在页面上的，肯定是红色的。

## 层叠冲突

前提：选中同一个元素

这里也有三个小规则：

1. 重要性
2. 专用性
3. 源代码次序

**重要性**

其实就是`!important`有这个样式，秒杀一切。但是不能随便乱用，一般用于修改组件库的样式。

**专用性**

那么这个就是看权重，如果选中了同一个元素，使用权重高的

具体来说，

1. 内联样式 `style`（1000 分）
2. id 选择器（100 分）
3. class 选择器、属性选择器、伪类选择器（10 分）
4. 元素选择器、伪元素选择器（1 分）

注：通用选择器（*）, 复合选择器（+、>、~、空格）和否定伪类（:not）在专用性中无影响。

举个例子：

|   |   |   |   |   |   |
|---|---|---|---|---|---|
|**选择器**|**千位**|**百位**|**十位**|**个位**|**合计值**|
|h1|0|0|0|1|0001|
|#indentifier|0|1|0|0|0100|
|h1 + p::first-letter|0|0|0|3|0003|
|li > a[href*=” zh-CN”] > .inline-warning|0|0|2|2|0022|
|内联样式 style|1|0|0|0|1000|

**源代码次序**

这个简单，如果权重一样，那么后面的覆盖前面的

## 使用继承值

前提：没有选择器选中这个属性

那么，这个元素会去继承**离它最近**的父类的**可继承**的内容。

比如：

```css
<style>
  .one {
    color: red;
  }

  .two {
    color: gray;
  }
</style>

<body>
  <div class="one">
    <div class="two">
      <p>star</p>
    </div>
  </div>
</body>
```

那么呈现在页面上的，肯定是灰色的。

## 使用默认值

如果说，经过上面的步骤，还没有值的话。那么这些元素就使用浏览器默认样式。

# 十三、CSS 引入的方式有哪些？

两个`link` 和 `@import`

使用，

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <link rel="stylesheet" href="./index.css" />
    <style>
       @import url('./index.css');
      
    </style>
  </head>
  <body></body>
</html>
```

一个是 HTML 的标签（<link ...>），另一个是 CSS 中提供的属性在 style 标签中使用

`@import`还可以引入媒体

```css
@import "mobstyle.css" screen and (max-width: 768px);
/* 只在媒体为 screen 且视口最大宽度 768 像素时导入 "mobstyle.css" 样式表 */
```

区别：

**加载顺序的差别**

比如，在 _a.css_ 中使用 _import_ 引用 _b.css_，只有当使用当使用 _import_ 命令的宿主 _css_ 文件 _a.css_ 被下载、解析之后，浏览器才会知道还有另外一个 _b.css_ 需要下载，这时才去下载，然后下载后开始解析、构建 _render tree_ 等一系列操作.

**当使用** _**JS**_ **控制** _**DOM**_ **去改变样式的时候，只能使用** _**link**_ **标签，因为** _**@import**_ **不是** _**DOM**_ **可以控制的**

# 十四、媒体查询

这个是**响应式设计**的关键，具体是，根据什么设备用哪一个样式

_**Media Type**_ **设备类型**

在 _W3C_ 中共列出了 _10_ 种媒体类型，如下表所示：

|   |   |
|---|---|
|**值**|**设备类型**|
|All|所有设备|
|Braille|盲人用点字法触觉回馈设备|
|Embossed|盲文打印机|
|Handheld|便携设备|
|Print|打印用纸或打印预览视图|
|Projection|各种投影设备|
|Screen|电脑显示器|
|Speech|语音或音频合成器|
|Tv|电视机类型设备|
|Tty|使用固定密度字母栅格的媒介，比如电传打字机和终端|

当然，虽然上面的表列出来了这么多，但是常用的也就是 _all_（全部）、_screen_（屏幕）和 _print_（页面打印或打印预览模式）这三种媒体类型。

**媒体类型引用方法**

1. _link_ 方法

_link_ 方法引入媒体类型其实就是在 _link_ 标签引用样式的时候，通过 _link_ 标签中的 _media_ 属性来指定不同的媒体类型，如下：

```
<!-- 只有设备是screen时，才将index.css引入 -->
<link rel="stylesheet" href="index.css" media="screen" />
<link rel="stylesheet" href="print.css" media="print" />
```

2. `@import`也可以，但不推荐

```
@import url('./index.css') screen
```

3. `@media`方式

_@media_ 是 _CSS3_ 中新引进的一个特性，称为媒体查询。_@media_ 引入媒体也有两种方式，如下：

在样式文件中引入媒体类型：

```
@media screen{
  /* 具体样式 */
}
```

**媒体查询具体语法**

接下来我们来看一下媒体查询的具体语法。

这里我们可以将 _Media Query_ 看成一个公式：

```
Media Type（判断条件）+ CSS（符合条件的样式规则）
```

这里举例如下：

_link_ 的方式

```
<link rel="stylesheet" media="screen and (max-width:600px)" href="style.css" />
```

_@media_ 的方式

```
@meida screen and (max-width:600px){
  /* 具体样式 */
}
```

上面的两个例子中都是使用 _width_ 来进行的样式判断，但是实际上还有很多特性都可以被用来当作样式判断的条件。

具体如下表：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751446745321-b337545f-f747-4e23-bcc6-f03d8bf37d99.png)

例子，

1. **最大宽度** **_max-width_**

该特性是指媒体类型小于或等于指定宽度时，样式生效，例如：

```
/* 当视口屏幕宽度小于或等于 480px 时，样式生效 */
@media screen and (max-width:480px) {
  /* 具体样式 */
}
```

2. **最小宽度** **_min-width_**

该特性和上面相反，及媒体类型大于或等于指定宽度时，样式生效，例如：

```
/* 当视口屏幕宽度大于或等于 480px 时，样式生效 */
@media screen and (min-width:480px) {
  /* 具体样式 */
}
```

3. **多个媒体特性混合使用**

当需要多个媒体特性时，使用 _and_ 关键字将媒体特性结合在一起，例如：

```
/* 当视口屏幕大于等于 480px 并且小于等于 900px 时，样式生效 */
@media screen and (min-width:480px) and (max-width:900px){
  /* 具体样式 */
}
```

4. **设备屏幕的输出宽度** **_Device Width_**

在智能设备上，例如 _iphone、ipad_ 等，可以通过 _min-device-width_ 和 _max-device-width_ 来设置媒体特性，例如：

```
/* 当设配屏幕小于等于 480px 时样式生效 */
@media screen and (max-device-height:480px) {
  /* 具体样式 */
}
```

5. **_not_** **关键字**

_not_ 关键词可以用来排除某种制定的媒体特性，示例如下：

```
/* 运用在除了打印设备和屏幕宽度小于或等于 900px 的所有设备中 */
@media not print and (max-width:900px) { 
  /* 具体样式 */
}·
```

6. **未指明** **_Media Type_**

如果在媒体查询中没有明确的指定 _Media Type_，那么其默认值为 _all_

```
/* 适用于屏幕尺寸小于或等于 900px 的所有设备 */
@media (max-width: 900px){
  /* 具体样式 */
}
```

# 十五、渐进增强和优雅降级

渐进增强，英语全称 _progressive enhancement_，指的是针对**低版本浏览器**进行构建页面，保证最基本的功能，然后再针对高级浏览器进行效果、交互等改进和追加功能达到更好的用户体验。

优雅降级，英语全称 _graceful degradation_，一开始就**构建完整**的功能，然后再针对低版本浏览器进行兼容。

也就是说，渐进增强考虑的是，向下兼容。优雅降级考虑的是，向上兼容。

# 十六、渐进式渲染

渐进式渲染，英文全称 _progressive rendering_，也被称之为惰性渲染，指的是为了提高用户感知的加载速度，以尽快的速度来呈现页面的技术。

有一点需要弄明白的是，这不是指的某一项技术，而是各种技术的一种集合。

具体的应用：

1. 骨架屏
2. 图片懒加载：先加载部分数据，之后的数据有需要在加载
3. 图片占位符：在网页加载的时候，某些图片还在请求中或者还未请求，这个时候就先找一个临时代替的图像，放在最终图像的位置上，但是这只是临时替代的图形，当图片数据准备好以后，会重新渲染真正的图形数据。
4. 资源拆分

# 十七、如何提升 CSS 的渲染性能？

_CSS_ 渲染性能的优化来自方方面面，这里列举一些常见的方式：

1. 使用 _id_ 选择器非常高效，因为 _id_ 是唯一的
2. 避免深层次的选择器嵌套
3. 尽量避免使用属性选择器，因为匹配速度慢
4. 使用渐进增强的方案
5. 遵守 _CSSLint_ 规则
6. 不要使用 `@import`
7. 避免过分重排（_Reflow_）
8. 依赖继承
9. 避免耗性能的属性：box-shadow、border-radius、:nth-child
10. 文件压缩：使用打包工具：webpack

常见的重排元素：

```
width 
height 
padding 
margin 
display 
border-width 
border 
top 
position 
font-size 
float 
text-align 
overflow-y 
font-weight 
overflow 
left 
font-family 
line-height 
vertical-align 
right 
clear 
white-space 
bottom 
min-height
```