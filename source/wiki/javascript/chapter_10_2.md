---
layout: wiki
wiki: JavaScript
title: 10.2 函数参数
order: 101
---

# 10.2.1 理解参数

ECMAScript的参数内部表现为一个{% mark 数组 color:purple %}，  
但函数并不关心数组中包含什么

{% border arguments color:blue %}
arguments是一个{% mark 类数组对象 color:green %}，  
可以在函数内部访问arguments对象，  
{% border 声明参数写法  color:cyan %}
```javascript
function sayHi(name,message) {
  console.log("Hello" + name + "," + message);
}
```
{% endborder %}
{% border 使用arguments重写 color:yellow %}
```javascript
function sayHi() {
  console.log("Hello" + arguments[0] + ', ' + arguments[1]);
}
```
{% endborder %}

{% border 使用arguments.length检查传入的参数个数 color:green %}
```javascript
function doAdd() {
  if (arguments.length === 1) {
    console.log(arguments[0] + 10);
  } else if (arguments.length === 2) {
    console.log(arguments[0] + arguments[1]);
  }
}
doAdd(10); //20
doAdd(30, 20); //50
```
{% endborder %}

{% border arguments对象也可以和命名参数一起使用 color:purple %}
```javascript
function doAdd(num1, num2) {
  if (arguments.length === 1) {
    console.log(num1 + 10);
  } else if (arguments.length === 2) {
    console.log(num1 + num2);
  }
}
doAdd(0); //10
doAdd(5, 10); //15
```
{% endborder %}

{% border arguments始终与对应的命名参数同步 color:orange %}
但它们并不访问同一个内存地址，  
只是保持同步而已。  
```javascript
function doAdd(num1, num2) {
  num2 = 1000;
  arguments[0] = -1000;
  console.log(num1 + arguments[1]);
}
doAdd(10, 20); //0
```
但如果访问arguments的索引超过了实际传参的数量，则无法同步。  
并且访问未传的参数默认值为undefined。
{% endborder %}

{% border strict mode color:warning %}
- arguments和命名参数无法再同步
- 函数中重写arguments对象会报错
{% endborder %}
{% endborder %}

{% border 箭头函数中的参数  %}

{% border 箭头函数参数不能使用arguments关键字访问 color:cyan %}
```javascript
function foo() {
  console.log(arguments[0]);
}
foo(5); //5

let bar = () => {
  console.log(arguments[0]);
}
bar(5); //ReferenceError: arguments is not defined
```
{% endborder %}

{% border 可以在包装函数中把arguments提供给箭头函数 color:green %}
```javascript
function foo() {
  let bar = () => {
    console.log(arguments[0]);
  };
  bar();
}
foo(5); //5
```
{% endborder %}

{% endborder %}

# 10.2.2 没有重载

{% note ECMAScript函数参数内部是由数组表示的，没有函数签名，自然也就无法重载 %}

{% border 存在同名函数，后定义的覆盖先定义的 color:yellow %}
```javascript
function addSomeNumber(num) {
  return num + 100;
}
function addSomeNumber(num) {
  return num + 200;
}
console.log(addSomeNumber(0)); //200
```
{% note 可以通过检查参数的类型和数量，分别执行不同逻辑来模拟函数重载。 color:cyan %}
{% endborder %}

# 10.2.3 默认参数值

{% border ECMAScript5.1以前设置默认参数值 color:cyan %}

**简单粗暴** 检测参数是否等于undefined，是的话就给它赋一个值
```javascript
function makeKing(name) {
  name = (typeof name !== 'undefined') ? name : 'Henry';
  return `King ${name} VIII`;
}
console.log(makeKing()); // King Henry VIII
console.log(makeKing('Louis')); // King Louis VIII
```
{% endborder %}

{% border ECMAScript6：支持=实现设置参数默认值  color:blue %}
```javascript
function makeKing(name = 'Henry') {
  return `King ${name} VIII`;
}
console.log(makeKing()); //King Henry VIII
console.log(makeKing('Louis')); // King Louis VIII
```
{% endborder %}

{% border 通过传值为undefined的参数占位 color:purple %}
```javascript
function makeKing(name = 'Henry', numerals = 'VIII') {
  return `King ${name} ${numerals}`
}
console.log(makeKing()); //King Henry VIII
console.log(makeKing('Louis')); //King Louis VIII
console.log(makeKing(undefined, 'VI')); //King Henry VI
```
{% endborder %}

{% border arguments不反映参数的默认值 color:green %}
arguments始终以调用函数时传入的值为准，  
所以修改命名参数也不会影响arguments对象
```javascript
function makeKing(name = 'Henry') {
  name = 'Louis';
  return `King ${arguments[0]}`;
}
console.log(makeKing()); //King undefined
console.log(makeKing('Louis')); //King Louis
```
{% endborder %}

