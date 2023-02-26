---
title: 面向牛客学习
date: 2023-02-24 21:22:00
tags:
- 牛客
categories:
- 面试准备
---

持续更新，希望能找到实习，内容大都来自互联网，如有侵权，请联系我删除。
<!--more-->


### 滚动列表优化

不能使用分页方式来加载列表数据的业务情况，对于此，我们称这种列表叫做`长列表`

页面渲染会耗费资源，在计算样式和布局耗费巨大

- 分片渲染（通过浏览器事件环机制，也就是 EventLoop，分割渲染时间）

  【分片渲染】 启用使用API setTimeout 分片渲染 每次渲染50条，进入宏任务列队进行页面渲染以提高效率。（最好用requestAnimationFrame）setTimeout会出现白屏闪烁

  其思想是**建立一个队列，通过定时器来进行渲染**，比如说一共有3次，先把这三个放入到数组中，当第一个执行完成后，并剔除执行完成的，在执行第二个，直到全部执行完毕，渲染队列清空。

- 虚拟列表（只渲染可视区域）

  只对`可见区域`进行渲染，对`非可见区域`中的数据不渲染或部分渲染的技术，从而达到极高的渲染性能。

  > 长列表的渲染有以下几种情况:
  >  1、列表项高度固定，且每个列表项高度相等
  >  2、列表项高度固定不相等，但组件调用方可以明确的传入形如(index: number)=>number的getter方法去指定每个索引处列表项的高度
  >  3、列表项高度不固定，随内容适应，且调用方无法确定具体高度

  虚拟列表的实现，实际上就是在首屏加载的时候，只加载`可视区域`内需要的列表项，当滚动发生时，动态通过计算获得`可视区域`内的列表项，并将`非可视区域`内存在的列表项删除。

  设置一个视图容器，用来展示可视区域的列表，内部设置两个div，一个用来撑大容器，高度是总列表数*列表高度，一个div用来存放列表，计算出totalHeight撑起容器，并在滚动事件触发时根据scrollTop值不断更新startIndex以及endIndex，以此从列表数据listData中截取元素。

    - 计算当前`可视区域`起始数据索引(`startIndex`)
    - 计算当前`可视区域`结束数据索引(`endIndex`)
    - 计算当前`可视区域的`数据，并渲染到页面中
    - 计算`startIndex`对应的数据在整个列表中的偏移位置`startOffset`并设置到列表上

  对于高度固定不相等的情况，维护一个positions数组来存储各列表位置信息，可以设置一个估计高度传过去进行初始化，在渲染完成后再获取列表每项的位置信息并缓存，由于缓存的position的top值是有顺序的，可以用二分查找的方法来得到偏移值。

  **时间分片、虚拟长列表、分页就是为了解决上述问题，将渲染过程分割成一个个的小任务，每隔一段时间，浏览器就渲染一部分DOM，避免渲染线程长时间执行导致其他互斥进程得不到执行机会；**

### 浏览器渲染时机

- 在 JS 的`Event Loop`中，当JS引擎所管理的执行栈中的事件以及所有微任务事件全部执行完后，才会触发渲染线程对页面进行渲染

- `setTimeout`的执行时间并不是确定的。在JS中，`setTimeout`任务被放进事件队列中，只有主线程执行完才会去检查事件队列中的任务是否需要执行，因此`setTimeout`的实际执行时间可能会比其设定的时间晚一些。

  刷新频率受屏幕分辨率和屏幕尺寸的影响，因此不同设备的刷新频率可能会不同，而`setTimeout`只能设置一个固定时间间隔，这个时间不一定和屏幕的刷新时间相同。

### DocumentFragment

从MDN的说明中，我们得知`DocumentFragments`是DOM节点，但并不是DOM树的一部分，可以认为是存在内存中的，所以将子元素插入到文档片段时不会引起页面回流。

