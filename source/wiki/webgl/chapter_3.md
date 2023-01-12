---
layout: wiki
wiki: WebGL
title: 绘制一个点
order: 11
---

# HelloPoint1

{% border 着色器（shader） color:yellow %}
如果要在绘图区域中心绘制一个{% mark 红点 color:red %},  
理论上只需要两步：
1. 指定点的颜色
2. 指定点的位置和大小

```javascript
gl.drawColor(1.0, 0.0, 0.0, 1.0);
gl.drawPoint(0, 0, 0, 10);
```
不幸的是，WebGL依赖一种称为**着色器（shader）**的绘图机制，  
**shader**强大且复杂，仅用一条简单绘图命令无法操作。  
{% endborder %}

{% border HelloPoint1 Code child:tabs %}
{% image ../../assets/image/WebGL/HelloPoint1.png 点绘效果 %}

{% tabs %}
<!--tab HelloPoint1.html-->
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="./HelloPoint1.js"></script>
  <script src="../examples/lib/webgl-debug.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="../examples/lib/cuon-utils.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="400" height="400"></canvas>
</body>
</html>
```
<!--tab HelloPoint1.js -->
```javascript
// 顶点着色器程序
var VSHADER_SOURCE =
  `
    void main(){
      gl_Position = vec4(0.0,0.0,0.0,1.0); //设置坐标
      gl_PointSize = 10.0; //设置尺寸
    }
  `;
//片源着色器程序
var FSHADER_SOURCE =
  `
    void main(){
      gl_FragColor = vec4(1.0, 0.0 ,0.0 ,1.0); //设置颜色
    }
  `;
function main() {
  // 获取canvas元素
  var canvas = document.getElementById('webgl');
  // 获取WebGL绘图上下文
  var gl = getWebGLContext(canvas);
  if (!gl) {
    console.log('绘图上下文获取失败');
    return;
  }
  // 初始化着色器
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) { 
    console.log('着色器初始化失败');
    return;
  }
  // 设置画板背景色
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  // 清空画板
  gl.clear(gl.COLOR_BUFFER_BIT);
  // 绘制点
  gl.drawArrays(gl.POINTS, 0, 1);
}
```
{% endtabs %}
{% endborder %}

# 什么是着色器？

{% border What is shader? child:tabs color:yellow %}
{% tabs %}
<!-- tab 顶点着色器 -->
{% border Vertex shader color:blue %}
- **描述范围：** 顶点特性（如位置、颜色等）
- **顶点（vertex）：** 二维或三维空间中的一个点（如端点、交点等）
  {% endborder %}
<!-- tab 片元着色器 -->
{% border Fragment shader color:blue %}
- **描述范围：** 进行逐片元处理过程（如光照等）
- **片元（fragment）：** 类似于像素（图像的单元）
  {% endborder %}
  {% endtabs %}
  {% note shader程序以String的形式嵌入在JavaScript文件中。 color:blue %}
  {% image ../../assets/image/WebGL/shader绘制过程.png Browser执行JavaScript的绘制流程 %}
  {% image ../../assets/image/WebGL/shader绘制过程简化版.png 简化版流程 %}

{% endborder %}

{% border 使用shader的WebGL程序的结构 color:yellow child:tabs %}
{% tabs %}

<!-- tab 顶点着色器 -->
{% border Vertex shader program color:blue %}
```javascript
var VSHADER_SOURCE =
  `
    void main(){
      gl_Position = vec4(0.0,0.0,0.0,1.0);
      gl_PointSize = 10.0;
    }
  `;
```
{% endborder %}
<!--tab 片元着色器-->
{% border Fragment shader program color:blue %}
```javascript
var FSHADER_SOURCE =
  `
    void main(){
      gl_FragColor = vec4(1.0,0.0,0.0,1.0);
    }
  `;
