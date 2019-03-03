### 前言

本文纯属个人平时实践过程中的一些经验总结，算是一点点小技巧吧，不是多么高明的技术，如果对你有帮助，那么不胜荣幸。

本文不涉及`罕见API`使用方法等，大部分内容都是基于对vue的一些实践而已。由于涉嫌投机取巧，可能会带来一些不符合规范的副作用，请根据项目要求酌情使用。

1. **多个页面都使用的到方法，放在 `vue.prototype` 上会很方便**

    刚接触 `vue` 的时候做过一件傻事，因为封装了一个异步请求接口`post`,放在 `post.js` 文件里面，然后在每个需要使用异步请求的页面引入
    ```javascript
    import port from './xxxx/xxxx/post'
    ```
    如果只是这样，还没什么，我们可以写好一个页面以后再复制，可以保证每个页面都有上面的语句。但是如果每个文件所在的目录层级不一样呢？
    ```javascript
    // 假设正常是这样
    import port from '../xxxx/xxxx/post'
    // 目录加深一级，就变成这样
    import port from '../../xxxx/xxxx/post'
    // 再加深一级的样子
    import port from '../../../xxxx/xxxx/post'
    ```
    当然，这个时候，我们可以用 别名 `@/xxxx/post`,但是还是少不了要每个页面引用。
    那我们来看看，用`vue.prototype` 有多方便？
    首先，你得在 `vue` 的入口文件( `vue-cli` 生成的项目的话，默认是 `/src/main.js`)里面做如下设置
    ```javascript
     import port from './xxxx/xxxx/post'

     vue.prototype.$post = post
    ```
    这样，我们就可以在所有的 `vue` 组件（页面）里面使用 `this.post()` 方法了，就像 `vue` 的亲儿子一样

    > tip: 把方法挂在到 `prototype` 上的时候，最好加一个 `$` 前缀，避免跟其他变量冲突

    > til again: 不要挂载太多方法到 `prototype` 上，只挂载一些使用频率非常高的

1. **需要响应的数据，在获取到接口数据的时候，先设置**

    大家有没有很经常碰到这样都一种情况，在循环列表的时候，我们需要给列表项一个控制显示的属性，如 是否可删除，是否已选中等等，而后端接口一般不会返回这种字段，因为这属于纯前端展示的，跟后端没啥关系，比如后端给的数据如下
    ```javascript
    [
      {name: 'abc', age: 18},
      {name: 'def', age: 20},
      {name: 'ghi', age: 22},
    ]
    ```
    我们不妨假设以上数据为学生列表

    然后我们需要渲染这个列表，在每一项后面显示一个勾选按钮，如果用户打勾，则这个按钮是绿色，默认这个按钮是灰色，这个时候，上表是没有满足这个渲染条件的数据，而如果我们在用户打勾的时候，再去添加这个数据的话，正常的做法是无法及时响应的。

    如果我们在获取到数据的时候，先给数组的每一项都加一个是否打勾的标示，就可以解决这个问题，我们假设我们获取到的数据是 `res.list`
    ```javascript
    res.list.map(item => {
      item.isTicked ＝ false
    })
    ```
    这么做的原理是 `vue` 无法对不存在的属性作响应，所以我们在获取到数据的时候，先把需要的属性加上去，然后在赋值给 `data` , 这样 `data` 接收到数据的时候，已经是存在这个属性了，所以会响应。当然还有其他方法可以实现。不过对于一个强迫症来说，我还是比较倾向于这种做法

