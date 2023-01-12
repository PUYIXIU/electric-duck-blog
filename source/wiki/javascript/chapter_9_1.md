---
layout: wiki
wiki: JavaScript
title: 9.1 代理基础
order: 90
---

{% border  代理简单来说 color:dark %}
就是指给目标对象定义一个代理对象，然后把它当做抽象的目标对象使用  
因此它能够用作目标对象的替身，又完全独立于目标对象。
{% endborder %}
{% border 代理需要平台支持 color:light %}
代理是一种新的基础性语言能力，转义程序不能直接将其转换为ES6之前的代码，
因此代理和反射需要在百分百支持它们的平台上使用。
{% endborder %}

# 9.1.1 创建空代理

{% border  Proxy函数 color:blue%}
- **功能：** 创建代理
- **参数：**
  - *param 1：* 目标对象
  - *param 2：* 处理程序对象
- **区分代理和目标：**
  - 可以使用严格相等（===）来区分
  - 不能使用
{% endborder %}instanceof Proxy，因为Proxy.prototype是undefined
    
```javascript
const target = {
  id: 'target'
};

const handler = {};

const proxy = new Proxy(target, handler);

console.log(target.id); //target
console.log(proxy.id); //target

target.id = `foo`;
console.log(target.id); //foo
console.log(proxy.id); //foo

proxy.id = `bar`;
console.log(target.id); //bar
console.log(proxy.id); //bar

console.log(target.hasOwnProperty('id')); //true
console.log(proxy.hasOwnProperty('id')); //true

console.log(proxy === target); //false

// TypeError: Function has non-object 
// prototype 'undefined' in instanceof check
console.log(target instanceof Proxy);
console.log(proxy instanceof Proxy)

```

# 9.1.2 定义捕获器

使用代理的主要目的就是可以定义捕获器（trap）。  
代理能在操作传播到目标对象之前先调用捕获器函数，从而拦截并修改相应的行为。  
{% border 注意 color:warning%}
只有在代理对象上执行这些操作才会触发捕获器。
{% endborder %}
```javascript
const target = {
  foo: 'bar'
}
const handler = {
  get() { 
    return 'handler override';
  }
}
const proxy = new Proxy(target, handler);

console.log(target.foo); //bar
console.log(proxy.foo); //handler override

console.log(target['foo']) //bar
console.log(proxy['foo']) //handler override

console.log(Object.create(target)['foo']); //bar
console.log(Object.create(proxy)['foo']); //handler override
```

# 9.1.3 捕获器参数和反射API

{% border get()参数 color:blue %}
- *trapTarget:* 目标对象
- *property:* 要查询的属性
- *receiver:* 代理对象
{% endborder %}
```javascript
const target = {
  foo:'bar'
}

const handler = {
  get(trapTarget, property, receiver) { 
    console.log(trapTarget === target);
    console.log(property);
    console.log(receiver === proxy);
  }
}

const proxy = new Proxy(target, handler);
// true
// foo
// true
proxy.foo;
```
通过这些参数就可以重建目标对象的原始行为了：
```javascript
const target = {
  foo:'bar'
}
const handler = {
  get(trapTarget, property, receiver) { 
    return trapTarget[property];
  }
}
const proxy = new Proxy(target, handler);
console.log(proxy.foo); //bar
console.log(target.foo); //bar
```

{% border 全局对象Reflect color:cyan %}
通过全局对象Reflect的同名方法也可以轻松实现重建。
{% border 简洁一点... color:blue %}
```javascript
const handler = {
  get() { 
    return Reflect.get(...arguments);
  }
}
```
{% endborder%}
{% border 再简洁一点... color:purple %}
```javascript
const handler = {
    get:Reflect.get
}
```
{% endborder %}
{% border 更简洁点...（直接捕获所有方法） color:orange %}
```javascript
const proxy = new Proxy(target, Reflect);
```
{% endborder %}
{% border 使用反射API的样板代码 color:yellow %}
```javascript
const target = {
  foo: 'bar',
  baz: 'qux'
};
const handler = {
  get(trapTarget, property, receiver) { 
    let decoration = '';
    if (property === 'foo') { 
      decoration = '!!!';
    }
    return Reflect.get(...arguments) + decoration;
  }
}
const proxy = new Proxy(target, handler);
console.log(proxy.foo);  //foo!!!
console.log(target.foo); //foo
console.log(proxy.baz);  //qux
console.log(target.baz); //qux
```
{% endborder %}
{% endborder %}

# 9.1.4 捕获器不变式
当捕获器定义出现过于反常的行为时（如get和writable的定义出现冲突时）  
捕获器不变式会进行报错。
```javascript
const target = {};
Object.defineProperty(target, 'foo', {
  configurable: false,
  writable: false,
  value: 'bar'
});
const handler = {
  get() {
    return 'qux';
  }
};
const proxy = new Proxy(target, handler);
// 'get' on proxy: property 'foo' is a read-only 
// and non - configurable data property on the proxy 
// target but the proxy did not return its 
// actual value(expected 'bar' but got 'qux')
console.log(proxy.foo);
```

