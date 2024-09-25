# 使用canvas实现图片标注功能

前言：最近在做一些图片的算法识别，需要在前端对图片进行人工复和，因此需要一款识别标注工具

##  1、初始化内容

#### 初始化html

创建canvas 画布以及图形选择框

```html
<div class="canvas-container">
  <canvas id="canvas" width="1600" height="800"></canvas>
  <div class="sharp">
    <button id="rect" onclick="setShape('rect')">矩形</button>
    <button id="circle" onclick="setShape('circle')">圆形</button>
    <button id="line" onclick="setShape('line')">线条</button>
    <button id="text" onclick="setShape('text')">文字</button>
    <input type="color" id="color" value="#000000" />
    <select id="select">
      <option value="标记1">标记1</option>
      <option value="标记2">标记2</option>
      <option value="标记3">标记3</option>
    </select>
  </div>
</div>
```

#### 初始化js

```javascript
let shape = null; //当前形状
let color = "#000000"; //当前颜色
// 设置形状
function setShape(type) {
  shape = type;
}
const colorBtn = document.getElementById("color");
colorBtn.addEventListener("input", (e) => {
  color = e.target.value;
});
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
// 初始化画布
function init() {
  // 设置画布的宽度
  let width = 1400;
  let height = 800;
  canvas.width = width * devicePixelRatio;
  canvas.height = height * devicePixelRatio;
  canvas.style.width = width + "px";
  canvas.style.height = height + "px";
}
init();
```



## 2、创建矩形类

#### 获取矩形的两个顶点

矩形的两个顶点不能点出的以鼠标点击的位置为起始点

因此我们需要手动获取顶点

```javascript
class RectShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.endX = startX;
    this.endY = startY;
  }
  get minX() {
    return Math.min(this.startX, this.endX);
  }
  get minY() {
    return Math.min(this.startY, this.endY);
  }
  get maxX() {
    return Math.max(this.startX, this.endX);
  }
  get maxY() {
    return Math.max(this.startY, this.endY);
  }

}
```

#### 绘制矩形框

绘制的时候需要考虑到一个dpi 的值 

```javascript
draw() {
    // 绘制填充
    // ctx.fillStyle = this.color;
    // ctx.fillRect(
    //   this.minX * devicePixelRatio,
    //   this.minY * devicePixelRatio,
    //   (this.maxX - this.minX) * devicePixelRatio,
    //   (this.maxY - this.minY) * devicePixelRatio
    // );
    // 绘制边框
    ctx.strokeStyle = this.color; // 设置边框颜色
    ctx.lineWidth = 3 * devicePixelRatio; // 设置边框宽度
    ctx.strokeRect(
      this.minX * devicePixelRatio,
      this.minY * devicePixelRatio,
      (this.maxX - this.minX) * devicePixelRatio,
      (this.maxY - this.minY) * devicePixelRatio
    );
  }
```

#### 判断当前是创建图形还是拖动图形

点击空白处是创建图形，点击图形时就需要拖动图形了

```javascript
//判断是否点击此图形
  isIn(x, y) {
    return x > this.minX && x < this.maxX && y > this.minY && y < this.maxY;
  }
```



#### 输出图形信息

```javascript
//输出图形信息 
getInfo() {
    return {
      type: "rect",
      startX: this.startX,
      startY: this.startY,
      endX: this.endX,
      endY: this.endY,
      color: this.color,
    };
  }
```



## 3、绘制图形

此时我们需要监听canvas的鼠标按下事件 而不是点击事件

```js
//因为需要绘制多个图形
// 图形集合
const shapes = [];
canvas.addEventListener("mousedown", (e) => {
  // 判断是否点击到图形
	// 移动图形
  // 创建新的图形
});
```



#### 创建图形

```js
const currentShape = new RectShape(e.offsetX, e.offsetY, color);
shapes.push(currentShape);
const cvsRact = canvas.getBoundingClientRect();
window.onmousemove = (e) => {
  const x = e.clientX - cvsRact.left;
  const y = e.clientY - cvsRact.top;
  currentShape.endX = x;
  currentShape.endY = y;
};
// 监听鼠标抬起 停止移动
window.onmouseup = () => {
  window.onmousemove = null;
  window.onmouseup = null;
};
```

#### 绘制图形

