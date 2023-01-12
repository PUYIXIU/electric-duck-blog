---
layout: wiki
wiki: JavaScript
title: 10.3 函数特性
order: 102
---

# 10.3.1 函数声明与函数表达式

{% note 函数声明和函数表达式的主要区别就在于是否会进行函数声明提升（function declaration hoisting） %}

{% border 函数声明会进行函数声明提升 color:blue %}
函数声明提升 即 函数声明会在任何代码执行之前先被读取并添加到执行上下文。
```javascript
console.log(sum(1, 2)); //3
function sum(num1, num2) {
  return num1 + num2;
}
```
{% endborder %}

{% border 函数表达式 必须要等到代码执行到那一行才会执行函数定义 color:cyan %}
这和let还是var声明无关。
```javascript
//ReferenceError: Cannot access 'num' before initialization 
console.log(sum(10, 20)); 
let sum = function (num1, num2) {
  return num1 + num2;
}
```
{% endborder %}
# 10.3.2 函数作为值

因为函数名本身就是**变量**，因此函数可以作为**参数**和**返回值**。  
```javascript
function callSomeFunction(someFunction, someArgument) {
    return someFunction(someArgument);
}

function add10(num) {
    return num + 10;
}
console.log(callSomeFunction(add10, 10)); //20

function getGreeting(name) {
    return 'Hello, ' + name;
}
console.log(callSomeFunction(getGreeting, 'Nicholas')); //Hello, Nicholas

```

{% border 从一个函数返回另一个函数可以非常有用 color:yellow %}
下面的程序中创建了一个可以按照对象数组中任意对象属性对数组进行排序的函数
```javascript
function createComparisonFunction(propertyName) {
  return function (object1, object2) {
    let value2 = object2[propertyName];
    let value1 = object1[propertyName];
    if (value1 < value2) {
      return -1;
    } else if (value1 > value2) {
      return 1;
    } else {
      return 0;
    }
  };
}
let data = [
  { name: 'Zachary', age: 28 },
  { name: 'Nicholas', age: 29 }
];
data.sort(createComparisonFunction('name'));
console.log(data[0].name); //Nicholas
```

{% endborder %}

# 10.3.3 函数内部
{% border 函数内部存在三个特殊对象 %}
- *arguments*
- *this*
- *new.target*
{% endborder %}
  
{% border arguments color:yellow %}
*arguments.callee* 是一个指向arguments对象所在函数的指针。  
{% border 经典阶乘函数 color:cyan %}
```javascript
function factorial(num) {
  if (num <= 1) {
    return 1;
  } else { 
    return num * factorial(num - 1);
  }
}
console.log(factorial(10)); //3628800
```
{% endborder %}

{% border 使用arguments.callee重写迭代 color:purple %}
这样的话就可以让函数逻辑与函数名解耦，解决了硬编码带来的问题。  
```javascript
function factorial(num) {
    if (num <= 1) {
        return 1;
    } else {
        return num * arguments.callee(num - 1);
    }
}
console.log(factorial(5)); //120
let trueFactorial = factorial;
factorial = function () {
    return 0;
}
console.log(trueFactorial(5)); //120
console.log(factorial(10)) //0
```
{% endborder %}

{% endborder %}

{% border this color:green %}
{% border 标准函数中的this color:orange %}
引用的是把函数当成方法调用的上下文对象，  
全局上下文中调用函数，this指向 *windows*。  
```javascript
window.color = 'red';
let o = {
  color: 'blue'
};
function sayColor() {
  console.log(this.color);
}
sayColor(); //red
o.sayColor = sayColor;
o.sayColor(); //blue
```
{% endborder %}

{% border 箭头函数中的this color:yellow %}
引用的是定义箭头函数的上下文。  
```javascript
window.color = 'red';
let o = {
  color: 'blue'
}
let sayColor = () => {
  console.log(this.color);
}
sayColor(); //red
o.sayColor = sayColor;
o.sayColor(); //red
```
因此在使用**事件回调**或**定时回调**时，  
为了保证回调函数内this值指向定义函数的上下文，  
常使用箭头函数定义回调函数。
```javascript
function King() {
  this.royaltyName = 'Henry';
  setTimeout(() => console.log(this.royaltyName), 1000);
}
function Queen() {
  this.royaltyName = 'Elizabeth';
  setTimeout(function () { console.log(this.royaltyName); }, 1000);
}
new King(); //Henry
new Queen(); //undefined
```
{% endborder %}

