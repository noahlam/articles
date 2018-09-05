### 前言

之前几个项目中,都多多少少碰到一些组件之间需要通信的地方,而因为种种原因,
event bus 的成本反而比vuex还高, 所以技术选型上选用了 vuex, 但是不知道为什么,
团队里的一些新人一听到vuex,就开始退缩了, 因为vuex 很难? 真的很难吗?
今天我们用简单的3步来证明一下,vuex有多简单.

> #### 纯属个人经验,难免有不正确的地方,如有发现,欢迎指正!

> #### 这是一个针对新手的入门级教程、入门级教程、入门级教程

### 第零步
新建一个vue项目,安装vuex,这里不做过多介绍,能点进来的,默认你具备这些技能 ^_^

### 第一步

新建一个`.js` 文件,名字位置任意,按照惯例,建议在`/src/store` 目录下(没有的话自己新建一个呗)

`文件位置 /src/store/index.js`

```js
// 引入vue 和 vuex
import Vue from 'vue'
import Vuex from 'vuex'

// 这里需要use一下,固定写法,记住即可
Vue.use(Vuex)

// 直接导出 一个 Store 的实例
export default new Vuex.Store({
  // 类似 vue 的 data
  state: {
    name: 'oldName'
  },
  // 类似 vue 里的 mothods(同步方法)
  mutations: {
    updateName (state) {
      state.name = 'newName'
    }
  }
})
```

代码看起来稍微有那么一点点多,不过看起来是不是很熟悉? 跟普通的 vue 没多大差别嘛.
这一步其实就是新建一个store,但是我们还没在项目中使用.

### 第二步

在入口文件引入上述文件, 并稍微改一下传给 new Vue()的参数,**新增的行后面有备注**

`文件位置 /src/main.js` (vue-cli自动生成的入口,如果你能不用脚手架,那么也就不需要我说明了)

```
import Vue from 'vue'
import App from './App'
import vuexStore from './store'   // 新增

new Vue({
  el: '#app',
  store:vuexStore                 // 新增
  components: { App },
  template: '<App/>'
})
```

> Tip: import store from './store' 后面的地址,就是上面我们新建那个文件的位置(`/src/store/index.js`),
因为我这里是index.js,所以可以省略.

### 第三步

以上2步,其实已经完成了vuex的基本配置,接下来就是使用了

`文件位置 /src/main.js` (同样是vue-cli生成的app.vue,这里为了方便演示,我去掉多余的代码)

```js
<template>
  <div>
    {{getName}}
    <button @click="changeName" value="更名">更名</button>
  </div>
</template>

<script>
export default {
  computed:{
    getName(){
      return this.$store.state.name
    }
  },
  methods:{
    changeName () {
      this.$store.commit('updateName')
    }
  }
}
</script>
```

这里就是一个很普通的vue文件了,有区别的地方是这里我们需要用computed属性去获取 `store 里的 "data"`

还有就是我们要改变数据的话,不再用 `this.xxx = xxx` 改成 this.$store.commit('updateName')


### 总结

**你可能会觉得,上例这样做的意义何在,为何不直接用vue的data跟methods?**

上例只是为了简单讲解如何使用vuex,所以简化了一些流程,试想一下,如果你有这样一个页面:
一共嵌套了10层组件(即子组件里面还有子子组件,子子组件下面还有子子子组件,以此类推10层)
然后最后一层组件一个数据改变了,要通知第一层组件的时候,我们只需在最底层组件里`this.$store.commit()`,
然后再最外层组件上用computed属性获取对应的值,就能做到实时更新.无需层层$emit上去.

### 最后

本来想在最后再扩展一下getter,action+dispatch,模块化等等,不过为了对得起这个标题,
只好放在 [下一篇:vuex其实超简单,喝完这3步,还有3步](https://github.com/noahlam/articles/blob/master/vuex%E5%85%B6%E5%AE%9E%E8%B6%85%E7%AE%80%E5%8D%95%2C%E5%96%9D%E5%AE%8C%E8%BF%993%E6%AD%A5%2C%E8%BF%98%E6%9C%893%E6%AD%A5.md)

