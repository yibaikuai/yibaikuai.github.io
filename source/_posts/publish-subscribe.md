---
title: 手写发布订阅模式
date: 2023-02-14 17:05:00
tags:
- 发布订阅模式
categories:
- 设计模式
---
```ts
interface List {
   [key:string]:Array<Function>
}
interface EventType{
    list:List
    on:(name:string,fn:Function)=>void
    emit:(name:string,...args:any)=>void
    once:(name:string,fn:Function)=>void
    off:(name:string,fn:Function)=>void
}

class Event implements EventType{
    list:List = {}
    constructor() {
        this.list = {}
    }
    on(name:string,fn:Function){
        let funArr = this.list[name] || []
        funArr.push(fn)
        this.list[name] = funArr
    }
    emit(name:string,...args:Array<any>){
        let funArr = this.list[name]
        if(funArr){
            funArr.forEach(fn=>fn.apply(this,args))
        }else{
            console.log('未找到注册的函数名')
        }
    }
    once(name:string,fn:Function){
        let tmpFn = (...args:Array<any>)=>{
            fn.apply(args)
            this.off(name,tmpFn)
        }
        this.on(name,tmpFn)
    }
    off(name:string,fn:Function){
        let funArr = this.list[name]
        if(funArr){
            let index = funArr.findIndex(singleFn=>singleFn===fn)
            if(index>=0){
                funArr.splice(index,1)
                this.list[name] = funArr
                console.log('删除成功')
            }else{
                console.log('未找到要删除的函数')
            }
        }else{
            console.log('未找到注册的函数名')
        }
    }
}
```
需要注意的是,在off方法中比较函数是否相等，比较的是函数的地址，如果是订阅的是匿名函数将无法删除，因为匿名函数的地址是不同的，在订阅的时候应该使用命名函数。