```
{% endborder %}
<!--tab 主程序-->
{% border Main program color:blue %}
```javascript
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
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.POINTS, 0, 1);
}
```
{% endborder %}
{% endtabs %}
{% endborder %}

{% border 步骤分解 color:yellow %}

{% folders  %}

<!--folder 1.获取canvas元素-->
```javascript
let canvas = document.getElementById('webgl');
```
<!--folder 2.获取WebGL绘图上下文-->
```javascript
let gl = getWebGLContext(canvas);
```
<!--folder 3.初始化着色器-->
```javascript
initShaders(gl,VSHADER_SOURCE,FSHADER_SOURCE);
```
{% border initShaders color:blue %}
该函数被定义在 *cuon.util.js* 中。
{% image ../../assets/image/WebGL/initShaders.png %}
{% image ../../assets/image/WebGL/initShader的行为.png initShaders()的行为 %}
上图有几点需要重点注意的：

- 着色器运行在**WebGL**系统中，而不是JavaScript系统中。
- 片元着色器接收到的是经过**光栅化处理**后的片元值。
- WebGL程序包括**运行于Browser的JS**和**运行于WebGL的shader**两个部分
  {% endborder %}

<!--folder 4.设置canvas背景色-->
```javascript
gl.clearColor(0.0,0.0,0.0,1.0);
```
<!--folder 5.清除canvas-->
```javascript
gl.clear(gl.COLOR_BUFFER_BIT);
```
<!--folder 6.绘图-->
```javascript
gl.drawArrays(gl.POINTS,0,1)
```
{% endfolders %}

{% endborder %}

{% border 初始化shader后如何绘图？ color:blue %}
要画一个点，你需要3个关键信息：
- 位置
- 尺寸
- 颜色

{% border 顶点着色器 color:yellow %}
shader程序本身必须包含一个 *main* 函数。
{% note 注意：gl_Position必须赋值，gl_PointSize可选，默认值为1.0 color:red %}
{% image ../../assets/image/WebGL/顶点着色器的内置变量.png 顶点着色器的内置变量 %}
{% image ../../assets/image/WebGL/顶点着色器内置变量对应的GLSE数据类型.png 顶点着色器内置变量对应的GLSE数据类型 %}

{% border float color:blue %}
表示浮点数，因此必须使用浮点形式的数字进行赋值。  
这样写就会出错：
```javascript
gl_PointSize = 10;
```
{% endborder %}

{% border vec4 color:blue %}
**矢量（vector）**，表示由3个浮点数组成的矢量，  
shader提供了内置函数 **vec4()** 创建vec4类型的变量。
{% image ../../assets/image/WebGL/vec4.png%}
{% border 齐次坐标 color:yellow %}
有4个分量组成的矢量被称为**齐次坐标**。
{% image ../../assets/image/WebGL/齐次坐标.png%}
{% endborder %}
{% endborder %}
{% endborder %}

{% border 片元着色器 color:yellow %}
片元就是显示在屏幕上的一个像素。  
*gl_FragColor* 是片元着色器唯一的内置变量。
{% image ../../assets/image/WebGL/片元着色器的内置变量.png%}
{% endborder %}

{% border 绘制操作 color:yellow %}
*drawArrays()* 可以用来绘制各种图形。  
{% image ../../assets/image/WebGL/drawArrays.png%}
```javascript
gl.drawArrays(gl.POINTS,0,1);
```

- 第一个参数（gl.POINTS）：表示绘制单点
- 第二个参数（0）：从第1个顶点开始绘制
- 第三个参数（1）：表示仅绘制1个点

{% note 一旦Vertex shader执行完毕，Fragment shader就开始执行 %}
{% endborder %}
{% endborder %}

# WebGL坐标系统

{% border color:yellow %}
WebGL使用的是**三维坐标系统（笛卡尔坐标系）**。
{% image ../../assets/image/WebGL/WebGL坐标系.png WebGL坐标系 %}

{% border 右手坐标系 color:purple %}
WebGL默认使用右手坐标系，  
但实际上，WebGL本身不是左手也不是右手坐标系，要复杂得多。
{% image ../../assets/image/WebGL/右手坐标系.png 右手坐标系 %}
{% endborder %}

{% border 坐标映射 color:blue %}
需要将WebGL的坐标系映射到canvas绘图区中。  
坐标对应关系如下：
|WebGL坐标|canvas坐标|
|:---:|:---:|
|( 0.0, 0.0, 0.0)|中心点|
|(-1.0, 0.0, 0.0)|上边缘|
|( 1.0, 0.0, 0.0)|下边缘|
|( 0.0, -1.0, 0.0)|左边缘|
|( 0.0, 1.0, 0.0)|右边缘|

{% image ../../assets/image/WebGL/WebGL和canvas坐标映射关系.png WebGL和canvas坐标映射关系 %}
{% endborder %}

{% endborder %}

# HelloPoint2

{% border 对比版本1和版本2 color:yellow %}
- **版本1**： 点位**硬编码**在Vertex Shader中
- **版本2**： 点位坐标从**JS**传到Shader程序中
{% endborder %}
  
{% border attribute与uniform color:blue %}
要实现从JavaScript程序向Vertex_Shader传输位置信息，  
有两种方式可以实现（使用哪一个取决于需传输的数据本身）：
{% split bg:card %}
<!-- cell left -->
{% border attribute变量 color:yellow %}
传输的是那些**与顶点相关**的数据。
{% endborder %}
<!-- cell right -->
{% border uniform变量 color:yellow %}
传输的是那些**与顶点无关**的数据。
{% endborder %}
{% endsplit %}
{% image ../../assets/image/WebGL/attribute与uniform.png  %}

{% endborder %}

{% border 使用attribute变量 color:yellow %}
attribute变量实际上是一种 {% mark GLSL ES color:purple %} 变量，  
功能就是被用来 **从外部向Vertex_Shader内部传输数据** 。
{% border 使用attribute需要经过以下步骤： color:blue %}
1. **声明：** 在Vertex_Shader中声明attribute变量。
2. **赋值：** 将attribute变量赋值给gl_Position变量。
3. **传输：** 向attribute变量传输数据。
{% endborder %}

{% endborder %}
  
{% border HelloPoint2 Code child:tabs color:blue %}

{% image ../../assets/image/WebGL/HelloPoint2.png HelloPoint2运行结果 %}

{% tabs %}
<!-- tab HelloPoint2.html -->
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
  <script src="./HelloPoint2.js"></script>
</head>
<body onload="main()">
  <canvas id="webgl" width="400" height="400"></canvas>
</body>
</html>
```
<!-- tab HelloPoint2.js -->
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
    void main(){
      gl_FragColor = vec4(0.0,1.0,0.0,1.0);
    }
  `;
function main() {
  // 获取canvas元素
  var canvas = document.getElementById('webgl');

  // 获取WebGL上下文
  var gl = getWebGLContext(canvas);
  if (!gl) {
    console.log('获取上下文失败');
    return;
  }

  // 初始化着色器
  if (!initShaders(gl, VSHADER_SOURCE, FSHADER_SOURCE)) {
    console.log('初始化着色器失败');
    return;
  }

  // 获取attribute变量的存储位置
  var a_Position = gl.getAttribLocation(gl.program, 'a_Position');
  if (a_Position < 0) {
    console.log('Failed to get the storage location of a_Position');
    return;
  }

  // 将顶点位置传输给attribute变量
  gl.vertexAttrib3f(a_Position, 0.0, 0.0, 0.0);

  // 设置canvas背景色
  gl.clearColor(0.0, 0.0, 0.0, 1.0);

  // 清除canvas
  gl.clear(gl.COLOR_BUFFER_BIT);

  // 绘制点位
  gl.drawArrays(gl.POINTS, 0, 1);
}

```

