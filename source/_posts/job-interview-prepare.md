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