```javascript
// 绘制图形
function draw() {
  // 使用requestAnimationFrame来实现动画
  requestAnimationFrame(draw);
  // 清空画布
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // 绘制所有图形
  shapes.forEach((shape) => {
    shape.draw();
  });
}
draw();
```

此时已经可以在canvas上进行绘制了

#### 移动图形

```javascript
// 判断点击的那个图形
function getShape(x, y) {
  // 从后往前遍历图形 因为后绘制的图形会覆盖先绘制的图形
  for (let i = shapes.length - 1; i >= 0; i--) {
    const shape = shapes[i];
    if (shape.isIn(x, y)) {
      return shape;
    }
  }
  return null;
}
```

此时的监听事件就应该修改一下

```javascript
canvas.addEventListener("mousedown", (e) => {
  // 判断是否点击到图形
  const shapeI = getShape(e.offsetX, e.offsetY);
  if (shapeI) {
    // 移动图形
    // 获取鼠标点击的位置
    const [sx, sy] = [e.offsetX, e.offsetY];
    // 获取图形的起始位置和结束位置
    const { startX, startY, endX, endY } = shapeI;
    // 获取画布的边界
    const cvsRact = canvas.getBoundingClientRect();
    // 监听鼠标移动
    window.onmousemove = (e) => {
      // 获取鼠标移动偏移量
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      const dx = x - sx;
      const dy = y - sy;
      //重新赋值图形的起始位置和结束位置
      shapeI.startX = startX + dx;
      shapeI.startY = startY + dy;
      shapeI.endX = endX + dx;
      shapeI.endY = endY + dy;
    };
  } else {
    // 创建新的图形
    const currentShape = new RectShape(e.offsetX, e.offsetY, color);
    shapes.push(currentShape);
    const cvsRact = canvas.getBoundingClientRect();
    window.onmousemove = (e) => {
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      currentShape.endX = x;
      currentShape.endY = y;
    };
  }
  // 监听鼠标抬起 停止移动
  window.onmouseup = () => {
    window.onmousemove = null;
    window.onmouseup = null;
  };
});
```

#### 双击图形则删除

```javascript
//监听双击事件
canvas.addEventListener("dblclick", (e) => {
  const shapeI = getShape(e.offsetX, e.offsetY);
  if (shapeI) {
    shapes.splice(shapes.indexOf(shapeI), 1);
  }
});
```



## 4、创建其他的类

#### 矩形类

```javascript
// 矩形类
class RectShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.endX = startX;
    this.endY = startY;
  }
  get minX() {
    return Math.min(this.startX, this.endX);
  }
  get minY() {
    return Math.min(this.startY, this.endY);
  }
  get maxX() {
    return Math.max(this.startX, this.endX);
  }
  get maxY() {
    return Math.max(this.startY, this.endY);
  }
  draw() {
    // 绘制填充
    // ctx.fillStyle = this.color;
    // ctx.fillRect(
    //   this.minX * devicePixelRatio,
    //   this.minY * devicePixelRatio,
    //   (this.maxX - this.minX) * devicePixelRatio,
    //   (this.maxY - this.minY) * devicePixelRatio
    // );
    // 绘制边框
    ctx.strokeStyle = this.color; // 设置边框颜色
    ctx.lineWidth = 3 * devicePixelRatio; // 设置边框宽度
    ctx.strokeRect(
      this.minX * devicePixelRatio,
      this.minY * devicePixelRatio,
      (this.maxX - this.minX) * devicePixelRatio,
      (this.maxY - this.minY) * devicePixelRatio
    );
  }
  //判断是否点击此图形
  isIn(x, y) {
    return x > this.minX && x < this.maxX && y > this.minY && y < this.maxY;
  }
  //输出图形信息
  getInfo() {
    return {
      type: "rect",
      startX: this.startX,
      startY: this.startY,
      endX: this.endX,
      endY: this.endY,
      color: this.color,
    };
  }
}
```

#### 圆形

