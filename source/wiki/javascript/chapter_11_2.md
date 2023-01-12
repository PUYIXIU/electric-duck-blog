---
layout: wiki
wiki: JavaScript
title: 11.2 期约
order: 111
---
{% note 期约是对尚不存在的结果的一个替身。 color:yellow %}
{% border 期约（Promise）还有以下别称 color:yellow %}
- 终局 *（eventual）*
- 期许 *（future）*
- 延迟 *（delay）*
- 迟付 *（deferred）*
{% endborder %}
  
# 11.2.1 Promises/A+规范
{% border Promise起源 color:yellow %}
- **早期** 期约机制在jQuery和Dojo中以 **Deferred API** 的形式出现  
- **2010** CommonJS项目实现了 **Promises/A**
- **2012** **Promiese/A+** 组织fork了CommonJS的 *Promises/A* 建议
- **最终** ECMAScript6增加了 **Promise** 类型
{% endborder %}
  
# 11.2.2 期约基础

{% border new Promise() color:purple %}
Promise作为引用类型时可以通过new操作符实例化，  
实例化时需要传入 **执行器（executor）** 函数作为参数。  
{% note 如果不提供executor参数会报错。 color:red %}
```javascript
let p = new Promise(() => {});
setTimeout(console.log, 0, p); //Promise{<pending>}
```
{% endborder %}

{% folders %}
<!--folder 1. 期约状态机-->

{% border 期约可能处在下面3种状态之一： color:yellow %}
- 待定 *（pending）* 表示尚未开始或正在执行
- 兑现/解决 *（fulfilled/resolved）* 表示已经成功完成
- 拒绝 *（rejected）* 表示没有成功完成

{% timeline %}
<!-- node 初始状态 -->
期约处于 **待定（pending）** 状态中。
<!-- node 待定状态 -->
期约可以 **落定（settled）** 为2种状态：  
- **兑现（fulfilled）** 状态代表成功
- **拒绝（rejected）** 状态代表失败
{% endtimeline %}

{% border 期约状态是私有的 color:blue %}
这主要是为了避免根据读取到的期约状态，以同步方式处理期约对象。  
期约故意将异步行为封装起来，从而隔离外部的同步代码。
{% endborder %}

{% endborder %}

<!-- folder 2. 解决值、拒绝理由及期约用例 -->
{% border 期约的两个主要用途 color:yellow %}
- 抽象表示一个异步操作是否完成
- 期约封装异步操作会实际生成某个状态改变后需要访问的值

为了支持这两个用途，期约提供了两个私有的内部属性（可选，默认值为undefined）：
- **value（值）**： 期约状态切换为兑现时提供
- **reason（理由）**： 期约状态切换为拒绝时提供

{% endborder %}

<!-- folder 3. 通过执行函数控制期约状态 -->
{% border executor的两项职责 color:yellow %}
- 初始化期约的异步行为
- 控制状态的最终转换

控制期约转换的2个函数参数通常这样命名：
- *resolve()* 将状态切换为fulfilled
- *reject()* 将状态切换为rejected

{% note executor函数是Promise的初始化程序。 color:blue %}

下面这段程序并非异步操作，  
因为在初始化Promise时，executor已经改变了Promise的状态。
```javascript
let p1 = new Promise((resolve, reject) => resolve());
// Promise{<fulfilled>: undefined}
setTimeout(console.log, 0, p1);

let p2 = new Promise((resolve, reject) => reject());
// Promise{<rejected>: undefined}
setTimeout(console.log, 0, p2);  
// Uncaught (in promise)
```

使用setTimeout推迟切换状态，可以看到Promise仍处在pending状态中：
```javascript
let p = new Promise((resolve, reject) => setTimeout(resolve, 1000));
setTimeout(console.log, 0, p); //Promise{<pending>}
```

Promise状态的转换不可撤销，修改状态操作静默失败：
```javascript
let p = new Promise((resolve, reject) => {
    resolve();
    reject();
});
// Promise {<fulfilled>: undefined}
setTimeout(console.log, 0, p); 
```

为了防止长时间卡在pending状态，  
可以添加一个定时退出的功能：
```javascript
let p = new Promise((resolve, reject) => {
  setTimeout(reject, 5000);
})
// Promise {<pending>}
setTimeout(console.log, 0, p); 
//Uncaught: (in promise)
// Promise {<rejected>}
setTimeout(console.log, 6000, p);
```

{% endborder %}

<!-- folder 4. Promise.resolve() -->
{% border color:yellow %}