当`append`元素到`document`中时，被`append`进去的元素的样式表的计算是同步发生的，此时调用 getComputedStyle 可以得到样式的计算值。 而`append`元素到`documentFragment` 中时，是不会计算元素的样式表，所以`documentFragment` 性能更优。当然现在浏览器的优化已经做的很好了， 当`append`元素到`document`中后，没有访问 getComputedStyle 之类的方法时，现代浏览器也可以把样式表的计算推迟到脚本执行之后。

### token过期重新登录

### 基本类型

number string boolen null undefined bigint symbol

### 检测数据类型的方法

- typeof 检测基本类型和function 但是不能检测null（底部机器码）引用类型不能检测

- instanceof 检测引用类型 原理是沿着原型链向上查找（所以也可以用来判断一个实例是否是其父类型或祖先类型的实例）

#### 手写instanceof（左值是实例，右值是类）

```js
function new_instance_of(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
// 其实 instanceof 主要的实现原理就是只要右边变量的 prototype 在左边变量的原型链上即可。因此，instanceof 在查找的过程中会遍历左边变量的原型链，直到找到右边变量的 prototype，如果查找失败，则会返回 false，告诉我们左边变量并非是右边变量的实例。
```

#### 如何给typeof拓展instanceof

- Object.prototype.toString.call()

### MVVM MVC MVP

***MVC---Model-View-Controller***

##### MVC各个部分之间的通信：M->V->C->M之间单向通信

m:model数据模型层 v:view 视图层 c:controller控制器

view传送指令到c层，c层完成业务逻辑后，要求model改变状态，model将新的数据发送到 View

也可以用户直接操纵c层

痛点问题：

1、 开发者在代码中大量调用相同的 DOM API，处理繁琐 ，操作冗余，使得代码难以维护。

2、大量的DOM 操作使页面渲染性能降低，加载速度变慢，影响用户体验。

3、 当 Model 频繁发生变化，开发者需要主动更新到View ；当用户的操作导致 Model 发生变化，开发者同样需要将变化的数据同步到Model 中，这样的工作不仅繁琐，而且很难维护复杂多变的数据状态。

***MVVM---Model-View-ViewModel***

Model 层代表数据模型，也可以在Model中定义数据修改和操作的业务逻辑

**在MVVM架构下，View 和 Model 之间并没有直接的联系，而是通过ViewModel进行交互，Model 和 ViewModel 之间的交互是双向的， 因此View 数据的变化会同步到Model中，而Model 数据的变化也会立即反应到View 上。**

ViewModel 通过双向数据绑定把 View 层和 Model 层连接了起来，而View 和 Model 之间的同步工作完全是自动的，无需人为干涉，因此开发者只需关注业务逻辑，不需要手动操作DOM， 不需要关注数据状态的同步问题，复杂的数据状态维护完全由 MVVM 来统一管理。

***MVP---model-view-presenter***

<img src="https://images2015.cnblogs.com/blog/1017946/201707/1017946-20170708121332565-769447954.png" alt="image" style="zoom: 50%;" />

view层和model层不相互通信，其他各部分通信是双向的，View 非常薄，不部署任何业务逻辑，称为"被动视图"（Passive View），即没有任何主动性，而 Presenter非常厚，所有逻辑都部署在那里。

### Vue的响应式原理
vue2中，我们通过`Object.defineProperty`为对象`obj`添加属性，可以设置对象属性的`getter`和`setter`函数。之后我们每次通过点语法获取属性都会执行这里的`getter`函数，在这个函数中我们会把调用此属性的依赖收集到一个集合中 ；而在我们给属性赋值(修改属性)时，会触发这里定义的`setter`函数，在次函数中会去通知集合中的依赖更新，做到数据变更驱动视图变更。

3.x的与2.x的核心思想一致，只不过数据的劫持使用`Proxy`而不是`Object.defineProperty`，只不过Proxy相比Object.defineProperty在处理数组和新增属性的响应式处理上更加方便。

Vue响应原理的核心是**数据劫持和依赖收集**，主要是利用`Object.defineProperty()`或者`Proxy`实现对数据存取操作的拦截，我们把这个实现称为数据代理；同时我们通过对数据get方法的拦截，可以获取到对数据的依赖，并将出所有的依赖收集到一个集合中。