```javascript
class CircleShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.radius = 0;
    this.endX = startX;
    this.endY = startY;
  }
  get getRadius() {
    return Math.sqrt(
      (this.endX - this.startX) ** 2 + (this.endY - this.startY) ** 2
    );
  }
  draw() {
    ctx.beginPath(); // 开始绘制
    this.radius = this.getRadius * devicePixelRatio; //计算半径
    ctx.arc(
      this.startX * devicePixelRatio,
      this.startY * devicePixelRatio,
      this.radius,
      0,
      Math.PI * 2
    );
    ctx.strokeStyle = this.color; // 设置边框颜色
    ctx.lineWidth = 3 * devicePixelRatio; // 设置边框宽度
    ctx.stroke(); // 描边
  }
  //判断是否点击此图形
  isIn(x, y) {
    return (
      Math.sqrt((x - this.startX) ** 2 + (y - this.startY) ** 2) <= this.radius
    );
  }
  //输出图形信息
  getInfo() {
    return {
      type: "circle",
      startX: this.startX,
      startY: this.startY,
      color: this.color,
      radius: this.radius,
    };
  }
}
```

#### 线条

```javascript
// 线条类
class LineShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.endX = startX;
    this.endY = startY;
  }
  draw() {
    ctx.beginPath();
    ctx.moveTo(this.startX, this.startY);
    ctx.lineTo(this.endX, this.endY);
    ctx.strokeStyle = this.color;
    ctx.lineWidth = 3 * devicePixelRatio;
    ctx.stroke();
  }
  //判断是否点击此图形
  isIn(x, y) {
    const dx = this.startX - this.endX;
    const dy = this.startY - this.endY;
    const k = dy / dx;
    const b = this.startY - k * this.startX;
    return y >= k * x + b - 3 && y <= k * x + b + 3;
  }
  //输出图形信息
  getInfo() {
    return {
      type: "line",
      startX: this.startX,
      startY: this.startY,
      endX: this.endX,
      endY: this.endY,
      color: this.color,
    };
  }
}
```



#### 文字

```js
class TextShape {
  constructor(startX, startY, color, text) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.text = text; // 保存文本
  }
  draw() {
    ctx.fillStyle = this.color;
    // 计算文本宽度和高度
    const metrics = ctx.measureText(this.text);
    const textWidth = metrics.width;
    const textHeight = 30; // 根据字体大小手动设置
    //设置成微软雅黑
    ctx.font = "30px 微软雅黑";
    ctx.textAlign = "left";
    ctx.textBaseline = "top";

    // 绘制文字
    ctx.fillText(this.text, this.startX, this.startY);

    // 绘制外边框
    ctx.strokeStyle = this.color;
    const lineWidth = 1;
    ctx.lineWidth = lineWidth;
    ctx.strokeRect(
      this.startX,
      this.startY - 2 * lineWidth, // 让矩形框顶部对齐文字顶部
      textWidth,
      textHeight
    );
  }

  //判断是否点击此图形
  isIn(x, y) {
    const metrics = ctx.measureText(this.text);
    return (
      x > this.startX &&
      x < this.startX + metrics.width &&
      y > this.startY && // 字体高度
      y < this.startY + 30
    );
  }
  //输出图形信息
  getInfo() {
    return {
      type: "text",
      startX: this.startX,
      startY: this.startY,
      color: this.color,
      text: this.text,
    };
  }
}
```



## 5、不同的形状创建不同的实例

```javascript
canvas.addEventListener("mousedown", (e) => {
  // 判断是否点击到图形
  const shapeI = getShape(e.offsetX, e.offsetY);
  if (shapeI) {
    // 移动图形
    // 获取鼠标点击的位置
    const [sx, sy] = [e.offsetX, e.offsetY];
    // 获取图形的起始位置和结束位置
    const { startX, startY, endX, endY } = shapeI;
    // 获取画布的边界
    const cvsRact = canvas.getBoundingClientRect();
    // 监听鼠标移动
    window.onmousemove = (e) => {
      // 获取鼠标移动偏移量
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      const dx = x - sx;
      const dy = y - sy;
      //重新赋值图形的起始位置和结束位置
      shapeI.startX = startX + dx;
      shapeI.startY = startY + dy;
      shapeI.endX = endX + dx;
      shapeI.endY = endY + dy;
    };
  } else {
    // 创建新的图形
    let currentShape = null;
    switch (shape) {
      case "rect":
        currentShape = new RectShape(e.offsetX, e.offsetY, color);
        break;
      case "circle":
        currentShape = new CircleShape(e.offsetX, e.offsetY, color);
        break;
      case "line":
        currentShape = new LineShape(e.offsetX, e.offsetY, color);
        break;
      case "text":
        const selectValue = select.value || "默认标记";
        currentShape = new TextShape(e.offsetX, e.offsetY, color, selectValue);
        break;

      default:
        alert("请选择形状");
        return;
    }
    shapes.push(currentShape);
    const cvsRact = canvas.getBoundingClientRect();
    window.onmousemove = (e) => {
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      currentShape.endX = x;
      currentShape.endY = y;
    };
  }
  // 监听鼠标抬起 停止移动
  window.onmouseup = () => {
    window.onmousemove = null;
    window.onmouseup = null;
  };
});
```