{% timeline %}
<!-- node 功能 -->
调用 **Promise.resolve()** 静态方法可以实例化一个 **resolved Promise** ，  
实际上它可以 **把任何值都转换为一个Promise对象** 。
<!-- node 参数 -->
**Promise.resolve()** 的第一个参数表示解决的期约的返回值 **value** 。  
多余的参数会被忽略。
```javascript
// Promise<resolved>: undefined
setTimeout(console.log, 0, Promise.resolve());
// Promise<resolved>: 3
setTimeout(console.log, 0, Promise.resolve(3));
// Promise<resolved>: 4
setTimeout(console.log, 0, Promise.resolve(4, 5, 6));
```
<!-- node 幂等性 -->
传入参数如果本身就是Promise对象，就相当于{% mark 空包装 color:blue %}。
```javascript
let p = Promise.resolve(7);
setTimeout(console.log, 0, p === Promise.resolve(p)); //true
setTimeout(console.log, 0, p === Promise.resolve(Promise.resolve(p))); //true
```
这种幂等性还会**保留传入期约的状态**。  
```javascript
let p = new Promise(() => { });
// Promise {<pending>}
setTimeout(console.log, 0, p);
// Promise {<pending>}
setTimeout(console.log, 0, Promise.resolve(p));
// true
setTimeout(console.log, 0, p === Promise.resolve(p));
```

{% endtimeline %}
{% endborder %}

<!-- folder 5. Promise.reject() -->
{% border color:yellow %}
{% timeline %}
<!-- node 功能 -->
实例化一个 **rejective Promise** 并抛出一个**异步错误**，  
**异步错误** 不能通过try/catch捕捉，只能通过 **拒绝程序** 捕获。  

<!-- node 参数 -->
**Promise.reject()** 的第一个参数作为 **拒绝期约的理由**，  
该理由会顺次传给后续的拒绝程序。  
```javascript
let p = Promise.reject(3);
// Promise {<rejected>: 3}
setTimeout(console.log, 0, p);
p.then(null, (e) => setTimeout(console.log, 0, e)); //3
```
<!-- node 非幂等性 -->
传给 **Promise.reject** 一个Promise对象作为参数，  
该对象会作为拒绝期约的理由。
```javascript
//Promise {<rejected>: Promise}
setTimeout(console.log, 0, Promise.reject(Promise.resolve(3)));
```
{% endtimeline %}


{% endborder %}

<!--folder 6. 同步/异步执行的二元性 -->
{% border 对比同步/异步模式下捕捉错误的区别 color:yellow %}
{% split bg:card %}
<!-- cell left -->
```javascript
try { 
  throw new Error('foo');
} catch (e) {
  console.log(e); //Error: foo
}
```
<!-- cell right -->
```javascript
try { 
  Promise.reject(new Error('bar'));
} catch (e) {
  console.log(e);
}
//Uncaught (in promise) Error: bar
```
{% endsplit %}
{% border Promise真正的异步性在于： color:blue %}
Promise是 **同步对象** ，也是 **异步执行模式** 的媒介。
- **同步模式：** 错误直接抛到执行同步代码的线程中
- **异步模式：** 错误通过浏览器异步消息队列来处理
{% endborder %}
{% endborder %}

{% endfolders %}

# 11.2.3 期约的实例方法

{% folders %}
<!-- folder 1. 实现Thenable接口 -->
{% border 如何实现？ color:yellow %}
答：在对象上实现 **then** 方法。  
```javascript
class MyThenable{
    then(){}
}
```
ECMAScript暴露的 **异步结构中的任何对象** 都有一个then方法。  
{% endborder %}

<!-- folder 2. Promise.prototype.then() -->

{% border  then方法的参数与返回值 color:yellow %}
then方法接收2个可选的参数：
- **param1** onResolved处理程序（如果选择不传，需要使用**null/undefined**占位）
- **param2** onRejected处理程序

```javascript
function onResolved(id) {
  setTimeout(console.log, 0, id, 'resolved');
};
function onRejected(id) {
  setTimeout(console.log, 0, id, 'rejected');
}
let p1 = new Promise((resolve, reject) => setTimeout(resolve, 1000));
let p2 = new Promise((resolve, reject) => setTimeout(reject, 1000));

// p1 resolved
p1.then(()=>onResolved('p1'), ()=>onRejected('p1')); 
// p2 rejected
p2.then(()=>onResolved('p2'), ()=>onRejected('p2')); 
```

如果参数非函数类型，会被 **静默忽略**。  
如果只想提供onRejected参数，需要对onResolved参数进行 **占位**。

```javascript
function onResolved(id) {
  setTimeout(console.log, 0, id, 'resolved');
}
function onRejected(id) {
  setTimeout(console.log, 0, id, 'rejected');
}
let p1 = new Promise((resolve, reject) => setTimeout(resolve, 1000));
let p2 = new Promise((resolve, reject) => setTimeout(reject, 1000));

// 非函数处理程序：静默
p1.then('notFunction');

// onResolved占位
p2.then(undefined, () => onResolved('p2')); //p2 rejected
```

