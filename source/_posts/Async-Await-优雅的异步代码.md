title: Async/Await 优雅的异步代码
date: 2017-04-08 18:14:54
tags:
- JavaScript
- Promise
---

JavaScript的异步编程方式有效提高了应用性能；但有时候使用回调函数来组织代码往往会导致代码难以阅读。Promise让我们告别回调函数，可以将嵌套的回调函数展平，写出更优雅的异步代码；在实践过程中，却发现Promise并不完美，这时，我们有了Async/Await。

ES7 中有了更加标准的解决方案，新增了 async/await 两个关键词。async 可以声明一个异步函数，此函数需要返回一个 Promise 对象。await 可以等待一个 Promise 对象 resolve，并拿到结果。

### Async/Await简介

- async/await是基于Promise实现的，它不能用于普通的回调函数。
- async/await与Promise一样，是非阻塞的。
- async/await使得异步代码看起来像同步代码。

<!--more-->

### 说说Promise

先以 Promise 实例展示
```javascript
const results = {
  data: [{
      content: 'test content1'
    },
    {
      content: 'test content2'
    },
  ]
}

function getPosts() {
  return new Promise(function (resolve) {
    setTimeout(function () {
      resolve(results)
    }, 1000)
  })
}

getPosts().then(function (result) {
  console.log(result)
}).catch(function (err) {
  console.log(err)
})
```
在 1 秒后 resolve 一個模拟的 result，catch能处理JSON.parse错误

### 如果是 Async/Await

有了 async/await 之后，我们就可以这样实现了：
```javascript
async function printResult() {
  try {
    console.log('执行请求前')
    var result = await getPosts()
    console.log('执行请求后')
    console.log(result)
  } catch (err) {
    console.log(err);
  }
}

printResult()
```
await getPosts()表示会等到getPosts的promise成功reosolve之后再执行。async 函数可以正常的返回结果和抛出异常。await 函数调用即可拿到结果，在外面包上 try/catch 就可以捕获异常。async/await使得异步代码看起来像同步代码

### 结论

Async/Await是近年来JavaScript添加的最革命性的特性之一。它会让你发现Promise的语法不完美，而且提供了一个直观的替代方法。

ES7 还在草案阶段，那现在想用这个特性：

#### 在前端

例如在写React的时候，基本上已经使用了 babel。如果要使用 Async/Await，presets 除了原本的 es2015 外，只要加上 stage-3：
```
.bebalrc
{
  "presets": ["es2015", "stage-3"]
}
```

#### 在后端

Node 7.0.0 起已经支持 Async/Await，Node 7不是LTS（长期支持版本），但是，Node 8很快就会发布，将代码迁移到新版本会非常简单。