1. **封装全局基于 `promise` 的异步请求方法**

    看过很多项目的源码，发现大部分的异步请求都是直接使用 `axios` 之类的方法，如下
    ```javascript
    axios({
      method: 'post',
      url: '/user/12345',
      data: {
        firstName: 'Fred',
        lastName: 'Flintstone'
      }
    })
     .then(function (response) {
        console.log(response);
      })
      .catch(function (error) {
        console.log(error);
      });
    ```
    如果有跨域，或者需要设置 `http` 头等，还需要加入更多的配置，而这些配置，对于同一个项目来说，基本都是一样的，不一样的只有 `url` 跟参数，既然这样，那我吗为什么不把它封装成一个方法呢？
    ```javascript
    function post (url,param) {
      return new Promise((resolve, reject) => {
        axios({
          method: 'post',
          url: url,
          data: param
        })
         .then(function (response) {
            resolve(response);
          })
          .catch(function (error) {
            reject(error);
          });
      })
    }

    ```
    再结合第一点，我们就可以再任意 `vue` 实例中这样使用
    ```javascript
    let param = {
      firstName: 'Fred',
      lastName: 'Flintstone'
    }
    this.post('/user/12345',param)
    .then(...)
    .catch(...)
    ```
    有没有比原始的简单很多呢？如果你的项目支持 `async` `await`，还可以这样用
    ```javascript
    let param = {
      firstName: 'Fred',
      lastName: 'Flintstone'
    }
    let res  = await this.post('/user/12345',param)
    console.log(res) // res 就是异步返回的数据

    ```
<<<<<<< HEAD
1. **如果你觉得有时候，你真的需要父子组件共享一个值，不如试试传个引用类型过去**

    `vue` 的父子组件传值，有好多种方法，这里就不一一列举了，但是今天我们要了解的，是利用 `javascript` 的引用类型特性，还达到另一种传值的目的

    假设有这么一个需求，父组件需要传 3 个值到子组件，然后再子组件里面改动后，需要立马再父组件上作出响应，我们通常的做法上改完以后，通过 `this.$emit` 发射事件，然后再父组件监听对应的事件，然而这么做应对一两个数据还好，如果传的数据多了，会累死人。
    我们不妨把这些要传递的数据，包再一个对象／数组 里面，然后在传给子组件

    ```html
    <subComponent :subData="subData"></subComponent>
    ```
    ```javascript
    data () {
      return {
        subData: {
          filed1: 'field1',
          filed2: 'field2',
          filed3: 'field3',
          filed4: 'field4',
          filed5: 'field5',
        }
      }
    }
    ```

    这样，我们在子组件里面改动 `subData` 的内容，父组件上就能直接作出响应，无需 `this.$emit` 或 `vuex` 而且如果有其他兄弟组件的话，只要兄弟组件也有绑定这个 `subData` ，那么兄弟组件里面的 `subData` 也能及时响应

    > tip: 首先，这么做我个人上感觉有点不符合规范的，如果没有特别多的数据，还是乖乖用 `this.$emit` 吧，其次，这个数据需要有特定的条件才能构造的出来，并不是所有情况都适用。

1. **异步请求的参数在 `data` 里面构造好，用一个对象包起来，会方便很多**

    有做过类似 `ERP` 类型的系统的同学，一定碰到过这样的一个场景，一个列表，有 N 个过滤条件，这个时候通常我们这么绑定
    ```html
     <input type="text" v-model="field1">
     <input type="text" v-model="field2">
     <input type="text" v-model="field3">
     ....
     <input type="text" v-model="fieldn">
    ```

    ```javascript
    data () {
     return {
       field1: 'value1',
       field2: 'value2',
       field3: 'value3',
       ...
       fieldn:'valuen'
     }
    }
    ```
    然后提交数据的时候这样：
    ```javascript
     var param = {
       backend_field1: this.field1,
       backend_field2: this.field2,
       backend_field3: this.field3,
       ...
       backend_fieldn: this.fieldn
     }
     this.post(url,param)
    ```

    如你看到的，每次提交接口，都要去构造参数，还很容易遗漏，我们不妨这样：先去接口文档里面看一下后端需要的字段名称，然后

     ```html
         <input type="text" v-model="queryParam.backend_field1">
         <input type="text" v-model="queryParam.backend_field2">
         <input type="text" v-model="queryParam.backend_field3">
         ....
         <input type="text" v-model="queryParam.backend_fieldn">
        ```

        ```javascript
        data () {
         return {
           queryParam:{
             backend_field1: 'value1'
             backend_field2: 'value2'
             backend_field3: 'value3'
             ...
             backend_fieldn: 'valuen'
           }
         }
        }
        ```
        然后提交数据的时候这样：
        ```javascript
         this.post(url,this.queryParam)
        ```
     是的，这样做也是有局限性的，比如你一个数据在 2 个地方共用，比如前端组件绑定的是一个数组，你需要提交给后端的是 2 个字符串（例：`element ui` 的时间控件）,不过部分特殊问题稍微处理一下，也比重新构建一个参数简单不是吗？


