---
layout: wiki
wiki: WebGL
title: 绘制多边形
order: 21
---

# Hello Triangle

{% border HelloTriangle Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/HelloTriangle.png HelloTriangle运行效果图 %}

{% tabs %}
<!-- tab HelloTriangle.html -->
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
  <script src="./HelloTriangle.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="500" height="400"></canvas>
</body>
</html>
```
<!-- tab HelloTriangle.js -->
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
      gl_FragColor = vec4(0.0,1.0,0.0,0.4);
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
  // 设置顶点位置
  let n = initVertexBuffer(gl);
  if (n < 0) {
    console.log('缓存区初始化失败');
    return;
  }
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, n);
}  

function initVertexBuffer(gl) {
  let vertices = new Float32Array([
    0.0,0.5,-0.5,-0.5,0.5,-0.5
  ]);
  let n = vertices.length / 2;
  // 创建缓冲区
  let vertexBuffer = gl.createBuffer();
  // 绑定缓冲区
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  // 写入缓冲区
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
  // 分配变量给缓冲区
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  // 连通变量与缓冲区
  gl.enableVertexAttribArray(a_Position);

  return n;
};
```
{% endtabs %}

{% endborder %}

# 基本图形

{% border gl.drawArrays()第一个参数mode color:yellow %}

通过给第一个参数mode指定不同的值，  
可以以7种不同的方式来绘制图形：

{% folders %}
<!-- folder gl.POINTS -->
{% border  %}
{% image ../../assets/image/WebGL/gl.POINTS.png  %}
{% image ../../assets/image/WebGL/gl.POINTS2.png  %}
{% image ../../assets/image/WebGL/gl.POINTS3.png  %}
{% endborder %}

<!-- folder gl.LINES -->
{% border  %}
{% image ../../assets/image/WebGL/gl.LINES.png  %}
{% image ../../assets/image/WebGL/gl.LINES2.png  %}
{% image ../../assets/image/WebGL/gl.LINES3.png  %}
{% endborder %}

<!-- folder gl.LINE_STRIP -->
{% border  %}
{% image ../../assets/image/WebGL/gl.LINE_STRIP.png  %}
{% image ../../assets/image/WebGL/gl.LINE_STRIP2.png  %}
{% image ../../assets/image/WebGL/gl.LINE_STRIP3.png  %}
{% endborder %}

<!-- folder gl.LINE_LOOP -->
{% border  %}
{% image ../../assets/image/WebGL/gl.LINE_LOOP.png  %}
{% image ../../assets/image/WebGL/gl.LINE_LOOP2.png  %}
{% image ../../assets/image/WebGL/gl.LINE_LOOP3.png  %}
{% endborder %}

<!-- folder gl.TRIANGLES -->
{% border  %}
{% image ../../assets/image/WebGL/gl.TRIANGLES.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLES2.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLES3.png  %}
{% endborder %}

<!-- folder gl.TRIANGLES_STRIP -->
{% border  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_STRIP.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_STRIP2.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_STRIP3.png  %}
{% endborder %}

<!-- folder gl.TRIANGLES_FAN -->
{% border  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_FAN.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_FAN2.png  %}
{% image ../../assets/image/WebGL/gl.TRIANGLE_FAN3.png  %}
{% endborder %}

{% endfolders %}

{% endborder %}

# Hello Rectangle

{% border HelloQuad Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/HelloQuad.png HelloQuad运行结果 %}

{% tabs %}
<!-- tab HelloQuad.html -->
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
  <script src="./HelloQuad.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="300" height="300"></canvas>
</body>
</html>
```
<!-- tab HelloQuad.js -->
```javascript
var VSHADER_SOURCE =
  `
    attribute vec4 a_Position;
    void main(){
      gl_Position = a_Position;
    }
  `;
var FSHADER_SOURCE =
  `
    void main(){
      gl_FragColor = vec4(0.0,1.0,0.0,0.6);
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
  let n = initVertexBuffer(gl);
  if (n < 0) {
    console.log('缓冲区初始化失败');
    return;
  }
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLE_STRIP, 0, n);
}
function initVertexBuffer(gl) {
  let vertices = new Float32Array([
    -0.5, 0.5,
    -0.5, -0.5,
    0.5, 0.5,
    0.5, -0.5,
  ]);
  let n = vertices.length / 2;
  let vertexBuffer = gl.createBuffer();
  if (!vertexBuffer) { 
    return -1;
  }
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  gl.enableVertexAttribArray(a_Position);
  return n;
}
```
{% endtabs %}

{% endborder %}
