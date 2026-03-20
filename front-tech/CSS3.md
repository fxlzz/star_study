## 定位
position: absolute;  
参考系：父类中的加了定位的  
position: fixed;  
参考系：整个浏览器视图  
【注意】

1. 加了定位的元素，就会变成块级盒子
2. 如果定位没有成功的话，可以检查一下父级的高度

百分号 %  
参考系：父类内容区域

## 浮动 --- 高度坍塌

解决思路：利用BFC特性解决

```css
.left {
    float: left;
}

.right {
    float: right;
}

/* 解决高度坍塌 */
/* 1. 使用BFC 在父元素上加上 overflow: hidden或其他能产生BFC效果的属性 */
/* 2. 在浮动元素后面加上一个不是浮动元素的块盒子 */
.clearfix::after {
    content: "";
    display: block;
    clear: both; 
}
```

## 文字对齐

```css
    /* 
        这个属性的使用前提是 
        两个 inline 或 inline-block 在垂直方向上不平行
        就可以用，一般情况下不要用（里面的水很深）
    */
    vertical-align: 2px;
```

## 属性的计算过程

前提：要知道每一个标签的css属性都是有值的

1. 确定声明值
2. 解决冲突  
    1.1 作者样式表覆盖浏览器默认样式表  
    1.2 看权重  
    1.3 看顺序（在后面定义的会覆盖前面的）
3. 没有声明且能被继承的属性（文字、颜色）进行样式继承
4. 确定默认值

# CSS3

1. 兼容性问题  
    这里要知道的是，为什么有些属性需要加上浏览厂商的前缀，有些就不需要  
    (这里就不详细说了...)
2. 语言扩展性问题  
    出现了 预编译器、后编译器  
    例如：scss、less、CssNext等一些增加css语言的‘环境’  
    预编译器：提供一套新的语法，让开发者更容易编辑，最后scss编译器将我们写的scss代码转化为css代码  
    后编译器：提供一些工具，比如，一些兼容性插件，将css文件更加完整

## CSS3之伪类选择器

真的是花里胡哨，确实有很多很多牛逼的伪类选择器，不过还是那句话（可以不用，但是要知道有什么用）

1. E:not(last-child) 除了最后一个元素xxx，其他元素要xxx
2. E:target() 其id的当前的URL片段匹配，不用js就可以改变其他元素，确实牛逼！！！
3. E:root 表示更标签 HTML文件中表示的是 html 标签
4. E:last-child() E:first-child() E:nth-child(an+b)表示E元素是第几个孩子才会被选中
5. E:last-of-type() E:frist-of-type() E:nth-of-type(an+b) 表示某个元素的 第几个类型
6. E:empty 元素为空的时候选中
7. E:checked 元素被选中的时候选中
8. E:disabled 元素被禁用时选中

【补充知识】  
body是没有高度的，但是他可以继承html的高度

```css
<style>
p:target {
  background-color: gold;
}

/* 在目标元素中增加一个伪元素*/
p:target::before {
  font: 70% sans-serif;
  content: "►";
  color: limegreen;
  margin-right: 0.25em;
}

/*在目标元素中使用 italic 样式*/
p:target i {
  color: red;
}
</style>
<body>
  <h3>目录</h3>
  <ol>
    <li><a href="#p1">跳转到第一个段落！</a></li>
    <li><a href="#p2">跳转到第二个段落！</a></li>
    <li><a href="#nowhere">此链接不会跳转，因为目标不存在。</a>
    </li>
  </ol>
  <h3>我的趣味文章</h3>
  <p id="p1">你可以使用 URL 片段定位此<i>段落</i>。点击上面的链接试试吧！</p>
  <p id="p2">这是<i>另一个段落</i>，也可以从上面的链接访问。这不是很愉快吗？</p>
</body>
```

## CSS3之伪类属性

```css
/* 控制输入框占位符文字的样式 */
input::placeholder {
    color: ;
    font-size: ;
}
/* 控制选中文字的样式 */
p::selection {
    color: ;
    background-color: ;
}
```

### border-radius

其实是下面这些的简单写法： X轴 Y轴

1. border-top-left-radius: 20px 20px;
2. border-top-right-radius: 20px 20px;
3. border-buttom-right-radius: 20px 20px;
4. border-buttom-left-radius: 20px 20px;

### box-shadow

