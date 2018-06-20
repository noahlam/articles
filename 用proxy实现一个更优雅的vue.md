### 前言
如果你有读过Vue的源码，或者有了解过Vue的响应原理，那么你一定知道Object.defineProperty(),
那么你也应该知道，Vue 2.x里，是通过 `递归 + 遍历 data`对象来实现对数据的监控的，
你可能还会知道，我们使用的时候，直接通过数组的下标给数组设置值，不能实时响应，是因为Object.defineProperty()
无法监控到数组下标的变化，而我们平常所用的数组方法 `push`, `pop`, `shift`, `unshift`, `splice`, `sort`, `reverse`,
其实不是真正的数组方法，而是被修改过的,这些都是因为 Object.defineProperty() 提供的能力有限，无法做到完美。
  
网上看过很多关于Vue的源码解读或者实现一个简易版的Vue的教程，还都是用 Object.defineProperty (大概是为兼容性考虑吧), 
而 Object.defineProperty() 确实存在诸多限制, 据说Vue的3.x版本会改用Proxy，那么今天我们就先来尝尝鲜，用Proxy实现一个简单版的Vue

### proxy 介绍

> Proxy 用于修改某些操作的默认行为，等同于在语言层面做出修改，所以属于一种“元编程”（meta programming），即对编程语言进行编程。
  
> Proxy 可以理解成，在目标对象之前架设一层“拦截”，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。Proxy 这个词的原意是代理，用在这里表示由它来“代理”某些操作，可以译为“代理器”。

以上引用内容来自阮一峰的es6教程的Proxy章节 原文地址[请戳这里](http://es6.ruanyifeng.com/#docs/proxy)。

我们来看看如何用Proxy去定义一个监听数据的函数

### 定义 observe
```javascript
 _observe (data){
    var that = this
    
    // 把代理器返回的对象存到 this.$data 里面
    this.$data = new Proxy(data, {
    set(target,key,value){
      // 利用 Reflect 还原默认的赋值操作
      let res =  Reflect.set(target,key,value)
      // 这行就是监控代码了
      that.handles[key].map(item => {item.update()})
      return res
    }
    })
}
```

当触发set的时候，就会执行 `that.handles[key].map(item => {item.update()})` ，这句代码的作用就是执行 `该属性名下的所有 "监视器" `

那么，监视器怎么来的呢？ 请继续看以下代码

### 定义 compile
```javascript
  _compile (root){
       const nodes = Array.prototype.slice.call(root.children)
       let data = this.$data
       nodes.map(node => {
         // 如果不是末尾节点，就递归
         if(node.children.length > 0) this._complie(node)
         //  处理 v-bind 指令
         if(node.hasAttribute('v-bind')) {
           let key = node.getAttribute('v-bind')
           this._pushWatcher(new Watcher(node,'innerHTML',data,key))
         }
         //  处理 v-model 指令
         if(node.hasAttribute('v-model')) {
           let key = node.getAttribute('v-model')
           this._pushWatcher(new Watcher(node,'value',data,key))
           node.addEventListener('input',() => {data[key] = node.value})
         }
         //  处理 v-click 指令
         if(node.hasAttribute('v-click')) {
           let methodName = node.getAttribute('v-click')
           let mothod = this.$methods[methodName].bind(data)
           node.addEventListener('click',mothod)
         }
       })
     }
```
上面这段代码，看起来很长，可是实际上，只做了一件很简单的事情，
就是 `"编译" html 模板`，把有 `v-bind`、`v-model`、`v-click` 都给加上对应的 `通知` 和 `监控`

1. **通知** 就是 `this._pushWatcher(...)` , 相当于是安装一个监听器，这样只要 this.$data 有发生 set 操作的话，就会执行
`this._pushWatcher` 括号里面传的函数，来通知节点更新数据

2. **监控** 就是 ` node.addEventListener(...)` 监听相应的事件，然后执行对应的 `watcher` 或者 `methods`

`this._pushWatcher` 又做了什么呢？

```javascript
 _pushWatcher (watcher) {
      if (!this._binding[watcher.key]) this._binding[watcher.key] = []
      this._binding[watcher.key].push(watcher)
    }
```

这个就更简单了，如果 `this._binding[watcher.key]` 为空，就初始化，然后向里面添加一个 监听器

最后，我们再来看看，监听器是怎么实现的

### 定义 Watcher
```javascript
 class Watcher {
     constructor (node,attr,data,key) {
       this.node = node
       this.attr = attr
       this.data = data
       this.key = key
     }
     update () {
       this.node[this.attr] = this.data[this.key]
     }
   }
```
`Watcher` 是一个类，只有一个方法，就是更新数据，怎么知道要更新哪个节点，更新为什么数据呢？
在实例化（new）的时候，传的参数就是定义这些的

这样，我们就实现初步的双向绑定了，整个代码大概只有50行。其实还可以更少，
但是更少的话，就是继续阉割功能了(虽然目前实现的也是严重阉割版的)，
但是我觉得实现这些，刚好可以不多不少帮我我们理解vue的本质。

### 最后

本文最终实现代码已经放在 [github](https://github.com/noahlam/practice-truth/blob/master/code/vue-class-proxy.html)上，想要直接看效果的同学，可以上去直接copy，运行。

如果觉得本文对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，有兴趣的童鞋可以看看，你们的支持就是我最大的动力。