{% note Promise.prototype.then返回值：**一个新的期约实例**。 color:blue %}

```javascript
let p1 = new Promise(() => { });
let p2 = p1.then();
setTimeout(console.log, 0, p1); //Promise <pending>
setTimeout(console.log, 0, p2); //Promise <pending>
setTimeout(console.log, 0, p1 === p2); false;
```
对于不同返回值的几种情况：
{% folding onResolved函数提供了返回值 color:blue %}
会将返回值通过Promise.resolve()包装成新的期约。
```javascript
let p6 = p1.then(() => 'bar');
let p7 = p1.then(() => Promise.resolve('bar'));
setTimeout(console.log, 0, p6); // Promise{<fulfilled>: bar}
setTimeout(console.log, 0, p7); // Promise{<fulfilled>: bar}

let p8 = p1.then(() => new Promise(() => { }));
let p9 = p1.then(() => Promise.reject());
//Uncaught (in promise) undefined
setTimeout(console.log, 0, p8); //Promise {<pending>}
setTimeout(console.log, 0, p9); //Promise {<rejected>: undefined}
```
{% endfolding %}

{% folding 没有提供onResolved函数 color:blue %}
会包装上一个期约解决之后的值。  
```javascript
let p1 = Promise.resolve('foo');
let p2 = p1.then();
setTimeout(console.log, 0, p2); //Promise{<fulfilled: 'foo'>}
let p8 = p1.then(() => new Promise(() => { }));
let p9 = p1.then(() => Promise.reject());
//Uncaught (in promise) undefined
setTimeout(console.log, 0, p8); //Promise {<pending>}
setTimeout(console.log, 0, p9); //Promise {<rejected>: undefined}
```
{% endfolding %}

{% folding onResolved函数没有显式返回语句 color:blue %}
会包装默认返回值 **undefined**。
```javascript
let p3 = p1.then(() => undefined); 
let p4 = p1.then(() => { });
let p5 = p1.then(() => Promise.resolve());
setTimeout(console.log, 0, p3); // Promise{<fulfiled>: undefined}
setTimeout(console.log, 0, p4); // Promise{<fulfiled>: undefined}
setTimeout(console.log, 0, p5); // Promise{<fulfiled>: undefined}
```
{% endfolding %}

{% folding 抛出异常 color:blue %}
会返回 **rejective Promise**。
```javascript
// Uncaught (in promise) baz
let p10 = p1.then(() => { throw 'baz'; });
setTimeout(console.log, 0, p10); //Promise <rejected> baz
```
{% endfolding %}

{% folding 返回错误值 color:blue %}
不会触发reject，会将错误对象包装在一个 **resolved Promise** 中。
```javascript
let p11 = p1.then(() => Error('qux'));
setTimeout(console.log, 0, p11); //Promise {<fulfilled>: Error:qux}
```
{% endfolding %}

*onRejected*处理程序返回值也会被 **Promise.resolve()** 包装，  
虽然有点诡异，但是 **onRejected** 本身的任务就是 **捕获异步错误**，    
也就是 **捕获错误后不抛出异常** 。 

{% folding onRejected返回值案例 color:purple %}
```javascript
let p1 = Promise.reject('foo');

let p2 = p1.then();
// Promise {<rejected>: 'foo'}
// Uncaught (in promise) foo
setTimeout(console.log, 0, p2);

let p3 = p1.then(null, () => undefined);
let p4 = p1.then(null, () => { });
let p5 = p1.then(null, () => Promise.resolve());

setTimeout(console.log, 0, p3); // Promise {<rejected>: undefined}
setTimeout(console.log, 0, p4); // Promise {<rejected>: undefined}
setTimeout(console.log, 0, p5); // Promise {<rejected>: undefined}

let p6 = p1.then(null, () => 'bar');
let p7 = p1.then(null, () => Promise.resolve('bar'));

setTimeout(console.log, 0, p6); // Promise {<rejected>: bar}
setTimeout(console.log, 0, p7); // Promise {<rejected>: bar}

let p8 = p1.then(null, () => new Promise(() => { }));
let p9 = p1.then(null, () => Promise.reject());

setTimeout(console.log, 0, p8); // Promise {<pending>}
// Promise {<rejected>: undefined}
// Uncaught (in promise) undefined
setTimeout(console.log, 0, p9); 

let p10 = p1.then(null, () => { throw 'baz' });
// Promise {<rejected>:'baz'}
// Uncaught (in promise) baz
setTimeout(console.log, 0, p10); 

let p11 = p1.then(null, () => Error('qux'));
// Promise {<rejected>:Error:qux}
setTimeout(console.log, 0, p11);
```
{% endfolding %}

{% endborder %}

<!-- folder 3. Promise.prototype.catch() -->

{% border catch的功能与参数 color:yellow %}