**box-shadow: (inset / outset 默认外部) 0px 0px 0px 0px #fff;**  
x轴偏移量 y轴偏移量 模糊像素 扩大像素（阴影） 阴影颜色

注意：

1. box-shadow: 后面的属性可以无限叠加
2. 模糊是从四边从中模糊，同时也向外模糊
3. 阴影扩大的像素是四面都同时加上指定的像素值

### background 系列

说实话，我也没想到会有那么那么多的属性，涨知识了~

1. background-image: url(), url()... 可以有多个图片，也可以使用渐变 linear-gradient()，radial-gradient()
2. background-origin: border-box padding-box(默认) content-box 规定了图片从哪开始渲染
3. background-repeat: no-repeat 图片不重复
4. background-position: 10px 20px; 确定图片的位置(x,y) 也可以用(center,left,right,top... 百分号) 参考系是origin是一样的
5. background-size: over(图片等比例扩大到与容器等大) contian(图片全貌都展示出来) 100px / 10%(也可以用像素和比分比)
6. background-clip: border-box(默认) padding-box content-box text 规定图片是从哪结尾

```css
background-clip: text;
color: transparent;
这两个属性配合起来，可以让图片嵌入文字
```

7. background-attchment: scroll(图片不动相对于内容), fixed(图片不动相对于视口), loacl(图片随着内容动)

### text-shadow

**text-shadow: 0px 0px 0px #fff** 文字阴影 x轴 y轴 模糊半径 阴影颜色

### overflow 系列

1. overflow: hidden(将溢出隐藏)
2. overflow: scroll(溢出加滚动条)
3. overflow: visible(默认)
4. overflow: auto(自动检测看是否需要滚动条)

注意：overflow 是 overflow-x 和 overflow-y 的简写形式，如果设置了overflow-x: scroll; 只要不是visible属性, 那么另一个方向的值就会被设为 auto;

### flex 系列

1. flex-wrap: wrap / nowrap(defualt) 允许换行
2. flex-direction: column; 切换主轴方向
3. justify-content: center / space-between / space-around / flex-end / flex-start; 规定主轴的对齐方式
4. align-items: center / flex-start / flex-end / stretch(子元素没有设置高度，与父级同高); 规定单行在交叉轴的对齐方式
5. align-content: center / space-between / space-around ; 多行在交叉轴的对齐方式

【对于子元素】  
1. align-self: end / start /stretch / center; 对于子元素中的单个元素进行对齐  
注意： align-self 会超过父级加的 align-items 上的权重，但不会超过 align-content 上的权重  
2. flex-grow: 0(defualt); 剩余空间按比例伸展  
3. flex-shrink: 1(defualt); 内容区域按比例压缩  
计算公式：  
真实的内容区域 * shrink + ... * = 加权值  
元素内容px  
---------- * 超过区域 = 元素应该被压缩的px  
加权值px  
4. flex-basis: auto(defualt);  
注意：元素内容是英文 并且没有换行 才会出现盒子扩展

- 只写了basis或者basis > width，那么flex-basis代表的是最小宽度(确定了下限)
- 设置了 width 并且 basis < width，那么元素中的内容扩展的范围为：basis <= realWidth <= width

【注意】：  
flex 是 flex-grow flex-shrink flex-basis的缩写  
flex: 1 --> flex: 1 1 0%;

### transition 系列

过渡效果，可以从一个状态转化成另一个状态

1. transition-property: all(监控可以进行动画的所有属性) / 指定某个属性
2. transition-duration: .2s 持续时间
3. transition-timing-function: cubic-bezier(0,0,1,1) 三次贝塞尔函数
4. transition-delay: 1s 过渡延时时间

### animation 系列

1. animation-name: 关键帧的名字
2. animation-duration: 持续时间
3. animation-timing-function: 运动函数 steps(1, end / start)
4. animation-delay: 延时时间
5. animation-iteration-count: 1 / infinite(无限次) 动画循环几次
6. animation-direction: reverse(与定义关键帧的反向运动) / alternate(交替运动) 运动方向
7. animation-fill-mode: forwards(初始状态使用最开始的帧的状态) / backwards(结束状态使用最后一帧的状态) / both(都使用) 动画开始和结尾呈现的形式
8. animation-play-state: paused(暂停) / running(运动) 可以设置动画是运动还是暂停的

### transform 系列

