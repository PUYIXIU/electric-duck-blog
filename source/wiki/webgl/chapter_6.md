---
layout: wiki
wiki: WebGL
title: 绘制多个点
order: 20
---

{% border 三维模型的基本单位是什么？ color:yellow %}
答： **三角形 △**  
不管三维模型的形状多么复杂，基本组成单位都是三角形。  
只不过复杂的模型由更多的三角形构成而已。
{% endborder %}

# MultiPoint

{% border 缓冲区对象（buffer object） color:yellow %}
如果你要绘制一个图形，你需要**一次性的将图形的顶点全部传入Vertex_Shader**。  
**缓冲区对象** 支持一次性地向shader传入多个顶点的数据。  
它是WebGL系统中的一块**内存区域**。
{% endborder %}

{% border MultiPoints Code child:tabs color:yellow %}

{% image ../../assets/image/WebGL/MultiPoints.png MultiPoints绘制效果 %}

{% tabs %}
<!-- tab MultiPoints.html -->
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
  <script src="./MultiPoints.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="500" height="300"></canvas>
</body>
</html>
```
<!-- tab MultiPoints.js -->
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
  `
function main() { 
  var canvas = document.getElementById('webgl');
  var gl = getWebGLContext(canvas);
  initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE);
  
  // 设置顶点位置
  var n = initVertexBuffers(gl);
  if (n < 0) {
    console.log('设置顶点位置失败');
    return;
  }
  
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.POINTS, 0, n);
}

