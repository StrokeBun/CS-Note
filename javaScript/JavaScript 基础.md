[toc]

## JavaScript 基础

### 1. 数据类型概述

**比较运算符**

``` javascript
== 等于，类型不一样但值相同也为true
=== 绝对等于，要求类型与值都相等
```

**NaN**

``` javascript
NaN 不等于任何值，包括自己，只能用 isNaN()进行判断
```

**null 和 undefined**

``` javascript
null 空值
undefined 未定义，使用不存在的变量
```

### 2. 严格检查

严格检查模式 (ES 6 标准)，预防随意性的问题；必须写在第一行

``` javascript
'use strict';
var a = 1; // 将会变灰，进行提醒
let b = 2;
```



### 3.数据类型

#### 3.1 字符串

基本 API 同 java，字符串也是不可变的，支持模板字符串(ES 6 标准)

``` javascript
let name = "bzzb"
let msg = `hello, ${name}`
name.legnth // 字符串长度
// 大小写转换
var upper = name.toUpperCase()
var lower = upper.toLowerCase()
```

#### 3.2 数组

可以包含任意数据类型

``` javascript
var arr = [1, 2, 3, 4]
arr.length = 6; // 可以通过length直接修改数组大小
var element = arr[5]; // undefined
```

#### 3.3 对象

``` javascript
var 对象名 = {
    属性名: 属性值,
    属性名: 属性值,
    属性名: 属性值
}
delete person.name; // 可以通过 delete 直接删除对象属性
person.university = "seu"; // 动态添加属性
```

#### 3.4 Map 和 Set 集合

ES 6标准

``` javascript
// Map和Set即为哈希表和集合
var map = new Map[['bzzb', 12], ['xuan', 13]];
var age = map.get('bzzb');
var set = new Set([1, 1, 1, 2, 3]);
```

#### 3.5 迭代器 iterator

ES 6标准，通过 for of 实现

``` javascript
var arr = [3, 4, 5]
for (let x of arr) {
    console.log(x);
}
```

### 4. 函数与 OOP







