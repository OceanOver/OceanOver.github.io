title: ES6函数拓展 - 箭头函数
date: 2017-01-07 11:35:16
tags:
- JavaScript
- ES6
---
ES6允许使用“箭头”（=>）定义函数，且总是匿名函数。

#### 1.基本语法
```
(param1, param2, …, paramN) => { statements }

(param1, param2, …, paramN) => expression
//等同于
(param1, param2, …, paramN) => { return expression; }

// 如果只有一个参数，圆括号是可选的:
singleParam => { statements }

// 无参数的函数需要使用圆括号:
() => { statements }
```
<!--more-->
例如以下箭头函数:
```
var f = v => v;
```
等价于:
```
var f = function(v) {
  return v;
};
```
如果箭头函数的代码块部分多于一条语句，就要使用大括号将它们括起来，并且使用return语句返回。
```
var sum = (string1, string2) => {
    let num1 = Number.parseInt(string1);
    let num2 = Number.parseInt(string2);
    return num1 + num2;
}
```
大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在对象外面加上括号。
```
var getItem = id => ({ id: id, name: "Item" });
```
箭头函数极大的简化了回调函数
```
// 正常函数写法
[1,2,3].map(function (x) {
  return x * x;
});

// 箭头函数写法
[1,2,3].map(x => x * x);
```

#### 2.箭头函数体内的this对象
在箭头函数出现之前，每个新定义的函数都有自己的this值。

构造函数的this指向新的实例对象，如果函数是作为对象的方法被调用的，则this指向调用它的那个对象

箭头函数内:

函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象。且箭头函数不可以当作构造函数，也就是说，不可以使用new命令
```
// ES6
function foo() {
    setTimeout(() => {
        console.log('id:', this.id);
    }, 100);
}

// ES5
function foo() {
    var _this = this;

    setTimeout(function() {
        console.log('id:', _this.id);
    }, 100);
}
```
从转换后的ES5代码说明了，箭头函数里面是引用外层的this。
