### 美团小程序框架mpvue(花名：没朋友)蹲坑指南

第一次接触小程序大概是17年初,当时小程序刚刚内侧,当时就被各种限制折腾的死去活来的,单向绑定,
没有promise,请求数限制,包大小限制,各种反人类,...反正我是感受到了满满的恶意.
最近接到一个工程类的小程序项目,做技术选型的时候,又把以前的东西捡起来看了看,重新熟悉了一下,
感觉小程序还是有在努力的,支持大部分es6语法了,还出了一个类Vue的mvvm框架wepy,还支持redux状态管理,
就大致建了个demo,跑了起来,赶紧虽然没有vue那么酸爽,但是还是挺ok的,至少比原生的小程序语法亲民很多.

然后就开始用wepy搭项目,写静态页面(由于公司的开发模式是先写静态页面,
等待后端的同学接口出来了再绑定数据),虽然wepy用起来比原生的顺手,
但是还是有很多坑的,这里就不列举了.....

就在我们静态页面快写完的时候,某天晚上论坛的时候看到一条消息, 美团出了个小程序框架mpVue
（不知道为什么，每次看到这个名字，我只想到3个字，没朋友）,
大致看了一下[官方的介绍](http://mpvue.com/),主要有一下亮点:
1. **跟vue一样的开发体验,包括vuex**
2. **H5代码转换编译成小程序目标代码的能力**

也就是说,不但可以用我们熟悉的vue语法开发,还有可能直接把你的h5页面编译成小程序.
该项目到目前位置，开源不到20天，已经收到将近7000个star,可见天下苦秦已久。

建了个demo,跑了一下,感觉简直就是开发界的良心之作啊.顺便把之前写的wepy的静态页面代码复制过来看了一下,
发现只要改动一点点,就可以顺利从wepy切换到mpvue上来(整个项目的切换时间在半天左右).
说做就做,当天就切到mpvue.一直到现在项目接近尾声了,整个开发过程,真是令人愉悦.

Bug....我今天好像不是来给mpvue做广告的,我是来找茬的..

下面就盘点一下我最近用mpvue开发,遇到的一些需要需要注意的细节.(或者说跟vue不同的地方)

**一,** 这个个人感觉是最大的坑,除了缺少文件会报错外,其他的代码语法错误等,
控制台大部分时间都是安静的(偶尔也会报一个xxx is undefined)
比较经常碰到的是这样 this.xxx =5 有些情况下会报错,而有些情况下则没有任何反应,
具体什么情况下会,什么情况下不会,我现在还没摸出规律来..

有一次我把

    this.dataObject.map(() => { ...这里省略... })

结果map前面的 **.** 不小心给露掉了,实际代码变成

    this.dataObjectmap(() => { ...这里省略... })

结果找了半天没找到问题的原因  

**二,** 这个也是比较难受的地方,就是模板的数据绑定里面,没办法在模板语法里面调用methods方法
(或者说没办法调用computed以外的函数),有人也许会说,那可以用computed属性,那如果我想给函数传参怎么办?
看下面代码:

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

这个时候 `{{formatCost(item)}}`里面的内容,会渲染成空字符串,理由就是因为不支持函数,而且这中情况,
也无法使用computed属性,除非你想为每个数组元素写一个computed

这种情况,我的解决方案是在在获取到数据的时候,就先把数据改了.如上面的例子,我们可以在 getData方法里面这样写

    let arr = [3.255,4.1,5,15]
    // 遍历数组里面的元素,然后格式化一下,添加到 costList里去
    arr.map(item => {
        this.costList.push = this.formatCost(item)
    })
	


**三,** 所有页面里面的created生命周期函数  都会在小程序加载的时候，
一次性执行，而不是没进入一个页面执行一次，如，我有3个页面
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

这个其实很好解决，用mounted或者onLoad或者onReady代替，说到这几个函数，那就顺便提一下,
这里的created和mounted是vue(mpvue)的生命周期，而onLoad、onReady是小程序的生命周期，mpvue官方给的说明是：
> 除了 Vue 本身的生命周期外，mpvue 还兼容了小程序生命周期，这部分生命周期钩子的来源于微信小程序的
 Page， **除特殊情况外，不建议使用小程序的生命周期钩子。**

但是官方给的生命周期图示里面，也表明了，小程序的onLoad、onReady比created、mounted执行的早，
也就是说如果我们在和onLoad onReady里面去请求数据的话，会相对的减少白屏时间（这里说的白屏是指数据未渲染的界面），
而且官方没说明为什么不建议使用小程序的生命周期，我们也尝试了，用小程序的生命周期，没发现生命问题，
所以我们还是比较倾向优先使用小程序的生命周期，毕竟用户体验才是王道。

mpvue的生命周期函数里面，还有一个比较容易坑到人的地方，就是onShow,onShow这个函数，在page页面里面，
是按我们的预期执行，**当小程序启动，或从后台进入前台显示** 都会执行，没问题。可是如果是组件里面的onShow就不
同了，在组件里面，onShow在组件第一次加载的时候，是不会执行的，
只有在**该组件所在的页面从后台进入前台显示**的时候，才会执行。

说到这里，我们顺便看看小程序的页面跳转方式，小程序在一个页面跳转（调用wx.navigateTo）到另一个页面的时候，
并不会销毁原来的页面，而是转到后台去，并且执行原页面里面的onHide里的代码，
这也是为什么小程序的页面路径最多只能十层，因为你访问过的页面，正常都会保存在内存里，相当于vue里的keep-alive，
如果允许跳转非常多页面的话，很容易导致使用过大。

当然，我们也可以使用wx.navigateBack wx.redirectTo wx.reLaunch 来销毁页面，这3个方法，会调用页面的onUnload函数

四、组件名称不能用tabBar list 等等，这个是我碰到的比较经典的坑，自己手写了一个tabBar，内容可以正常显示，
页面里面的js逻辑，死活不会执行，加上 本文 第一点提到的，不会报错，让我一顿好找啊...

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

五、用 v-for循环的时候，如果要给当前项指定一个索引，在vue下，为了省事，我通常喜欢这样做

    v-for="item,index in list"

因为多打一对括号真的是很烦人。但是在mpvue下面却不行，你必须老老实实这样写，否则会报错。

    v-for="(item,index) in list"


六、单独为每个页面的设置页面头部信息，有提供这个功能，不过文档不是很详细，几经尝试，才试出来。

我们的入口文件main.js（延续vue的叫法，暂且这么称呼吧，其实我觉得应该叫配置文件）里面可以这样配置，
官方文档大概也是这么说的

>这部分内容来源于 app 和 page 的 entry 文件，通常习惯是 main.js，你需要在你的入口文件中
export default { config: {} }，这才能被我们的 loader 识别为这是一个配置，需要写成 json 文件。

    import Vue from 'vue';
    import App from './app';

    const vueApp = new Vue(App);
    vueApp.$mount();

    // 这个是我们约定的额外的配置
    export default {
        // 这个字段下的数据会被填充到 app.json ／ page.json
        config: {
            pages: ['static/calendar/calendar', '^pages/list/list'], // Will be filled in webpack
            window: {  // 顶部栏的统一配置
                backgroundTextStyle: 'light',
                navigationBarBackgroundColor: '#455A73',
                navigationBarTitleText: '美团汽车票',
                navigationBarTextStyle: '#fff'
            }
        }
    };

>同时，这个时候，我们会根据 entry 的页面数据，自动填充到 app.json 中的 pages 字段。
pages 字段也是可以自定义的，约定带有 ^ 符号开头的页面，会放到数组的最前面。

我们看到，可以在config.window下面配置全局的顶部栏样式，但是如果我们想为每个页面指定一个样式呢？事实上，
以上方法只适合配置app.json里面的内容，如果你想要为你的每个页面都添加一种样式，那么应该这样做：
在页面所属的入口文件（main.js）里面添加以下内容，比如我想为 userCenter/index页面设置一个标题，
应该在userCenter/main.js里面加入

    export default {
      config: {
        navigationBarTitleText: '个人中心',
      }
    }

**注意** 这里跟上面那个全局配置不同的是，配置内容navigationBarTitleText是config的属性，
而全局配置里，则是config.window的属性

七、组件的命名问题，有一次，我写了一个局部的组件，为什么叫局部的组件呢，因为我只在某个页面里面使用，
所以为了简单化，我给这个组件取了个名字叫`list.vue`,然后在父组件引用：

    <template>
    <!-- 省略其他代码 -->
        <list />
    </template>
    <script>
      import list from './components/list'
      export default {
        components: {list},
        // 省略其他代码
      }
    </script>

组件能正常显示，样式也没问题，一切看上去都是那么的正常，然而组件里面的逻辑就是不会执行。
经过排查发现，跟组件的引入名称有关，应该是跟微信的关键字同名了。

    <template>
    <!-- 省略其他代码 -->
        <listA />
    </template>
    <script>
      import listA from './components/list'
      export default {
        components: {listA},
        // 省略其他代码
      }
    </script>

这样就能正常运行，出了list,我目前踩到的还有tabbar,搞得我现在命名的时候，看到一些疑似关键的字眼，心理都有点阴影。。
这个应该是微信的问题吧，总之遇到了，就一块写出来。

八、组件第一次加载的时候不能执行onShow里面的内容，只有在隐藏又显示后，才会显示，而页面却每次进入都会显示
例如我们在一个组件里有一下代码

    onLoad () {
      console.log('onLoad')
    },
    onShow () {
      console.log('onShow')
    },
    mounted () {
      console.log('mounted')
    },

页面加载的时候，我们期望打印出来的是

    onLoad
    onShow
    mounted

然后实际上，只打印出

    onLoad
    mounted
这个问题，我已给官方提[Issue](https://github.com/Meituan-Dianping/mpvue/issues/179),不过目前还没得到回应

九、同一个子组件,在2个不同的地方引用,会导致2个地方的样式都加载不了,而如果只在一个地方引用却没问题，
为什么把这个问题放到最后？ 因为这只是前几个版本的脚手架有这个问题，后面的应该就没有这个问题了。
这个问题我也给官方提过Issue,官方给的回答是用新版本的脚手架重新生成项目，但是项目都快做完了，
这个时候重新生成，然后拷贝代码，感觉心太累了，所以抱着不折腾不罢休的态度，终于找到原因，是因为早期版本的脚手架，
缺少了 webpack-mpvue-asset-plugin 这个插件,新版的cli里面会自动添加这个插件。具体看[Issue #180](https://github.com/Meituan-Dianping/mpvue/issues/180)

还有一些官方明确指出的问题，这里就不一一列举了，有兴趣的童鞋可以直接查看[mpvue官方文档](http://mpvue.com/mpvue/)

另外，最近正在做一个mpvue的基础教程，有兴趣的童鞋请前往我的
[github](https://github.com/noahlam) [mpvue-tutorials](https://github.com/noahlam/mpvue-tutorials),
您的一个Star,就是我最大的动力了。


