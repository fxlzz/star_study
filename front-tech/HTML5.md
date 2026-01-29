可编辑属性，`contenteditable`只要加上了这个属性，那么，那个元素内部就可以编辑了

比如，

  `<div contenteditable="true" id="content"></div>`

# 拖拽元素

也是加一个属性`draggable`，大部分元素默认为 false，像 a 标签、img 标签默认是 true

对于拖拽元素，有几个事件：

1. `dragstart`：拖拽开始（鼠标按下那一刻是不算的）
2. `drag`：进行中
3. `dragend`：拖拽结束

有一个小的 demo：

```html
<style>
  .box {
    position: absolute;
    left: 0;
    top: 0;
    width: 100px;
    height: 100px;
    background-color: #adcdef;
  }
</style>
<body>
  <div class="box" draggable="true"></div>
  <script>
    const dragBox = document.querySelector(".box");
    let dragX = 0;
    let dragY = 0;
    dragBox.addEventListener("dragstart", (e) => {
      console.log("start");
      dragX = e.clientX;
      dragY = e.clientY;
    });
    dragBox.addEventListener("drag", (e) => {
      console.log("draging");
    });
    dragBox.addEventListener("dragend", (e) => {
      console.log("end");
      let x = e.clientX - dragX;
      let y = e.clientY - dragY;
      dragBox.style.left = dragBox.offsetLeft + x + "px";
      dragBox.style.top = dragBox.offsetTop + y + "px";
    });

    // 鼠标事件写法
     let mouseStartPos = { x: 0, y: 0 };
      const mouseDown = (e) => {
        mouseStartPos.x = e.clientX;
        mouseStartPos.y = e.clientY;

        document.addEventListener("mousemove", mouseMove);
        document.addEventListener("mouseup", mouseUp);
      };
      dragBox.addEventListener("mousedown", mouseDown);

      const mouseMove = (e) => {
        let mouseDir = {
          x: e.clientX - mouseStartPos.x,
          y: e.clientY - mouseStartPos.y,
        };

        mouseStartPos.x = e.clientX;
        mouseStartPos.y = e.clientY;

        dragBox.style.left = dragBox.offsetLeft + mouseDir.x + "px";
        dragBox.style.top = dragBox.offsetTop + mouseDir.y + "px";
      };

      const mouseUp = (e) => {
        document.removeEventListener("mousedown", mouseDown);
        document.removeEventListener("mousemove", mouseMove);
      };
  </script>
</body>
```

对于目标拖拽区别，有几个事件：

1. `dragenter`： 只有拖拽元素的鼠标进入target这个框了，才会触发事件
2. `dragover`：类似于 drag 事件，在target区域一致触发事件
3. `drop`：这个事件要触发，必须要dragover事件阻止了它的默认行为（over -> drag(默) -> drop）
4. `dragleave`：鼠标离开 target 区域

一个 demo：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      div {
        width: 200px;
        height: 150px;
        border: 1px solid #333;
      }
      .box1 ul {
        padding: 0;
      }
      li {
        list-style: none;
        width: 180px;
        height: 30px;
        background-color: #abcdef;
        margin: 5px auto;
      }
      .box2 {
        position: absolute;
        left: 500px;
        top: 8px;
      }
    </style>
  </head>
  <body>
    <div class="box1">
      <ul>
        <li></li>
        <li></li>
        <li></li>
      </ul>
    </div>
    <div class="box2"></div>
    <script>
      const dragBox = document.querySelector(".box1");
      const targetBox = document.querySelector(".box2");
      const lis = dragBox.querySelectorAll("li");
      let curDragDom = null;
      for (const li of lis) {
        li.setAttribute("draggable", "true");
        li.addEventListener("dragstart", () => {
          curDragDom = li;
        });
      }
      targetBox.addEventListener("dragover", (e) => {
        e.preventDefault();
      });
      targetBox.addEventListener("drop", (e) => {
        targetBox.appendChild(curDragDom);
        curDragDom = null;
      });

      dragBox.addEventListener("dragover", (e) => {
        e.preventDefault();
      });
      dragBox.addEventListener("drop", (e) => {
        dragBox.appendChild(curDragDom);
        curDragDom = null;
      });
    </script>
  </body>
</html>
```

# canvas

使用，

创建一个 canvas`<canvas id="canvas" width="500" height="300"></canvas>`

如果没有指定宽高的话，默认是 width: 300 height: 150

用 JS 来操作 canvas

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d"); // 获取渲染上下文
```

## 画线

