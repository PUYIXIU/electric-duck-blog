---
layout: wiki
wiki: WebGL
title: WebGL入门
order: 10
---
# Canvas是什么？

{% quot canvas标签定义了网页上的绘画区域 %}

{% border canvas标签的引用就像这样： color:cyan %}
```html
<canvas id="example" width="500" height="500">
对于不支持canvas标签的老式浏览器，会显示这条错误信息
</canvas>>
```
{% endborder %}

{% border 使用canvas绘制矩形 color:yellow %}

{% image ../../assets/image/WebGL/DrawRectangle.png DrawRectangle绘制结果 %}

{% border HTML color:purple %}
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body onload="main()">
<canvas id="example" width="400" height="400">
    Please use a browser that supports "canvas"
</canvas>
<script src="DrawRectangle.js"></script>
</body>
</html>
```
{% endborder %}
{% border JavaScript color:blue %}
```javascript
// DrawRecrangle.js
function main(){
    var canvas = document.getElementById('example');
    if(!canvas){
        console.log('Failed to retrieve the <canvas> element')
        return false;
    }
    // get the rendering context for 2DCG
    var ctx = canvas.getContext('2d'); 
    ctx.fillStyle = 'rgba(0,0,255,1.0)'; //set color
    ctx.fillRect(120,10,150,150); //fill a rectangele
}
```
{% endborder%}

{% border 步骤分解 color:green %}
{% folders %}
<!-- folder 步骤1 获取canvas元素 -->
```javascript
var canvas = document.getElementById('example');
if(!canvas){
    console.log('Failed to retrieve the <canvas> element');
    return false;
}
```
<!-- folder 步骤2 向该元素请求二维/三维图形的“绘图上下文”-->
```javascript
var ctx = canvas.getContext('2d');
```
{% border 上下文 context color:yellow %}
*getContext\()* 的参数用于指定上下文类型，  
有两种类型：  
- 二维
- 三维
{% endborder %}
<!-- folder 步骤3 在绘图上下文调用相应绘图函数-->
```javascript
ctx.fillStyle = 'rgba(0,0,255,1.0)';
ctx.fillRect(120,10,150,150);
```
{% border canvas的坐标系统 color:blue %}
- **x轴** 坐标系横轴，正方向朝→
- **y轴** 坐标系纵轴，正方向朝↓
- **原点** 落在左上方
{% image ../../assets/image/WebGL/canvas的坐标系统.png %}  
{% endborder %}

{% border fillRect的参数 color:cyan %}  
- *param1* 矩形左上顶点x坐标
- *param2* 矩形左上顶点y坐标
- *param3* 矩形宽度
- *param4* 矩形高度
{% endborder %}

{% endfolders %}
{% endborder %}

{% endborder%}


# 最短的WebGL程序：清空绘图区

{% border HelloCanvas code  child:tabs %}
{% image ../../assets/image/WebGL/HelloCanvas.png 清空绘图区效果 %}
{% tabs %}
<!-- tab HelloCanvas.html -->
```html
<body onload="main()">
  <canvas id="webgl" width="400" height="400">
    Come on! Why you still use your grammy's browser?
  </canvas>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="../examples/lib/webgl-utils.js"></script>
  <script src="../examples/lib/cuon-utils.js"></script>
  <script src="./HelloCanvas.js"></script>
</body>
```
<!-- tab HelloCanvas.js -->
```javascript
function main() {
  // 获取<canvas>元素
  var canvas = document.getElementById('webgl');
  
  // 获取WebGL绘图上下文
  var gl = getWebGLContext(canvas);
  if (!gl) {
    console.log("Congratulate! You meet a bug!")
    return;
  }

  // 指定清空<canvas>颜色
  gl.clearColor(0.0, 0.0, 0.0, 1.0);

  // 清空<canvas>
  gl.clear(gl.COLOR_BUFFER_BIT);
}
```
{% endtabs %}

{% border 步骤分解 color:purple %}

{% folders %}
<!-- folder 1. 获取canvas元素 -->
```javascript
let canvas = document.getElementById('webgl');
```
<!-- folder 2. 获取WebGL绘图上下文 -->
{% border  getContext 和 getWebGLContext color:yellow %}
{% split bg:card %}
<!--cell left-->
**getContext**  
接收的参数（'2d'/'3d''）在不同的浏览器中不同。
```javascript
let gl = canvas.getContext('2d');
```
<!--cell right-->
**getWebGLContext**  
在引入的*cuon-utils.js*中，  
**用于隐藏不同浏览器之间的差异。**
```javascript
let gl = getWebGLContext(canvas);
```
{% endsplit %}
{% image ../../assets/image/WebGL/getWebGLContext.png %}  
{% endborder %}

{% border 上下文命名技巧 color:yellow %}
之所以有人习惯将绘图上下文命名为gl，  
是因为 WebGL 是基于 OpenGL ES 的，  
这样可以将两者的函数名对应起来，比如：  
**gl.clearColor()** 与 **glClearColor()**
{% endborder %}

<!-- folder 3. 设置背景色 -->
```javascript
gl.clearColor(0.0, 0.0, 0.0, 1.0);
```
{% image ../../assets/image/WebGL/clearColor.png %}

{% note 颜色分量取值继承自OpenGL，因此范围在0.0-1.0，RGB颜色越高就越亮。 color:yellow %}

<!-- folder 4. 清空canvas -->
```javascript
gl.clear(gl.COLOR_BUFFER_BIT);
```
{% image ../../assets/image/WebGL/clear.png %}
{% border BUFFER color:yellow %}
由于clear继承自OpenGL，基于**多基本缓冲区模型**，比二维绘图上下文要复杂。  
清空绘图区域，实际上是清空**颜色缓冲区（color buffer）**。  

也可以通过下面方法清空对应缓冲区： 
{% image ../../assets/image/WebGL/清空缓冲区相关函数.png  %}
{% endborder %}

{% endfolders %}

{% endborder %}

{% endborder %}
