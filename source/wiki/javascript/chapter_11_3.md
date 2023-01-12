---
layout: wiki
wiki: JavaScript
title: 11.3 异步函数
order: 112
---

{% border ES8之前的异步调用 color:blue %}
任何要访问期约产生值的代码，  
都需要以处理程序的形式来接受此值：
```javascript
function handler(x) { console.log(x); }
let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3));
p.then(handler); //3
```
这种书写方式相当笨重，  
ES8新增了 **async/await** 关键字，  
让以同步方式写的代码能够异步执行。

{% endborder %}

# 11.3.1 异步函数

{% border child:tabs %}
{% tabs %}
<!-- tab async  -->
{% border async用于声明异步函数 color:yellow %}
{% folding async语法 color:blue %}
适用范围：
- 函数声明
- 函数表达式
- 箭头函数
- 方法

```javascript
async function foo() { } //函数声明
let bar = async function () { }; //函数表达式
let baz = async () => { }; //箭头函数
class Qux { 
  async qux() { } //方法
}
```

{% endfolding %}
{% folding async功能 color:blue %}
让函数具有异步特征，但总体上代码仍然是同步求值。
```javascript
async function foo() {
  console.log(1);
}
foo();
console.log(2); //1 2
```
{% endfolding %}
{% folding async返回值 color:blue %}
始终返回 **期约对象**。  
返回值会被 **Promise.resolve()** 包装为一个期约对象。
```javascript
async function foo() { 
  console.log(1);
  return 3; // 相当于 return Promise.resolve(3)
}
foo().then(console.log);
console.log(2); // 1 2 3
```

{% endfolding %}
{% folding async返回值的静默处理 color:blue %}
async函数的返回值实际期待的是：  
**一个实现thenable接口的对象**   
对于是否实现了这一条件，async会对返回值采取不同的静默处理方式：  
- **实现thenable接口** 在调用返回值的then方法时，可以提供onResolve/onRejected处理程序以供解包
- **未实现thenable接口** 返回值被当做已经已解决的期约

{% border child:codeblock %}
{% kbd 返回一个原始值 %}
```javascript
async function foo() {
    return 'foo';
}
foo().then(console.log); // foo
```
{% endborder %}

{% border child:codeblock %}
{% kbd 返回一个没有实现thenable接口的对象 %}
```javascript
async function bar() { 
  return ['bar'];
}
bar().then(console.log); //['bar']
```
{% endborder %}

{% border child:codeblock %}
{% kbd 返回一个实现了thenable接口的对象 %}
```javascript
async function baz() { 
  const thenable = {
    then(callback) { callback('baz'); }
  };
  return thenable;
}
baz().then(console.log); //baz
```
{% endborder %}

{% border child:codeblock %}
{% kbd 返回一个期约 %}
```javascript
async function qux() { 
  return Promise.resolve('qux');
}
qux().then(console.log); //qux
```
{% endborder %}

{% endfolding %}
{% folding 异步函数中抛出错误 color:blue %}
会返回拒绝期约
```javascript
async function foo() {
  console.log(1);
  throw 3;
}
foo().catch(console.log);
console.log(2); //1 2 3
```
{% endfolding %}
{% folding 拒绝期约的错误 color:blue %}
不会被异步函数捕获
```javascript
async function foo() {
  console.log(1);
  Promise.reject(3);
}
foo().catch(console.log);
console.log(2);
//1 2 
//Uncaught (in promise) 3
```
{% endfolding %}

{% endborder %}
<!-- tab await -->
{% border  await可以暂停异步函数代码的执行，等待期约解决 color:yellow %}

{% folding await功能 color:blue %}
await关键字会 **暂停执行异步函数后面的代码** 。  
{% timeline %}
<!-- node 第一步 -->
尝试解包对象的值。
<!-- node 第二步 -->
将值传给表达式。
<!-- node 第三步 -->
异步恢复异步函数的执行。
{% endtimeline %}