```js
// 画个线
ctx.moveTo(100, 100); // 线的起点
ctx.lineTo(200, 100);
ctx.lineTo(200, 200);
ctx.closePath(); // 可以使连续的图形闭合起来
ctx.stroke(); // 画到页面上
```

## 画矩阵

```js
ctx.rect(100, 100, 200, 100);  --> (x, y, width, height)
ctx.fill();  --> 填充颜色
ctx.stroke();  ---> 画

ctx.strokeRect(100, 100, 300, 100); --> 一步到位

ctx.fillRect(100, 100, 200, 100);
```

一个小 demo：自由落体运动

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

let height = 100;
let timer = setInterval(() => {
  ctx.clearRect(0, 0, 500, 300); // 从(0, 0)的位置，清除宽为500，高为300的范围，橡皮擦
  ctx.strokeRect(100, height, 50, 50);
  height += 5;

  if (height > 250) {
    ctx.clearRect(0, 0, 500, 300);
    clearInterval(timer);
    ctx.strokeRect(100, 250, 50, 50);
  }
}, 1000 / 30);
```

## 画圆

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751702724744-2f1631bf-fd34-4bcd-8534-8f084c9a4c1c.png)

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

// arc：画圆、画弧
// (x, y, 起始度数, 终止度数, 顺逆时针) -> 0：顺时针 1：逆时针
// 起始度数, 终止度数 -> 使用弧度制来描述的 Math.PI 表示 180°
ctx.arc(100, 100, 50, (Math.PI / 180) * 0, (Math.PI / 180) * 300, 0);
ctx.lineTo(100, 100);
ctx.closePath();
ctx.stroke();
```

结果是：

![](https://cdn.nlark.com/yuque/0/2025/png/51701855/1751702880276-9952103a-4114-4912-8f6b-237b3d60128b.png)

## 平移和旋转

`translate(x, y)`：本来坐标轴开头在左上角，这个属性可以将坐标轴移动到指定的(x, y)位置

`rotate(angle)`：旋转的角度

## save 和 restore

上面设置的平移和旋转都是全局起作用的，

可以使用 save() 保存没有设置平移和旋转的之前的状态，以及坐标轴的位置

然后在合适的地方使用 restore()恢复最开始的位置，不然它全部都被平移和旋转影响

## 填充颜色和图形

使用`fillStyle`这个属性

填充颜色：

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

ctx.fillStyle = "red";
ctx.fillRect(100, 100, 200, 100);
```

填充图片：

```js
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");

const img = new Image(); // 创建一个 image dom对象
img.src = "./hashiqi.webp";

// 因为img是异步加载，有可能在画canvas时还没加载完图片，所以要放到onload的事件中执行
img.onload = function () {
  ctx.beginPath();
  ctx.translate(100, 100); 
  // createPattern 这个是以坐标轴原点开始渲染的，
  // 如果要图片填充到矩阵的位置，那么就要调整坐标轴原点的位置
  const bg = ctx.createPattern(img, "no-repeat");
  ctx.fillStyle = bg;
  ctx.fillRect(0, 0, 200, 100);
};

const body = document.querySelector("body");
body.appendChild(img);
```

## 线性渐变

填充的颜色是线性渐变的，

```js
const canvas = document.getElementById("canvas");
const context = canvas.getContext("2d");

// 从那个点渐变到那个点
// createLinearGradient(x1, y1, x2, y2)
const color = context.createLinearGradient(0, 0, 200, 100);

// 添加颜色 addColorStop(number, color) -> number: 0~1中的值
// 可以添加多个颜色帧
color.addColorStop(0, "white");
color.addColorStop(1, "black");

context.fillStyle = color;
context.fillRect(0, 0, 200, 100);
```

还有辐射渐变，那个我没看，要用的时候在看吧

## 阴影

```js
const canvas = document.getElementById("canvas");
const context = canvas.getContext("2d");

context.beginPath();
context.shadowColor = "black"; // 设置阴影颜色
context.shadowBlur = 30; // 模糊半径
context.shadowOffsetX = 10; // 阴影偏移量
context.shadowOffsetY = 10; // 阴影偏移量
context.fillRect(0, 0, 200, 100);
```

## 渲染文字

```js
const canvas = document.getElementById("canvas");
const context = canvas.getContext("2d");

context.beginPath();
context.font = "40px Georgia"; // 全局设置字体大小和字体类型
context.strokeText("hello", 100, 100); // 空心的
context.fillStyle = "red";
context.fillText("world", 200, 100); // 实心的
```

# svg