#### proxy支持的拦截操作

- **get(target, propKey, receiver)**：拦截对象属性的读取，比如`proxy.foo`和`proxy['foo']`。

- **set(target, propKey, value, receiver)**：拦截对象属性的设置，比如`proxy.foo = v`或`proxy['foo'] = v`，返回一个布尔值。

- **has(target, propKey)**：拦截`propKey in proxy`的操作，返回一个布尔值。

  `has()`方法用来拦截`HasProperty`操作，即判断对象是否具有某个属性时，这个方法会生效。典型的操作就是`in`运算符。值得注意的是，`has()`方法拦截的是`HasProperty`操作，而不是`HasOwnProperty`操作，即`has()`方法不判断一个属性是对象自身的属性，还是继承的属性。也不会拦截for...in...操作

- **deleteProperty(target, propKey)**：拦截`delete proxy[propKey]`的操作，返回一个布尔值。

- **ownKeys(target)**：拦截`Object.getOwnPropertyNames(proxy)`、`Object.getOwnPropertySymbols(proxy)`、`Object.keys(proxy)`、`for...in`循环，返回一个数组。



拦截`Object.keys()`的返回结果仅包括目标对象自身的可遍历属性。过滤以下三个属性：

- 目标对象上不存在的属性
- 属性名为 Symbol 值
- 不可遍历（`enumerable`）的属性

拦截`for..in..`也必须返回存在的属性

拦截`Object.getOwnPropertyNames`不受影响

`ownKeys()`方法返回的数组成员，只能是字符串或 Symbol 值。如果有其他类型的值，或者返回的根本不是数组，就会报错。

如果目标对象包含不可配置的属性，则该属性必须被返回，如果目标对象是不可拓展的，返回的数组中必须包含原对象的所有属性，且不能包含多余属性



- **getOwnPropertyDescriptor(target, propKey)**：拦截`Object.getOwnPropertyDescriptor(proxy, propKey)`，返回属性的描述对象。

- **defineProperty(target, propKey, propDesc)**：拦截`Object.defineProperty(proxy, propKey, propDesc）`、`Object.defineProperties(proxy, propDescs)`，返回一个布尔值。

- **preventExtensions(target)**：拦截`Object.preventExtensions(proxy)`，返回一个布尔值。

- **getPrototypeOf(target)**：拦截`Object.getPrototypeOf(proxy)`，返回一个对象。

- **isExtensible(target)**：拦截`Object.isExtensible(proxy)`，返回一个布尔值。

- **setPrototypeOf(target, proto)**：拦截`Object.setPrototypeOf(proxy, proto)`，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

- **apply(target, object, args)**：拦截 Proxy 实例作为函数调用的操作，比如`proxy(...args)`、`proxy.call(object, ...args)`、`proxy.apply(...)`。

  `apply`方法可以接受三个参数，分别是目标对象、目标对象的上下文对象（`this`）和目标对象的参数数组。

- **construct(target, args)**：拦截 Proxy 实例作为构造函数调用的操作，比如`new proxy(...args)`。

#### Proxy.revocable()

`Proxy.revocable()`方法返回一个可取消的 Proxy 实例。

```javascript
let target = {};
let handler = {};

let {proxy, revoke} = Proxy.revocable(target, handler);

proxy.foo = 123;
proxy.foo // 123

revoke();
proxy.foo // TypeError: Revoked
```

`Proxy.revocable()`方法返回一个对象，该对象的`proxy`属性是`Proxy`实例，`revoke`属性是一个函数，可以取消`Proxy`实例。上面代码中，当执行`revoke`函数之后，再访问`Proxy`实例，就会抛出一个错误。

`Proxy.revocable()`的一个使用场景是，目标对象不允许直接访问，必须通过代理访问，一旦访问结束，就收回代理权，不允许再次访问。

#### 如何实现数组读取负数的索引呢？