catch用于 **给期约添加拒绝处理程序**。  
唯一参数是 **onRejected处理程序**。
相当于 {% mark Promise.prototype.then(null, onRejected) color:purple %} 的语法糖。
```javascript
let p1 = new Promise(() => { });
let p2 = p1.catch();
setTimeout(console.log, 0, p1); // Promise {<pending>}
setTimeout(console.log, 0, p2); // Promise {<pending>}
setTimeout(console.log, 0, p1 === p2); //false
```
{% endborder %}

<!-- folder 4. Promise.prototype.finally() -->

{% border finally的功能与参数 color:yellow %}
finally()方法用于 **给期约添加onFinally处理程序**。
{% border onFinally color:blue %}
无论Promise转化为 **fulfilled** 还是 **rejected** ，  
onFinally都会执行，  
用这个方法能够避免 **onResolved** 和 **onRejected** 程序中出现冗余代码。
{% note onFinally无法获知Promise的状态，所以这个方法主要用于添加清理代码。 color:purple %}
```javascript
let p1 = Promise.resolve();
let p2 = Promise.reject();
let onFinally = function () { 
  setTimeout(console.log, 0, 'Finally!');
}
p1.finally(onFinally); // Finally!
p2.finally(onFinally); // Finally!
```
{% folding finally()与catch()、then()的区别 color:blue %}
主要就是onFinally被设计为一个  **状态无关的方法**，  
重点在于 **父期约的传递** 上。
```javascript
let p1 = Promise.resolve('foo');
let p2 = p1.finally();
let p3 = p1.finally(() => undefined);
let p4 = p1.finally(() => { });
let p5 = p1.finally(() => Promise.resolve());
let p6 = p1.finally(() => 'bar');
let p7 = p1.finally(() => Promise.resolve('bar'));
let p8 = p1.finally(() => Error('qux'));

setTimeout(console.log, 0, p2); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p3); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p4); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p5); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p6); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p7); //Promise {<fulfilled>:foo}
setTimeout(console.log, 0, p8); //Promise {<fulfilled>:foo}

```
{% endfolding %}

{% folding 特殊情况下的finally返回值 color:blue %}
当出现下列情况时，会返回相应的期约：  
- 期约待定：Promise {\<pending>}
- onFinally程序抛出了错误
- onFinally程序显式抛出或返回了一个 rejective Promise

```javascript
let p1 = Promise.resolve('foo');

// 返回待定期约
let p2 = p1.finally(
    () => new Promise(
        (resolve, reject) =>
            setTimeout(() => resolve('bar'), 100)
    )
);
setTimeout(console.log, 0, p2); // Promise {<pending>}
setTimeout(() => setTimeout(console.log, 0, p2), 200); //Promise {<fulfilled>: foo}

let p9 = p1.finally(() => new Promise(() => { }));
let p10 = p1.finally(() => Promise.reject());
// Uncaught (in promise) undefined
setTimeout(console.log, 0, p9); // Promise {<pending>}
setTimeout(console.log, 0, p10); // Promise {<rejected>: undefined}
```
{% endfolding %}

{% endborder %}

{% endborder %}

<!-- folder 5. 非重入期约的方法 -->
{% border 非重入（non-reentrancy）特性 color:yellow %}
当期约进入 **fulfilled** 状态时，  
相关异步处理程序不会立即执行，仅仅会被 **排期** ，  
其后的同步代码一定会在它之前执行。  
```javascript
// 创建解决的期约
let p = Promise.resolve();

// 添加解决处理程序
p.then(() => console.log(`onResolved handler`));

// 同步操作
console.log('then() returns');

// then() returns
// onResolved handler
```

{% folding 先添加处理程序后解决期约。 color:blue %}
```javascript
let synchronousResolve;

// 创建一个期约并将解决函数保存在一个局部变量中
let p = new Promise((resolve) => {
  synchronousResolve = function () {
    console.log('1');
    resolve();
    console.log('2');
  };
});
p.then(() => console.log('4'));
synchronousResolve();
console.log('3'); //1 2 3 4
```
{% endfolding %}

{% folding 非重入适用范围。 color:blue %}
- onResolved/onRejected
- catch
- finally

```javascript
let p1 = Promise.resolve();
p1.then(() => console.log('p1.then() onResolved'));
console.log('p1.then() returns');

let p2 = Promise.reject();
p2.then(null, () => console.log('p2.then() onRejected'));
console.log('p2.then() returns')

let p3 = Promise.reject();
p3.catch(() => console.log('p3.catch() onRejected'));
console.log('p3.catch() returns')

let p4 = Promise.resolve();
p4.finally(() => console.log('p4.finally() onFinally'));
console.log('p4.finally() returns')

// p1.then() returns
// p2.then() returns
// p3.catch() returns
// p4.finally() returns
// p1.then() onResolved
// p2.then() onRejected
// p3.catch() onRejected
// p4.finally() onFinally
```
{% endfolding %}