```javascript
async function foo() { 
  let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3));
  console.log(await p);
}
foo(); //3
```

{% endfolding %}
{% folding await用法 color:blue %}
用法相当于 **一元操作符**：
{% border child:codeblock %}
{% kbd 异步打印”foo“ %}
```javascript
async function foo() { 
  console.log(await Promise.resolve('foo'));
}
foo(); //foo
```
{% endborder %}
{% border child:codeblock %}
{% kbd 异步打印”bar“ %}
```javascript
async function bar() { 
  return await Promise.resolve('bar');
}
bar().then(console.log); //bar
```
{% endborder %}
{% border child:codeblock %}
{% kbd 1000ms后异步打印”baz“ %}
```javascript
async function baz() {
  await new Promise((resolve, reject) => setTimeout(resolve, 1000));
  console.log('baz');
}
baz(); //baz
```
{% endborder %}
{% endfolding %}
{% folding await的静默处理 color:blue %}
await关键字实际期待的是：  
**一个实现了thenable接口的对象**  
对于是否实现thenable接口，await对其也有不同的静默处理方式：
- **实现了thenable接口：** await会对其进行解包 
- **未实现thenable接口：** 默认包装为已解决的期约

{% border child:codeblock %}
{% kbd await一个原始值 %}
```javascript
async function foo() {
  console.log(await 'foo');
}
foo(); //foo
```
{% endborder %}
{% border child:codeblock %}
{% kbd await一个没有实现thenable接口的对象 %}
```javascript
async function bar() {
  console.log(await ['bar']);
}
bar(); //['bar']
```
{% endborder %}
{% border child:codeblock %}
{% kbd await一个实现了thenable接口的对象 %}
```javascript
async function baz() {
  const thenable = {
    then(callback) { callback('baz'); }
  };
  console.log(await thenable);
}
baz(); //baz
```
{% endborder %}
{% border child:codeblock %}
{% kbd await一个期约 %}
```javascript
async function qux() {
  console.log(await Promise.resolve('qux'));
}
qux(); //qux
```
{% endborder %}
{% endfolding %}

{% folding await抛出错误的同步操作 color:blue %}
会返回拒绝的期约。
```javascript
async function foo() {
  console.log(1);
  await (() => { throw 3; }) ();
}
foo().catch(console.log);
console.log(2); // 1 2 3
```
{% endfolding %}
{% folding await拒绝的期约 color:blue %}
会 **释放（unwrap）错误值。**  
即直接将拒绝期约作为返回值return。  
```javascript
async function foo() {
  console.log(1);
  await Promise.reject(3);
  console.log(4); // 本行不会执行
}
foo().catch(console.log);
console.log(2); //1 2 3
```
{% endfolding %}
{% folding await的限制 color:blue %}
await关键字只能 **直接出现在异步函数的定义中** ，   
它的异步特质不会扩展到嵌套函数中。  
不能在顶级上下文中使用，但是可以定义并立即调用：
{% border child:codeblock  %}
{% kbd 定义并立即调用的异步函数 %}
```javascript
(async function () {
  console.log(await Promise.resolve(3));
})(); //3
```
{% endborder %}
{% border child:codeblock  %}
{% kbd 不允许：await出现在箭头函数中 %}
```javascript
function foo() {
  const syncFn = () => {
    return await Promise.resolve('foo');
  };
  console.log(syncFn());
}
```
{% endborder %}
{% border child:codeblock  %}
{% kbd 不允许：await出现在同步函数声明中 %}
```javascript
function bar() {
  function syncFn() {
    return await Promise.resolve('bar');
  }
  console.log(syncFn());
}
```
{% endborder %}
{% border child:codeblock  %}
{% kbd 不允许：await出现在同步函数表达式中 %}
```javascript
function baz() {
  const syncFn = function () {
    return await Promise.resolve('baz');
  }
  console.log(syncFn());
}
```
{% endborder %}
{% border child:codeblock  %}
{% kbd 不允许：IIFE使用同步函数表达式或箭头函数 %}
```javascript
function qux() {
  (function () {
    console.log(await Promise.resolve('qux'));
  })();
  (() => console.log(await Promise.resolve('qux')))();
}
```
{% endborder %}
{% endfolding %}