```js
function createArray(...elements) {
  let handler = {
    get(target, propKey, receiver) {
      let index = Number(propKey);
      if (index < 0) {
        propKey = String(target.length + index); // 核心
      }
      return Reflect.get(target, propKey, receiver);
    }
  };

  let target = [];
  target.push(...elements);
  return new Proxy(target, handler);
}

let arr = createArray('a', 'b', 'c');
arr[-1] // c
```

如何实现不允许修改对象的_开头的属性不可被修改呢，设置set拦截

#### Object.keys,Object.getOwnPropertyNames,**Reflect.ownKeys(obj)**和for...in.. 在遍历对象有什么区别

Object.keys

- 返回对象自身**可枚举属性**组成的数组
- 不会遍历对象原型链上的属性以及 Symbol 属性
- 对数组的遍历顺序和 for in 一致
- 不包含继承的属性方法

Object.getOwnPropertyNames

- 返回一个由指定对象的所有自身属性的属性名（**包括不可枚举属性但不包括Symbol值作为名称的属性**）组成的数组。

Reflect.ownKeys

- 返回一个数组，包含对象自身的所有属性，不管是属性名是Symbol或字符串，也不管是否可枚举。

for in

- 遍历**对象及其原型链**上可枚举的属性；
- 如果用于遍历数组，处理遍历其元素外，还会遍历开发者对数组对象自定义的可枚举属性及其原型链上的可枚举属性；
- 遍历对象返回的属性名和遍历数组返回的索引都是 string 类型；
- 某些情况下，可能按随机顺序遍历数组元素

如果for..in.. 想要过滤掉原型链上的属性，可以用**hasOwnProperty(key)**

**总结一下：只有for..in..能遍历原型链上的属性，Reflect.ownKeys能获得自身所有属性，Object.keys只能获取其可枚举属性，而Object.getOwnPropertyNames只多了不可枚举属性的获取，除了Reflect.ownKeys，其他都不能获取symbol属性，要获取symbol属性，使用Object.getOwnPropertySymbols方法**

设置属性的可枚举？

`Object.defineProperty(target,prop,{enumerable:false})`

检测可否枚举obj.propertyIsEnumerable(key)

所有 Class 的原型的方法都是不可枚举的

Object.assign()   *//会忽略enumerable为false的属性，只拷贝对象自身的可枚举的属性*

JSON.stringify()  // 会忽略不可枚举的属性



#### 对象的可拓展性

javascript 对象中的可扩展性指的是：是否可以给对象添加新属性。所有的内置对象和自定义对象显示的都是可扩展的，对于宿主对象，则有javascript 引擎决定。

**封存状态、冻结状态**

层层递进

```js
下面有几个函数是设置对象的可扩展性：

1、Object.isExtensible(Object); 检查对象是否可以扩展。

2、Object.preventExtensions(Object) 设置对象不可扩展，也就是不能添加新的属性，但如果该对象的原型，添加了新的属性，那么该对象也将继承该属性。

3、Object.seal(Object);它除了可以设置对象的不可扩展，还可以设置对象的自有属性都设置为不可配置的，不能删除和配置。对于它已经有的可写属性依然可以设置。
{ value: 0, writable: true, enumerable: true, configurable: false }
4、Object.isSealed(Object); 检查对象是否封闭。

5、Object.freeze();更严格的锁定对象（冻结）。除了将对象设置为不可扩展，属性设置为不可配置，所有的自有属性设置为只读的，（如果对象存储器属性有setter方法，存储器属性不受影响，依然可以通过属性赋值给他们）。
{ value: 0, writable: false, enumerable: true, configurable: false }
6、Object.isFrozen() 来检测对象是否冻结。
```

Object.preventExtensions()，Object.seal()，Object.freeze()都是不可逆的操作

**另外对象除了有普通的属性外还有一种叫访问器属性的东西（get、set），这种访问器属性没有value和writable特性**

#### 对象的属性的四个特性

**属性值（value）：该属性的值；**

