---
layout: wiki
wiki: JavaScript
title: 10.1 函数语法
order: 100
---

{% border 函数&&对象 color:yellow %}
- 函数实际上是对象
- 函数都是Function类型的实例
- Function也有属性和方法
- 函数名是指向函数对象的指针
{% endborder %}
  
{% border 定义函数的几种方式 color:purple %}
{% folders color:blue %}
<!-- folder ① 函数声明 -->
{% border color:yellow %}
```javascript
function fun(param1,param2){
    return param1 + param2;
}
```
{% endborder %}
<!-- folder ② 函数表达式 -->
{% border color:blue %}
```javascript
let fun = function(num1, num2){
    return num1 + num2;
}
```
{% endborder %}
<!-- folder ③ 箭头函数 -->
{% border color:green %}
```javascript
let fun = (param1, param2) => {
    return param1 * param2;
}
```
{% endborder %}
<!-- folder ④ Function构造函数 -->
{% border color:orange %}
```javascript
let fun = new Function('param1','param2','return param1 % param2');
```
{% folding Function构造函数的参数 color:yellow %}
- **最后一个参数** 始终被当做函数体
- **非最后一个参数** 作为新函数的参数
{% endfolding %}

{% folding 不推荐使用原因：两次解释代码，会影响性能 color:red %}
1. 将它当做常规ECMAScript代码
2. 解释传给构造函数的字符串
{% endfolding %}

{% endborder %}
{% endfolders %}

{% border 重要的是： color:blue %}
把函数想象为对象，  
把函数名想象为指针
{% endborder %}

{% endborder %}

# 10.1.1 箭头函数
{% border 箭头函数适合嵌入函数的场景 color:cyan %}
```javascript
let ints = [1, 2, 3];
console.log(ints.map(function (i) { return i + 1 }));
console.log(ints.map(i => i + 1));
```
{% endborder %}

{% border 只有一个参数的情况下可以不加括号 color:red %}
```javascript
// 一个参数可以省略括号
let triple = x => { return 3 * x; }
// 没有参数需要括号
let getRandom = () => { return Math.random(); }
// 多个参数需要括号
let sum = (a, b) => { return a + b; }
```
{% endborder %}

{% border 函数体单条语句隐藏大括号表示返回这行代码的值 color:purple %}
```javascript
let triple = x => 3 * x;
// 直接赋值
let value = {};
let setName = x => x.name = 'Matt';
setName(value);
console.log(value.name); //Matt
```
{% endborder %}

{% border 箭头函数局限 color:green %}
- 不能使用 *arguments、super、new.target*
- 不能用作构造函数
- 没有 *protortpe* 属性
{% endborder %}

# 10.1.2 函数名

函数名作为指针就意味着：  
一个函数可以有多个名称
```javascript
function sum(num1, num2) {
  return num1 + num2;
}
console.log(sum(10, 20)); //30
let anotherSum = sum;
console.log(anotherSum(10, 10)); //20
sum = null;
console.log(anotherSum(10, 10)); //20
```
{% border 函数的name属性 color:green %}
- name属性一般保存的是函数标识符
- name属性只读
- 没有名称的函数，name为空字符串
- Function构造的函数，name为'anonymous'
```javascript
function foo() { }
let bar = function () { };
let baz = () => { };
console.log(foo.name); //foo
console.log(bar.name); //bar
console.log(baz.name); //baz
console.log((() => { }).name); //''
console.log((new Function()).name); //anonymous
```
{% border 特殊函数的标识前缀 color:cyan %}
对于获取函数get、设置函数set、使用bind\()实例化的函数，  
标识符前会加上一个前缀
|函数|前缀|
|:---:|:---:|
|get|get|
|set|set|
|bind|bound|
{% endborder %}
```javascript
function foo() { }

console.log(foo.bind(null).name); //bound foo

let dog = {
  years: 1,
  get age() { 
    return this.years;
  },
  set age(newAge) {
    this.years = newAge;
  }
}
let propertyDescriptor = Object.getOwnPropertyDescriptor(dog, 'age');
console.log(propertyDescriptor.get.name); //get age
console.log(propertyDescriptor.set.name); //set age
```
{% endborder %}