# 9.1.5 可撤销代理

{% border  Proxy.revocable color:dark %}
这个方法支持撤销代理对象与目标对象的关联。
```javascript
const target = {
  foo:'bar'
}

const handler = {
  get() { 
    return 'interecpted'
  }
}

const { proxy, revoke } = Proxy.revocable(target, handler);

console.log(proxy.foo); //bar
console.log(target.foo); //bar

revoke();
// TypeError:
// Cannot perform 'get' on a proxy that has been revoked
console.log(proxy.foo);
```
{% endborder %}
# 9.1.6 实用反射API

{% folding 1. 反射API与对象API open:false color:green %}
1. 反射API并不限于捕获处理程序
2. 大多数反射API方法在Object类型上有对应的方法
    - Object上的方法适用于通用程序
    - 反射方法适用于细粒度的对象控制与操作
{% endfolding %}

{% folding 2. 状态标记 open:false color:cyan %}
- **数据类型：** 布尔值
- **表示：** 意图执行的操作是否成功

{% border 重构前 color:blue %}
```javascript
const o = {}
try {
  Object.defineProperty(o, 'foo', 'bar');
  console.log('success');
} catch (e) { 
  console.log('failure');
}
```
{% endborder %}

{% border 重构后（操作失败时不会报错） color:purple %}
```javascript
const o = {};
if (Reflect.defineProperty(o, 'foo', { value: 'bar' })) {
  console.log('success'); //success
} else { 
  console.log('failure');
}
```
{% endborder %}

{% border 提供状态标记的反射方法 color:yellow %}
- *Reflect.defineProperty()*
- *Reflect.preventExtensions()*
- *Reflect.setPrototypeof()*
- *Reflect.set()*
- *Reflect.deleteProperty()*
{% endborder %}
{% endfolding %} 
{% folding 3. 用一等函数替代操作符 color:orange %}
- *Reflect.get():* 代替对象属性访问操作符
- *Reflect.set():* 代替=赋值操作符
- *Reflect.has():* 代替in操作符或with()
- *Reflect.deleteProperty():* 代替delete操作符
- *Reflect.construct():* 代替new操作符
{% endfolding %}
  
{% folding 4. 安全地应用函数 color:purple %}
主要针对apply方法的调用:
{% border apply调用反面教材 color:yellow %}
```javascript
Function.prototye.apply.call(myFunc, thisVal, argumentList);
```
{% endborder %}
{% border 使用Reflect来避免 color:orange %}
```javascript
Reflect.apply(myFunc, thisVal, argumentsList);
```
{% endborder %}
{% endfolding %}

# 9.1.7 代理另一个代理
{% border 多层拦截网 color:cyan %}
代理完全可以代理另一个代理，因此可以在一个目标对象之上构建多层拦截网。
```javascript
const target = {
  foo:'bar'
}

const firstProxy = new Proxy(target, {
  get() {
    console.log('first proxy');
    return Reflect.get(...arguments);
  }
});
const secondProxy = new Proxy(firstProxy, {
  get() {
    console.log('second Proxy');
    return Reflect.get(...arguments);
  }
});
// second Proxy
// first Proxy
// bar
console.log(secondProxy.foo);
```
{% endborder %}
# 9.1.8 代理的问题与不足

{% border 1. 代理中的this color:orange %}
直觉上讲，this通常指向调用这个方法的对象。  
即调用代理上的任何方法，如proxy.outerMethod(),  
proxy.outerMethod()调用this.innerMethod(),  
实际上会调用proxy.innerMethod()

{% border 当目标对象依赖于对象标识时会存在问题 color:red %}
```javascript
const wm = new WeakMap();
class User { 
  constructor(userId) { 
    wm.set(this, userId);
  }
  set id(userId) {
    wm.set(this, userId);
  }
  get id() { 
    return wm.get(this);
  }
}

const user = new User(123);
console.log(user.id); //123

const userInstanceProxy = new Proxy(user, {});
console.log(userInstanceProxy.id); //undefined
```
这是因为User实例一开始使用目标对象作为WeakMao的键，  
代理对象却尝试从自身取得这个实例。  
{% border 解决方案 color:purple %}
把代理User实例改为代理User类本身，  
再创建代理的实例。
```javascript
const UserClassProxy = new Proxy(User, {});
const proxyUser = new UserClassProxy(456);
console.log(proxyUser.id); //456
```
{% endborder %}
{% endborder %}
{% endborder %}

{% border 2. 代理与内部槽位 color:green %}
有些ECMAScript内置类型可能会依赖代理无法控制的机制，  
原因是这些内置类型方法的执行依赖this值上的内部槽位，  
但是代理对象上不存在这个内部槽位。  
```javascript
const target = new Date();
const proxy = new Proxy(target, {});
console.log(proxy instanceof Date); //true
proxy.getDate(); //TypeError: 'this' is not a Date object
```
{% endborder %}
