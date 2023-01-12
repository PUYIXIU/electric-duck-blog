---
layout: wiki
wiki: WebGL
title: 复合变换
order: 30
---

# 平移，然后旋转
{% note 大多数WebGL开发者都使用**矩阵操作函数库**来隐藏矩阵计算的细节。 color:blue %}

{% border 矩阵变换库：cuon-matrix.js color:yellow %}
OpenGL提供了一系列创建变换矩阵的函数，但WebGL都没有。

{% border matrix4 color:blue %}
在 cuon-matrix.js中提供了一种 {% mark Matrix4 color:purple %} 类，  
其中封装了许多创建变换矩阵的方法。  
Matrix4 表示 **4×4的矩阵**，  
其实例内部一贯使用 **Float32Array类型** 来存储矩阵元素。

{% endborder %}

{% endborder %}

# RotatedTriangle_Matrix4

{% border RotatedTriangle_Matrix4 Code color:yellow child:tabs %}
{% image ../../assets/image/WebGL/RotatedTriangle_Matrix4.png RotatedTriganle_Matrix运行结果 %}

{% tabs %}
<!-- tab RotatedTriangle_Matrix4.html -->
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
  <script src="../examples/lib/cuon-matrix.js"></script>
  <script src="./RotatedTriangle_Matrix4.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="300" height="300"></canvas>
</body>
</html>
```
<!-- tab RotatedTriangle_Matrix4.js -->
```javascript
let VSHADER_SOURCE =
  `
  attribute vec4 a_Position;
  uniform mat4 u_xformMatrix;
  void main(){
    gl_Position = u_xformMatrix * a_Position;
  }
  `;
let FSHADER_SOURCE =
  `
  void main(){
    gl_FragColor = vec4(0.0,1.0,0.0,0.7);
  }
  `;
let ANGLE = 90.0;
function main() {
  let canvas = document.getElementById('webgl');
  let gl = getWebGLContext(canvas);
  if (!gl) {
    console.log('创建上下文失败');
    return;
  }
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('着色器初始化失败');
    return;
  }
  let n = initVertexBuffer(gl);
  if (n < 0) {
    console.log('顶点缓冲区初始化失败');
    return;
  }

  let xformMatrix = new Matrix4();
  xformMatrix.setRotate(ANGLE, 0, 0, 1);

  let u_xformMatrix = gl.getUniformLocation(gl.program, 'u_xformMatrix');
  if (!u_xformMatrix) {
    console.log('获取变量失败');
    return;
  }
  gl.uniformMatrix4fv(u_xformMatrix, false, xformMatrix.elements);

  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, n);
}