{% endborder %}

{% endtabs %}
{% endborder %}

# 11.3.2 停止和恢复执行

{% border await关键实际作用 color:yellow %}
异步函数中真正起作用的实际是 **await**，  
async更像是一个 **标识符**，  
异步函数如果不包含await关键字，执行起来实际和普通函数没有区别：
```javascript
async function foo() {
  console.log(await Promise.resolve('foo'));
}
async function bar() {
  console.log(await 'bar');
}
async function baz() {
  console.log('baz');
}
foo(); 
bar();
baz();
// baz bar foo
```
{% endborder %}

{% border await关键字作用原理 color:blue %}

{% border JavaScript运行时遇到await会进行如下操作 %}
{% timeline %}
<!-- node 第一步 -->
记录暂停执行的位置。
<!-- node 第二步 -->
等到对象值可用时，会向消息队列推送一个任务。
<!-- node 第三步 -->
该任务恢复异步函数的执行（回到之前记录的暂停位置）
{% endtimeline %}
{% endborder %}

{% tabs %}
<!-- tab await对象是一个立即可用的值 -->
{% split %}
<!-- cell left -->
```javascript
async function foo() {
  console.log(2);
  await null;
  console.log(3)
}
console.log(1);
foo();
console.log(4); 
// 1 2 3 4
```
<!-- cell right -->
{% timeline %}
<!-- node step 1 --> 打印1
<!-- node step 2 --> 进入foo()，打印2
<!-- node step 3 --> 遇到await关键字，记录暂停位置，向消息队列中添加一个任务
<!-- node step 4 --> 暂时退出foo()，打印3
<!-- node step 5 --> 同步线程代码执行完毕，从消息队列中取出任务，返回之前记录位置恢复异步函数执行
<!-- node stup 6 --> 打印4，退出foo()
{% endtimeline %}
{% endsplit %}

<!-- tab await对象是一个Promise的对象-->
{% split %}
<!-- cell left -->
```javascript
async function foo() { 
  console.log(2);
  console.log(await Promise.resolve(8));
  console.log(9);
}
async function bar() { 
  console.log(4)
  console.log(await 6);
  console.log(7);
}
// 1 2 3 4 5 8 9 6 7
console.log(1);
foo();
console.log(3);
bar();
console.log(5);
```
<!-- cell right -->
{% timeline %}
<!-- node step 1 --> 打印1
<!-- node step 2 --> 进入foo，打印2
<!-- node step 3 --> 遇到await关键字暂停执行，记录暂定位置，向消息队列中添加一个期约在落定之后执行的任务
<!-- node step 4 --> 期约立即落定，把给await提供值的任务添加到消息队列，暂时从foo中退出
<!-- node step 5 --> 打印3
<!-- node step 6 --> 进入bar，打印4
<!-- node step 7 --> 遇到await关键字暂停执行，向消息队列添加一个任务，暂时退出bar
<!-- node step 8 --> 打印5，线程执行完毕
<!-- node step 9 --> 退回到foo暂停位置，打印8，打印9，退出foo()
<!-- node step 10 --> 退回到bar暂停位置，打印6，打印7，退出bar()
{% endtimeline %}

{% endsplit %}
{% endtabs %}
{% endborder %}

# 11.3.3 异步函数策略

{% folders %}
<!-- folder 1. 实现sleep() -->
{% border color:yellow %}
可以实现类似Java中的 **Tread.sleep()** 函数，  
在程序中加入非阻塞的暂停：
```javascript
async function sleep(delay) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}
async function foo() { 
  const t0 = Date.now();
  await sleep(1500);
  console.log(Date.now() - t0);
}
foo(); //1507
```
{% endborder %}