{% endborder %}

<!-- folder 6. 邻近处理程序的执行顺序 -->
{% border 同一期约多个处理程序按照添加顺序执行 color:yellow %}
```javascript
let p1 = Promise.resolve();
let p2 = Promise.reject();

p1.then(() => setTimeout(console.log, 0, 1));
p1.then(() => setTimeout(console.log, 0, 2));
//1 2

p2.then(null,() => setTimeout(console.log, 0, 3));
p2.then(null, () => setTimeout(console.log, 0, 4));
// 3 4

p2.catch(() => setTimeout(console.log, 0, 5));
p2.catch(() => setTimeout(console.log, 0, 6));
// 5 6

p1.finally(() => setTimeout(console.log, 0, 7));
p1.finally(() => setTimeout(console.log, 0, 8));
// 7 8
```
{% endborder %}

<!-- folder 7. 传递解决值和拒绝理由 -->
{% border 在进入fulfilled状态后， color:yellow %}
期约会提供value/reason给相关状态的处理程序。  
```javascript
let p1 = new Promise((resolve, reject) => resolve('foo'));
p1.then((value) => console.log(value)); //foo

let p2 = new Promise((resolve, reject) => reject('bar'));
p2.catch((reason) => console.log(reason)); //bar

let p3 = Promise.resolve('foo');
p3.then((value) => console.log(value)); //foo

let p4 = Promise.reject('bar');
p4.catch((reason) => console.log(reason)); //bar
```
{% endborder %}

<!-- folder 8. 拒绝期约与拒绝错误处理 -->

{% border 在期约执行/处理时抛出错误会导致拒绝 color:yellow %}
```javascript
let p1 = new Promise((resolve, reject) => reject(Error('foo')));
let p2 = new Promise((resolve, reject) => { throw Error('foo') });
let p3 = Promise.resolve().then(() => { throw Error('foo') });
let p4 = Promise.reject(Error('foo'));

setTimeout(console.log, 0, p1); //Promise {<rejected>: Error:foo}
setTimeout(console.log, 0, p2); //Promise {<rejected>: Error:foo}
setTimeout(console.log, 0, p3); //Promise {<rejected>: Error:foo}
setTimeout(console.log, 0, p4); //Promise {<rejected>: Error:foo}
```

{% folding 异步错误的副作用 color:blue %}
正常情况下抛出的错误会阻塞后续指令的执行：
```javascript
// Uncaught Error: foo
throw Error('foo');
console.log('bar'); // 无效代码
```

期约中抛出的错误不会阻塞后续同步指令的执行：
```javascript
// Error: foo
// bar
Promise.reject(Error('foo'));
console.log('bar');
```

{% endfolding %}

{% folding 对比同步错误与异步错误处理 color:blue %}

{% split bg:card %}
<!-- cell left -->
同步错误处理
```javascript
// 1
// 3 Error:2
// 4
console.log('1');
try {
  throw Error('2');
} catch (e) { 
  console.log('3',e);
}
console.log('4'); 
```
<!-- cell right -->
异步错误处理
```javascript
// 1
// 3 Error:2
// 4
new Promise((resolve, reject) => {
  console.log('1');
  reject(Error('2'));
}).catch((e) => {
  console.log('3', e);
}).then(() => {
  console.log('4');
})
```
{% endsplit %}

{% endfolding %}

{% endborder %}

{% endfolders %}

# 11.2.4 期约连锁与期约合成

{% split bg:card %}
<!-- cell left -->
{% border 期约连锁 %}
一个期约接一个期约的拼接。
{% endborder %}
<!-- cell right -->
{% border 期约合成 %}
将多个期约组合为一个期约。
{% endborder %}
{% endsplit %}

{% folders %}
<!-- folder 1. 期约连锁 -->
{% border color:yellow %}
通过连缀方法调用就可以构成所谓的 **期约连锁**：
```javascript
let p = new Promise((resolve, reject) => {
  console.log('first');
  resolve();
});
// first second third fourth
p.then(() => console.log('second'))
  .then(() => console.log('third'))
  .then(() => console.log('fourth'));
```

{% folding 串行化异步任务 color:blue %}
每个执行器都返回一个期约实例，  
每个后续期约都等待之前的期约。  
```javascript
let p1 = new Promise((resolve, reject) => {
  console.log('p1 executor');
  setTimeout(resolve, 1000);
})
p1.then(() => new Promise((resolve, reject) => {
  console.log('p2 executor');
  setTimeout(resolve, 1000);
})).then(() => new Promise((resolve, reject) => {
  console.log('p3 executor');
  setTimeout(resolve, 1000);
})).then(() => new Promise((resolve, reject) => {
  console.log('p4 executor');
  setTimeout(resolve, 1000);
}));
// p1 executor
// p2 executor
// p3 executor
// p4 executor
```
{% endfolding %}

