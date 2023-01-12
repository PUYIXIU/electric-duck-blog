---
layout: wiki
wiki: JavaScript
title: 9.3 代理模式
order: 92
---

{% quot 代理可以用来实现一些有用的编程模式 %}

# 9.3.1 跟踪属性访问

{% border color:yellow %}
把实现相应捕获器的某个对象代理放到应用中，  
可以监控这个对象何时在何处被访问过
```javascript
const user = {
  name:'Jake'
}
const proxy = new Proxy(user, {
  get(target, property, receiver) {
    console.log(`Getting ${property}`);
    return Reflect.get(...arguments);
  },
  set(target, property, value, receiver) {
    console.log(`Setting ${property} = ${value}`);
    return Reflect.set(...arguments);
  }
});
proxy.name; // Getting name
proxy.age = 27; // Setting age = 27
```
{% endborder %}

# 9.3.2 隐藏属性
{% border color:green %}
代理内部实现对外部代码不可见，  
可以借此隐藏目标对象上的属性
```javascript
const hiddenProperties = ['foo', 'bar'];
const targetObject = {
  foo: 1,
  bar: 2,
  baz: 3
};
const proxy = new Proxy(targetObject, {
  get(target, property) {
    if (hiddenProperties.includes(property)) {
      return undefined;
    } else {
      return Reflect.get(...arguments);
    }
  },
  has(target, property) {
    if (hiddenProperties.includes(property)) {
      return false;
    } else {
      return Reflect.has(...arguments);
    }
  }
});
console.log(proxy.foo); //undefined
console.log(proxy.bar); //undefined
console.log(proxy.baz); //3

console.log('foo' in proxy); // false
console.log('bar' in proxy); // false
console.log('baz' in proxy); // true
```
{% endborder %}

# 9.3.3 属性验证

{% border color:cyan %}
在赋值操作触发set捕获器时，  
可以根据所赋的值决定是否允许赋值。  
```javascript
const target = {
  onlyNumbersGoHere: 0 
}
const proxy = new Proxy(target, {
  set(target, property, value) {
    if (typeof value !== 'number') {
      return false;
    } else { 
      return Reflect.set(...arguments);
    }
  }
})

proxy.onlyNumbersGoHere = 1;
console.log(proxy.onlyNumbersGoHere); //1
proxy.onlyNumbersGoHere = '2';
console.log(proxy.onlyNumbersGoHere); //1
```
{% endborder %}

# 9.3.4 函数与构造函数参数验证

{% border color:blue %}
可以对函数和构造函数的**参数**进行审查，  
如让函数接收指定类型的值。  
```javascript
function median(...nums) {
  return nums.sort()[Math.floor(nums.length / 2)];
}
const proxy = new Proxy(median, {
  apply(target, thisArg, argumentsList) {
    for (const arg of argumentsList) {
      if (typeof arg !== 'number') {
        throw 'Error: Non-number argument provided';
      }
    }
    return Reflect.apply(...arguments);
  }
})
console.log(proxy(4, 7, 1)); //4
console.log(proxy(4, '7', 1)); //Error: Non-number argument provided
```
又或者是在实例化时必须给构造函数传参
```javascript
class User { 
  constructor(id) {
    this.id_ = id;
  }
}
const proxy = new Proxy(User, {
  construct(target, argumentsList, newTarget) {
    if (argumentsList[0] === undefined) {
      throw 'User cannot be instantiated without id';
    } else { 
      return Reflect.construct(...arguments);
    }
  }
})
new proxy(1);
new proxy(); //Error: ...
```
{% endborder %}

# 9.3.5 数据绑定与可观察对象

{% border color:purple %}
代理可以把运行时不相关的部分联系到一起，让不同的代码互操作。  
{% border 可以将被代理的类绑定到一个全局实例集合中 color:yellow %}
```javascript
const userList = [];
class User { 
  constructor(name) {
    this.name_ = name;
  }
}
const proxy = new Proxy(User, {
  construct() {
    const newUser = Reflect.construct(...arguments);
    userList.push(newUser);
    return newUser;
  }
});

new proxy('John');
new proxy('Jacob');
new proxy('Jingleheimerschmidt');
console.log(userList); //[User,User,User]
```
{% endborder %}

{% border 可以创建一个每次插入新实例都会发送消息的事件分派程序 color:orange %}
```javascript
const userList = [];
function emit(newValue) {
  console.log(newValue);
}
const proxy = new Proxy(userList, {
  set(target, property, value, receiver) {
    const result = Reflect.set(...arguments);
    if (result) {
      emit(Reflect.get(target, property, receiver));
    }
    return result;
  }
})
proxy.push('John'); //John 1
proxy.push('Jacob'); //Jacob 2
```
{% endborder %}
{% endborder %}
