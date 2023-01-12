---
layout: wiki
wiki: WebGL
title: 颜色与纹理
order: 40
---

# MutiAttributeSize
 
{% note 本案例主要介绍**如何将多个顶点数据通过缓冲区对象传入顶点着色器** color:blue %}

{% border MutiAttributeSize Code color:yellow child:tabs %}

{% image ../../assets/image/WebGL/MutiAttributeSize.png MutiAttributeSize运行结果 %}

{% tabs %}
<!-- tab MutiAttributeSize.html -->
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
  <script src="./MutiAttributeSize.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="300" height="300"></canvas>
</body>
</html>
```
<!-- tab MutiAttributeSize.js -->
```javascript
let VSHADER_SOURCE =
    `
  attribute vec4 a_Position;
  attribute float a_PointSize;
  void main(){
    gl_Position = a_Position;
    gl_PointSize = a_PointSize;
  }
  `;
let FSHADER_SOURCE =
    `
  void main(){
    gl_FragColor = vec4(0.0,1.0,0.0,0.5);
  }
  `;
function initVertexBuffer(gl) {
    let vertices = new Float32Array([
        0.0, 0.5, -0.5, -0.5, 0.5, -0.5
    ]);
    let sizes = new Float32Array([
        10.0, 20.0, 30.0
    ]);
    let vertexBuffer = gl.createBuffer();
    let sizeBuffer = gl.createBuffer();

    gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
    let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
    gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(a_Position);

    gl.bindBuffer(gl.ARRAY_BUFFER, sizeBuffer);
    gl.bufferData(gl.ARRAY_BUFFER, sizes, gl.STATIC_DRAW);
    let a_PointSize = gl.getAttribLocation(gl.program, 'a_PointSize');
    gl.vertexAttribPointer(a_PointSize, 1, gl.FLOAT, false, 0, 0);
    gl.enableVertexAttribArray(a_PointSize);

    return sizes.length
}
function main() {
    let canvas = document.getElementById('webgl');
    let gl = getWebGLContext(canvas);
    if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
        console.log('初始化着色器失败');
        return;
    }
    let n = initVertexBuffer(gl);
    gl.clearColor(0.0, 0.0, 0.0, 1.0);
    gl.clear(gl.COLOR_BUFFER_BIT);
    gl.drawArrays(gl.POINTS, 0, n);
}
```
{% endtabs %}

{% endborder %}