{% folding 将生成期约代码提取成工厂函数 color:blue %}
```javascript
function delayedResolve(str) { 
  return new Promise((resolve, reject) => {
    console.log(str);
    setTimeout(resolve, 1000);
  })
}

delayedResolve('p1 executor')
  .then(() => delayedResolve('p2 executor'))
  .then(() => delayedResolve('p3 executor'))
  .then(() => delayedResolve('p4 executor'));
// p1 executor
// p2 executor
// p3 executor
// p4 executor
```
{% endfolding %}

{% folding 使用回调函数重写连锁期约（回调地狱） color:blue %}
```javascript
function delayedExecute(str, callback = null) {
  setTimeout(() => {
    console.log(str);
    callback && callback();
  }, 1000);
}
delayedExecute('p1 callback', () => {
  delayedExecute('p2 callback', () => {
    delayedExecute('p3 callback', () => {
      delayedExecute('p4 callback');
    })
  })
});
// p1 callback
// p2 callback
// p3 callback
// p4 callback
```
{% endfolding %}

{% folding 串联期约的相关方法 color:blue %}
```javascript
let p = new Promise((resolve, reject) => {
  console.log('initial promise rejects');
  reject();
});

p.catch(() => console.log('reject handler'))
  .then(() => console.log('resolve handler'))
  .finally(() => console.log('finally handler'));

// initial promise rejects
// reject handler
// resolve handler
// finally handler
```
{% endfolding %}

{% endborder %}
<!-- folder 2. 期约图 -->
{% border color:yellow %}
期约连锁的结构类似于 {% mark 有向非循环图 color:purple %}：
- **一对多关系：** 一个期约可以有任意多个处理程序
- **节点：** 组成连锁期约的每一个期约
- **有向顶点：** 使用实例添加的处理程序
- **层序遍历：** 期约处理程序是按照添加顺序执行的

```javascript
//     A
//    / \
//   B   C
//  / \ / \
// D  E F  G

let A = new Promise((resolve, reject) => {
  console.log('A');
  resolve();
});
let B = A.then(() => console.log('B'));
let C = A.then(() => console.log('C'));
B.then(() => console.log('D'));
B.then(() => console.log('E'));
C.then(() => console.log('F'));
C.then(() => console.log('G'));
// A B C D E F G
```
{% endborder %}
<!-- folder 3. Promise.all()和Promise.race() -->

{% border 合成期约方法 color:yellow child:tabs %}

{% tabs %}
<!-- tab Promise.all() -->
{% border %}
- **功能：**  创建一个期约，该期约会在一组期约全部解决后再解决
    - 存在一个待定期约，合成期约也待定
    - 存在一个拒绝期约，合成期约也拒绝
- **参数：** 可迭代对象
- **返回值：** 一个新期约
    - **合成期约解决值：** 所有包含期约解决值的数组，按照迭代器顺序
    - **合成期约拒绝理由：** 即第一个拒绝的期约的理由

{% folding  Promise.all()参数 color:blue %}
```javascript
let p1 = Promise.all([
  Promise.resolve(),
  Promise.resolve()
]);

// 可迭代对象中的元素会通过Promise.resolve()转换为期约
let p2 = Promise.all([3, 4]);

// 空的可迭代对象等价于Promise.resolve()
let p3 = Promise.all([]);

// 无效语法
// TypeError: undefined is not iterable 
let p4 = Promise.all();
```
{% endfolding %}

{% folding 合成期约在每个期约都解决后才解决 color:blue %}
```javascript
let p = Promise.all([
  Promise.resolve(),
  new Promise((resolve, reject) => setTimeout(resolve, 1000))
]);
setTimeout(console.log, 0, p); //Promise {<pending>}
// after 1s: all() resolved! 
p.then(() => setTimeout(console.log, 0, 'all() resolved!'));
```
{% endfolding %}

{% folding 合成期约的pendinng和rejected条件 color:blue %}
```javascript
let p1 = Promise.all([new Promise(() => { })]);
setTimeout(console.log, 0, p1); // Promise {<pending>}

let p2 = Promise.all([
  Promise.resolve(),
  Promise.reject(),
  Promise.resolve()
]);
setTimeout(console.log, 0, p2); // Promise {<rejected>}
```
{% endfolding %}

{% folding 合成期约的解决值 color:blue %}
```javascript
let p = Promise.all([
  Promise.resolve(3),
  Promise.resolve(),
  Promise.resolve(4)
]);
p.then((value) => setTimeout(console.log,0,value)); // [3,undefined,4]
```
{% endfolding %}

