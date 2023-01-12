---
layout: wiki
wiki: JavaScript
title: 10.4 函数调用技巧
order: 103
---

# 10.4.1 递归

{% note 递归函数通常是指一个函数通过名称调用自己 %}

但如果不将函数逻辑和函数名称解耦合，就可能会出现问题：
```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else { 
    return num * factorial(num - 1);
  }
}

let anotherFactorial = factorial;
factorial = null;
//TypeError: factorial is not a function
console.log(anotherFactorial(10));  
```
{% border 使用arguments.callee可以解决硬编码问题 color:cyan %}
```javascript
function factorial(num) {
  if (num <= 1) {
    return num;
  } else { 
    return num * arguments.callee(num - 1);
  }
}
```
{% endborder %}

{% border 严格模式下解耦递归函数 color:blue %}
严格模式下不能访问arguments.callee,  
可以使用**命名函数表达式（named function expression）**。
```javascript
const factorial = (function f(num) {
  if (num <= 1) {
    return 1;
  } else { 
    return num * f(num - 1);
  }
})
```
{% endborder %}

# 10.4.2 尾调用优化

{% border 尾调用 color:purple %}
*即外部函数返回值是一个内部函数的返回值*
```javascript
function outerFunction() {
  return innerFunction();
}
```
{% note ES6新增的一项内存管理优化机制：JS引擎在满足条件时可以重用栈帧，使尾调用更具优势。 color:green %}
{% endborder %}

{% border 比较ES6优化前后尾调用的执行步骤 color:green child:tabs %}

{% tabs %}

<!-- tab 优化前 -->

{% border color:cyan %}
1. outerFunction执行，推上①号栈帧
2. 执行到return，开始进入innerFunction
3. innerFunction执行，推上②号栈帧
4. 执行到return，计算返回值result
5. result传回outerFunction
6. outerFunction返回result
7. 弹出①②号栈帧
{% endborder %}

{% note 结论：每多调用一次嵌套函数，就会多增加一个栈帧。 color:blue %}

<!-- tab 优化后 -->
{% border color:cyan %}
1. outerFunction执行，推上①号栈帧
2. 执行到return，开始进入innerFunction
3. JS引擎检测innerFunction和outerFunction的返回值是否一致（结果一致）
4. 弹出①号栈帧
5. 执行到innerFunction，推上②号栈帧
6. 执行到return，计算返回值result
7. 弹出②号栈帧
{% endborder %}
{% note 结论：无论调用多少次嵌套函数，都只有一个栈帧 color:blue %}

{% endtabs %}

{% endborder %}

{% border 尾调用优化的条件：确认外部栈帧真的没有存在必要了 color:blue %}
涉及条件如下：
{% folders  %}
<!-- folder "use strict" -->
{% note 原因：非严格模式下函数存在arguments属性和caller方法，都会引用外部函数的栈帧。 color:green %}
<!-- folder 外部函数返回值是对尾调用函数的调用 -->
{% split bg:card %}
<!-- cell left -->
**违反优化条件**
```javascript
"use strict";

// 尾调用没有返回
function outerFunction(){
    innerFunction();
}
```
<!-- cell right -->
**符合优化条件**
```javascript
"use strict";
function outerFunction(a,b){
    return innerFunction(a+b);
}
```

{% endsplit %}
<!-- folder 尾调用函数返回后不需要执行额外的逻辑 -->

{% split bg:card %}
<!-- cell left -->
**违反优化条件**
```javascript
"use strict";
// 尾调用没有直接返回
function outerFunction(){
    let innerFunctionRsult = innerFunction();
    return innerFunctionResult;
}
// 尾调用返回后必须转型为字符串
function outerFunction(){
    return innerFunction().toString();
}
```
<!-- cell right -->
**符合优化条件**
```javascript
"use strict";
function outerFunction(a,b){
    if(a<b){
        return a;
    }
    return innerFunction(a+b);
}
```
{% endsplit %}

<!-- folder 尾调用函数不是引用外部函数作用域中自由变量的闭包 -->

