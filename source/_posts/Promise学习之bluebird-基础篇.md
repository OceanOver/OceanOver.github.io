title: Promise学习之bluebird(基础篇)
date: 2017-01-07 11:31:26
tags:
- JavaScript
- Promise
---

>bluebird是一个第三方的Promise实现，除了Promise对象中的方法外，bluebird还有很多扩展方法。如，可以通过.spread()展开结构集、可以通过Promise.promisify()方法将一个Node回调函数包装成一个Promise实例。

安装bluebird模块后，可以就可以通过require获取对模块的引用：
```
var Promise = require('bluebird');
```
<!--more-->
### 1. 核心方法(常用的方法)
#### 1.1 new Promise - 创建实例
```
new Promise(function(function resolve, function reject) resolver) -> Promise
```
两个参数：resolve和reject，这两个参数被封装到所创建的Promise实例中，分别用于执行成功和执行失败时的调用。例：
```
function requestAsync(url) {
    return new Promise(function(resolve, reject) {
        var xhr = new XMLHttpRequest;
        xhr.addEventListener("error", reject);
        xhr.addEventListener("load", resolve);
        xhr.open("GET", url);
        xhr.send(null);
    });
}
```
调用：
```
requestAsync('http://######')
    .then(function(result) {
        // 请求成功的处理
    })
    .catch(function(err) {
        // 请求失败的处理
    })
```

#### 1.2 实例的then()方法
```
promise.then(onFulfilled, onRejected)-> Promise
```
* resolve(成功)时，onFulfilled方法会被调用
* reject(失败)时，onRejected方法会被调用

#### 1.3 实例的catch()方法
```
promise.catch(function(any error) handler) -> Promise
```
catch()方法是promise中.then(null, handler)的便捷方法，.then链中异常会被传递到.catch中。
为了提高代码的可读性，我们可以使用promise.then处理操作成功的情况，使用promise.catch处理操作失败的情况，示例如下：
```
promise.then(function(result) {
    // 操作成功
}).catch(function(err) {
    // 操作失败
});
```

#### 1.4 .spread - 展开实例返回的结果数组
```
function readFilesAsync() {
    return [fs.readFileAsync("file1.txt"),
        fs.readFileAsync("file2.txt")
    ];
}
readFilesAsync().spread(function(file1text, file2text) {
    //deal with fileText
});
```

#### 1.5 Promise.resolve()方法
```
Promise.resolve(obj)
```
* 参数是Promise实例，那么Promise.resolve将不做任何修改、原封不动地返回这个实例
* 参数是一个thenable对象,thenable对象指的是具有then方法的对象(个人在实践中没有用到)
* 参数是一个原始值，则Promise.resolve方法返回一个新的Promise对象，状态为Resolved。
* 不带参数，直接返回一个Resolved状态的Promise对象。

#### 1.6 Promise.reject()方法
```
Promise.reject(obj)
```
该方法也会返回一个新的Promise实例，该实例的状态为rejected。它的参数用法与Promise.resolve方法完全一致。
```
var p = Promise.resolve('出错了');

p.catch(function (s){
  console.log(s);   //出错了
});
```

### 2. Promise包装器
>将一个没有Promise的API对象包装成有Promise的API对象，即将其Promise化
>bluebird中有用于单个函数包装的Promise.promisify方法，也有用于对象属性包装的Promise.promisifyAll方法

#### 2.1 Promise.promisify - 单个函数对象的Promise化
```
const fs = require('fs');

// 回调形式，这里的callback 必须是最后一个参数
fs.readFile('./test.js', function(err, data) {
    console.log(data);
});

// promisify 形式
const readFileAsync = Promise.promisify(fs.readFile);

readFileAsync('./test.js').then(function(data) {
    //deal with data
}).catch(function(err) {
    //deal with err
});
```
callBack 一定出现在异步函数的最后一个

callBack 函数的调用参数中，第一个参数一定传入的是 err ，而不能是别的参数。

#### 2.2 Promise.promisifyAll - 对象属性的Promise化
将传入的对象实体的属性包装成Promise对象
```
const fs = Promise.promisifyall(require('fs'));

fs.readFileAsync('./test.js').then(function(data) {
    //deal with data
}).catch(function(err) {
    //deal with err
});
```
 经过 promisifyall , fs所有的函数都已经被 promise 化了。而被 promise 化的函数名变成了：原来的函数名+Async。

 如果想将对象原型链上的方法也加上的话，则传入对象的原型即可，例如 Promise.promisifyAll(obj.prototype)