**可写（writable）：是否可以修改该属性的值;**

**可枚举（enumerable）：是否可以通过for/in循环或者Object.keys()方法枚举该属性；**

**可配置（configurable）：是否可以删除该属性、是否可以修改该属性的特性；**

Object.getOwnPropertyDescriptor(obj,prop) 查询对象某属性的特性

如果writable为false**并且configurable为true**，不能通过obj['a']或obj.a的方式修改值，但仍然可以用

Object.defineProperty(obj,'a',{value:**100**})这样来修改

**属性不可配置时，不能再将该属性该为可配置（所以不可配置是不可逆的）**，**属性不可配置时不能删除该属性（不报错但不成功）**

如果configurable和writable均为false，则不可修改value和writable属性

#### proxy代理web的原理？

### nextTick 原理

#### 为什么要用nextTick?

**如果有频繁的数据更新导致dom的更新，这样的性能成本是昂贵的，开发中应当减少dom操作，应该在数据变化之后不是立刻执行dom更新，而是要把数据变化的动作缓存起来，在合适的时机执行dom的更新操作，这里就需要设置一个合适的时间间隔**

数据变化缓存可以依赖事件循环来完成；因为每次事件循环之间都有一次视图渲染，我们只需要在render之前完成对dom的更新即可，因此我们为了避免无效的DOM操作，需要将数据变更缓存起来，只保存最后一次数据最终的变更结果

当数据变更时，会触发Watcher对象的update方法，但是此时我们不再直接在update中触发run方法执行更新，而是把这个变更的Watcher保存到一个待更新的队列（数组实现）中（**异步更新队列**），同时我们为这个待更新的队列**创建一个微任务**来执行它里面保存的更新。

```js
let callbacks=[];//事件队列,包含异步dom更新队列和用户添加的异步事件
let pending=false;//控制变量，每次宏任务期间执行一次flushCallbacks清空callbacks
funciton nextTick(cb){
   callbacks.push(cb);
   if(!pending){
      pending=true;
      //这里也可以使用Promise，Promise创建的是微任务，微任务会在本次事件循环同步代码执行结束后执行，使用setTimeout创建的是宏任务，同样会在此次同步代码执行完成后执行，区别是在setTimeout代码执行之前会穿插一次无效的视图渲染，因此我们尽量使用Promise创建微任务实现异步更新。
      if(Promise){
          Promise.resovle().then(()=>{
              flushCallbacks();
          })
      }else{
          setTimeout(()=>{
              flushCallbacks();
          })
      }
   }
}
function flushCallbacks(){
    pending=false;//状态重置
    callbacks.forEach(cb=>{
        callbacks.shift()(this); // callbacks:[flushUpdateQueue方法,用户的异步事件]
    })			// 要取出第一个方法执行，this是传递的vue实例
}
```

```js
flushUpdateQueue(vm){
    while(updateQueue.length!=0){
        updateQueue.shift().run();
    }
    has={};
    vm.waiting=false; // 在添加的时候设置true，在清空异步队列后设置为false，保证只添加一次
}
// updateQueue是全局的
```

同时设置wait的flag，每次事件循环只添加一次flushUpdateQueue事件进callbacks队列

添加has对象保证不添加重复的watcher对象到异步队列中

添加pendding使每次事件循环只执行一次flushCallbacks

### 浏览器内核

简单来说浏览器内核是通过取得页面内容、整理信息（应用CSS）、计算和组合最终输出可视化的图像结果，通常也被称为渲染引擎。

浏览器内核是多线程，在内核控制下各线程相互配合以保持同步，一个浏览器通常由以下常驻线程组成：

- GUI 渲染线程
- JavaScript引擎线程
- 定时触发器线程
- 事件触发线程
- 异步http请求线程

#### 1.GUI渲染线程

- 主要负责页面的渲染，解析HTML、CSS，构建DOM树，布局和绘制等。
- 当界面需要重绘或者由于某种操作引发回流时，将执行该线程。
- **该线程与JS引擎线程互斥**，当执行JS引擎线程时，GUI渲染会被挂起，当任务队列空闲时，主线程才会去执行GUI渲染。

