### 前言  

本文纯属个人实践经验总结，如果对你有帮助，那么我会感到荣幸。

本文不涉及 罕见API 使用方法等，大部分内容都是基于对vue

1. 多个页面都使用的到方法，放在 vue.prototype上会很方便
2. 需要响应的数据，在获取接口的时候，先设置
3. 封装全局基于promise的方法
4. 参数用一个对象包起来会让你很方便
5. data里面的数据多的时候，给每个数据加一个备注，会让你后期往回看的时候很清晰
6. 逻辑复杂的，拆成组件
7. 大部分情况下，生命周期里面，不要有太多行代码，可以封装成方法，再调用
8. 少用watch，如果你觉得你好多地方都需要用到watch，那十有八九是你对vue的API还不够了解
9.

### 最后

好了，以上就是 `promise` 的核心逻辑，当然还有很多功能没实现，不过本文的目的是帮助新手更好的了解 `promise`,
不是要实现一个完整的，符合 `promise A+`规范的 `promise`, 想要了解更多的童鞋，这里推荐几个链接
1. [阮一峰：ECMAScript 6 入门 - Promise 对象](http://es6.ruanyifeng.com/#docs/promise)
1. [深入 Promise(一)——Promise 实现详解](https://zhuanlan.zhihu.com/p/25178630)
1. [浅谈Promise之按照Promise/A+规范实现Promise类](https://juejin.im/post/5a5c0c04518825734978e1a6)


如果觉得本文对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，
有兴趣的童鞋可以看看，你们的支持就是我最大的动力。