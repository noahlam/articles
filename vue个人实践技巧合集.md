### 前言  

本文纯属个人平时实践过程中的一些经验总结，算是一点点小技巧吧，不是多么高明的技术，如果对你有帮助，那么不胜荣幸。

本文不涉及`罕见API`使用方法等，大部分内容都是基于对vue的一些实践而已。  

由于投机取巧，可能会带来一些不符合规范的副作用，请根据项目要求酌情使用。  

1. #### **多个页面都使用的到方法，放在 vue.prototype上会很方便**   
    刚接触vue的时候很傻，因为封装了一个异步请求接口`post`,放在post.js文件里面，然后在每个需要使用异步请求的页面引入
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
    首先，你得在vue的入口文件(vue-cli生成的项目的话，默认是/src/main.js)里面做如下设置
    ```javascript
     import port from './xxxx/xxxx/post'
  
     vue.prototype.$post = post   
    ```
    这样，我们就可以在所有的vue组件（页面）里面使用this.post() 方法了，就像vue的亲儿子一样
    
    > tip: 把方法挂在到prototype上的时候，最好加一个$前缀，避免跟其他变量冲突
    
    > til again: 不要挂载太多方法到prototype上，只挂载一些使用频率非常高的
    
1. #### **需要响应的数据，在获取到接口数据的时候，先设置** 
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
    
    如果我们在获取到数据的时候，先给数组的每一项都加一个是否打勾的标示，就可以解决这个问题，我们假设我们获取到的数据是 res.list
    ```javascript
    res.list.map(item => { 
      item.isTicked ＝ false
    })
    ```
    这么做的原理是 vue 无法对不存在的属性作响应，所以我们在获取到数据的时候，先把需要的属性加上去，然后在赋值给 `data` , 这样 `data` 接收到数据的时候，已经是存在这个属性了，所以会响应。当然还有其他方法可以实现。不过对于一个强迫症来说，我还是比较倾向于这种做法

1. #### **封装全局基于promise的方法** 
    看过很多项目的源码，发现大部分的异步请求都是直接使用 axios之类的方法，如下
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
    如果有跨域，或者需要设置http头等，还需要加入更多的配置，而这些配置，对于同一个项目来说，基本都是一样的，不一样的只有url跟参数，既然这样，那我吗为什么不把它封装成一个方法呢？
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
    再结合第一点，我们就可以再任意vue实例中这样使用
    ```javascript
    let param = {
      firstName: 'Fred',
      lastName: 'Flintstone'
    }
    this.post('/user/12345',param)
    .then(...)
    .catch(...)
    ```
    有没有比原始的简单很多呢？如果你的项目支持async await，还可以这样用
    ```javascript
    let param = {
      firstName: 'Fred',
      lastName: 'Flintstone'
    }
    let res  = this.post('/user/12345',param)
    console.log(res) // res 就是异步返回的数据
    
    ```
1. #### **如果你觉得有时候，你真的需要父子组件共享一个值，不如试试传个引用类型过去** 
    vue 的父子组件传值，有好多种方法，这里就不一一列举了，但是今天我们要了解的，是利用javascript的引用类型特性，还达到另一种传值的目的  
    假设有这么一个需求，父组件需要传3个值到子组件，然后再子组件里面改动后，需要立马再父组件上作出响应，我们通常的做法上改完以后，通过this.$emit发射事件，然后再父组件监听对应的事件，然而这么做应对一两个数据还好，如果传的数据多了，会累死人。
    我们不妨把这些要传递的数据，包再一个对象／数组 里面，然后在传给子组件
    
    ```html
    <subComponent :subData="subData"></subComponent>
    ```
    ```javascript
    data () {
      return {
        subData: {
          filed1:'field1',
          filed2:'field2',
          filed3:'field3',
          filed4:'field4',
          filed5:'field5',
        }
      }
    }
    ```
    
    这样，我们在子组件里面改动 `subData` 的内容，父组件上就能直接作出响应，无需 `this.$emit` 或 `vuex` 而且如果有其他兄弟组件的话，只要兄弟组件也有绑定这个 `subData` ，那么兄弟组件里面的 `subData` 也能及时响应
    
    > tip: 首先，这么做我个人上感觉有点不符合规范的，如果没有特别多的数据，还是乖乖用 `this.$emit` 吧，其次，这个数据需要有特定的条件才能构造的出来，并不是所有情况都适用。 

1. #### **参数用一个对象包起来会让你很方便** 
1. #### **data里面的数据多的时候，给每个数据加一个备注，会让你后期往回看的时候很清晰** 
1. #### **逻辑复杂的，拆成组件** 
1. #### **如果你只在子组件里面改变父组件的一个值，不妨试试$emit('input'),会直接改变v-model** 
1. #### **大部分情况下，生命周期里面，不要有太多行代码，可以封装成方法，再调用** 
1. #### **少用watch，如果你觉得你好多地方都需要用到watch，那十有八九是你对vue的API还不够了解**
1.

### 最后

如果觉得本文对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢

另外，[github](https://github.com/noahlam/articles)上还有其他一些关于前端的教程和组件，
有兴趣的童鞋可以看看，你们的支持就是我最大的动力。