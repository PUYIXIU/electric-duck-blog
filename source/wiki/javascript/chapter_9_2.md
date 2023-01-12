---
layout: wiki
wiki: JavaScript
title: 9.2 代理捕获器与反射方法
order: 91
---

代理可以捕获13种不同的基本操作。  
这些操作各自有不同的反射API方法、参数、关联ECMAScript操作和不变式。

# 9.2.1 get()
{% border color:red %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget,{
  get(target, property, receiver) { 
    console.log('get');
    return Reflect.get(...arguments);
  }
})
proxy.foo; //getS
```
{% endborder %}
- **调用场景：** 获取属性值的操作
- **反射API：** *Reflect.get()*
- **返回值：** 无返回值
- **拦截的操作：** 
  - *proxy.property*
  - *proxy\[property\]*
  - *Object.create\(proxy\)\[property\]*
  - *Reflect.get\(proxy, property, receiver\)*
- **捕获器处理程序参数：**
  - *target:* 目标对象
  - *property:* 字符串键属性
  - *receiver:* 代理对象
- **捕获器不变式：**
  - target.property不可写且不可配置，get返回值必须匹配target.property
  - target.property不可配置且\[\[Get\]\]=undefined，get返回值也必须是undefined

# 9.2.2 set()
{% border color:orange %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  set(target, property, value, reciever) { 
    console.log('set');
    return Reflect.set(...arguments);
  }
})
proxy.foo = 'bar'; //set
```
{% endborder %}
- **调用场景：** 在设置属性值的操作中被调用
- **反射API：** *Reflect.set\(\)*
- **返回值：** 
  - *true* 设置成功 
  - *false* 设置失败，strict模式下会抛出TypeError
- **拦截的操作：**
  - *proxy.property = value*
  - *proxy\[property\] = value*
  - *Object.create\(proxy\)\[property\] = value*
  - *Reflect.set\(proxy, property, value, receiver\)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *property* 键属性
  - *value* 属性值
  - *receiver* 接收赋值的对象
- **捕获器不变式：**
  - target.property不可写且不可配置，不能修改目标属性值
  - target.property不可配置且\[\[Set\]\] = undefined，不能修改目标属性值
  - strict模式下，return false会抛出TypeError

# 9.2.3 has()
{% border color:yellow %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  has(target, property) { 
    console.log('has');
    return Reflect.has(...arguments);
  }
})
'foo' in proxy; //has 
```
{% endborder %}
- **调用场景：** 在in操作符中被调用
- **反射API：** *Reflect.has\(\)*
- **返回值：** 必须为Boolean，表示属性是否存在
- **拦截的操作：** 
  - *property in proxy*
  - *property in Obejct.create\(proxy\)*
  - *with\(proxy\) {\(property\);}*
  - *Reflect.has\(proxy, property\)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *property* 键属性
- **捕获器不变式：**
  - target.property存在且不可配置，return true
  - target.property存在且目标对象不可扩展，return true

# 9.2.4 defineProperty()
{% border color:green %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  defineProperty(target, property, descriptor) {
    console.log('defineProperty');
    return Reflect.defineProperty(...arguments);
  }
})
// defineProperty
Object.defineProperty(proxy, 'foo', { value: 'bar' });
```
{% endborder %}
- **调用场景：** 在 *Obejct.defineProperty* 中被调用
- **反射API：** *Reflect.defineProperty\(\)*
- **返回值：** Boolean，表示属性是否成功定义
- **拦截的操作：** 
  - *Object.defineProperty\(proxy, property, descriptor)*
  - *Reflect.defineProperty\(proxy, property, descriptor)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *property* 键属性
  - *descriptor* 一个包含可选的 *enumerable、configurable、writable、value、get、set* 的对象
- **捕获器不变式：**
  - target.property\[\[Configurable]] = false，无法定义属性
  - target.property\[\[Configurable]] = true，不能添加不可配置的同名属性
  - target.property\[\[Configurable]] = false，不能添加可配置同名属性
  