## 6 、完整代码

#### html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <style>
      .container {
        display: flex;
        flex-direction: row;
        justify-content: space-between;
      }
      #canvas {
        border: 1px solid black;
        display: block;
      }
      .canvas-container {
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
      }
      .sharp {
        width: 100px;
        margin-top: 10px;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
      }
      /* sharp点击后样式  */
      .sharp button {
        margin: 10px 10px;
        border: none;
        cursor: pointer;
      }
      .active {
        background-color: #5785ee;
      }
      .info-container {
        height: 100%;
      }
      /* 给表格加样式 */
      .info {
        width: 800px;
        border-collapse: collapse;
        margin: 0 auto;
      }
      .info th,
      .info td {
        border: 1px solid black;
        padding: 5px;
      }
      .info th {
        width: 100px;
        background-color: #f2f2f2;
      }
      .info td {
        text-align: center;
      }
      /* 给提交按钮加样式 */
      #submit {
        width: 80px;
        height: 30px;
        border-radius: 5px;
        font-size: 16px;
        font-weight: bold;
        line-height: 30px;
        /* 居中 */
        text-align: center;
        display: block;
        background-color: #5785ee;
        color: white;
        border: none;
        cursor: pointer;
      }
      #color {
        height: 30px;
        border: none;
        margin-top: 10px;
        cursor: pointer;
      }
      #select {
        height: 30px;
        border: none;
        margin-top: 10px;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <!-- 创建一个画布支持在画布上绘制矩形 圆形 线条 文字 -->
    <div class="container">
      <div class="canvas-container">
        <canvas id="canvas" width="1600" height="800"></canvas>
        <div class="sharp">
          <button id="rect" onclick="setShape('rect')">矩形</button>
          <button id="circle" onclick="setShape('circle')">圆形</button>
          <button id="line" onclick="setShape('line')">线条</button>
          <button id="text" onclick="setShape('text')">文字</button>
          <input type="color" id="color" value="#000000" />
          <select id="select">
            <option value="标记1">标记1</option>
            <option value="标记2">标记2</option>
            <option value="标记3">标记3</option>
          </select>
        </div>
      </div>

      <div class="info-container">
        <div id="submit">提交信息</div>
        <div class="info">
          <table id="info-table">
            <thead>
              <tr>
                <th>类型</th>
                <th>startX</th>
                <th>startY</th>
                <th>endX</th>
                <th>endY</th>
                <th>颜色</th>
                <th>半径</th>
                <th>文本</th>
              </tr>
            </thead>
          </table>
        </div>
      </div>
    </div>
  </body>
  <script src="./canva.js"></script>
  <script>
    // 给几个button添加事件 点击后背景变色
    const buttons = document.querySelectorAll("button");
    buttons.forEach((item) => {
      item.addEventListener("click", () => {
        buttons.forEach((item) => {
          item.classList.remove("active");
        });
        item.classList.add("active");
      });
    });
  </script>
</html>

