---
layout: wiki
wiki: WebGL
title: 变换矩阵
order: 23
---

# 变换矩阵：旋转

{% border 变换矩阵（Transformation matrix） color:yellow %}

{% folding 为什么需要变换矩阵？ color:blue %}

如果我们要实现一个 **旋转后平移**的操作，需要以下的等式：
```javascript
let VSHADER_SOURCE =
  `
  attribute vec4 a_Position;
  uniform vec4 u_Transition;
  uniform vec2 u_SinCos;
  void main(){
    gl_Position.x = a_Position.x*u_SinCos.x - a_Position.y*u_SinCos.y + u_Transition.x;
    gl_Position.y = a_Position.y*u_SinCos.x + a_Position.x*u_SinCos.y + u_Transition.y;
    gl_Position.z = a_Position.z + u_Transition.z;
    gl_Position.w = a_Position.w;
  }
  `;
```
可以看到变换式变得越来越繁琐，  
但是如果用变量矩阵来实现就能简单很多，  
**变换矩阵（Transformation matrix）** 相当适合操作计算机图形。

{% endfolding %}

{% folding 一些矩阵乘法基本条件 color:red %}
- 矩阵乘法不满足交换律
- 矩阵列数必须等于矢量行数
{% endfolding %}

{% folding 矩阵乘法等式 color:blue %}
{% image ../../assets/image/WebGL/矩阵乘法等式1.png %}
{% image ../../assets/image/WebGL/矩阵乘法等式2.png %}
{% endfolding %}

{% endborder %}

{% border 旋转矩阵（rotation matrix） color:blue %}

{% image ../../assets/image/WebGL/旋转矩阵等式1.png %}
{% image ../../assets/image/WebGL/旋转矩阵等式2.png %}
{% endborder %}

# 变换矩阵：平移

{% border 需要一个4×4的变换矩阵 color:yellow %}
因为在平移的等式中存在 **常量项** ，  
因此需要给三维矢量添加 **第4个分量** ，  
同时需要对变换矩阵进行扩充。  

{% image ../../assets/image/WebGL/平移矩阵等式1.png %}
{% image ../../assets/image/WebGL/平移矩阵等式2.png %}

{% endborder %}

# 4×4的旋转矩阵

{% border 旋转矩阵的维度扩充 color:yellow %}
{% image ../../assets/image/WebGL/四维旋转矩阵等式1.png %}
{% image ../../assets/image/WebGL/四维旋转矩阵等式2.png %}
{% endborder %}

# RotatedTriangle_Matrix

{% border RotatedTriangle_Matrix Code color:yellow %}

{% image ../../assets/image/WebGL/RotatedTriangle_Matrix.png RotatedTriangle_Matrix运行效果 %}

{% tabs %}
<!-- tab RotatedTriangle_Matrix.html -->
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="../examples/lib/cuon-utils.js"></script>
  <script src="../examples/lib/webgl-debug.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="./RotatedTriangle_Matrix.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width='300' height="300"></canvas>
</body>
</html>
```
<!-- tab RotatedTriangle_Matrix.js -->
```javascript
var VSHADER_SOURCE =
  `
  attribute vec4 a_Position;
  uniform mat4 u_xformMatrix;
  void main(){
    gl_Position = u_xformMatrix * a_Position;
  }
  `;
var FSHADER_SOURCE =
  `
  void main(){
    gl_FragColor = vec4(0.0,1.0,0.0,0.4);
  }
  `;
// 旋转角度
let ANGLE = 90.0;
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
  // 设置顶点位置
  let n = initVertexBuffers(gl);
  if (n < 0) {
    console.log('缓冲区初始化失败');
    return;
  }
  // 创建旋转矩阵
  let radian = Math.PI * ANGLE / 180.0;
  let cosB = Math.cos(radian), sinB = Math.sin(radian);

  // WebGL中矩阵是列主序的
  let xformMatrix = new Float32Array([
    cosB, sinB, 0, 0,
    -sinB, cosB, 0, 0,
    0, 0, 1, 0,
    0, 0, 0, 1
  ]);
  let u_xformMatrix = gl.getUniformLocation(gl.program, 'u_xformMatrix');
  if (!u_xformMatrix) {
    console.log('获取变量失败');
    return;
  }
  gl.uniformMatrix4fv(u_xformMatrix, false, xformMatrix);

  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, n);
}


function initVertexBuffers(gl) {
  let vertices = new Float32Array([
    0.0, 0.3, -0.3, -0.3, 0.3, -0.3
  ]);
  let vertexBuffer = gl.createBuffer();
  if (!vertexBuffer) {
    return -1;
  }
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  gl.enableVertexAttribArray(a_Position);
  return vertices.length / 2;
}
```
{% endtabs %}

{% endborder %}

{% border 一些小问题 color:yellow %}

{% folding 传入着色器内的矩阵变量的处理 color:blue %}
使用一行就能够实现矢量与矩阵的加法：
```javascript
gl_Position = u_xformMatrix * a_Position ;
```
因为shader内置了常用的矢量和矩阵运算功能。  

{% note mat4类型指的是4×4的矩阵类型 color:blue %}

{% endfolding %}

{% folding 两种在数组中存储矩阵元素的方式 color:blue %}

{% note WebGL和OpenGL的矩阵元素都是**按列主序**存储在数组中的。 color:yellow %}

{% split bg:card %}
<!-- cell left -->
**按行主序（row major order）**
{% image ../../assets/image/WebGL/按行主序.png %}

<!-- cell right -->
**按列主序（column major order）**
{% image ../../assets/image/WebGL/按列主序.png %}

{% endsplit %}
{% endfolding %}

{% folding gl.uniformMatrix4fx() color:blue %}
函数名最后一个字母是v，  
表示**可以向shader传输多个数据值**。
{% image ../../assets/image/WebGL/uniformMatrix4fv.png %}
{% border 第二个参数Transpose之所以必须是false color:red %}
是由于该参数表示 **是否转置矩阵**，  
因为WebGL没有提供矩阵转置的方法，  
所以该参数必须是false。
{% endborder %}
{% endfolding %}

{% folding 平移：相同的策略 color:blue %}

{% image ../../assets/image/WebGL/TranslatedTriangle_Matrix.png 运行结果 %}

{% border child:codeblock %}
{% kbd 平移使用的变换矩阵 %}
```javascript
  let xformMatrix = new Float32Array([
    1.0, 0.0, 0.0, 0.0,
    0.0, 1.0, 0.0, 0.0,
    0.0, 0.0, 1.0, 0.0,
    Tx, Ty, 0.0, 1.0
  ]);
```
{% endborder %}
{% endfolding %}

{% folding 变换矩阵：缩放 color:blue %}

{% timeline %}
<!-- node 图解 -->
{% image ../../assets/image/WebGL/缩放变换.png %}

<!-- node 等式 -->
{% image ../../assets/image/WebGL/缩放等式.png %}

<!-- node 变换矩阵 -->
{% image ../../assets/image/WebGL/缩放矩阵.png %}

<!-- node 矩阵数组 -->
```javascript
  let xformMatrix = new Float32Array([
    Sx, 0, 0, 0,
    0, Sy, 0, 0,
    0, 0, Sz, 0,
    0, 0, 0, 1.0
  ])
```
<!-- node 运行结果 -->
{% image ../../assets/image/WebGL/ScaleTriangle_Matrix.png %}


{% endtimeline %}

{% endfolding %}

{% endborder %}
