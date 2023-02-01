---
title: js中的隐式转换
date: 2023-02-01 21:00:00
tags:
- 隐式转换
categories:
- javascript
---

## 隐式转换
在js中，当运算符在运算时，如果两边数据不统一，CPU就无法计算,编译器会自动将运算符两边的数据做一个数据类型转换，
转成一样的数据类型再计算,这种无需程序员手动转换，而由编译器自动转换的方式就称为隐式转换。

- 字符串拼接，其他所有类型都会被传唤为字符串，对于引用类型，会调用toString方法再进行拼接，
  null和undefined和NaN会被转换为字符串null和undefined
- 和数字做+运算，**基本类型**都会被转换为数字，能转化的字符串会转，不能转的变成NaN，null变成0，undefined变成NaN，**引用类型**的数据会valueOf查看原始值，不是Number类型，再调用toString方法,由于优先级字符串拼接更高，所以变成了字符串拼接，而不是数字运算
- 和数字做其他运算，**引用类型**会先valueOf看原始值，发现不是Number类型，就调用toString方法，变成字符串，看能不能转数字，不能就变NaN，
  []=>''=>0，{}=>'[object Object]'=>NaN
-  !（逻辑非运算符）发生Boolean的隐式转换,除了0、NaN、''、null、undefined、false，其他都是true，**空的数组还是true**


引用类型转换Number类型的转换步骤,图片来自 [vitamin-乐乐](https://blog.csdn.net/vita_min123/article/details/121359602?spm=1001.2101.3001.6650.5&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-5-121359602-blog-126843398.pc_relevant_default&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7EOPENSEARCH%7ERate-5-121359602-blog-126843398.pc_relevant_default&utm_relevant_index=10)

![images](https://img-blog.csdnimg.cn/99145d0f7f4149f3ad1cfbcdf9398322.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBAdml0YW1pbi3kuZDkuZA=,size_16,color_FFFFFF,t_70,g_se,x_16)

面试题
```
var a = ? ? ? ;
 
if (a == 1 && a == 2 && a == 3) {
 
console.log(1)
 
}
//给a设置内容  让他实现值同时等于1 2 3
```
重写valueOf方法
```
var a = {
    i: 1,
    valueOf: function() {
        return a.i++;
    }
}
```

## 取正负号会被当作数字运算
可以理解成全都正负号右边都必须转化为数字，不能转化就是NaN

console.log(+'a') //NaN  
console.log(-'a') //NaN

console.log('a' + +'a') //NaNa

## ==运算符 和 逻辑非运算符! 存在坑
> 都是在做数字的转换，!优先转换成Boolean类型，再转换成数字类型，最终还是做数字上的比较

> 引用类型的地址都是存在堆内存中，所以引用类型的比较，比较的是地址，而不是值
```
==运算符会发生隐式转换，逻辑非运算符!也会发生隐式转换
{} == false //false  //{}转换为字符串[object Object]，再转换为数字NaN，NaN==false为false
[] == false //true   //[]转换为字符串''，再转换为数字0，0==false为true
[] == [] // false   地址不同
{} == {} // false   地址不同
[] == false //true
[] == ![] //true
{} == 0   // false {} 转换成NaN
[] == 0   // true  [] 转换成0
[] == ![] // true  [] 转换成0
```
## == 和 === 的区别
为了防止出现意外类型的转换，所以使用 === 代替 ==  
=== 会比较类型和值是否相等（不存在隐式转换），== 只会比较值是否相等（存在隐式转换）