{% endtabs %}

{% endborder %}

{% border attribute使用步骤详解 color:blue %}

{% folding 步骤一：声明 color:yellow %}
attribute又称 {% mark 存储限定符 color:purple %} （storage qualifier）。  
attribute变量必须声明成**全局变量**，  
attribute变量声明格式如下：
> <存储限定符>  <类型> <变量名>

{% image ../../assets/image/WebGL/attribute变量声明格式.png attribute变量声明格式 %}

{% endfolding %}

{% folding 步骤二：获取 color:yellow %}
每个变量都具有一个**存储地址**，  
以便通过**存储地址**向变量**传输数据**。  
{% border getAttribLocation() color:purple %}
用于获取attribute变量的地址。  
其接收2个参数：  

{% border 参数1：程序对象（program object）%}
只要引用**gl.program**即可，  
包括了Vertex_Shader和Frag_Shader，  
由 **initShader\()** 函数创建，  
因此必须在initShader函数调用之后访问。
{% endborder %}
{% note **参数2：attribute变量的存储地址** %}

{% image ../../assets/image/WebGL/getAttribLocation.png  %}

{% endborder %}
{% endfolding %}

{% folding 步骤三：赋值 color:yellow %}
{% image ../../assets/image/WebGL/vertexAttrib3f.png %}
{% border %}
如果对于**vec4类型**的变量通过**vertexAttrib3f**传参，  
自然会省略掉1个参数，则默认为**1.0**。
{% endborder %}
{% endfolding %}
{% endborder %}

{% border gl.vertexAttrib3f()同族函数 color:blue %}
gl.vertextAttrib\[1-4]f有一系列函数，  
专门负责从JavaScript向Vertex_Shader中的attribute变量传参。  

{% image ../../assets/image/WebGL/vertexAttribnf.png %}

**gl.vertextAttrib\[1-4]f**也提供矢量版本**gl.vertextAttrib\[1-4]fv**：
- 名字以v结尾
- 接受类型化数组参数

```javascript
var position = new Float32Array([1.0, 2.0, 3.0, 1.0]);
gl.vertexAttrib4fv(a_Position, position);
```
{% endborder %}

{% border WebGL函数命名规范 color:yellow %}
与{% mark OpenGL ES 2.0 color:purple %}一致。  
由三个部分组成：
- 基础函数名
- 参数个数
- 参数类型

{% image ../../assets/image/WebGL/WebGL函数命名规范.png %}

{% endborder %}