#### 2.JS引擎线程

- 该线程当然是主要负责处理 JavaScript脚本，执行代码。
- 也是主要负责执行准备好待执行的事件，即定时器计数结束，或者异步请求成功并正确返回时，将依次进入任务队列，等待 JS引擎线程的执行。
- 当然，该线程与 GUI渲染线程互斥，当 JS引擎线程执行 JavaScript脚本时间过长，将导致页面渲染的阻塞。

#### 3.定时器触发线程

- 负责执行异步定时器一类的函数的线程，如： setTimeout，setInterval。
- 主线程依次执行代码时，遇到定时器，会将定时器交给该线程处理，当计数完毕后，事件触发线程会将计数完毕后的事件加入到任务队列的尾部，等待JS引擎线程执行。

#### 4.事件触发线程

- 主要负责将准备好的事件交给 JS引擎线程执行。

比如 setTimeout定时器计数结束， ajax等异步请求成功并触发回调函数，或者用户触发点击事件时，该线程会将整装待发的事件依次加入到任务队列的队尾，等待 JS引擎线程的执行。

#### 5.异步http请求线程

- 负责执行异步请求一类的函数的线程，如： Promise，axios，ajax等。
- 主线程依次执行代码时，遇到异步请求，会将函数交给该线程处理，当监听到状态码变更，如果有回调函数，事件触发线程会将回调函数加入到任务队列的尾部，等待JS引擎线程执行。

### 浏览器event loop

- 一开始执行栈空,我们可以把**执行栈认为是一个存储函数调用的栈结构，遵循先进后出的原则**。micro 队列空，macro 队列里有且只有一个 script 脚本（整体代码）。

- 全局上下文（script 标签）被推入执行栈，同步代码执行。在执行的过程中，会判断是同步任务还是异步任务，通过对一些接口的调用，可以产生新的 macro-task 与 micro-task，它们会分别被推入各自的任务队列里。同步代码执行完了，script 脚本会被移出 macro 队列，这个过程本质上是队列的 macro-task 的执行和出队的过程。

- 上一步我们出队的是一个 macro-task，这一步我们处理的是 micro-task。但需要注意的是：当 macro-task 出队时，任务是**一个一个**执行的；而 micro-task 出队时，任务是**一队一队**执行的。因此，我们处理 micro 队列这一步，会逐个执行队列中的任务并把它出队，直到队列被清空。

- 检查是否存在 Web worker 任务，如果有，则对其进行处理
- 上述过程循环往复，直到两个队列都清空

### Node.js运行机制

- V8引擎解析JavaScript脚本。

- 解析后的代码，调用Node API。

- libuv库负责Node API的执行。它将不同的任务分配给不同的线程，形成一个Event Loop（事件循环），以异步的方式将任务的执行结果返回给V8引擎。

- V8引擎再将结果返回给用户。

### Vue的双向数据绑定

双向数据绑定通常是指我们使用的`v-model`指令的实现，是`Vue`的一个特性，也可以说是一个`input`事件和`value`的语法糖。 `Vue`通过`v-model`指令为组件添加上`input`事件处理和`value`属性的赋值。

### Vue的一些指令

v-pre 用于跳过这个元素及其子元素的编译过程

v-cloak 用于解决模板字符在页面闪烁的问题

v-once 只渲染一次

v-memo 用于缓存一个模板的子树，该指令接收一个固定长度的数组作为依赖值进行记忆比对。如果数组中的每个值都和上次渲染的时候相同，则整个该子树的更新会被跳过

#### 自定义指令

可以在app实例上注册全局指令，也可以在组件中配置`directives`选项来注册局部指令

```js
directives: {
  focus: {
    // 指令的定义
    mounted(el) {
      el.focus()
    }
  }
}
```

有7个钩子函数，四个参数都是可选，分别是

- `el`：用于直接操作 DOM，表示指令绑定到的元素
- `binding`对象：包含以下6个属性