{% split bg:card %}
<!-- cell left -->
**违反优化条件**
```javascript
"use strict";
// 尾调用是一个闭包
function outerFunction(){
    let foo = 'bar';
    function innerFunction(){return foo;}
    return innerFunction();
}
```
<!-- cell right -->
**符合优化条件**
```javascript
function outerFunction(condition){
    return condition?innerFunctionA():innerFunctionB();
}
```
{% endsplit %}

{% endfolders %}

{% note Tips：尾调用优化在递归场景下效果最好。 color:yellow %}

{% endborder %}

{% border 尾调用优化的代码 color:yellow %}

下面是一段待优化的代码：
```javascript
function fib(num) {
  if (num < 2) {
    return num;
  }
  // 存在相加逻辑操作，不符合优化条件
  return fib(num - 1) + fib(num - 2);
}

console.log(fib(1)); //1
console.log(fib(5)); //5
let startTime = new Date();
console.log(fib(20)); //6765
console.log(new Date() - startTime); //1
```
可以改写成迭代循环形式
```javascript
function fib(n) {
  return fibImpl(0, 1, n);
}
function fibImpl(a, b, n) {
  if (n === 0) { return a; }
  return fibImpl(b, a + b, n - 1);
}
console.log(fib(1)); //1
console.log(fib(5)); //5
let startTime = new Date();
console.log(fib(100)); //354224848179262000000
console.log(new Date() - startTime); //0
```

{% endborder %}

# 10.4.3 闭包（closure）

{% note 闭包指的是引用了另一个函数作用域中变量的函数。 color:blue %}

{% border 作用域链 color:blue %}
在调用一个函数时，会进行这2步操作：
- 创建一个执行上下文
- 创建一个作用域链

```javascript
function outerFunction(){
    function innerFunction(){}
}
```
**innerFunction作用域链：**  
innerFunction - outerFunction - \[ more ]  
- outerFunction的活动对象是innerFunction作用域链上的第二个对象
- 作用域链一直向外串起所有包含函数的活动对象
- 作用域链终止于全局执行上下文
{% endborder %}
  
{% border 变量对象和活动对象 color:yellow %}
每个执行上下文中都会有一个包含其中变量的对象。  
- 在全局上下文中就叫做**变量对象**
- 在函数局部上下文中就叫做**活动对象**，只在函数执行期间存在
{% endborder %}

{% border 普通函数的作用域链 color:green %}
```javascript
function compare(value1, value2){
    if(value1<value2){
        return -1;
    }else if(value1>value2){
        return 1;
    }else{
        return 0;
    }
}
let result = compare(5,10);
```
{% split bg:card %}
<!--cell left-->
**定义compare函数经历步骤**
1. 创建作用域链
2. 预装载全局变量对象
3. 全局变量对象保存在内部的\[\[Scope]]中
<!--cell right-->
**调用compare函数经历步骤**
1. 创建相应执行上下文
2. 复制函数的\[\[Scope]]创建作用域链
3. 创建函数的活动对象
4. 将函数活动对象推入作用域链首
5. 函数执行完毕，销毁局部活动对象
{% endsplit %}

{% image ../../assets/image/JavaScript/普通函数作用域链.png 普通函数作用域链指针示意图 %}   

{% endborder %}

{% border 匿名函数的作用域链 color:yellow %}
```javascript
function createComparisionFunction(propertyName){
    return function(object1,object2){
        let value1 = object1[propertyName];
        let value2 = object2[propertyName];
        if(value1<value2){
            return -1;
        }else if(value1>value2){
            return 1;
        }else{
            return 0;
        }
    }
}
let compare = createComparisionFunction('name');
let result = compare({name:'Nicolas'},{name:'Matt'});
```
{% border 匿名函数与普通函数的不同在于： color:blue %}
外部函数（createComparisonFunction）执行完毕后，  
执行上下文的作用域链会销毁，  
但是活动对象仍然会保留在内存中，  
直到匿名函数被销毁后才能完全被销毁。  
**这是因为匿名函数的作用域链中仍然有对它的引用。**
{% endborder %}

{% image ../../assets/image/JavaScript/匿名闭包函数作用域链.png 作为返回值的匿名闭包函数作用域链指针示意图 %}