```



#### js

```javascript
let shape = null; //当前形状
let color = "#000000"; //当前颜色
// 设置形状
function setShape(type) {
  shape = type;
}
// 设置颜色
const colorBtn = document.getElementById("color");
colorBtn.addEventListener("input", (e) => {
  color = e.target.value;
});
// 设置画布
const textInput = document.getElementById("text-input");
const select = document.getElementById("select");
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
// 初始化画布
function init() {
  // 设置画布的宽度
  let width = 1400;
  let height = 800;
  canvas.width = width * devicePixelRatio;
  canvas.height = height * devicePixelRatio;
  canvas.style.width = width + "px";
  canvas.style.height = height + "px";
}
init();
// 图形集合
const shapes = [];
// 矩形类
class RectShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.endX = startX;
    this.endY = startY;
  }
  get minX() {
    return Math.min(this.startX, this.endX);
  }
  get minY() {
    return Math.min(this.startY, this.endY);
  }
  get maxX() {
    return Math.max(this.startX, this.endX);
  }
  get maxY() {
    return Math.max(this.startY, this.endY);
  }
  draw() {
    // 绘制填充
    // ctx.fillStyle = this.color;
    // ctx.fillRect(
    //   this.minX * devicePixelRatio,
    //   this.minY * devicePixelRatio,
    //   (this.maxX - this.minX) * devicePixelRatio,
    //   (this.maxY - this.minY) * devicePixelRatio
    // );
    // 绘制边框
    ctx.strokeStyle = this.color; // 设置边框颜色
    ctx.lineWidth = 3 * devicePixelRatio; // 设置边框宽度
    ctx.strokeRect(
      this.minX * devicePixelRatio,
      this.minY * devicePixelRatio,
      (this.maxX - this.minX) * devicePixelRatio,
      (this.maxY - this.minY) * devicePixelRatio
    );
  }
  //判断是否点击此图形
  isIn(x, y) {
    return x > this.minX && x < this.maxX && y > this.minY && y < this.maxY;
  }
  //输出图形信息
  getInfo() {
    return {
      type: "rect",
      startX: this.startX,
      startY: this.startY,
      endX: this.endX,
      endY: this.endY,
      color: this.color,
    };
  }
}
// 圆形类
class CircleShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.radius = 0;
    this.endX = startX;
    this.endY = startY;
  }
  get getRadius() {
    return Math.sqrt(
      (this.endX - this.startX) ** 2 + (this.endY - this.startY) ** 2
    );
  }
  draw() {
    ctx.beginPath(); // 开始绘制
    this.radius = this.getRadius * devicePixelRatio; //计算半径
    ctx.arc(
      this.startX * devicePixelRatio,
      this.startY * devicePixelRatio,
      this.radius,
      0,
      Math.PI * 2
    );
    ctx.strokeStyle = this.color; // 设置边框颜色
    ctx.lineWidth = 3 * devicePixelRatio; // 设置边框宽度
    ctx.stroke(); // 描边
  }
  //判断是否点击此图形
  isIn(x, y) {
    return (
      Math.sqrt((x - this.startX) ** 2 + (y - this.startY) ** 2) <= this.radius
    );
  }
  //输出图形信息
  getInfo() {
    return {
      type: "circle",
      startX: this.startX,
      startY: this.startY,
      color: this.color,
      radius: this.radius,
    };
  }
}
// 线条类
class LineShape {
  constructor(startX, startY, color) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.endX = startX;
    this.endY = startY;
  }
  draw() {
    ctx.beginPath();
    ctx.moveTo(this.startX, this.startY);
    ctx.lineTo(this.endX, this.endY);
    ctx.strokeStyle = this.color;
    ctx.lineWidth = 3 * devicePixelRatio;
    ctx.stroke();
  }
  //判断是否点击此图形
  isIn(x, y) {
    const dx = this.startX - this.endX;
    const dy = this.startY - this.endY;
    const k = dy / dx;
    const b = this.startY - k * this.startX;
    return y >= k * x + b - 3 && y <= k * x + b + 3;
  }
  //输出图形信息
  getInfo() {
    return {
      type: "line",
      startX: this.startX,
      startY: this.startY,
      endX: this.endX,
      endY: this.endY,
      color: this.color,
    };
  }
}
// 文字类
class TextShape {
  constructor(startX, startY, color, text) {
    this.startX = startX;
    this.startY = startY;
    this.color = color;
    this.text = text; // 保存文本
  }
  draw() {
    ctx.fillStyle = this.color;
    // 计算文本宽度和高度
    const metrics = ctx.measureText(this.text);
    const textWidth = metrics.width;
    const textHeight = 30; // 根据字体大小手动设置
    //设置成微软雅黑
    ctx.font = "30px 微软雅黑";
    ctx.textAlign = "left";
    ctx.textBaseline = "top";

    // 绘制文字
    ctx.fillText(this.text, this.startX, this.startY);

    // 绘制外边框
    ctx.strokeStyle = this.color;
    const lineWidth = 1;
    ctx.lineWidth = lineWidth;
    ctx.strokeRect(
      this.startX,
      this.startY - 2 * lineWidth, // 让矩形框顶部对齐文字顶部
      textWidth,
      textHeight
    );
  }

  //判断是否点击此图形
  isIn(x, y) {
    const metrics = ctx.measureText(this.text);
    return (
      x > this.startX &&
      x < this.startX + metrics.width &&
      y > this.startY && // 字体高度
      y < this.startY + 30
    );
  }
  //输出图形信息
  getInfo() {
    return {
      type: "text",
      startX: this.startX,
      startY: this.startY,
      color: this.color,
      text: this.text,
    };
  }
}