1. transform: rotate(0deg) 2d旋转  
    rotatex(0deg) 3d-x轴旋转 rotatey(0deg) 3d-y轴旋转 rotatez(0deg) 3d-z轴旋转 rotate3d(x, y ,z 10deg); (x,y,z)是3旋转的矢量轴
2. transform: scale(1.5, 1) 2d将刻度值扩大1.5倍,但是内容没变，也就是之前1px能够描述10cm，现在扩大之后，1px能够描述20cm, 并且要注意的是：变化轴会随着旋转变化
3. transform: skew(0deg, 0deg) 让盒子发生倾斜
4. transform: translateX(100px) / translateY() / translateZ() 分别向X,Y,Z轴进行平移，参考的是自身的轴(有待考量)
5. transform-origin: center center; 控制旋转中心
6. transform-style: flat(defualt) / preserve-3d(需要作用于直接父元素，子元素才会保留3d效果)

**perspective** 景深

1. perspective: 800px; 说明人的眼睛距离平面有800px的距离
2. perspective-origin: center center; 规定眼睛在哪里  
    【注意】
3. 眼睛看屏幕的物体，都是物体的**投影**，改变景深距离会使得物体在视觉层面上放大或缩小
4. 如果某个父级加了景深，那么那个元素就会变成定位的参考系

# flex布局

弹性布局（flex布局），一般运用在移动端

### 1.1 父项的属性

`flex-direction` 决定主轴的方向，有row(x轴)，column(y轴)

默认：row 从左到右 row-reverse 从右到左

column 从上到下 column-reverse 从下到上

`jusitfy-content` 定义了项目在主轴上的对齐方式

flex-start 默认值，如果主轴是x轴就是从左到右

如果主轴是y轴就是从上到下

flex-end 从尾部开始排列

enter 在主轴居中对齐

space-around 平分剩余空间

```css
       space-between   先两边贴边，在平分剩余空间
       space-evenly    每个子项之间的距离是相同的
```

`flex-wrap` 设置子元素是否换行

默认 nowrap 不换行

flex-wrap: wrap 换行

`align-content` 设置侧轴上的子元素的排列方式（多行）

必须有换行的属性，才能用这个属性

flex-start 默认值，如果主轴是x轴就是从左到右

如果主轴是y轴就是从上到下

flex-end 从尾部开始排列

enter 在主轴居中对齐

space-around 平方剩余空间

```css
       space-between 先两边贴边，在平方剩余空间
```

stretch 子项没有高度，与父项一样高

`align-items` 设置**侧轴**上子元素排列方式（单行）

flex-start 默认值，如果主轴是x轴就是从左到右

如果主轴是y轴就是从上到下

flex-end 从尾部开始排列

enter 在主轴居中对齐

stretch 子项没有高度，与父项一样高

`flex-flow` 是flex-wrap和flex-dirction 复合属性

要是**换列并换行**，那么只要

flex-flow: column, wrap

### 1.2 子项属性

`flex: <number>` 在剩余空间中，子项占的份数

`align-self` 控制子项在侧轴上的排列方式，就是控制他**自己怎么排**

flex-start 默认值，如果主轴是x轴就是从左到右

·如果主轴是y轴就是从上到下

flex-end 从尾部开始排列

enter 在主轴居中对齐

stretch 子项没有高度，与父项一样高

`order` 定义**子项的排列顺序**

默认值为0 小的值在前面

# grid布局

常用于九宫格布局，对齐的方块，一般**不给**子级宽度，给**父级一个宽度**

## 九宫格布局

**三行代码就可以让你实现布局：这简直太棒了**

当然是常用的属性：

```css
display: grid;
gap: 15rpx; // 盒子之间的间隔
grid-template-columns: repeat(3, 1fr) // 1fr表示：平均分配父盒子的宽度
```

# 多行省略号

```css
overflow: hidden;
text-overflow: ellipsis; // 超出文字使用省略号表示
display: -webkit-box; // 将对象作为弹性伸缩盒子模型显示。
-webkit-line-clamp: 2;// 这个属性不是css的规范属性，需要组合上面两个属性，表示显示的行数
overflow:hidden;
-webkit-box-orient: vertical;   // 从上到下垂直排列子元素（设置伸缩盒子的子元素排列方式）


text-overflow: ellipsis;
display: -webkit-box; 
-webkit-line-clamp: 2;
overflow:hidden;
-webkit-box-orient: vertical; 
```