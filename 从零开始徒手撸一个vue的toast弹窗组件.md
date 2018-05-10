效果如图

![step_3](https://github.com/noahlam/practice-truth/blob/master/src/toast/img/step_3.gif)

### 一. 先写一个普通的vue组件

文件位置 `/src/toast/toast.vue`
```
<template>
  <div class="wrap">我是弹窗</div>
</template>

<style scoped>
  .wrap{
    position: fixed;
    left: 50%;
    top:50%;
    background: rgba(0,0,0,.35);
    padding: 10px;
    border-radius: 5px;
    transform: translate(-50%,-50%);
    color:#fff;
  }
</style>


```
### 二. 在我们需要使用的页面引入组件,方便看效果和错误
```
<template>
  <div id="app">
    <toast></toast>
  </div>
</template>

<script>
  import toast from './toast/toast'
  export default {
    components: {toast},
  }
</script>
```
 ![step_1](https://github.com/noahlam/practice-truth/blob/master/src/toast/img/step_1.png)

### 三. 实现动态加载组件

可以看到,已经显示出一个静态的弹出层了,接下来我们就来看看如何实现动态弹出.  

我们先在 `/src/toast/` 目录下面,新建一个`index.js`, 然后在index.js里面,敲入以下代码(由于该代码耦合比较严重,所以就不拆开一行一行讲解了,改成行内注释)

文件位置 `/src/toast/index.js`

```
import vue from 'vue'

// 这里就是我们刚刚创建的那个静态组件
import toastComponent from './toast.vue'

// 返回一个 扩展实例构造器
const ToastConstructor = vue.extend(toastComponent)

// 定义弹出组件的函数 接收2个参数, 要显示的文本 和 显示时间
function showToast(text, duration = 2000) {

  // 实例化一个 toast.vue
  const toastDom = new ToastConstructor({
    el: document.createElement('div'),
    data() {
      return {
        text:text,
        show:true
      }
    }
  })

  // 把 实例化的 toast.vue 添加到 body 里
  document.body.appendChild(toastDom.$el)

  // 过了 duration 时间后隐藏
  setTimeout(() => {toastDom.show = false} ,duration)
}

// 注册为全局组件的函数
function registryToast() {
  // 将组件注册到 vue 的 原型链里去,
  // 这样就可以在所有 vue 的实例里面使用 this.$toast()
  vue.prototype.$toast = showToast
}

export default registryToast
```

> 附一个传送门 [vue.extend 官方文档](https://cn.vuejs.org/v2/api/#Vue-extend)



### 四. 试用

到这里,我们已经初步完成了一个可以全局注册和动态加载的toast组件,接下来我们来试用一下看看

1. 在vue的入口文件(脚手架生成的话是`./src/main.js`) 注册一下组件

文件位置 `/src/main.js`
```
import toastRegistry from './toast/index'

// 这里也可以直接执行 toastRegistry()
Vue.use(toastRegistry)
```

2. 我们稍微修改一下使用方式,把`第二步` 的引用静态组件的代码,改成如下

```
<template>
  <div id="app">
    <input type="button" value="显示弹窗" @click="showToast">
  </div>
</template>

<script>
  export default {
    methods: {
      showToast () {
        this.$toast('我是弹出消息')
      }
    }
  }
</script>
```

  ![step_2](https://github.com/noahlam/practice-truth/blob/master/src/toast/img/step_2.gif)

可以看到,我们已经`不需要`在页面里面`引入`跟`注册`组件,就可以直接使用`this.$toast()`了.

### 五. 优化

现在我们已经初步实现了一个弹窗.不过离成功还差一点点,缺少一个动画,现在的弹出和隐藏都很生硬.

我们再对 `toast/index.js` 里的`showToast`函数稍微做一下修改(有注释的地方是有改动的)

文件位置 `/src/toast/index.js`
```
function showToast(text, duration = 2000) {
  const toastDom = new ToastConstructor({
    el: document.createElement('div'),
    data() {
      return {
        text:text,
        showWrap:true,    // 是否显示组件
        showContent:true  // 作用:在隐藏组件之前,显示隐藏动画
      }
    }
  })
  document.body.appendChild(toastDom.$el)

  // 提前 250ms 执行淡出动画(因为我们再css里面设置的隐藏动画持续是250ms)
  setTimeout(() => {toastDom.showContent = false} ,duration - 1250)
  // 过了 duration 时间后隐藏整个组件
  setTimeout(() => {toastDom.showWrap = false} ,duration)
}

```

然后,再修改一下toast.vue的样式

文件位置 `/src/toast/toast.vue`

```
<template>
  <div class="wrap" v-if="showWrap" :class="showContent ?'fadein':'fadeout'">{{text}}</div>
</template>

<style scoped>
  .wrap{
    position: fixed;
    left: 50%;
    top:50%;
    background: rgba(0,0,0,.35);
    padding: 10px;
    border-radius: 5px;
    transform: translate(-50%,-50%);
    color:#fff;
  }
  .fadein {
    animation: animate_in 0.25s;
  }
  .fadeout {
    animation: animate_out 0.25s;
    opacity: 0;
  }
  @keyframes animate_in {
    0% {
      opacity: 0;
    }
    100%{
      opacity: 1;
    }
  }
  @keyframes animate_out {
    0% {
      opacity: 1;
    }
    100%{
      opacity: 0;
    }
  }
</style>
```

大功告成,一个toast组件初步完成

![step_3](https://github.com/noahlam/practice-truth/blob/master/src/toast/img/step_3.gif)

### 总结
1. vue.extend 函数可以生成一个 `组件构造器` 可以用这个函数构造出一个 vue组件实例
2. 可以用 document.body.appendChild() 动态的把组件加到 body里面去
3. vue.prototype.$toast = showToast  可以在全局注册组件
4. 显示动画比较简单,隐藏动画必须要在隐藏之前预留足够的动画执行时间
5. 本文源码地址 [在这里](https://github.com/noahlam/practice-truth/tree/master/src/toast)
5. 以上都不重要,重要的是 给本文来个[star](https://github.com/noahlam/articles)