上一篇 [vuex其实超简单,只需3步](https://github.com/noahlam/articles/blob/master/vuex%E5%85%B6%E5%AE%9E%E8%B6%85%E7%AE%80%E5%8D%95%2C%E5%8F%AA%E9%9C%803%E6%AD%A5.md)
简单介绍了vuex的3步入门,不过为了初学者容易消化,我削减了很多内容,这一节,就是把少掉的内容补上,
如果你没看过上篇,请戳链接过去先看一下再回来,否则,你会觉得本文摸不着头脑.

> #### 纯属个人经验,难免有不正确的地方,如有发现,欢迎指正!

> 还是一样,本文针对初学者.

一、 **Getter**

我们先回忆一下上一篇的代码
```js
computed:{
    getName(){
      return this.$store.state.name
    }
}
```

这里假设现在逻辑有变,我们最终期望得到的数据(getName),是基于 `this.$store.state.name`
上经过复杂计算得来的,刚好这个getName要在好多个地方使用,那么我们就得复制好几份.

vuex 给我们提供了 getter,请看代码 (`文件位置 /src/store/index.js`)

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  // 类似 vue 的 data
  state: {
    name: 'oldName'
  },
  // 类似 vue 的 computed -----------------以下5行为新增
  getters:{
    getReverseName: state => {
        return state.name.split('').reverse().join('')
    }
  },
  // 类似 vue 里的 mothods(同步方法)
  mutations: {
    updateName (state) {
      state.name = 'newName'
    }
  }
})
```

然后我们可以这样用 `文件位置 /src/main.js`
```
computed:{
    getName(){
      return this.$store.getters.getReverseName
    }
}
```
事实上, getter 不止单单起到封装的作用,它还跟vue的computed属性一样,会缓存结果数据,
只有当依赖改变的时候,才要重新计算.

二、 **actions和$dispatch**

细心的你,一定发现我之前代码里 `mutations` 头上的注释了 `类似 vue 里的 mothods(同步方法)`

为什么要在 `methods` 后面备注是同步方法呢? mutation只能是同步的函数,只能是同步的函数,只能是同步的函数!!
请看vuex的解释:

> 现在想象，我们正在 debug 一个 app 并且观察 devtool 中的 mutation 日志。每一条 mutation 被记录，
devtools 都需要捕捉到前一状态和后一状态的快照。然而，在上面的例子中 mutation 中的异步函数中的回调让这不
可能完成：因为当 mutation 触发的时候，回调函数还没有被调用，devtools 不知道什么时候回调函数实际上被调
用——实质上任何在回调函数中进行的状态的改变都是不可追踪的。

那么如果我们想触发一个异步的操作呢? 答案是: action + $dispatch, 我们继续修改store/index.js下面的代码

`文件位置 /src/store/index.js`

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
  // 类似 vue 的 data
  state: {
    name: 'oldName'
  },
  // 类似 vue 的 computed
  getters:{
    getReverseName: state => {
        return state.name.split('').reverse().join('')
    }
  },
  // 类似 vue 里的 mothods(同步方法)
  mutations: {
    updateName (state) {
      state.name = 'newName'
    }
  },
  // 类似 vue 里的 mothods(异步方法) -------- 以下7行为新增
  actions: {
    updateNameAsync ({ commit }) {
      setTimeout(() => {
        commit('updateName')
      }, 1000)
    }
  }
})
```

然后我们可以再我们的vue页面里面这样使用
```js
methods: {
    rename () {
        this.$store.dispatch('updateNameAsync')
    }
}
```

三、 **Module 模块化**

当项目越来越大的时候,单个 store 文件,肯定不是我们想要的, 所以就有了模块化.
假设 `src/store` 目录下有这2个文件

`moduleA.js`

```
export default {
    state: { ... },
    getters: { ... },
    mutations: { ... },
    actions: { ... }
}
```

`moduleB.js`

```
export default {
    state: { ... },
    getters: { ... },
    mutations: { ... },
    actions: { ... }
}
```

那么我们可以把 `index.js` 改成这样

```
import moduleA from './moduleA'
import moduleB from './moduleB'

export default new Vuex.Store({
    modules: {
        moduleA,
        moduleB
    }
})
```

这样我们就可以很轻松的把一个store拆分成多个.

四、 **总结**

1. actions 的参数是 `store` 对象,而 getters 和 mutations 的参数是 `state` .
2. actions 和 mutations 还可以传第二个参数,具体看vuex官方文档
3. getters/mutations/actions 都有对应的map,如: mapGetters , 具体看vuex官方文档
1. 模块内部如果怕有命名冲突的话,可以使用命名空间, 具体看vuex官方文档
4. vuex 其实跟 vue 非常像,有data(state),methods(mutations,actions),computed(getters),还能模块化.

如果觉得本文对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，有兴趣的童鞋可以看看，你们的支持就是我最大的动力。