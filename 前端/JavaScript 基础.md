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



### 4. 函数

函数与方法：函数类外，方法类内

#### 4.1 定义函数

因为没有类型，故不需要定义返回值

``` javascript
function abs(x) {
    if (x >= 0) {
        return x; // js每行会加';', 故不要在return之后换行
    } else {
        return -x;
    }
}

// 或者
var abs = function(x) {
    if (x >= 0) {
        return x; 
    } else {
        return -x;
    }
}
```

#### 4.2 变量作用域

**全局变量**

所有全局变量都会绑定到全局对象 window上，为减少冲突，自己定义全局变量实现(类似 C++ 的命名空间)

``` javascript
// 唯一全局变量
var MyApp = {};
// 定义全局变量
MyApp.name = 'bzzb';
MyApp.add = function (a, b) {
    return a + b;
}
```

**局部变量**

局部变量使用 **let** 定义

**常量**

ES 6 引入 **const** 关键字实现常量



### 5. 内部对象

#### 5.1 Date

具体使用查文档

#### 5.2 JSON

``` javascript
var user = {
    name : 'bzzb',
    age : 20,
    sex : '男',
}
// 对象 to JSON
var jsonUser = JSON.stringify(user);
// JSON to 对象
var obj = JSON.parse(jsonUser);
```

#### 5.3 Ajax



### 6. OOP

**原型** 

js 通过原型来实现 oop，就是个父类

``` javascript
var student = {
    name: "student",
    age: 3
}
var bzzb = {
    name : "bzzb"
}
// bzzb的原型是 student
bzzb.__proto__ = student;
```



**class继承**

class 在 ES 6 引入，实现类定义，**<font color=blue>本质还是原型</font>**

``` javascript
class Student {
    // 构造器固定为constructor
    constructor(name) {
        this.name = name;
    }
    // 成员方法
    hello() {
        alert('hello');
    }
}

var bzzb = new Student('bzzb');

// 继承
class underGraduate extends Student {
    constructor(name, age) {
        super(name); // 同样使用super调用父类方法
        this.age = age;
    }
}
```



### 7. 操作 BOM 对象(重点)

BOM：Browser Object Model

#### 主要对象

- window： 代表浏览器窗口
- navigator：封装了浏览器信息，大多数不会使用，因为会被认为修改
- screen：全屏属性
- location (重要)：代表当前的 URL 信息，

``` javascript
host: "www.baidu.com"
href: "https://www.baidu.com/"
protocol: "https"
reload // 刷新网页
location.assign('http://101.132.96.161/') // 跳转
```

- document：代表当前的页面， HTML DOM 文档树

- history：代表历史记录，底层用栈实现，不建议使用