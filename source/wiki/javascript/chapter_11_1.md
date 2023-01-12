---
layout: wiki
wiki: JavaScript
title: 11.1 异步编程
order: 110
---

{% border 同步与异步 color:green %}
在JavaScript这种单线程事件循环模型中，  
**同步操作**与**异步操作**是代码要依赖的核心机制。  
异步行为是为了**优化因计算量大而花费时间长的操作**。
{% endborder %}

# 11.1.1 同步与异步

{% split bg:card %}
<!-- cell left -->
{% border 同步行为 color:yellow %}
对应内存中顺序执行的处理器指令。  
**每条指令严格按照出现顺序执行**，  
执行后立即获得存储在系统本地的信息。
```javascript
let x = 3;
x = x + 4;
```
{% endborder %}
<!-- cell right -->
{% border 异步行为 color:purple %}
异步行为类似于**系统中断**。  
即**当前进程外部的实体可以触发代码执行**，  
常用于访问一些高延迟的资源。  
```javascript
let x = 3;
setTimeout(() => x = x + 4, 1000);
```
{% endborder %}
{% endsplit %}

{% border 预知异步执行的变化  %}
异步执行由**系统计时器**触发，  
会生成一个入队执行的中断，中断何时触发则是无法预知的，
*但可以保证中断发生在当前线程的同步代码执行以后*。
{% endborder %}

# 11.1.2 以往的异步编程模式

{% border 回调地狱 color:red %}
早期JS中只支持**回调函数**来表明*异步操作完成*，  
因此常常会串联多个异步操作，俗称**回调地狱（深度嵌套的回调函数）**。
```javascript
function double(value) {
  setTimeout(() => setTimeout(console.log, 0, value * 2), 1000);
}
double(3); //6
```
上面的例子中，double函数在setTimeout成功调度异步操作之后就会立即退出。
{% endborder %}

{% folding 1. 异步返回值 color:yellow %}
常用的策略是：  
给异步操作提供一个包含要使用异步返回值代码的回调函数。  
```javascript
function double(value, callback) {
  setTimeout(() => callback(value * 2), 1000);
}
// I was given 6
double(3, (x) => console.log(`I was given: ${x}`));
```
{% endfolding %}

{% folding 2. 失败处理 color:green %}
考虑到异步操作也可能存在失败处理，  
因此就分化出了**成功回调**和**失败回调**。  
这种模式必须在初始化异步操作时定义回调。  
```javascript
function double(value, success, failure) {
  setTimeout(() => {
    try {
      if (typeof value !== 'number') {
        throw 'Must provide number as first argument';
      }
      success(2 * value);
    } catch (e) {
      failure(e);
    }
  }, 1000);
}
const successCallback = (x) => console.log(`Success: ${x}`);
const failureCallback = (e) => console.log(`Failure: ${e}`);
// Success: 6
double(3, successCallback, failureCallback);
// Failure: Must provide number as first argument
double('b', successCallback, failureCallback);
```
{% endfolding %}

{% folding 3. 嵌套异步回调 color:yellow %}
在异步返回值需要依赖另一个异步返回值时，    
回调情况还会进一步变复杂：  
```javascript
function double(value, success, failure) {
  setTimeout(() => {
    try { 
      if (typeof value !== 'number') {
        throw 'Give me a number.'
      }
      success(value * 2);
    } catch (e) {
      failure(e);
    }
  }, 1000);
}
const successCallback = (x) => {
  double(x, (y) => console.log(`Success:${y}`));
}
const failureCallback = (e) => console.log(`Failure: ${e}`);
double(3, successCallback, failureCallback); //Syccess:12
```
{% note 回调策略不具有扩展性，“回调地狱”实至名归。 color:red %}
{% endfolding %}