# 9.2.5 getOwnPropertyDescriptor()
{% border color:cyan %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  getOwnPropertyDescriptor(target, property) { 
    console.log('getOwnPropertyDescriptor');
    return Reflect.getOwnPropertyDescriptor(...arguments);
  }
})
// getOwnPropertyDescriptor
Object.getOwnPropertyDescriptor(proxy, 'foo');
```
{% endborder %}
- **调用场景：** 在 *Object.getOwnPropertyDescriptor()* 中被调用
- **反射API：** *Reflect.getOwnPropertyDescriptor()*
- **返回值：** 必须返回Object，属性不存在时返回undefined
- **拦截的操作：**
  - *Object.getOwnPropertyDescriptor\(proxy, property\)*
  - *Reflect.getOwnPropertyDescriptor\(proxy, property\)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *property* 键属性
- **捕获器不变式：**
  - target.property存在
    - target.property可配置，返回表示该属性可配置的对象
    - target.property不可配置，返回表示该属性存在的对象
    - target可扩展，返回一个该属性存在的对象
    - target不可扩展，返回undefined表示该属性不存在
  - target.property不存在，不能返回表示该属性可配置的对象

# 9.2.6 deleteProperty()
{% border color:blue %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  deleteProperty(target,property) { 
    console.log('delete');
    return Reflect.deleteProperty(...arguments);
  }
})
delete proxy.foo; //delete
```
{% endborder %}
- **调用场景：** 在delete操作符中调用
- **反射API：** *Reflect.deleteProperty\()*
- **返回值：** *Boolean* 表示删除是否成功
- **拦截的操作：** 
  - *delete proxy.property*
  - *delete proxy\[property]*
  - *Reflect.deleteProperty\(proxy, property)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *property* 键属性
- **捕获器不变式：** target.property存在且不可配置则不能删除

# 9.2.7 ownKeys()
{% border color:red %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  ownKeys(target) {
    console.log('ownKeys');
    return Reflect.ownKeys(...arguments);
  }
})
Object.keys(proxy); //ownKeys
```
{% endborder %}
- **调用场景：** 在Object.keys及类似方法中调用
- **反射API：** *Reflect.ownKeys\()*
- **返回值：** *String/Symbol* 可枚举对象
- **拦截的操作：**
  - *Object.getOwnPropertyNames\(proxy)*
  - *Object.getOwnPropertySymbols\(proxy)*
  - *Object.keys\(proxy)*
  - *Reflect.ownKeys\(proxy)*
- **捕获器处理程序参数：**
  - *target* 目标对象
- **捕获器不变式：** 
  - 返回的可枚举对象必须包含target所有不可配置的自有property
  - 如果target不可扩展，返回的可枚举对象必须准确包含自有属性键

# 9.2.8 getPrototypeOf()
{% border color:purple %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  getPrototypeOf(target) {
    console.log('getPrototypeOf');
    return Reflect.getPrototypeOf(...arguments);
  }
})
// getPrototypeOf
Object.getPrototypeOf(proxy);
```
{% endborder %}
- **调用场景：** 在*Object.getPropertyOf*中调用
- **反射API：** *Reflect.getPropertyOf\()*
- **返回值：** *Object/null* 
- **拦截的操作：**
  - *Object.getPrototypeOf\(proxy)*
  - *Reflect.getPrototypeOf\(proxy)*
  - *proxy.\_\_proto\_\_*
  - *Object.prototype.isPrototypeOf\(proxy)*
  - *proxy instanceof Object*
- **捕获器处理程序参数：**
  - *target* 目标对象
- **捕获器不变式：** 若target不可扩展，proxy和target的getPrototypeOf值相等