canvas.addEventListener("mousedown", (e) => {
  // 判断是否点击到图形
  const shapeI = getShape(e.offsetX, e.offsetY);
  if (shapeI) {
    // 移动图形
    // 获取鼠标点击的位置
    const [sx, sy] = [e.offsetX, e.offsetY];
    // 获取图形的起始位置和结束位置
    const { startX, startY, endX, endY } = shapeI;
    // 获取画布的边界
    const cvsRact = canvas.getBoundingClientRect();
    // 监听鼠标移动
    window.onmousemove = (e) => {
      // 获取鼠标移动偏移量
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      const dx = x - sx;
      const dy = y - sy;
      //重新赋值图形的起始位置和结束位置
      shapeI.startX = startX + dx;
      shapeI.startY = startY + dy;
      shapeI.endX = endX + dx;
      shapeI.endY = endY + dy;
    };
  } else {
    // 创建新的图形
    let currentShape = null;
    switch (shape) {
      case "rect":
        currentShape = new RectShape(e.offsetX, e.offsetY, color);
        break;
      case "circle":
        currentShape = new CircleShape(e.offsetX, e.offsetY, color);
        break;
      case "line":
        currentShape = new LineShape(e.offsetX, e.offsetY, color);
        break;
      case "text":
        // const text = textInput.value || "默认文本"; // 获取输入框中的文本
        const selectValue = select.value || "默认标记";
        currentShape = new TextShape(e.offsetX, e.offsetY, color, selectValue);
        break;

      default:
        alert("请选择形状");
        return;
    }
    shapes.push(currentShape);
    const cvsRact = canvas.getBoundingClientRect();
    window.onmousemove = (e) => {
      const x = e.clientX - cvsRact.left;
      const y = e.clientY - cvsRact.top;
      currentShape.endX = x;
      currentShape.endY = y;
    };
  }
  // 监听鼠标抬起 停止移动
  window.onmouseup = () => {
    window.onmousemove = null;
    window.onmouseup = null;
  };
});
//监听双击事件
canvas.addEventListener("dblclick", (e) => {
  const shapeI = getShape(e.offsetX, e.offsetY);
  if (shapeI) {
    shapes.splice(shapes.indexOf(shapeI), 1);
  }
});
// 绘制图形
function draw() {
  // 使用requestAnimationFrame来实现动画
  requestAnimationFrame(draw);
  // 清空画布
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  // 绘制所有图形
  shapes.forEach((shape) => {
    shape.draw();
  });
}
draw();

// 获取图形
function getShape(x, y) {
  // 从后往前遍历图形 因为后绘制的图形会覆盖先绘制的图形
  for (let i = shapes.length - 1; i >= 0; i--) {
    const shape = shapes[i];
    if (shape.isIn(x, y)) {
      return shape;
    }
  }
  return null;
}
//输出图形信息
const onSubmit = document.getElementById("submit");
onSubmit.addEventListener("click", () => {
  outputInfo();
});
function outputInfo() {
  const infoTable = document.getElementById("info-table");
  // 清空表格 不清除表头
  infoTable.innerHTML = `<thead>
          <tr>
            <th>类型</th>
            <th>startX</th>
            <th>startY</th>
            <th>endX</th>
            <th>endY</th>
            <th>颜色</th>
            <th>半径</th>
            <th>文本</th>
          </tr>
        </thead>`;
  shapes.forEach((shape) => {
    const row = infoTable.insertRow();
    const typeCell = row.insertCell();
    typeCell.textContent = shape.getInfo().type;
    const startXCell = row.insertCell();
    startXCell.textContent = shape.getInfo().startX;
    const startYCell = row.insertCell();
    startYCell.textContent = shape.getInfo().startY;
    const endXCell = row.insertCell();
    endXCell.textContent = shape.getInfo().endX;
    const endYCell = row.insertCell();
    endYCell.textContent = shape.getInfo().endY;
    const colorCell = row.insertCell();
    colorCell.textContent = shape.getInfo().color;
    const radiusCell = row.insertCell();
    radiusCell.textContent = shape.getInfo().radius;
    const textCell = row.insertCell();
    textCell.textContent = shape.getInfo().text;
  });
}

```

