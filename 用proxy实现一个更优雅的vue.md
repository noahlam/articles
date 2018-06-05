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

```javascript
 _observe (data) {
    var that = this
    this.$data = new Proxy(data, {
    set(target,key,value){
      // 利用 Reflect 还原默认的赋值操作
      let res =  Reflect.set(target,key,value)
      // 这行是监听代码，后面会解释
      that.handles[key].map(item => {item.update()})
      return res
    }
    })
}
```

-------未完待续---------------