{% border 解除对函数引用对策：手动释放内存 color:green %}
```javascript
let compareNames = createComparisonFunction('name');
let result = compareNames({name:'Nicholas'},{name:'Rachel'});
// 解除占用
compareNames = null;
```
{% endborder %}

{% note 因为闭包函数会保留其外部函数的活动对象，因此比其他函数更占用内存，需谨慎使用。 color:red %}

{% endborder %}

{% border 闭包中的this color:blue%}
闭包中使用this会让代码变得复杂。
```javascript
window.identity = 'The Window';
let object = {
  identity: 'My Object',
  getIdentityFunc() {
    return function () {
      return this.identity;
    }
  }
}
console.log(object.getIdentityFunc()()); //The Window
console.log(object.getIdentityFunc().apply(object)); //My Object
```
{% border 保留this color:green %}
可以保证闭包函数的this与其外部函数的this一致
```javascript
window.identity = 'The Window';
let object = {
  identity: 'My Object',
  getIdentityFunc() {
    let that = this;
    return function () {
      return that.identity;
    }
  }
}
console.log(object.getIdentityFunc()()); //My Object
```
{% endborder %}

{% border 特殊情况分析 color:green %}
```javascript
window.identity = 'The Window';
let object = {
  identity: 'My Object',
  getIdentity() {
    console.log(this.identity);
  }
}
object.getIdentity(); //My Object
(object.getIdentity)(); //My Object
(object.getIdentity = object.getIdentity)(); //The Window
```
{% endborder %}
{% folders %}
<!-- folder 第一行：obejct.getIdentity() -->
普通函数中this指向调用函数的上下文，  
因为这里调用函数的上下文是object，  
因此这里的this就是object
<!-- folder 第二行：(object.getIdentity)() -->
按照规范，object.getIdenitty = (object.getIdentity)，  
因此this还是object
<!-- folder 第三行：(object.getIdentity =  object.getIdentity)() -->
因为赋值表达式右侧的值是函数本身，  
并非函数引用，  
因此这里this的值不再与任何对象绑定，  
所以this指向的是window
{% endfolders %}

{% endborder %}

{% border 内存泄漏 color:green %}
```javascript
function assignHandler() {
  let element = document.getElementById('someElement');
  element.onclick = () => console.log(element.id);
}
```
上面的代码中匿名函数的存在导致element始终被引用，其内存不会被回收。  
因此可以进行手动回收。  
```javascript
function assignHandler() {
  let element = document.getElementById('someElement');
  let id = element.id;
  element.onclick = () => console.log(id);
  element = null;
}
```
{% endborder %}

# 10.4.4 立即调用的函数表达式

立即调用的函数表达式（IIFE，Immediately Invoked Function Expression）
```javascript
(function(){
    // 块级作用域
})();
```
{% border IIFE主要用于模拟块级作用域 color:blue %}

{% note 主要针对于ECMAScript5.1之前不支持块级作用域的情况。 color:yellow %}

```javascript
let count = 5;
(function () {
    for (let i = 0; i < count; i++) {
        console.log(i);
    }
})(); // 0 1 2 3 4
```
因为不存在对这个匿名函数的引用，  
因此只要函数执行完毕其作用域链就会被销毁。  
{% endborder %}

{% border ECMAScript6以后出现了块级作用域： color:green %}
```javascript
{ 
  let i;
  for (i = 0; i < couhnt; i++){
    console.log(i);
  }
}
console.log(i); //Error
```
{% endborder %}

{% border IIFE还可以用来锁定参数值 color:blue %}
```javascript
let divs = document.querySelectorAll('div');
for (var i = 0; i < divs.length; ++i){
  divs[i].addEventListener('click', function () {
    console.log(i);
  })
}
```
以上这段代码会导致每次触发click都打印同样的值，  
可以通过下面方式锁定索引值：
```javascript
let divs = document.querySelectorAll('div');
for (var i = 0; i < divs.length; ++i){
  divs[i].addEventListener('click', (function (frozenCounter) {
    return function () {
      console.log(frozenCounter);
    }
  })(i));
}
```
使用 ECMAScript6 块级作用域就简单那多了：
```javascript
let divs = document.querySelectorAll('div');
for (let i = 0; i < divs.length; ++i){
  divs[i].addEventListener('click', function () {
    console.log(i);
  })
}
```
当然要保证for循环使用块级作用域变量关键字：
```javascript
let divs = document.querySelectorAll('div');
let i;
for (i = 0; i < divs.length; ++i){
  divs[i].addEventListener('click', function () {
    console.log(i); //same output
  })
}
```
{% endborder %}

