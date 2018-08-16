### JS中的原型对象

> 白天写了一篇[【JS中创建对象的方法】](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%96%B9%E6%B3%95.md)，写完以后感觉意犹未尽（实际情况是感觉原型那块内容没有交
代清楚），所以开这一篇继续聊聊关于JavaScript中的原型对象

相信用过vue的童鞋，都经常这样做，用Vue.prototype.xxx = xxx 把一个方法或者属性添加到Vue对象的原型上，
这样，我们在vue实例的任何地方，都可以用这个方法或属性了，我最喜欢用的，就是把异步请求库
（我比较喜欢用axios）挂载到vue原型上：
```javascript
   // 一般是./src/mian.js
    // 这里为了方便理解就直接引入axios，实际使用，我们可以先用axios封装一个异步请求模块
    // 在模块里做一些拦截或者处理，然后再导入这个模块。具体做法看
    import axios from 'axios'
    import Vue from 'vue'
    import App from './App'
    // 为所有Vue实例添加一个post模块，可以在vue实例中直接使用this.post
    Vue.prototype.post = axios
    new Vue({
      el: '#app',
      components: { App },
      template: '<App/>'
    })
```
 

这了是prototype就是我们所说的原型，那什么是原型呢？ 我们看看MDN给的解释
> 当谈到继承时，JavaScript 只有一种结构：对象。每个对象都有一个私有属性（称之为 \[\[Prototype\]\]），
它指向它的原型对象（prototype）。该 prototype 对象又具有一个自己的 prototype ，
层层向上直到一个对象的原型为 null。根据定义，null 没有原型，并作为这个原型链中的最后一个环节。

视乎看起好有点拗口，没关系，我用自己的话总结了一下
> 1.prototype其实就是存在于对象中的一个特殊的对象，你可以把它理解为对象的一个属性或方法，
如 a.prototype,看起来是不是很像对象a的一个属性呢？

> 2.每个对象都有一个prototype,除了null

那这个prototype是干嘛的呢？ 其实回头看看上面关于vue的代码就知道了，
prototype最主要的作用就是该原型所属对象的所有实例，都能共享prototype里的属性和方法
上面的代码中，通过向Vue.prototype中添加一个post方法，然后就可以在所有vue实例中使用该方法，就是个简单的实践。

我们回头看看 [【JS中创建对象的方法】](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E5%88%9B%E5%BB%BA%E5%AF%B9%E8%B1%A1%E7%9A%84%E6%96%B9%E6%B3%95.md)里面的原型模式
```javascript
    function Student(){}   // 声明一个空函数
    Student.prototype.name = 'xiaohong'
    Student.prototype.age = 17
    Student.prototype.gender = 'f'
    Student.prototype.study = fucntion() { console.log('我在学习...')}
```


我们先定义了一个空函数，注意：这个时候，我们并没有认为的给函数添加一个prototype属性/方法，
而Student却自动有了prototype，然后我们往prototype里面添加了name,age,gender属性和study方法，
然后我们用new实例化2个Student对象出来

```javascript
    var studentA = new Student()
    console.log(studentA.name)    // xiaohong
    console.log(studentA.age)     // 17
    console.log(studentA.gender)  // f
    studentA.study()              // 我在学习...

    var studentB = new Student()
    console.log(studentB.age)     // xiaohong
    console.log(studentB.name)    // 17
    console.log(studentB.gender)  // f
    studentB.study()              // 我在学习...
```

上面的例子可以看出，对象的prototype里面的属性和方法，在该对象的所有实例里面，都是共享的
那如果我们想要让实例对象有自己的属性/方法，该怎么办呢？ 比如，我想让studentB的名字是'lili',
很简单，直接在实例对象上添加该属性/方法：

```javascript
    studentA.name = 'lili'
    studentA.study = function () {
        console.log('我在偷懒')
    }

    console.log(studentA.name)  // lili
    console.log(studentB.name)  // xiaohong
    studentA.study()            // 我在偷懒
    studentB.study()            // 我在学习...
```

可以看出，studentA的属性/方法被改变的时候，studentB没有对应的跟着改变，这是为什么呢？
不是说好的全所有prototype里的属性/方法都是共享的吗？事实上，prototype里的属性/方法，确实是共享的，
问题出在我们是在实例对象上赋值，所以这个属性/方法，是属于实例的，而不是属于prototype的，prototype的属性，
也无法在实例对象上写入，也就是说，实例对象和prototype上，同时存在了 name属性和study方法，那么，
为什么 studentA 和 studentB 访问到的属性/方法 会不一样呢？ 其实每次访问一个属性/方法的时候，
都会先从实例对象开始查找，如果实例上有，就直接返回，如果实例上没有，就继续往prototype上查找,有就返回，
如果prototype上还有 prototype，那么还会继续网上查找，直到原型链的最顶层。如果都没有查到，则会返回undefined。

那么新的问题来了，我们该如何判断一个属性，是属于实例本身的，还是属于prototype的？ 答案是hasOwnProperty方法，
hasOwnProperty方法可以检测到实例对象里面有没有给定的属性，**该方法只能检测到实例里面的属性，检测不到prototype上的**

```javascript
    studentA.hasOwnProperty('name')  // true
    studentB.hasOwnProperty('name')  // false
```

那如果想同时查找实例对象和原型对象prototype呢？我们可以用 in 操作符

```javascript
    'name' in studentA    // true
    'name' in studentB    // true
```

有了这2个方法，我们就可以组合起来判断属性是属于实例还是原型了。

> 这里老是说到实例，不得不提一下，**实例对象**虽然是构造函数“构造”出来的，但是其实跟构造函数没有直接联系，
实例对象内部指向的是构造函数的prototype（原型）。 实例跟构造函数的一个间接关系是
**实例.prototype.constructor --> 构造函数**

关于原型的介绍就到这里，有需要更深入的童鞋，建议去读一下javascript权威指南。里面关于原型的介绍比我这详细。

下面列几个javascript权威指南里面介绍的关于原型的方法

1. 获取一个实例对象的原型 (ES5才支持)

    `Object.getPrototypeOf(studentA)  // Student.prototype `

    部分浏览器（chrome,safari,firefox）也支持一个属性 `__proto__`

    `studentA.__proto__ ==  Student.prototype`

2. 判断一个构造函数是否是`指定实例对象`的原型

    `Student.prototype.isPrototypeOf(studentA) // true`

好了，关于原型对象的话题，就到这里了，感谢收看！
