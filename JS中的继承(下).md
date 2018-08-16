### JS中的继承(下)
> 在上一篇 [JS中的继承(上)](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E7%9A%84%E7%BB%A7%E6%89%BF(%E4%B8%8A).md)
 我们介绍了3种比较常用的js继承方法，如果你没看过，那么建议你先看一下，因为接下来要写的内容，
是建立在此基础上的.另外本文作为我个人的读书笔记,才疏学浅,如有错误,敬请指正.

接下来我们要介绍另外3种相对比较奇葩的继承  

#### **一. 原型式继承**
```javascript
function clone (proto) {
    function F () {}
    F.prototype = proto
    return new F()
}
```
clone 内部首先是创建了一个空的构造函数F,然后把F的prototype指向参数proto,最后返回一个F的实例对象,完成继承.
`原型式继承`看起来跟`原型继承`很像,事实上,两者因为都是基于prototype继承的,所以也有一些相同的特性,比如`引用属性共享问题`,
那`原型式继承`跟`原型继承`有什么区别呢? 一个比较明显的区别就是`clone`函数接收的参数不一定要是构造函数,也可以是其他任何对象,
这样我们就相当于是**浅复制**了一个对象.  

> es5的`Object.create()`函数,就是基于原型式继承的  

> 原型式继承是`道格拉斯-克罗克福德` 2006 年在 `Prototypal Inheritance in JavaScript`一文中提出的

#### **二. 寄生式继承**

寄生式继承其实就是在原型式继承的基础上,做了一些增强.
```javascript
function cloneAndStrengthen(proto){
    function F () {}
    F.prototype = proto
    let f = new F()
    f.say = function() { 
        console.log('I am a person')
    }
    return f
}
```
我们看到上面的代码,跟`原型式继承`比,差别就是:在实例对象f返回之前,给f添加了一个say函数.
但是这样在实例对象上添加的引用属性(比如函数),跟`构造函数模式`一样,
实例对象的引用类型属性无法共享,尽管这既是缺点也是优点.

#### **三. 寄生组合式继承**    

上一篇我们提到的`组合继承`其实也有个缺点,就是**父类构造函数里面的代码会执行2遍**,第一遍是在原型继承的时候实例化父类,
第二遍是在子类的构造函数里面借用父类的构造函数,我们可以用寄生组合式继承来解决这个问题
```javascript
function inherit(sub, super){
    let prototype = clone(super.prototype)
    prototype.constructor = sub    
    sub.prototype = prototype      
}
```
这样我们就实现了一个寄生组合式继承的函数inherit,接下来我们来使用一下:
```javascript
function Person(name){}

function Student(){
    SuperType.call(this)
}

inherit(Student, Student)
 ```   
我们用 `inherit` 函数替换了 `Student.prototype = new Person() `,从而避免了执行 `new Person()`.

> 书上说: 寄生组合式继承是引用类型最理想的继承范式.
