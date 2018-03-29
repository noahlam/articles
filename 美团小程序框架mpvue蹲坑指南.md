###美团小程序框架mpvue蹲坑指南

第一次接触小程序大概是17年初,当时小程序刚刚内侧,当时就被各种限制折腾的死去活来的,单向绑定,没有promise,请求数限制,包大小限制,各种反人类,...反正我是感受到了满满的恶意.
最近接到一个工程类的小程序项目,做技术选型的时候,又把以前的东西捡起来看了看,重新熟悉了一下,感觉小程序还是有在努力的,支持大部分es6语法了,还出了一个类Vue的mvvm框架wepy,还支持redux状态管理,就大致建了个demo,跑了起来,赶紧虽然没有vue那么酸爽,但是还是挺ok的,至少比原生的小程序语法亲民很多.

然后就开始用wepy搭项目,写静态页面(由于公司的开发模式是先写静态页面,等待后端的同学接口出来了再绑定数据),虽然wepy用起来比原生的顺手,但是还是有很多坑的,这里就不列举了.....就在我们静态页面快写完的时候,某天晚上论坛的时候看到一条消息, 美团出了个小程序框架mpVue,大致看了一下官方(http://mpvue.com/)的介绍,主要有一下亮点:
1. **跟vue一样的开发体验,包括vuex**
2. **H5代码转换编译成小程序目标代码的能力**

也就是说,不但可以用我们熟悉的vue语法开发,还有可能直接把你的h5页面编译成小程序.
建了个demo,跑了一下,感觉简直就是开发界的良心之作啊.顺便把之前写的wepy的静态页面代码复制过来看了一下,发现只要改动一点点,就可以顺利从wepy切换到mpvue上来(整个项目的切换时间在半天左右).
说做就做,当天就切到mpvue.一直到现在项目接近尾声了,整个开发过程,真是令人愉悦.

Bug....我今天好像不是来给mpvue做广告的,我是来找茬的..

下面就盘点一下我最近用mpvue开发,遇到的一些需要需要注意的细节.(或者说跟vue不同的地方)

一,这个个人感觉是最大的坑,除了缺少文件会报错外,其他的代码语法错误等,控制台大部分时间都是安静的(偶尔也会报一个xxx is undefined)
比较经常碰到的是这样 this.xxx =5 有些情况下会报错,而有些情况下则没有任何反应,具体什么情况下会,什么情况下不会,我现在还没摸出规律来..

有一次我把 
this.dataObject.map(()=>{...这里省略...})   
结果map前面的**.**不小心给露掉了,实际代码变成  
this.dataObjectmap(()=>{...这里省略...})   
结果找了半天没找到问题的原因

二,这个也是比较难受的地方,就是模板的数据绑定里面,没办法在模板语法里面调用methods方法(或者说没办法调用computed以外的函数),有人也许会说,那可以用computed属性,那如果我想给函数传参怎么办? 看下面代码:

<template>
  <view v-for="item in costList" >
	{{formatCost(item)}}
  </view>
</template>

<script>
export default {
  data(){
    return{
      costList:[]
    }
  },
  methods: {
    formatCost(item){
	return item.toFixed(2)
    },
    getData(){
	let arr = [3.255,4.1,5,15]
	this.costList = arr
    }
  }
</script>

这个时候 {{formatCost(item)}}里面的内容,会渲染成空字符串,理由就是因为不支持函数,而且这中情况,也无法使用computed属性,除非你想为每个数组元素写一个computed

这种情况,我的解决方案是在在获取到数据的时候,就先把数据改了.如上面的例子,我们可以在 getData方法里面这样写
let arr = [3.255,4.1,5,15]
// 遍历数组里面的元素,然后格式化一下,添加到 costList里去
arr.map(item => {
    this.costList.push = this.formatCost(item)
})
	


三、所有页面里面的created生命周期函数  都会在小程序加载的时候，一次性执行，而不是没进入一个页面执行一次，如，我有3个页面
pageA  
    ...省略一些代码...
    creatted(){
        console.log('pageA 的 created函数执行')
    }
pageB  
    ...省略一些代码...
    creatted(){
        console.log('pageB 的 created函数执行')
    }
pageC  
    ...省略一些代码...
    creatted(){
        console.log('pageC 的 created函数执行')
    }

然后，启动小程序，不进入这3个页面，假设我现在有一个index页面，我们打开这个页面，会有一下输出
pageA 的 created函数执行
pageB 的 created函数执行
pageC 的 created函数执行

这个其实很好解决，用mounted或者onLoad或者onReady代替，说到这几个函数，那就顺便提一下,这里的created和mounted是vue(mpvue)的生命周期，而onLoad、onReady是小程序的生命周期，mpvue官方给的说明是：
> 除了 Vue 本身的生命周期外，mpvue 还兼容了小程序生命周期，这部分生命周期钩子的来源于微信小程序的 Page， **除特殊情况外，不建议使用小程序的生命周期钩子。**

但是官方给的生命周期图示里面，也表明了，小程序的onLoad、onReady比created、mounted执行的早，也就是说如果我们在和onLoad onReady里面去请求数据的话，会相对的减少白屏时间（这里说的白屏是指数据未渲染的界面），而且官方没说明为什么不建议使用小程序的生命周期，我们也尝试了，用小程序的生命周期，没发现生命问题，所以我们还是比较倾向优先使用小程序的生命周期，毕竟用户体验才是王道。

mpvue的生命周期函数里面，还有一个比较容易坑到人的地方，就是onShow,onShow这个函数，在page页面里面，是按我们的预期执行，**当小程序启动，或从后台进入前台显示** 都会执行，没问题。可是如果是组件里面的onShow就不同了，在组件里面，onShow在组件第一次加载的时候，是不会执行的，只有在**该组件所在的页面从后台进入前台显示**的时候，才会执行。

说到这里，我们顺便看看小程序的页面跳转方式，小程序在一个页面跳转（调用wx.navigateTo）到另一个页面的时候，并不会销毁原来的页面，而是转到后台去，并且执行原页面里面的onHide里的代码，这也是为什么小程序的页面路径最多只能十层，因为你访问过的页面，正常都会保存在内存里，相当于vue里的keep-alive，如果允许跳转非常多页面的话，很容易导致使用过大。
当然，我们也可以使用wx.navigateBack wx.redirectTo wx.reLaunch 来销毁页面，这3个方法，会调用页面的onUnload函数

四、组件名称不能用tabBar list 等等，这个是我碰到的比较经典的坑，自己手写了一个tabBar，内容可以正常显示，页面里面的js逻辑，死活不会执行，加上 本文 第一点提到的，不会报错，让我一顿好找啊...

五、挂载在Vue.prototype上的属性，在模板语法里面是undefined，必须经过computed计算过一下才能用。
在用vue的时候，我喜欢把图片的服务器路径存到vue的原型里面：
	import config from './config'
	Vue.prototype.$serverPath = config.serverPath
然后 我们在页面里面这样用
	<img :src="$serverPath + 'logo.png'" />
这样 就可以避免在每个页面导入config文件，后期如果我们发布正式版的时候，只要在这边修改一下config配置文件就可以了
然额，这样写在mpvue里面，实际渲染出来的会是 
 <image src="undefinedlogo.png" ></image>
要想在每个页面里面使用，只能乖乖在每个页面里面导入，或者在computed里面返回this.$serverPath

v-for=(item,index)



官方给出的不同地方
在父组件里面,不能给子组件加class style
任何地方,:class不能太复杂,可以用计算属性,不能三元.最好也不要三元
没有过滤器和自定义指令.

页面底部的 tabBar 由页面决定，即只要是定义为 tabBar 的页面，底部都有 tabBar





