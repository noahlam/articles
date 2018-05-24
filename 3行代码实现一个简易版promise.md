### 前言  

作为一个后端过来的同学,刚入门前端的时候,被js种种「反人类」的概念折腾的死去活来的.
其中一个印象比较深刻的,就是promise,感觉实在太难理解了...所有就有了写个简单的promise的想法.
希望能帮助到一些跟我一样,感觉promise很难理解的新同学.

promise的教程网上多如牛毛,其中写的比较通俗易懂的莫过于阮一峰的es6,反正我是他的书才懂的.
所以今天,我们也不会来复述一遍如何使用promise,今天我们从另一个角度学习promise,
先自己动手造一个轮子--实现一个最简单的promise,解决 `回调地狱` 的问题.

请看代码
```js
function easyPromise (fn) {
    this.then = cb => this.cb = cb
    this.resolve = data => this.cb(data)
    fn(this.resolve)
}
```
这里请原谅我,为了缩短代码量,用了es6的简写,实际展开应该是这样

```js
function easyPromise (fn) {
this.then = function (cb) {
this.cb = cb
}

this.resolve = function(data) {
this.cb(data)
}
fn(this.resolve)
}
```


```
new easyPromise((resolve) => {
console.log('马上执行')
setTimeout(() => {
resolve("延时执行")
}, 1000)
}).then((res) => {
console.log(res)
})
```

我们看到,这里真正起作用的,只有3行代码,那这3行代码,分别起什么作用呢?下面我们就来分析一下:
#### 第一行. `this.then = callback => this.cb = callback`