# 10.4.5 私有变量

{% note JavaScript任何定义在函数或块中的变量，都可以认为是私有的。 color:blue %}

{% border 闭包与私有变量 color:yellow %}
如果在函数中创建一个能通过作用域链访问外部函数变量的**闭包**，  
就能够创建出访问私有变量的公有方法。
{% endborder %}

{% border 特权方法（privileged method） color:purple %}
**特权方法**指的是*能够访问函数私有变量/方法的公有方法*。
{% border 方法一：在构造函数中实现 color:yellow %}
即在构造函数中定义私有变量和私有方法，  
再创建一个能够访问这些私有成员的特权方法：
```javascript
function Person(name) {
  this.getName = function () {
    return name;
  };
  this.setName = function (value) {
    name = value;
  }
}
let person = new Person('Nicholas');
console.log(person.getName()); //Nicholas
person.setName('Greg');
console.log(person.getName()); //Greg
```
{% endborder %}

{% border 方法二：使用私有作用域定义私有变量和函数 color:yellow %}
```javascript
(function () {
  let privateVariable = 10;
  function privateFunction() {
    return false;
  }
  MyObject = function () { };
  MyObject.prototype.publicMethod = function () {
    privateVariable++;
    return privateFunction();
  }
})();
```
这种方式存在的问题是：实例共享私有变量和私有函数：
```javascript
(function () {
  let name = '';
  Person = function (value) {
    name = value;
  };
  Person.prototype.getName = function () {
    return name;
  }
  Person.prototype.setName = function (value) {
    name = value;
  }
})();
let person1 = new Person('Nicholas');
console.log(person1.getName()); //Nicholas
person1.setName('Matt');
console.log(person1.getName()); //Matt

let person2 = new Person('Michael');
console.log(person2.getName()); //Michael
console.log(person1.getName()); //Michael
```

{% endborder %}

{% endborder %}

{% border 模块模式 color:yellow %}
{% note 模块模式是在单例对象基础上的扩展。 color:blue %}
```javascript
let singleton = function () {
  let proivateVariable = 10;
  function privateFunction() {
    return false;
  }
  return {
    publicProperty: true,
    publicMethod() {
      privateVariable++;
      return privateFunction();
    }
  }
}();
```
如果单例对象需要进行某种初始化，  
并且需要访问私有变量时，  
可以采用这个模式：  
```javascript
let application = function () {
  let components = new Array();
  components.push(new BaseComponent());
  return {
    getComponentCount() {
      return components.length;
    },
    registerComponent(component) {
      if (typeof component == 'object') {
        components.push(component);
      }
    }
  }
}();
```
{% border 单例模式与组件管理 color:blue %}
Web开发中，经常需要单例对象管理应用程序级的信息，    
比如在页面组件的管理上。
{% endborder %}
{% endborder %}
{% border 模块增强模式 color:blue %}
增强模式即在返回对象之前对其进行增强，  
这适合单例对象需要时某个特定类型的实例，  
但又必须给它添加额外属性或方法的场景。  
```javascript
let singleton = function () {
  let privateVariable = 10;
  function privateFunction() {
    return false;
  }
  let object = new CustomType();
  object.publicProperty = true;
  object.publicMethod = function () {
    privateVariable++;
    return privateFunction();
  };
  return object;
}
```
重写单例模式中的application对象：
```javascript
let application = function () {
  let components = new Array();
  components.push(new BaseComponent());
  let app = new BaseComponent();
  app.getComponentCount = function () {
    return components.length;
  };
  app.getComponentCount = function (component) {
    if (typeof component == 'object') {
      components.push(component);
    }
  };
  return app;
}
```
{% endborder %}