1. **`data` 里面的数据多的时候，给每个数据加一个备注，会让你后期往回看的时候很清晰**

    续上一点，`data` 里面有很多数据的时候，可能你写的时候是挺清晰的，毕竟都是你自己写的东西，可是过了十天半个月，或者别人看你的代码，相信我，不管是你自己，还是别人，都是一头雾水（记忆力超出常人的除外），所以我们不妨给每个数据后面加一个备注
    ```javascript
    data () {
     return {
       field1: 'value1',  // 控制xxx显示
       field2: 'value2',  // 页面加载状态
       field3: [],        // 用户列表
       ...
       fieldn: 'valuen'   // XXXXXXXX
     }
    }
    ```
1. **逻辑复杂的内容，尽量拆成组件**

    假设我们有一个这样的场景：
    ```html
    <div>
       <div>姓名：{{user1.name}}</div>
       <div>性别：{{user1.sex}}</div>
       <div>年龄：{{user1.age}}</div>
       ...此处省略999个字段...
       <div>他隔壁邻居的阿姨家小狗的名字：{{user1.petName}}</div>
    </div>
    <-- 当然，显示中我们不会傻到不用 v-for,我们假设这种情况无法用v-for -->
    <div>
        <div>姓名：{{user2.name}}</div>
        <div>性别：{{user2.sex}}</div>
        <div>年龄：{{user2.age}}</div>
        ...此处省略999个字段...
        <div>他隔壁邻居的阿姨家小狗的名字：{{user2.petName}}</div>
    </div>
    ```
    这种情况，我们不妨把[用户]的代码，提取到一个组件里面：
    假设如下代码，在 `comUserInfo.vue`
    ```vue
    <template>
     <div>
       <div>姓名：{{user.name}}</div>
       <div>性别：{{user.sex}}</div>
       <div>年龄：{{user.age}}</div>
       ...此处省略999个字段...
       <div>他隔壁邻居的阿姨家小狗的名字：{{user.petName}}</div>
     </div>
    </template>

    <script >
    export  default {
     props:{
       user:{
         type:Object,
         default: () => {}
       }
     }
    }
    </script>
    ```
    然后原来的页面可以改成这样(省略掉导入和注册组件，假设注册的名字是 `comUserInfo` )：
    ```html
    <comUserInfo :user="user1"/>
    <comUserInfo :user="user2"/>
    ```
    这样是不是清晰很多？不用看注释，都能猜的出来，这是2个用户信息模块， 这样做，还有一个好处就是出现错误的时候，你可以更容易的定位到错误的位置。

1. **如果你只在子组件里面改变父组件的一个值，不妨试试 `$emit('input')` ,会直接改变 `v-model`**

    我们正常的父子组件通信是 父组件通过 `props` 传给子组件，子组件通过 `this.$emit('eventName',value)` 通知父组件绑定在 `@eventName` 上的方法来做相应的处理。
    但是这边有个特例，`vue` 默认会监听组件的 `input` 事件，而且会把子组件里面传出来的值，赋给当前绑定到 `v-model` 上的值

    正常用法 - 父组件
    ```vue
    <template>
      <subComponent :data="param" @dataChange="dataChangeHandler"></subComponent>
    </template>

    <script >
      export default {
        data () {
          return {
            param:'xxxxxx'
          }
        },
        methods:{
          dataChangeHandler (newParam) {
            this.param = newParam
          }
        }
      }
    </script>
    ```
    正常用法 - 子组件
    ```vue
    <script >
      export default {
        methods:{
          updateData (newParam) {
            this.$emit('dataChange',newParam)
          }
        }
      }
    </script>
    ```

    **利用默认 `input` 事件 - 父组件**
    ```vue
    <template>
      <subComponent  v-model="param"></subComponent>
    </template>
    ```
    **利用默认 `input` 事件 - 子组件**
    ```vue
    <script >
      export default {
        methods:{
          updateData (newParam) {
            this.$emit('input',newParam)
          }
        }
      }
    </script>
    ```
    这样，我们就能省掉父组件上的一列席处理代码，`vue` 会自动帮你处理好

    > tip: 这种方法只适用于改变单个值的情况，且子组件对父组件只需简单的传值，不需要其他附加操作(如更新列表)的情况。