function initVertexBuffer(gl) {
  let vertices = new Float32Array([
    0.0, 0.3, -0.3, -0.3, 0.3, -0.3
  ]);
  let vertexBuffer = gl.createBuffer();
  if (!vertexBuffer) { return -1; }
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

{% border Matrix4.setRotate(ANGLE, x, y ,z) color:blue %}
- **ANGLE：** 表示旋转角，**是角度制**不是弧度制
- **（x，y，z）：** 表示旋转轴
    - **旋转角度是正值** 表示逆时针方向旋转
    - **绕x轴逆时针旋转** (1,0,0)
    - **绕y轴逆时针旋转** (0,1,0)
    - **绕z轴逆时针旋转** (0,0,1)
{% endborder %}

# Matrix4类型

{% folders %}
<!-- folder setIdentity() -->
{% border color:yellow %}
将Matrix4实例初始化为 **单位阵**。
```javascript
let mat4 = new Matrix4();
console.log(mat4.setIdentity().elements);
/**
 * [
 *  1,0,0,0,
 *  0,1,0,0,
 *  0,0,1,0,
 *  0,0,0,1
 * ]
 */
```
{% endborder %}

<!-- folder setTranslate(x,y,z) -->
{% border color:yellow %}
将Matrix实例设置为 **平移变换矩阵**，    
(x,y,z)表示在xyz轴上平移距离。
```javascript
  let mat4 = new Matrix4();
  console.log(mat4.setTranslate(0.3, 0.3, 0.0).elements);
  /**
   * [1, 0, 0, 0,
   *  0, 1, 0, 0,
   *  0, 0, 1, 0,
   *  0.30000001192092896, 0.30000001192092896, 0, 1]
   */
```
{% endborder %}

<!-- folder setRotate(angle,x,y,z) -->
{% border color:yellow %}
将Matrix4实例设置为 **旋转变换矩阵**。
- **angle** 表示旋转角度
- **(x,y,z)** 表示旋转轴，无需进行归一化

```javascript
  let mat4 = new Matrix4();
  console.log(mat4.setRotate(90.0, 0, 0,1).elements);
  /**
   * [
   *  6.123234262925839e-17, 1, 0, 0,
   *  -1, 6.123234262925839e-17, 0, 0,
   *  0, 0, 1, 0,
   *  0, 0, 0, 1
   * ]
   */
```
{% endborder %}
 
<!-- folder setScale(x,y,z) -->
{% border color:yellow %}
将Matrix4实例设置为 **缩放变换矩阵** ，
(x,y,z)表示**三个轴上的缩放因子**。
```javascript
  let mat4 = new Matrix4();
  console.log(mat4.setScale(3.0, 2.0, 1.0).elements);
  /**
   * [
   *  3, 0, 0, 0, 
   *  0, 2, 0, 0, 
   *  0, 0, 1, 0, 
   *  0, 0, 0, 1
   * ]
   */
```
{% endborder %}

<!-- folder translate(x,y,z) -->
{% border color:yellow %}
将Matrix4实例 **乘以一个平移变换矩阵**
```javascript
  let mat4 = new Matrix4();
  mat4.setScale(1, 2, 3);
  console.log(mat4.translate(0.1, 0.5, 0.0).elements);
  /**
   * [
   *  1, 0, 0, 0,
   *  0, 2, 0, 0,
   *  0, 0, 3, 0,
   *  0.10000000149011612, 1, 0, 1,
   * ]
   */
```
{% endborder %}

<!-- folder rotate(angle,x,y,z) -->
{% border color:yellow %}
将Matrix4实例 **乘以一个旋转变换矩阵**
```javascript
  let mat4 = new Matrix4();
  mat4.setScale(1, 2, 3);
  console.log(mat4.rotate(90, 0, 1, 0).elements);
  /**
   * [
   *  6.123234262925839e-17, 0, -3, 0,
   *  0, 2, 0, 0,
   *  1, 0, 1.8369702788777518e-16, 0,
   *  0, 0, 0, 1
   * ]
   */
```
{% endborder %}

<!-- folder scale(x,y,z) -->
{% border color:yellow %}
将Matrix4实例 **乘以一个缩放变换矩阵**
```javascript
  let mat4 = new Matrix4();
  mat4.setScale(1, 2, 3);
  console.log(mat4.scale(1, 0.5, 0.3).elements);
  /**
   * [
   *  1, 0, 0, 0,
   *  0, 1, 0, 0,
   *  0, 0, 0.8999999761581421, 0,
   *  0, 0, 0, 1
   * ]
   */
```
{% endborder %}

<!-- folder set(m) -->
{% border color:yellow %}
将Matrix4实例 **设置为m**，  
m必须是一个Matrix4实例。
```javascript
  let m = new Matrix4();
  m.setScale(1, 2, 3);
  let mat4 = new Matrix4();
  mat4.set(m);
  // true
  console.log(m.elements.toString() === mat4.elements.toString());
```
{% endborder %}

<!-- folder elements -->
{% border color:yellow %}
类型化数组 **Floated2Array**，  
包含了Matrix4实例的矩阵元素。  
{% endborder %}
{% endfolders %}

{% border 有无set前缀的区别 color:yellow %}
{% split bg:card %}
<!-- cell left -->
**有set前缀的方法**  
会根据参数计算出变换矩阵，然后将变换矩阵写入到自身中。
<!-- cell right -->
**没有set前缀的方法**  
会先根据参数计算出变换矩阵，然后将自身与刚刚计算得到的变换矩阵相乘，最终把结果写入到Matrix4对象中。
{% endsplit %}
{% endborder %}

# 复合变换

{% folding 3×3矩阵乘法 color:blue %}

{% image ../../assets/image/WebGL/三维矩阵乘法1.png %}
{% image ../../assets/image/WebGL/三维矩阵乘法2.png %}

{% endfolding %}

{% timeline %}
<!-- node 问题描述 -->
需要将图形先平移，再旋转。
<!-- node 图解 -->
{% image ../../assets/image/WebGL/先平移后旋转.png %}
<!-- node 变换等式 -->
> ( <旋转矩阵> × <平移矩阵> ) × <原始坐标>

{% endtimeline %}

# Rotated Translated Triangle

{% border RotatedTranslatedTriangle Code color:yellow %}

{% image ../../assets/image/WebGL/RotatedTranslatedTriangle.png  RotatedTranslatedTriangle运行结果 %}

{% tabs %}
<!-- tab RotatedTranslatedTriangle.html -->
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
  <script src="./RotatedTranslatedTriangle.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="300" height="300"></canvas>
</body>
</html>
```
<!-- tab RotatedTranslatedTriangle.js -->
```javascript
let ANGEL = 60.0;
let Tx = 0.5, Ty = 0.0, Tz = 0.0;

let VSHADER_SOURCE =
  `
  attribute vec4 a_Position;
  uniform mat4 u_ModelMatrix;
  void main(){
    gl_Position = a_Position * u_ModelMatrix ;
    gl_Position =  u_ModelMatrix *a_Position ;
  }
  `;
let FSHADER_SOURCE =
  `
  void main(){
    gl_FragColor = vec4(0.0,1.0,0.0,0.8);
  }
  `;

function initVertexBuffer(gl) {
  let vertices = new Float32Array([
    0.0, 0.2, -0.25, -0.25, 0.25, -0.25
  ]);
  let vertexBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);
  let a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);
  gl.enableVertexAttribArray(a_Position);
  return vertices.length / 2;
}

function main() { 
  let canvas = document.getElementById('webgl');
  let gl = getWebGLContext(canvas);
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('着色器初始化失败');
    return;
  }
  let n = initVertexBuffer(gl);

  let modelMatrix = new Matrix4();
  let u_ModelMatrix = gl.getUniformLocation(gl.program, 'u_ModelMatrix');
  modelMatrix.setRotate(ANGEL, 0, 0, 1);
  modelMatrix.translate(Tx, Ty, Tz);
  gl.uniformMatrix4fv(u_ModelMatrix, false, modelMatrix.elements);
  
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, n);
}
```
{% endtabs %}

{% folding 小案例：先旋转再平移 color:blue %}
{% timeline  %}
<!-- node 对比区别 -->
{% image ../../assets/image/WebGL/先旋转后平移1.png %}
<!-- node 需要调用的方法和顺序都要变换 -->
```javascript
  modelMatrix.setTranslate(Tx, Ty, Tz);
  modelMatrix.rotate(ANGEL, 0, 0, 1);
```

<!-- node 运行结果 -->
{% image ../../assets/image/WebGL/先旋转后平移2.png %}

{% endtimeline %}
{% endfolding %}

{% endborder %}
