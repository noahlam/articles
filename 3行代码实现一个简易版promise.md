### 前言  

作为一个后端过来的同学,刚入门前端的时候,被js种种「反人类」的概念折腾的死去活来的.
其中一个印象比较深刻的,就是promise,感觉实在太难理解了...所有就有了写个简单的promise的想法.
希望能帮助到一些跟我一样,感觉promise很难理解的新同学.

promise的教程网上多如牛毛,其中写的比较通俗易懂的莫过于阮一峰的es6,反正我是他的书才懂的.
所以今天,我们也不会来复述一遍如何使用promise,今天我们从另一个角度学习promise,
先自己动手造一个轮子--实现一个最简单的promise,解决 `回调地狱` 的问题.

### 简单实现

请看代码

```js
function easyPromise (fn) {
    this.then = cb => this.cb = cb
    this.resolve = data => this.cb(data)
    fn(this.resolve)
}
```

### 详解

上面的代码就实现了一个简单的,实现then回调的「promise」,这里为了缩短代码量,用了es6的简写,实际展开应该是这样

```js
function easyPromise (fn) {
    var that = this

    // 第一步,定义 then()
    this.then = function (cb) {
        //先将 then() 括号里面的参数(回调函数)保存起来
        that.cb = cb
    }

    // 定义一个 resolve
    this.resolve = function(data) {
        that.cb(data)
    }

    // 将 resolve 作为回调函数,传给fn
    fn(this.resolve)
}
```

接下来我们看看如何使用


```js
new easyPromise((resolve) => {
    setTimeout(() => {
        resolve("延时执行")
    }, 1000)
}).then((data) => {
    console.log(data)
})
```
结果： 控制台在1秒之后，输出 `延时执行`

同样为了方便理解,我们不妨把以上代码写好理解一点.

```js
// 定义一个要传给 promise 的函数，它接收一个函数（resolve）作为参数。
// resolve 的作用是在合适的时间，通知 promise 应该要执行 then 里面的回调函数了。
function promiseCallback (resolve) {
   setTimeout(() => {
      resolve("延时执行")
   }, 1000)
}

// 定义一个 要传给 then 的回调函数
function thenCallback (data) {
    console.log(data)
}

// 实例化 promis,并分别传入对应的回调
new easyPromise(promiseCallback)
.then(thenCallback)
```
> tips:  `promise.then()` 的时候,并没有马上执行括号里面的回调函数,只是把括号里面的回调函数保存起来.

我们来梳理一下执行流程

1. 先通过 `then` 把 `thenCallback` 存起来
```js
this.then = function (cb) {
  that.cb = cb
}
```
这里的 `cb` , 就是上例的 `thenCallback` 所以其实可以等价于 ` this.cb = thenCallback `

2. 执行 `promise` 括号里的函数，并把事先定义好的 `resolve` 函数作为参数传给他

```js
fn(this.resolve)
```
这里的 `fn` , 就是上例的 `promiseCallback`

3. 执行 `promiseCallback` 我们的逻辑就跳到 `promiseCallback` 函数内部去
```js
setTimeout(() => {
  resolve("延时执行")
}, 1000)
```
逻辑很简单，就是等待1秒后，执行 `resolve` 函数， 这个 `resolve` 哪来的呢？

`fn(this.resolve)` -> `promiseCallback (resolve)` -> `resolve`

4. 执行 `resolve` 我们的逻辑就跳到 `resolve` 函数内部去

```js
that.cb(data)
```
这个 `that.cb` 又是哪来的呢？ 就是我们第一步保存的 then括号里面的回调函数，也就是 `thenCallback`

```js
console.log(data)
```
所以就在1秒后输出 `延时执行`

### 最后

好了，以上就是 `promise` 的核心逻辑，当然还有很多功能没实现，不过本文的目的是帮助新手更好的了解 `promise`,
不是要实现一个完整的，符合 `promise A+`规范的 `promise`, 想要了解更多的童鞋，这里推荐几个链接
1. [阮一峰：ECMAScript 6 入门 - Promise 对象](http://es6.ruanyifeng.com/#docs/promise)
1. [深入 Promise(一)——Promise 实现详解](https://zhuanlan.zhihu.com/p/25178630)
1. [浅谈Promise之按照Promise/A+规范实现Promise类](https://juejin.im/post/5a5c0c04518825734978e1a6)


如果觉得本文对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，
有兴趣的童鞋可以看看，你们的支持就是我最大的动力。