{% folding 合成期约会静默处理所有包含期约的拒绝操作 color:blue %}
```javascript
let p = Promise.all([
  Promise.reject(3),
  new Promise((resolve, reject) => setTimeout(reject, 1000))
])

p.catch((reason) => setTimeout(console.log, 0, reason)); //3
```
{% endfolding %}

{% endborder %}
<!-- tab Promise.race() -->
{% border %}
- **功能：**  返回一个包装期约，该期约是一组集合中最先解决/拒绝的期约的镜像
    - 不会对解决或拒绝的期约区别对待
    - 会静默处理所有包含期约的拒绝操作
- **参数：** 可迭代对象
- **返回值：** 一个新期约

{% folding Promise.race()参数 color:blue %}
```javascript
let p1 = Promise.race([
  Promise.resolve(),
  Promise.resolve()
]);

// 可迭代对象中的元素会通过Promise.resolve转换为期约
let p2 = Promise.race([3, 4]);

// 空的迭代对象等价于new Promise(()=>{})
let p3 = Promise.race([]);

// 无效语法
// TypeError: undefined is not iterable
let p4 = Promise.race();
```
{% endfolding %}

{% folding Promise.race()返回值 color:blue %}
```javascript
// 先解决
let p1 = Promise.race([
  Promise.resolve(3),
  new Promise((resolve, reject) => setTimeout(reject, 1000))
]);
setTimeout(console.log, 0, p1); //Promise {<fulfilled>:3}

// 先拒绝
let p2 = Promise.race([
  Promise.reject(4),
  new Promise((resolve, reject) => setTimeout(resolve, 1000))
]);
setTimeout(console.log, 0, p2); //Promise {<rejected>:4}

// 迭代顺序决定落定顺序
let p3 = Promise.race([
  Promise.resolve(5),
  Promise.resolve(6),
  Promise.reject(7),
]);
setTimeout(console.log, 0, p3); //Promise {<fulfilled>:5}
```
{% endfolding %}

{% folding Promise.race()对拒绝操作的静默处理 color:blue %}
```javascript
let p = Promise.race([
  Promise.reject(3),
  new Promise((resolve, reejct) => setTimeout(reject, 1000))
]);
p.catch((reason) => setTimeout(console.log, 0, reason)); //3
```
{% endfolding %}
{% endborder %}

{% endtabs %}

{% endborder %}

<!-- folder 4. 串行期约合成 -->

{% border 基于期约的一个主要特性： color:yellow %}
异步产生值并将其传给处理程序。

{% folding 函数合成 color:blue %}
```javascript
function addTwo(x) { return x + 2; }
function addThree(x) { return x + 3; }
function addFive(x) { return x + 5; }
function addTen(x) {
  return addFive(addThree(addTwo(x)));
}
console.log(addTen(10)); //20
```
{% endfolding %}

{% folding 使用期约重现函数合成 color:purple %}
```javascript
function addTwo(x) { return x + 2; }
function addThree(x) { return x + 3; }
function addFive(x) { return x + 5; }
function addTen(x) {
  return Promise.resolve(x)
    .then(addTwo)
    .then(addThree)
    .then(addFive);
}
addTen(8).then(console.log); //18
```
{% endfolding %}

{% folding 使用Array.prototype.reduce()再简化 color:blue %}
```javascript
function addTwo(x) { return x + 2; }
function addThree(x) { return x + 3; }
function addFive(x) { return x + 5; }
function addTen(x) {
  return [addTwo, addThree, addFive]
    .reduce(
      (promise, fn) => promise.then(fn),
      Promise.resolve(x)
    );
}
addTen(5).then(console.log); //15
```
{% endfolding %}

{% folding 提炼通用合成函数 color:purple %}
```javascript
function addTwo(x) { return x + 2; }
function addThree(x) { return x + 3; }
function addFive(x) { return x + 5; }

function compose(...fns) {
  return (x) =>
    fns.reduce(
      (promise, fn) => promise.then(fn),
      Promise.resolve(x)
    )
}
compose(
  addTwo,
  addFive,
  addThree
)(11).then(console.log); //21
```
{% endfolding %}

{% endborder %}

{% endfolders %}

# 11.2.5 期约扩展

{% border 为什么ES6不支持取消期约和进度通知？ color:yellow %}
一个主要原因就是这样会导致 **期约连锁** 和 **期约合成** 过度复杂化。
{% endborder %}

{% note 下面涉及到两个第三方期约库中具备但是ECMAScript未涉及的特性。 color:blue %}

{% border child:tabs color:yellow %}

{% tabs %}
<!-- tab 期约取消 -->

{% border %}
一般针对于期约正在处理过程中，程序不再需要结果的情形。

{% border ES6被认为是“激进的” color:red %}
因为期约的逻辑只要开始执行，就无法阻止其执行到完成。
{% endborder %}

