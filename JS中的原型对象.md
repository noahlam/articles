### JS中的原型对象

> 白天写了一篇【JS中创建对象的方法】，写完以后感觉意犹未尽（实际情况是感觉原型那块内容没有交
代清楚），所以开这一篇继续聊聊关于JavaScript中的原型对象

相信用过vue的童鞋，都经常这样做，用Vue.prototype.xxx = xxx 把一个方法或者属性添加到Vue对象的原型上，
这样，我们在vue实例的任何地方，都可以用这个方法或属性了，我最喜欢用的，就是把异步请求库
（我比较喜欢用axios）挂载到vue原型上：

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

我们回头看看 【JS中创建对象的方法】里面的原型模式

    function Student(){}   // 声明一个空函数
    Student.prototype.name = 'xiaohong'
    Student.prototype.age = 17
    Student.prototype.gender = 'f'
    Student.prototype.study = fucntion() { console.log('我在学习...')}

我们先定义了一个空函数，注意：这个时候，我们并没有认为的给函数添加一个prototype属性/方法，
而Student却自动有了prototype，然后我们往prototype里面添加了name,age,gender属性和study方法，
然后我们用new实例化2个Student对象出来

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

上面的例子可以看出，对象的prototype里面的属性和方法，在该对象的所有实例里面，都是共享的

------未完待续------

判断位置，是否存在
----查找-屏蔽
实例与构造函数没有直接关系，与prototype有关
studentA.__proto__ ==  Student.prototype
构造函数指向 Student.prototype.constructor == Student

studentA.hasOwnProperty('name')  // 只有实例自己有name属性，才返回true
name in studentA  // 查找实例+原型
Student.prototype.isPrototypeOf(studentA) // 判断是否是它的原型
Object.getPrototypeOf(studentA)  // Student.prototype (ES5)
在实例之后加入prototype的，也能访问。
直接改变prototype(导致prototype.constructor指向Object)，单独改变prototype，Object.assign(Student.prototype,{aaa,bbb})
