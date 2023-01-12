---
layout: wiki
wiki: WebGL
title: 鼠标绘点
order: 12
---

# ClickedPoints

{% border ClickedPoints Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/ClickedPoints.png ClcikedPoints运行结果 %}

{% tabs %}
<!-- tab ClickedPoints.html -->
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="../examples/lib/webgl-debug.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="../examples/lib/cuon-utils.js"></script>
  <script src="./ClickedPoints.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="400" height="400"></canvas>
</body>
</html>
```
<!-- tab ClickedPoints.js -->
```javascript
// 顶点着色器
var VSHADER_SOURCE =
  `
    attribute vec4 a_Position;
    void main(){
      gl_Position = a_Position;
      gl_PointSize = 10.0;
    }
  `;
// 片元着色器
var FSHADER_SOURCE =
  `
    void main(){
      gl_FragColor = vec4(0.0,1.0,0.0,1.0);
    }
  `;
function main() { 
  // 获取canvas元素
  let canvas = document.getElementById('webgl');
  // 获取WebGL上下文
  let gl = getWebGLContext(canvas);
  if (!gl) { 
    console.log('获取上下文失败');
    return;
  }
  // 初始化着色器
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('着色器初始化失败');
    return;
  }
  // 获取attribute变量存储位置
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  if (a_Position < 0) {
    console.log('变量内存获取失败');
    return;
  }
  // 注册鼠标点击事件响应函数
  canvas.onmousedown = function (ev) {
    click(ev, gl, canvas, a_Position);
  };
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
}

var g_points = []; //鼠标点击位置数组
function click(ev, gl, canvas, a_Position) {
    var x = ev.clientX; //鼠标点击处x坐标
    var y = ev.clientY; //鼠标点击处y坐标
    var rect = ev.target.getBoundingClientRect();
    x = ((x - rect.left) - canvas.width / 2) / (canvas.width / 2);
    y = (canvas.height / 2 - (y - rect.top)) / (canvas.height / 2);
    // 将坐标存储到g_points数组中
    g_points.push(x);
    g_points.push(y);
    // 清除canvas
    gl.clear(gl.COLOR_BUFFER_BIT);
    var len = g_points.length;
    for (var i = 0; i < len; i += 2){
        // 将点的位置传递到变量中a_Position
        gl.vertexAttrib3f(a_Position, g_points[i], g_points[i + 1], 0.0);
        // 绘制点
        gl.drawArrays(gl.POINTS, 0, 1);
    }
}
```
{% endtabs %}

{% endborder %}


# 响应鼠标点击事件

{% border 下面是onmousedown触发的后续操作 color:yellow %}

{% timeline %}
<!-- node 第一步 -->
获取鼠标点击的位置，存储于一个数组中。
```javascript
var x = ev.clientX;
var y = ev.clientY;
var rect = ev.target.getBoundingClientRect();
x = ((x-rect.left-canvas.width/2))/(canvas.width/2);
y = ((rect.top+canvas.height/2-y))/(canvas.height/2);
gl_points.push(x);
gl_points.push(y);
```
<!-- node 第二步 -->
清空canvas
```javascript
gl.clear(gl.COLOR_BUFFER_BIT);
```
<!-- node 第三步 -->
在相应的位置绘制点
```javascript
var len = gl_points.length;
for(var i=0;i<len;i+=2){
    gl.vertexAttrib3f(a_Position,gl_points[i],gl_points[i+1],1.0);
    gl.drawArrays(gl.POINTS,0,1);
}
```

{% endtimeline %}
{% endborder %}

{% border 设置坐标的2个注意点 color:blue %}
- **ev.client\[X|Y]** 是相对 **浏览器客户区（client area）** 的坐标。
  {% image ../../assets/image/WebGL/浏览器坐标系下的鼠标点击坐标.png %}
- canvas坐标系和WebGL坐标系的 **原点** 和 **Y轴正方向** 都不同。
  {% image ../../assets/image/WebGL/canvas坐标系与WebGL坐标系.png %}
{% endborder %}
  
{% border 坐标映射步骤分解 color:yellow %}
{% timeline %}
<!-- node 第一步 -->
将鼠标点击位置的坐标从 **客户区坐标系** 转换到 **canvas坐标系** 上：
> ( x, y) → ( x - rect.left, y - rect.top)
<!-- node 第二步 -->
计算 **WebGL中心点** 相对 **canvas坐标系** 的坐标位置：
> ( canvas.width/2, canvas.height/2)
<!-- node 第三步 -->
计算 **鼠标点** 相对 ***WebGL中心点** 的坐标位置：
> ( x - rect.left - canvas.width/2 , canvas.height/2 - y + rect.top)
<!-- node 第四步 -->
将坐标进行 **归一化处理**：
>( ( x - rect.left - canvas.width )/( canvas.width/2 ), ( canvas.height/2 - y + rect.top )/(canvas.height/2) )

{% endtimeline %}
{% endborder %}

{% border 颜色缓冲区的重置 color:yellow %}
{% note **为什么要把每一次点击的点位都记录下来？** color:blue %}

**因为WebGL使用的是颜色缓冲区。**  
绘制操作实际上是在颜色缓冲区中进行绘制的，  
一旦绘制结束，颜色缓冲区就会被 **重置**，内容就会丢失。  
因此如果你希望保留已经绘制的内容，  
就需要保留涉及到绘制的变量所占用的内存地址。

{% endborder %}