{% border 取消令牌（cancel token） color:blue %}
提供一种临时性的封装。  
生成的令牌实例提供一个接口，可以用于取消期约。  
同时也提供一个期约实例，用来触发取消后操作并求值取消状态。
```javascript
class CancelToken { 
  constructor(cancelFn) {
    this.promise = new Promise((resolve, reject) => {
      cancelFn(resolve);
    })
  }
}
```
{% endborder %}

{% folding 取消令牌案例 color:blue %}
{% border 存在问题： color:red %}
尽管本案例的思路是在触发开始按钮的时候，  
对取消按钮进行事件绑定。  
但是由于没有手动进行事件解绑，  
因此导致取消按钮事件会不断堆叠。  
**解决方法：**  
在添加事件监听器时将option.once配置为true，表示仅触发一次事件。  
```javascript
cancelButton.addEventListener('click',cancelCallback,{once:true});
```
{% endborder %}

```html
  <button id="start">Start</button>
  <button id="cancel">Cancel</button>
```

```javascript
class CancelToken { 
  constructor(cancelFn) { 
    this.promise = new Promise((resolve, reject) => {
      cancelFn(() => { 
        setTimeout(console.log, 0, 'delay cancelled');
        resolve();
      })
    })
  }
}

const startButton = document.querySelector('#start');
const cancelButton = document.querySelector('#cancel');

function cancellableDelayedResolve(delay) {
  setTimeout(console.log, 0, 'set delay');
  return new Promise((resolve, reject) => { 
    const id = setTimeout(() => {
      setTimeout(console.log, 0, 'delayed resolve');
      resolve();
    }, delay);
    const cancelToken = new CancelToken(
      cancelCallback =>
        cancelButton.addEventListener('click', cancelCallback)
    );
    cancelToken.promise.then(() => clearTimeout(id));
  })
}
startButton.addEventListener('click', () => cancellableDelayedResolve(1000));
```
{% endfolding %}

{% folding 简化版取消期约 color:blue %}

{% border 实现思路 %}
相比取消令牌方法更加简单粗暴，  
即是将开始按钮触发事件包装成异步操作，  
给取消按钮添加提前resolve事件， 
在期约进入resolve状态时对取消按钮进行解绑。  
**相较官方案例扩展度更低，但简单粗暴。**
{% endborder %}

```javascript
function cancellableDelayedResolve(delay) {
  setTimeout(console.log, 0, 'start');
  let cancelCallback;
  new Promise((resolve, reject) => {
    let id = setTimeout(() => {
      console.log('finished');
      resolve();
    }, 1500);
    cancelCallback = () => {
      console.log('cancel');
      clearTimeout(id);
      resolve();
    }
    cancelButton.addEventListener('click',cancelCallback);
  }).finally(() => { 
    cancelButton.removeEventListener('click', cancelCallback);
  })
}
startButton.addEventListener('click', cancellableDelayedResolve);
```

{% endfolding %}

{% folding async/await实现取消期约 color:blue %}
```javascript
let startButton = document.querySelector('#start');
let cancelButton = document.querySelector('#cancel');

async function cancellableDelayedResolve(delay) {
  setTimeout(console.log, 0, 'start');
  let id = setTimeout(console.log, delay, await 'finished');
  cancelButton.addEventListener('click', () => {
    clearTimeout(id);
    console.log('cancel')
  }, { once: true });
}
startButton.addEventListener('click', () => cancellableDelayedResolve(1000));
```
{% endfolding %}

{% endborder %}
 
<!-- tab 进度追踪 -->
{% border  有种实现期约进度追踪的方法： %}
扩展Promise类，添加 **notify()** 方法：
```javascript
class TrackablePromise extends Promise { 
  constructor(executor) {
    const notifyHandlers = [];
    super((resolve, reject) => {
      return executor(resolve, reject, (status) => {
        notifyHandlers.map((handler) => handler(status));
      });
    });
    this.notifyHandlers = notifyHandlers;
  }
  notify(notifyHandler) {
    this.notifyHandlers.push(notifyHandler);
    return this;
  }
}

let p = new TrackablePromise((resolve, reject, notify) => {
  function countdown(x) {
    if (x > 0) {
      notify(`${20 * x}% remaining`);
      setTimeout(() => countdown(x - 1), 1000);
    } else {
      resolve();
    }
  }
  countdown(5);
});

p.notify((x) => setTimeout(console.log, 0, 'a:', x))
  .notify((x) => setTimeout(console.log, 0, 'b:', x));
p.then(() => setTimeout(console.log, 0, 'completed'));

// a: 80% remaining
// b: 80% remaining
// a: 60% remaining
// b: 60% remaining
// a: 40% remaining
// b: 40% remaining
// a: 20% remaining
// b: 20% remaining
// completed
```
{% endborder %}
{% endtabs %}

{% endborder %}