```bash
instance：使用指令的组件实例
value：传递给指令的值
oldValue：先前的值
arg：传递给指令的参数
modifiers：传递给指令的修饰符
dir：一个对象，其实就是注册指令时传递的配置对象
```

- `vnode`：虚拟DOM，一个真实 DOM 元素的蓝图，对应el
- `prevNode`：上一个虚拟节点

### cookie localStorage sessionStorage之间的区别

cookie数据始终在同源的http请求中携带（即使不需要），即cookie在浏览器和服务器间来回传递；cookie数据还有路径（path）的概念，可以限制cookie只属于某个路径下。存储大小限制也不同，cookie数据不能超过4k，同时因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。

1. 存储大小不同
   而sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。
2. 数据有效期不同
   sessionStorage：仅在当前浏览器窗口关闭前有效，自然也就不可能持久保持；localStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie只在设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭。
3. 作用域不同
   sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面；localStorage 在所有同源窗口中都是共享的；cookie也是在所有同源窗口中都是共享的。Web Storage 支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage 的 api 接口使用更方便。
4. Web Storage带来的好处
   1、减少网络流量：一旦数据保存在本地之后，就可以避免再向服务器请求数据，因此减少不必要的数据请求，减少数据在浏览器和服务器间不必要的来回传递。
   2、快速显示数据：性能好，从本地读数据比通过网络从服务器上获得数据快得多，本地数据可以及时获得，再加上网页本身也可以有缓存，因此整个页面和数据都在本地的话，可以立即显示。
   3、临时存储：很多时候数据只需要在用户浏览一组页面期间使用，关闭窗口后数据就可以丢弃了，这种情况使用sessionStorage非常方便

### cookie

#### 交互机制

1. 浏览器(客户端)发送一个请求到服务器
2. 服务器响应。并在HttpResponse里增加一个响应头：Set-Cookie
3. 浏览器保存此cookie在本地，然后以后每次请求都带着它，且请求头为：Cookie
4. 服务器收到请求便可读取到此Cookie，做相应逻辑后给出响应

缺省情况下，Cookie的生命周期是Session级别(会话级别)。若想用Cookie进行状态保存、资源共享，服务端一般都会给其设置一个过期时间maxAge，短则1小时、1天，长则1星期、1个月甚至永久，这就是Cookie的生命(周期)

1. maxAge > 0：cookie不仅内存里有，还会持久化到硬盘，也叫持久Cookie。这样的话即使你关机重启(甚至过几天再访问)，这个cookie依旧存在，请求时依旧会携带
2. maxAge < 0：一般值为-1，也就临时Cookie。该Cookie只在内存中有(如session级别)，一旦关闭浏览器此Cookie将不复存在。值得注意的是：若使用无痕模式访问也是不会携带此Cookie的哟
3. maxAge = 0：内存中没有，硬盘中也没有了，也就立即删除Cookie。此种case存在的唯一目的：服务浏览器可能的已存在的cookie，让其立马失效(消失)

#### 劣势

- 每次请求都会携带Cookie，这无形中增加了流量开销，这在移动端对流量敏感的场景下是不够友好的
- Http请求中Cookie均为明文传输，所以安全性成问题(除非用Https)
- Cookie有大小限制，一般最大为4kb，对于复杂的需求来讲就捉襟见肘

#### cookie跨域

Cookie是不可以跨域的，隐私安全机制禁止网站非法获取其他网站(域)的Cookie。概念上咱不用长篇大论，举个例子你应该就懂了：

淘宝有两个页面：A页面a.taotao.com/index.html和B页面b.taotao.com/index.html，默认情况下A页面和B页面的Cookie是互相独立不能共享的。若现在有需要共享(如单点登录共享token )，我们只需要这么做：将A/B页面创建的Cookie的path设置为“/”，domain设置为“.taobtao.com”，那么位于a.taotao.com和b.taotao.com域下的所有页面都可以访问到这个Cookie了。

