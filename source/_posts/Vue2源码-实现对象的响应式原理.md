---
title: Vue2源码-实现对象的响应式原理
date: 2023-01-29 21:27:48
tags:
- Vue2
- 响应式原理
  categories:
- Vue2源码学习
---
## Vue2底层的一些设计
由于Vue2设计的时候没有用class，用的是构造函数function，防止原型方法耦合在一起，
将这些原型方法扩展成函数，这些函数都放在不同文件里，分别在这些函数里扩展原型方法。
```
// src/index.js
function Vue(options) {    // options是用户传入的参数
  this._init(options);     // _init是初始化的函数，该原型方法在src/init.js里
}
```
```
// src/init.js
Vue.prototype._init =function(options){  //初始化
    let vm = this        // 原型中的this都是实例
    vm.$options = options  // 将用户的选项挂载到实例上
    initState(vm) // 初始化状态
}
```
写到这问题来了，在init.js文件里并没有Vue这个构造函数，为了传入Vue,可以使用一个高阶函数，
将Vue作为参数传入进来。如下。
```
// src/index.js
import {initMixin} from "./init";
function Vue(options) {  
  this._init(options);  
}
initMixin(Vue)          //Vue作为参数传出
```
```
// src/init.js
export function initMixin(Vue){
    Vue.prototype._init =function(options){  
        let vm = this          // this指向实例
        vm.$options = options  // 将用户的选项挂载到实例上
        initState(vm)          // 初始化状态
    }
}
```
- 把options挂载在实例上，方便其他原型方法共用
- $是Vue的一个约定，表示这是Vue的私有属性

## 对象的响应式原理
### 初始化状态
initState(vm)进行初始化状态，对data、props、methods、computed、watch进行初始化，
initState里initData,initProps等操作。  
```
// src/initState.js
export function initData(vm){
    let data = vm.$options.data
    data = typeof data === 'function'? data.apply(vm) : data
    vm._data = data
    observe(data)       // 对数据进行劫持 vue2采用defineProperty
    // 将vm._data里的key 用vm来代理
    for(let key in data){
        proxy(vm,'_data',key)  // 对data里的数据进行劫持 代理
    }
}
```
- Vue2的data是一个函数，为了保证每个实例的data是独立的，所以需要用apply来改变this指向，
data()返回一个data对象，这个对象就是我们的数据对象。
- 此时挂载在实例上的data还是一个函数，不能直接获得里面的数据，便将数据对象data挂载在实例上的_data上。
- 对data数据进行劫持，实现响应式，核心实现是Object.defineProperty()方法。
- 通过vm._data.key来访问data里的数据，这样写起来比较麻烦，所以通过代理的方式，将vm._data.key代理为vm.key,核心也是Object.defineProperty()方法。

```
function proxy(vm,target,key){     
    Object.defineProperty(vm,key,{ 
        get() {
            return vm[target][key]  // vm._data.key
        },
        set(newVal) {
            vm[target][key] = newVal
        }

    })
}
```
- target代表 _data，key代表data里的各个key。
- 访问 vm.key时，实际上访问的是vm._data.key。

### 对象的响应式原理
Object.defineProperty()方法，可以给对象添加属性，或者修改对象的属性，给对象添加访问器属性，拦截对该属性的访问和设置。  
Object.defineProperty()方法接收三个参数，第一个参数是要定义属性的对象，第二个参数是要定义或修改的属性的名称，第三个参数是一个描述符对象,设置拦截。
```
// src/observe/index.js
export function observe(data){
    if(typeof data !== 'object' || data == null){
        return;
    }
    new Observer(data)
}
```
- observe方法，判断data是否是对象，如果不是对象，直接return，如果是对象，就new一个Observer实例，这个实例做数据劫持操作。
```
// src/observe/index.js
class Observer{
    constructor(data) {
        this.walk(data) 
    }
    walk(data){   
        Object.keys(data).forEach(key=>
            defineReactive(data,key,data[key])
        )
    }
}
```

- 由于Object.defineProperty()方法只能一个一个地对属性进行劫持，所以需要遍历对象，对对象的每个属性进行劫持。
- object.defineProperty只能劫持已经存在的属性,这些属性是来自data()函数的返回值,后续添加在_data上的属性是无法劫持的。

```
// src/observe/index.js
function defineReactive(obj,key,value){
    observe(value)  // 递归遍历，深度劫持
    Object.defineProperty(obj,key,{
        get(){
            return value
        },
        set(newValue){
            if(newValue !== value){
                observe(newValue)  // 如果用户将值设置为对象，需要对新值再次进行劫持
                value = newValue
            }
        }
    })
}
```
- Object.defineProperty()只能劫持一层，如果value是多层对象，需要递归遍历，深度劫持。  


如图，所有属性都被劫持。  

![image](https://user-images.githubusercontent.com/90198481/215334342-f68f4d26-dbc5-4626-b2ee-7a58cca2ac12.png)  