{% endborder %}

{% border caller color:purple %}
*caller* 引用的是**调用当前函数的函数**。  
```javascript
function outer() {
  inner();
}
function inner() {
  console.log(inner.caller);
}
outer();  //output:outer source code
```
为了降低耦合度，可以引用*arguments.callee.caller*。
```javascript
function outer() {
  inner();
}
function inner() {
  console.log(arguments.callee.caller);
}
outer(); //outer source code
```
{% border 严格模式下 color:blue %}
- 访问*arguments.callee*会报错
- 不能给*caller*属性赋值
{% endborder %}
{% endborder %}

{% border new.target（ES6） color:orange %}
用于检测函数是否是使用new关键字调用的。  
- **正常调用** *new.target = undefined*
- **new关键字调用** *new.target = 被调用的构造函数*
```javascript
function King() {
  if (!new.target) {
    throw 'King must be instantiated using "new"'
  }
  console.log('King instantiated using "new"');
}
new King(); //King instantiated using new
King(); //Error:King must be instantiated using "new"
```
{% endborder %}

# 10.3.4 函数属性与方法

{% border 函数固有属性（2个） color:cyan %}

{% folding length color:green %}
length保存函数定义的**命名参数个数**。  
```javascript
function sayName(name) {
  console.log(name)
}
function sum(num1, num2) {
  return num1 + num2;
}
function sayHi() {
  console.log('hi')
}
console.log(sayName.length) //1
console.log(sum.length) //2
console.log(sayHi.length) //0
```
{% endfolding %}

{% folding prototype color:green %}
prototype用于**保存引用类型所有实例方法**，    
在自定义类型时特别重要。  
注意prototype不可枚举。  
{% endfolding %}

{% endborder %}

{% border 函数方法（apply，call，bind） color:yellow %}
这3种方法都是用于 **设置调用函数时函数体内this对象的值**。

{% note strict模式下，没有指定上下文的调用函数，函数的this不会指向window，而会变成undefined %}

{% folding apply() color:purple %}
接收2个参数：
- **param1** 指定this
- **param2** 参数数组
```javascript
function sum(num1, num2) {
  return num1 + num2;
}
function callSum1(num1, num2) {
  return sum.apply(this, arguments);
}
function callSum2(num1, num2) {
  return sum.apply(this, [num1, num2]);
}
console.log(callSum1(10, 20)); //30
console.log(callSum2(20, 30)); //50
```

{% endfolding %}

{% folding call() color:purple %}
call方法和apply的不同之处在于：  
**需要传递的参数是逐个传递的**
```javascript
function sum(num1, num2) {
  return num1 + num2;
}
function callSum(num1, num2) {
  return sum.call(this, ...arguments);
}

console.log(callSum(10, 20)); //30
```
{% endfolding %}

apply和call最强大就是控制函数调用上下文this的能力。  
```javascript
window.color = 'red'
let o = {
  color:'blue'
}
function sayColor() {
  console.log(this.color)
}
sayColor();
sayColor.call(this); //red
sayColor.call(window); //red
sayColor.call(o); //blue
```

{% border bind（ES5） color:purple %}
bind和call、apply的区别在于  
**bind会创建一个绑定了指定this的函数实例**。
```javascript
window.color = 'red';
var o = {
  color:'blue'
}
function sayColor() {
  console.log(this.color);
}
let objectSayColor = sayColor.bind(o);
objectSayColor(); //blue
```
{% endborder %}

{% border 提示 color:red %}
*toLocaleString()* 和 *toString()* 始终返回函数代码，  
但是返回代码的具体格式会因浏览器而异，  
因此应该在重要功能是避免依赖这些方法的返回值。  
{% endborder %}

{% endborder %}

# 10.3.5 函数表达式

```javascript
let functionName = function (arg1, arg2, arg3) { };
```
像这样创建的函数叫做 **匿名函数（anonymous function）**，  
有时也被称为**兰姆达（λ）函数**

{% border 函数声明提升案例 color:blue %}
使用函数声明会进行声明提升，如下面这段危险的代码：
```javascript
let condition = true;
if (condition) {
  function sayHi() {
    console.log('Hi!')
  }
} else {
  function sayHi() {
    console.log('Yo!')
  }
}
sayHi() // Hi
```
也许浏览器会帮你纠正这段声明，  
关键在于不同浏览器纠正问题的方式不一致，  
所以尽量不要这么写。
{% endborder %}