- domain：创建此cookie的服务器主机名(or域名)，服务端设置。但是不能将其设置为服务器所属域之外的域(若这都允许的话，你把Cookie的域都设置为baidu.com，那百度每次请求岂不要“累死”)

注：端口和域无关，也就是说Cookie的域是不包括端口的

- path：域下的哪些目录可以访问此cookie，默认为/，表示所有目录均可访问此cookie

### 跨域Cookie共享的关键点

这里要讨论的是跨域中Cookie的存储问题：默认情况下，浏览器是不会去为你保存下跨域请求响应的Cookie的。具体现象是：跨域请求的Response响应了即使有Set-Cookie响应头(且有值)，浏览器收到后也是不会保存此cookie的。

要实现Cookie的跨域共享，有3个关键点：

1. 服务端负责在响应中将Set-Cookie发出来(由Access-Control-Allow-Credentials响应头决定)（允许客户端携带验证信息如cookie）
2. 浏览器端只要响应里有Set-Cookie头，就将此Cookie存储(由异步对象（XMLhttpRequest）的withCredentials属性决定
3. 浏览器端发现只要有Cookie，即使是跨域请求也将其带着(由异步对象的withCredentials属性决定)

为了满足这三个关键点，在实施层面就有三要素来指导我们开发来解决此类问题

### 如何理解 HTML 语义化？

- 让人更容易读懂（增加代码可读性）。

- 让搜索引擎更容易读懂，有助于爬虫抓取更多的有效信息，爬虫依赖于标签来确定上下文和各个关键字的权重（SEO）。

- 在没有 CSS 样式下，页面也能呈现出很好地内容结构、代码结构。

#### 触发重绘和重排

- 添加或删除可见的DOM元素

- 元素的位置发生变化

- 元素的尺寸发生变化（包括外边距、内边框、边框大小、高度和宽度等）

- 内容发生变化，比如文本变化或图片被另一个不同尺寸的图片所替代。

- 页面一开始渲染的时候（这肯定避免不了）

- 浏览器的窗口尺寸变化（因为回流是根据视口的大小来计算元素的位置和大小的）

### 最小化重绘和重排

合并改变一次处理，使用cssText `el.style.cssText +='margin:10px;'`修改css的class

当需要对dom多次修改时，可通过以下步骤

1. 使元素脱离文档流
2. 对其进行多次修改
3. 将元素带回到文档中。

有三种方式可以让DOM脱离文档流：

- 隐藏元素，应用修改，重新显示
- 使用文档片段(document fragment)在当前DOM之外构建一个子树，再把它拷贝回文档。
- 将原始元素拷贝到一个脱离文档的节点中，修改节点后，再替换原始的元素。ul.parentNode.replaceChild

```js 
const ul = document.getElementById('list');
const fragment = document.createDocumentFragment();
appendDataToElement(fragment, data);
ul.appendChild(fragment);
```

事实上对于这三种方案，现代浏览器会使用队列来存储多次修改，进行优化，我们不用优先考虑

#### 避免触发同步布局事件

大多数浏览器都会通过队列化修改并批量执行来优化重排过程。浏览器会将修改操作放入到队列里，直到过了一段时间或者操作达到了一个阈值，才清空队列。但是！**当你获取布局信息的操作的时候，会强制队列刷新**，应该避免在循环里获取offsetWidth/clientWidth等属性，这样会强制让浏览器刷新队列

#### 对于复杂动画效果,使用绝对定位让其脱离文档流

#### css3硬件加速（GPU加速）

- 使用css3硬件加速，可以让transform、opacity、filters这些动画不会引起回流重绘

- 对于动画的其它属性，比如background-color这些，还是会引起回流重绘的，不过它还是可以提升这些动画的性能。



- 如果你为太多元素使用css3硬件加速，会导致内存占用较大，会有性能问题。

- 在GPU渲染字体会导致抗锯齿无效。这是因为GPU和CPU的算法不同。因此如果你不在动画结束的时候关闭硬件加速，会产生字体模糊。