1. **`conponents`放在 `Vue options` 的最上面**

    不知道大家有没有这样的经历: 导入组件，然后在也页面中使用，好的，报错了，为啥？忘记注册组件了，为什么会经常忘记注册组件呢？因为正常的一个 `vue` 实例的结构大概是这样的：
    ```javascript
    import xxx form 'xxx/xxx'
    export default {
      name: 'component-name',
      data () {
        return {
          // ...根据业务逻辑的复杂程度，这里省略若干行
        }
      },
      computed: {
        // ...根据业务逻辑的复杂程度，这里省略若干行
      },
      created () {
        // ...根据业务逻辑的复杂程度，这里省略若干行
      },
      mounted () {
        // ...根据业务逻辑的复杂程度，这里省略若干行
      },
      methods () {
        // ...根据业务逻辑的复杂程度，这里省略若干行
      },
    }
    ```
    我不知道大家正常是把 `components` 属性放在哪个位置，反正我之前是放在最底下，结果就是导致经常犯上述错误。

    后面我把 `components` 调到第一个去了

    ```javascript
    import xxx form 'xxx/xxx'
    export default {
      components: {
        xxx
      },
      // 省略其他代码
    }
    ```
    从此以后，妈妈再也不用担心我忘记注册组件了，导入和注册都在同一个位置，想忘记都难。

1. **大部分情况下，生命周期里面，不要有太多行代码，可以封装成方法，再调用**

    看过很多代码，包括我自己之前的，在生命周期里面洋洋洒洒的写了一两百行的代码，如：把页面加载的时候，该做的事，全部写在 `created` 里面，导致整个代码难以阅读，完全不知道你在页面加载的时候，做了些什么，
    这个时候，我们不妨把那些逻辑封装成方法，然后在生命周期里面直接调用：
    ```javascript
    created () {
      // 获取用户信息
      this.getUserInfo()
      // 获取系统信息
      this.getSystemInfo()
      // 获取配置
      this.getConfigInfo()
    },
    methods:{
      // 获取用户信息
      getUserInfo () {...},
      // 获取系统信息
      getSystemInfo () {...},
      // 获取配置
      getConfigInfo () {...},
    }
    ```
    这样是不是一眼就能看的出，你在页面加载的时候做了些什么？

    > tip: 这个应该算是一个约定俗成的规范吧，只是觉得看的比较多这样写的，加上我自己初学的时候，也这么做了，所以写出来，希望新入坑的同学能避免这个问题

1. **少用 `watch`，如果你觉得你好多地方都需要用到 `watch`，那十有八九是你对 `vue` 的 `API` 还不够了解**

    `vue` 本身就是一个数据驱动的框架，数据的变动，能实时反馈到视图上去，如果你想要根据数据来控制试图，正常情况一下配合 `computed` 服用就能解决大部分问题了，而视图上的变动，我们一般可以通过监听 `input` `change` 等事件，达到实时监听的目的，
    所以很少有需求使用到 `watch` 的时候,至少我最近到的十来个项目里面，是没有用过 `watch` 当然，并不是说 `watch` 是肯定没用处, `vue` 提供这个api,肯定是有他的道理，也有部分需求是真的需要用到的，只是我觉得应该很少用到才对，如果你觉得到处都得用到的话，
    那么我觉得 **十有八九**你应该多去熟悉一下 `computed` 和 `vue` 的其他 `api` 了


### 最后

[本文的github地址](https://github.com/noahlam/articles) 欢迎随意[star](https://github.com/noahlam/articles),[follow](https://github.com/noahlam), 和 不随意的 [issue](https://github.com/noahlam/articles/issues)

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，
有兴趣的童鞋可以看看，你们的支持就是我最大的动力。