{% border 默认参数可以是原始类型或者引用类型 color:blue %}
计算默认值的函数只有在调用函数并且未传相应参数时才会被调用
```javascript
let romanNumerals = ['I', 'II', 'III', 'IV', 'V', 'VI'];
let ordinalty = 0;
function getNumerals() {
  return romanNumerals[ordinalty++];
}
function makeKing(name = 'Henry', numerals = getNumerals()) {
  return `King ${name} ${numerals}`;
}
console.log(makeKing()); //King Henry I
console.log(makeKing('Louis', 'XVI')); //King Louis XVI
console.log(makeKing()); //King Henry II 
console.log(makeKing()); //King Henry III
```
{% endborder %}

{% border 箭头函数得到默认参数 color:orange %}
在只有一个参数且设置了默认值的时候，箭头函数参数的括号就不能省略了
```javascript
let makeKing = (name = 'Henry') => `King ${name}`;
console.log(makeKing()); //King Henry
```
{% endborder %}

{% border 默认参数作用域与暂时性死区 %}
给参数赋默认值实际上等同于使用let顺序声明
```javascript
function makeKing(name = 'Henry', numerals = 'VIII') {
  return `King ${name} ${numerals}`;
}
console.log(makeKing()); //King Henry VIII
```
上面这段代码就等同于：
```javascript
function makeKing() {
  let name = 'Henry';
  let numerals = 'VIII';
  return `King ${name} ${numerals}`;
}
console.log(makeKing()); //King Henry VIII
```
默认参数赋值也是顺序的：
```javascript
function makeKing(name = 'Henry', numerals = name) {
  return `King  ${name} ${numerals}`;
}
console.log(makeKing()); //King Henry Henry
```

{% border 暂时性死区 color:red %}
**暂时性死区** 指的是 *前面定义的参数不能引用后面定义的*，  
否则会抛出错误：
```javascript
function makeKing(name = numerals, numerals = 'VIII') {
  return `King ${name} ${numerals}`;
}
// ReferenceError: Cannot access 'numerals' before initialization
console.log(makeKing());
```
参数也存在与自己的作用域中
```javascript
function makeKing(name = 'Henry', numerals = defaultNumers) {
  let defaultNumbers = 'VIII';
  return `King ${name} ${numerals}`;
}
console.log(makeKing()); //ReferenceError: defaultNumbers is not defined
```

{% endborder %}

{% endborder %}

# 10.2.4 参数扩展与收集

{% note ES6新增的扩展操作符最有用的场景就是函数定义中的参数列表 color:green %}

{% border child:tabs %}
{% tabs %}

<!-- tab 扩展参数 -->
{% border 使用场景：不需要传入一个数组，强调分别传入数组元素 color:orange %}
```javascript
let values = [1, 2, 3, 4];
function getSum() {
    let sum = 0;
    for (let i = 0; i < arguments.length; i++){
        sum += arguments[i];
    }
    return sum;
}
```
{% endborder %}

{% border 可以使用扩展操作符拆分可迭代对象参数 color:yellow %}
```javascript
console.log(getSum(...values)); //10
console.log(getSum(...values, 5)); //15
console.log(getSum(-1, ...values, 5)); //14
console.log(getSum(...values, ...[5, 6, 7])); //28
```
{% endborder %}

{% border 扩展操作符也可以用于命名参数中 color:blue %}
```javascript
function getProduct(a, b, c = 1) {
  return a * b * c;
}
let getSum = (a, b, c = 0) => {
  return a + b + c;
}
console.log(getProduct(...[1,2])); //2
console.log(getProduct(...[1,2,3])); //6
console.log(getProduct(...[1, 2, 3, 4])); //6
console.log(getSum(...[0, 1])); //1
console.log(getSum(...[0, 1, 2])); //3
console.log(getSum(...[0, 1, 2, 3])); //3
```
{% endborder %}

<!-- tab 收集参数 -->
{% border 扩展操作符可以把不同长度的独立参数组合为一个数组 color:purple %}
```javascript
function getSum(...values) {
  return values.reduce((x, y) => x + y, 0);
}
console.log(getSum(1, 2, 3)); //6
```
{% endborder %}

{% border 收集参数前如果还有命名参数，就只会收集剩余的参数 color:blue %}
没有剩余的话就只得到空数组
```javascript
function ignoreFirst(firstValue, ...values) {
  console.log(values);
}
ignoreFirst();  //[]
ignoreFirst(1); //[]
ignoreFirst(1, 2); //[2]
ignoreFirst(1, 2, 3); //[2,3]

function getProduct(...values, lastValue) {
  //SyntaxError: Rest parameter must be last formal parameter
}
```
{% endborder %}
{% border 箭头函数支持收集参数 color:green %}
```javascript
let getSum = (...values) => {
  return values.reduce((x, y) => x + y, 0);
}
console.log(getSum(1, 2, 3)); //6
```
{% endborder %}

{% border 收集参数不影响arguments color:cyan %}
```javascript
function getSum(...values) {
  console.log(arguments.length); //3
  console.log(arguments); //[1,2,3]
  console.log(values); //[1,2,3]
}
getSum(1, 2, 3);
```
{% endborder %}

{% endtabs %}
{% endborder %}
