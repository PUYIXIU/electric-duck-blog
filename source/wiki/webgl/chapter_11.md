---
layout: wiki
wiki: WebGL
title: 动画
order: 31
---

{% note 实现动画的关键就是：**不断擦除并重绘动画图形，并且在每次重绘时轻微的改变动画属性**。 color:blue %}

# Rotating Triangle

{% border RotatingTriangle Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/RotatingTriangle.gif RotatingTriangle运行效果 %}

{% tabs %}
<!-- tab RotatingTriangle.html -->
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="../examples/lib/cuon-matrix.js"></script>
  <script src="../examples/lib/cuon-utils.js"></script>
  <script src="../examples/lib/webgl-debug.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="./RotatingTriangle.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="300" height="300"></canvas>
</body>
</html>
```
<!-- tab RotatingTriangle.js -->
```javascript
let VSHADER_SOURCE =
  `
  attribute vec4 a_Position;
  uniform mat4 u_ModelMatrix;
  void main(){
    gl_Position = u_ModelMatrix * a_Position;
  }
  `;
let FSHADER_SOURCE =
  `
  void main(){
    gl_FragColor = vec4(0.0,1.0,0.0,0.5);
  }
  `;
let ANGLE_STEP = 45.0;
function main() {
  let canvas = document.getElementById('webgl');
  let gl = getWebGLContext(canvas);
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('着色器初始化失败');
    return;
  }
  let n = initVertexBuffer(gl);
  let u_ModelMatrix = gl.getUniformLocation(gl.program, 'u_ModelMatrix');
  gl.clearColor(0.0, 0.0, 0.0, 1.0);

  // 三角形当前旋转角度
  let currentAngle = 0.0;
  // 模型矩阵 Matrix4对象
  let modelMatrix = new Matrix4();

  // 开始绘制三角形
  var tick = function () {
    // 更新旋转角
    currentAngle = animate(currentAngle);
    draw(gl, n, currentAngle, modelMatrix, u_ModelMatrix);
    // 请求浏览器调用tick
    requestAnimationFrame(tick);
  };

  tick();
} 

function initVertexBuffer(gl) {
  let vertices = new Float32Array([
    0.0, 0.25, -0.25, -0.25, 0.25, -0.25
  ]);
  let vertexBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  gl.enableVertexAttribArray(a_Position);
  return vertices.length / 2;
}

function draw(gl, n, currentAngle, modelMatrix, u_ModelMatrix) {
  // 设置旋转矩阵
  modelMatrix.setRotate(currentAngle, 0, 0, 1);

  // 将旋转矩阵传输给顶点着色器
  gl.uniformMatrix4fv(u_ModelMatrix, false, modelMatrix.elements);
  
  // 清除canvas
  gl.clear(gl.COLOR_BUFFER_BIT);
  
  // 绘制三角形
  gl.drawArrays(gl.TRIANGLES, 0, n);
}

// 记录上一次调用函数的时刻
let g_last = Date.now();
function animate(angle) {
  // 计算距离上次调用经过多长时间
  var now = Date.now();
  var elapsed = now - g_last; // 毫秒
  g_last = now;
  // 根据距离上次调用的时间，更新当前旋转角度
  let newAngle = angle + (ANGLE_STEP * elapsed) / 1000.0;
  return newAngle %= 360;
}
```
{% endtabs %}

{% endborder %}

{% border 步骤分解 color:yellow %}

{% folding 反复调用绘制函数 tick() color:blue %}
tick中实现了关键的2个步骤：
1. 更新角度
2. 重绘

{% note 下图的基本步骤可以用来实现各种动画，是3D图形编程的关键技术。 color:purple %}
{% image ../../assets/image/WebGL/tick函数流程图.png %}

{% endfolding %}

{% folding 按照指定旋转角度绘制三角形 draw() color:blue %}
draw一共有四步操作：
{% timeline %}
<!-- node step1 -->
设置旋转矩阵。
```javascript
modelMatrix.setRotate(currentAngle,0,0,1);
```
<!-- node step2 -->
将旋转矩阵传入Vertex_Shader。
```javascript
gl.uniformMatrix4fv(u_ModelMatrix,false,modelMatrix.elements);
```
<!-- node step3 -->
清除canvas。
```javascript
gl.clear(gl.COLOR_BUFFER_BIT);
```
<!-- node step4 -->
开始绘制。
```javascript
gl.drawArrays(gl.TRIANGLES,0,n);
```
{% endtimeline %}
{% endfolding %}

{% folding 请求再次被调用 requestAnimationFrame() color:blue %}

{% border 为什么不用setInterval？ color:purple %}
如果要实现 **重复执行某个特定的任务**，  
**setInterval** 是比较常用的方法，  
但是它存在一个弊端：
**不管其被调用的标签页是否被激活，其中的setInterval()函数都会反复调用。**  
这会增加浏览器的负荷，  
requestAnimation只有在标签页激活时才生效。
{% endborder %}
{% image ../../assets/image/WebGL/requestAnimationFrame.png %}

也可以通过 **cancelAnimationFrame** 取消请求：
{% image ../../assets/image/WebGL/cancelAnimationFrame.png %}

{% endfolding %}

{% folding 更新旋转角 aniamte() color:blue %}
要注意的是：调用requestAnimationFrame(),  
**只是请求浏览器在适当的时机调用参数函数**，  
因此不能保证每次调用间隔的时间相同， 
**需要计算时间间隔，并由时间间隔计算这一帧中三角形的旋转角度。**

> g_last = 上一帧开始的时间
> elapsed = 两帧之间时间间隔 = 这一帧开始的时间 - g_last
> ANGLE_STEP/1000.0 = 每毫秒旋转度数
> 旋转变化度数 = ANGLE_STEP / 1000.0 * elapsed

```javascript
let newAngle = angle + (ANGLE_STEP * elapsed) / 1000.0;
```
{% endfolding %}

{% endborder %}

# Rotating Translated Triangle

{% border 结合本章内容的小动画 color:yellow %}

{% image ../../assets/image/WebGL/RotatingTranslatedTriangle.gif  %}
{% image ../../assets/image/WebGL/RotatingHeart.gif  %}

{% endborder %}