function initVertexBuffers(gl) {
  var vertices = new Float32Array([
    0.0, 0.5, -0.5, -0.5, 0.5, -0.5
  ]);
  var n = 3; //点数
  
  // 创建缓冲区对象
  var vertexBuffer = gl.createBuffer();
  if (!vertexBuffer) { 
    console.log('创建缓冲区对象失败');
    return -1;
  }

  // 将缓冲区对象绑定到目标
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);

  // 向缓冲区对象中写入数据
  gl.bufferData(gl.ARRAY_BUFFER, vertices, gl.STATIC_DRAW);

  var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  
  // 将缓冲区对象分配给a_Position变量
  gl.vertexAttribPointer(a_Position, 2, gl.FLOAT, false, 0, 0);

  // 连接a_Position变量与分配给它的缓冲区对象
  gl.enableVertexAttribArray(a_Position);
  return n;
}
```
{% endtabs %}

{% endborder %}

{% border 使用缓冲区对象流程 color:yellow %}
{% note 缓冲区对象是WebGL系统中的一块存储区。 color:blue %}

{% image ../../assets/image/WebGL/缓冲区对象.png %}

{% timeline %}
<!-- node 1. 创建缓冲区对象 -->
{% border gl.createBuffer() color:blue %}
```javascript
var vertexBuffer = gl.createBuffer();
if(!vertexBuffer){
    console.log('Failed to create the buffer object');
    return -1;
}
```
{% endborder %}
<!-- node 2. 绑定缓冲区对象 -->
{% border gl.bindBuffer() color:blue %}
```javascript
gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
```
{% endborder %}
<!-- node 3. 将数据写入缓冲区对象 -->
{% border gl.bufferData() color:blue %}
```javascript
gl.bufferData(gl.ARRAY_BUFFER,vertices,gl.STATIC_DRAW);
```
{% endborder %}
<!-- node 4. 将缓冲区对象分配给一个attribute变量 -->
{% border gl.vertexAttribPointer() color:blue %}
```javascript
var a_Position = gl.getAttribLocation(gl.program,'a_Position');
gl.vertexAttribPointer(a_Position,2,gl.FLOAT,false,0,0);
```
{% endborder %}
<!-- node 5. 开启attribute变量 -->
{% border gl.enableVertexAttribArray() color:blue %}
```javascript
gl.enableVertexAttribArray(a_Position);
```
{% endborder %}
{% endtimeline %}

{% endborder %}

{% border 步骤分解 color:yellow %}

{% folders %}

<!-- folder 1. 创建缓冲区对象 -->

{% border gl.createBuffer() color:blue %}

{% image ../../assets/image/WebGL/创建缓冲区对象.png %}

{% note 缓冲区对象可以被 **创建** 和 **删除** 。 color:purple %}

{% image ../../assets/image/WebGL/createBuffer.png createBuffer %}

{% image ../../assets/image/WebGL/deleteBuffer.png deleteBuffer %}


{% endborder %}

<!-- folder 2. 绑定缓冲区对象 -->
{% border gl.bindBuffer() color:blue %}

将 **缓冲区对象** 绑定到WebGL系统中已经存在的 **target** 上，  
这个 **target** 表示 **缓冲区对象的用途**。  

{% image ../../assets/image/WebGL/绑定缓冲区.png %}

{% image ../../assets/image/WebGL/bindBuffer.png bindBuffer %}

{% endborder %}

<!-- folder 3. 写入缓冲区对象 -->
{% border gl.bufferData() color:blue %}

{% note 不能直接向缓冲区写入数据，只能向与缓冲区绑定的目标写入数据。 color:purple %}

{% image ../../assets/image/WebGL/写入缓冲区对象.png %}

{% image ../../assets/image/WebGL/bufferData.png bufferData %}

{% endborder %}
<!-- folder 4. 分配attribute变量给缓冲区对象 -->
{% border gl.vertexAttribPointer() color:blue %}
{% note vertexAttribPointer可以将整个缓冲区对象的引用分配给attribute变量 color:purple %}

{% image ../../assets/image/WebGL/给缓冲区分配对象.png %}

{% image ../../assets/image/WebGL/vertexAttribPointer.png vertexAttribPointer %}

{% endborder %}
<!-- folder 5. 开启attribute对象 -->

{% border gl.enableVertexAttribArray() color:blue %}
通过enableVertexAttribArray，  
**缓冲区对象和attribute变量之间的连接就真正建立起来了。**  

{% image ../../assets/image/WebGL/变量与缓冲区对象建立连接.png %}

{% note enableVertexAttribArray函数名会让很多人误解它是用于处理顶点的，这是由于**继承自OpenGL**。 color:red %}

缓冲区对象与attribute变量之间的连接可以通过 **enableVertexAttribArray** 建立，  
同时也可以通过 **disableVertexAttribArray** 关闭。

{% image ../../assets/image/WebGL/enableVertexArray.png enableVertexArray %}

{% image ../../assets/image/WebGL/disableVertexArray.png disableVertexArray %}

{% endborder %}

<!-- folder 6. 开始绘制 -->
{% border gl.drawArrays() color:blue %}
{% note shader的执行次数与drawArrays的count参数相一致。 color:purple %}
因此在MultiPoints中，Shader实际上被执行了3次。

{% image ../../assets/image/WebGL/顶点着色器的多次执行.png 顶点着色器执行过程中缓冲区数据的传输过程 %}

下面是2个对于gl.drawArrays函数的小实验：

{% folding drawArrays(gl.POINTS,0,1) color:purple %}
{% image ../../assets/image/WebGL/MultiPoints2.png %}
{% endfolding %}

{% folding drawArrays(gl.POINTS,1,2) color:purple %}
{% image ../../assets/image/WebGL/MultiPoints3.png %}
{% endfolding %}

{% endborder %}
 
{% endfolders %}


{% endborder %}

# 类型化数组

{% note WebGL中的很多操作都要用到类型化数组。 color:yellow %}

{% border 提示: color:red %}
- 类型化数组不支持 **push** 和 **pop** 。
- 创建类型化数组的唯一方法就是使用 **new运算符** 。
{% endborder %}
  
{% image ../../assets/image/WebGL/WebGL使用的各种类型化数组.png WebGL使用的各种类型化数组 %}

{% image ../../assets/image/WebGL/类型化数组的方法、属性和常量.png 类型化数组的方法、属性和常量 %}
