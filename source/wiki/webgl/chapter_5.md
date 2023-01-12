---
layout: wiki
wiki: WebGL
title: 改变点的颜色
order: 13
---
# ColoredPoints 

{% border 本程序主要是介绍： color:yellow %}
如何使用 **uniform变量** 将值传给shader的。  

{% timeline %}
<!-- node 第一步：声明 -->
在 **Fragment_Shader** 中声明好 **uniform变量** 。
<!-- node 第二步：赋值 -->
用 **uniform变量** 向 **gl_FragColor** 赋值。
<!-- node 第三步：传输 -->
将颜色数据传输给 **uniform变量** 。

{% endtimeline %}

{% endborder %}

{% border ColoredPoints Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/ColoredPoints.png ColoredPoints运行结果 %}

{% tabs %}
<!-- tab ColoredPoints.html -->
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=, initial-scale=1.0">
  <title>Document</title>
  <script src="../examples/lib/cuon-utils.js"></script>
  <script src="../examples/lib/webgl-debug.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="./ColoredPoints.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="400" height="300"></canvas>
</body>
</html>
```
<!-- tab ColoredPoints.js -->
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
// 片源着色器
var FSHADER_SOURCE =
  `
  precision mediump float;
  uniform vec4 u_FragColor;
  void main(){
    gl_FragColor = u_FragColor;
  }
  `;
function main() { 
  let canvas = document.getElementById('webgl');
  let gl = getWebGLContext(canvas);
  if (!gl) { 
    console.log('获取上下文失败');
    return;
  }
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('着色器初始化失败');
    return;
  }
  // 获取a_Position变量的存储位置
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  // 获取u_FragColor变量的存储位置
  let u_FragColor = gl.getUniformLocation(gl.program, 'u_FragColor');
  if (a_Position < 0 || u_FragColor ) {
    console.log('获取变量存储位置失败');
    return;
  }

  // 注册鼠标点击事件
  canvas.onmousedown = function (ev) {
    click(ev, gl, canvas, a_Position, u_FragColor);
  }

  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);

}

let gl_points = [];
let gl_colors = [];
function click(ev,gl,canvas,a_Position,u_FragColor) { 
  let x = ev.clientX;
  let y = ev.clientY;
  let rect = ev.target.getBoundingClientRect();
  x = (x - rect.left - canvas.width / 2) / (canvas.width / 2);
  y = (rect.top + canvas.height / 2 - y) / (canvas.height / 2);
  gl_points.push([x, y]);
  gl_colors.push([
    Math.random().toFixed(1),
    Math.random().toFixed(1),
    Math.random().toFixed(1),
    1.0,
  ]);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl_points.forEach((point, index) => {
    // 将坐标存储到gl_points数组中
    gl.vertexAttrib2f(a_Position, point[0], point[1]);
    //
    gl.uniform4f(
      u_FragColor,
      gl_colors[index][0],
      gl_colors[index][1],
      gl_colors[index][2],
      gl_colors[index][3]
    )
    gl.drawArrays(gl.POINTS, 0, 1);
  })
}
```
{% endtabs %}
{% endborder %}

# uniform 变量

{% border color:yellow %}

{% image ../../assets/image/WebGL/向片源着色器传输数据的两种方式.png 向片源着色器传输数据的两种方式 %}

{% folding uniform的声明 color:blue %}
uniform用于传输 **一致不变的数据** ，  
其声明格式与 **attribute** 相同：
> <存储限定符> <类型> <变量名>

{% image ../../assets/image/WebGL/uniform变量的声明.png %}

{% border 精度限定词（precision qualifier） color:purple %}
用于指定变量的范围和精度。
```c
precision mediump float; //表示中等精度
```
{% endborder %}

{% endfolding %}

{% folding 获取uniform变量的存储地址 color:blue %}

{% image ../../assets/image/WebGL/getUniformLocation.png %}

{% border getUniformLocation与getAttribLocation返回值的区别 color:purple %}
{% split %}
<!-- cell left -->
{% border getAttribLocation： %}
如果attribute变量不存在，返回 **-1** 。
{% endborder %}
<!--cell right-->
{% border getUniformLocation： %}
如果uniform变量不存在，返回 **null** 。
{% endborder %}
{% endsplit %}
{% endborder %}
{% endfolding %}

{% folding 向uniform变量赋值 color:blue %}
**uniform\[1-4]f** 的功能与参数与 **vertexAttrib\[1-4]f** 很相近。  

{% image ../../assets/image/WebGL/uniform4f.png uniform4f的功能与参数 %}

{% image ../../assets/image/WebGL/uniformnf.png uniform4f的同族函数 %}

{% endfolding %}

{% endborder %}