# 9.2.9 setPrototypeOf()
{% border color:orange %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  setPrototypeOf(target, prototype) {
    console.log('setPrototypeOf');
    return Reflect.setPrototypeOf(...arguments);
  }
})
// setPrototypeOf
Object.setPrototypeOf(proxy, Object);
```
{% endborder %}
- **调用场景：** 在*Object.setPrototypeOf*中调用
- **反射API：** *Reflect.setPropertyOf\()*
- **返回值：** *Boolean* 表示原型赋值是否成功
- **拦截的操作：**
  - *Object.setPrototypeOf\(proxy)*
  - *Reflect.setPrototypeOf\(proxy)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *prototype* 代替原型，顶级原型则为null
- **捕获器不变式：** 若target不可扩展，唯一有效的prototype参数就是Object.getPrototypeOf(target)返回值

# 9.2.10 isExtensible()
{% border color:yellow %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  isExtensible(target) {
    console.log('isExtensible');
    return Reflect.isExtensible(...arguments);
  }
})
Object.isExtensible(proxy); //isExtensible
```
{% endborder %}
- **调用场景：** 在*Object.isExtensible*中调用
- **反射API：** *Reflect.isExtensible\()*
- **返回值：** *Boolean* 表示target是否可扩展
- **拦截的操作：**
  - *Object.isExtensible\(proxy)*
  - *Reflect.isExtensible\(proxy)*
- **捕获器处理程序参数：**
  - *target* 目标对象
- **捕获器不变式：** 
  - target可扩展，return true
  - target不可扩展，return false
  
# 9.2.11 preventExtensions()
{% border color:green %}
```javascript
const myTarget = {};
const proxy = new Proxy(myTarget, {
  preventExtensions(target) {
    console.log('preventExtensions');
    return Reflect.preventExtensions(...arguments);
  }
})
//preventExtensions
Object.preventExtensions(proxy);
```
{% endborder %}
- **调用场景：** 在*Object.preventExtensions*中调用
- **反射API：** *Reflect.preventExtensions\()*
- **返回值：** *Boolean* 表示target是否不可扩展
- **拦截的操作：**
  - *Object.preventExtensions\(proxy)*
  - *Reflect.preventExtensions\(proxy)*
- **捕获器处理程序参数：**
  - *target* 目标对象
- **捕获器不变式：** 必须返回 !Object.isExtensible\(proxy)
  
# 9.2.12 apply()
{% border color:cyan %}
```javascript
const myTarget = () => { };
const proxy = new Proxy(myTarget, {
  apply(target, thisArg, ...argumentsList) {
    console.log('apply');
    return Reflect.apply(...arguments);
  }
});
proxy(); //apply
```
{% endborder %}
- **调用场景：** 在调用函数时被调用
- **反射API：** *Reflect.apply\()*
- **返回值：** 无限制
- **拦截的操作：**
  - *proxy\(...argumentsList)*
  - *Function.prototype.apply\(thisArg, argumentsList)*
  - *Function.prototype.call\(thisArg, ...argumentsList)*
  - *Reflect.apply\(target, thisArgument, argumentsList)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *thisArg* 调用函数时this参数
  - *argumentsList* 调用函数时参数列表
- **捕获器不变式：** target必须是函数对象

# 9.2.13 construct()
{% border color:blue %}
```javascript
const myTarget = function () { };
const proxy = new Proxy(myTarget, {
  construct(target, argumentsList, newTarget) {
    console.log('construct');
    return Reflect.construct(...arguments);
  }
})
new proxy; //construct
```
{% endborder %}
- **调用场景：** 在new操作符中调用
- **反射API：** *Reflect.construct\()*
- **返回值：** *Object*
- **拦截的操作：**
  - *new proxy\(...argumentsList)*
  - *Reflect.construct\(target, argumentsList, newTarget)*
- **捕获器处理程序参数：**
  - *target* 目标对象
  - *argumentsList* 传给目标构造函数的参数列表
  - *newTarget* 最初被调用的构造函数
- **捕获器不变式：** target必须可以用做构造函数