<!-- folder 2. 利用平行执行 -->
{% border color:yellow child:tabs %}

{% tabs %}
<!-- tab 顺序等待多个异步操作 -->
```javascript
async function randomDelay(id) {
  const delay = Math.random() * 1000;
  return new Promise((resolve) => setTimeout(() => {
    console.log(id);
    resolve();
  },delay))
}

async function foo() {
  const t0 = Date.now();
  await randomDelay(0);
  await randomDelay(1);
  await randomDelay(2);
  await randomDelay(3);
  await randomDelay(4);
  console.log(Date.now() - t0);
}
foo(); //0 1 2 3 4 1994
```
<!-- tab 平行执行多个异步操作 -->
```javascript
async function randomDelay(id) {
  const delay = Math.random() * 1000;
  return new Promise((resolve) => setTimeout(() => {
    console.log(id);
    resolve();
  },delay))
}

async function foo() {
  const t0 = Date.now();
  // 一次性初始化所有期约，再分别等待结果
  const p0 = randomDelay(0);
  const p1 = randomDelay(1);
  const p2 = randomDelay(2);
  const p3 = randomDelay(3);
  const p4 = randomDelay(4);
  await p0;
  await p1;
  await p2;
  await p3;
  await p4;
  console.log(Date.now() - t0);
} 
foo(); // 3 1 4 2 0 977
```
{% endtabs %}

{% endborder %}
<!-- folder 3. 串行执行期约 -->
{% border color:yellow %}
```javascript
async function addTwo(x) { return x + 2; }
async function addThree(x) { return x + 3; }
async function addFive(x) { return x + 5; }
async function addTen(x) {
  for (const fn of [addTwo, addThree, addFive]) {
    x = await fn(x);
  }
  return x;
}
addTen(9).then(console.log); //19
```
{% endborder %}
<!-- folder 4. 栈追踪与内存管理 -->
{% border color:yellow %}
比较 **期约** 与 **内存管理** 在拒绝期约时的栈追踪信息：
{% split %}


<!-- cell left -->
{% border child:codeblock color:green %}
{% kbd 期约 %}
```javascript
function fooPromiseExecutor(resolve, reject) {
  setTimeout(reject, 1000, 'bar');
}
function foo() {
  new Promise(fooPromiseExecutor);
}
foo();
```
{% endborder %}
{% border child:codeblock color:cyan %}
{% kbd 栈追踪信息 %}
> Uncaught (in promise) bar
> setTimeout (async)
> fooPromiseExecutor
> foo

{% endborder %}


{% border color:blue %}
JavaScript引擎会在创建期约时 **尽可能保留完整的调用栈** ，    
这意味着栈追踪信息会占用内存，带来一些计算和存储成本。  
{% endborder %}

<!-- cell right -->
{% border child:codeblock color:green %}
{% kbd 异步函数 %}
```javascript
function fooPromiseExecutor(resolve, reject) {
    setTimeout(reject, 1000, 'bar');
}
async function foo() {
    await new Promise(fooPromiseExecutor);
}
foo();
```
{% endborder %}
{% border child:codeblock color:cyan %}
{% kbd 栈追踪信息 %}
> Uncaught (in promise) bar
> foo
> await in foo (async)

{% endborder %}
{% border color:blue %}
**栈追踪信息可以准确地反应当前的调用栈。**  
JavaScript在嵌套函数中存储指向包含函数的指针，  
指针存储在内存中，可以用于在出错时生成追踪信息。  
因此能够降低内存的额外消耗。
{% endborder %}

{% endsplit %}

{% note 栈追踪信息应该相当直接地表现JavaScript引擎当前栈内存中函数调用之间的嵌套关系。 color:blue %}



{% endborder %}
{% endfolders %}
