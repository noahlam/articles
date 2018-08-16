### JS中的继承(上)

> 学过java或者c#之类语言的同学,应该会对js的继承感到很困惑--不要问我怎么知道的,js的继承主要是基于原型(prototype)的,对js的原型感兴趣的同学,
可以了解一下我之前写的[JS中的原型对象](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1.md)

相信很多同学也跟我一样,刚开始接触js的面向对象编程的时候,都抱着一种排斥的心态--为什么js这么麻烦?
其实了解完原型链后,再来看js的继承,你会发现js的继承其实比其他OOP语言更简单,更灵活,我们来看一个基于原型链的继承
```javascript

// 父类
function Person() {}

// 子类
function Student(){}

// 继承
Student.prototype = new Person()
```

我们只要`把子类的prototype设置为父类的实例`,就完成了继承,怎么样? 是不是超级简单? 有没有比Java,C#的清晰?
事实上,以上就是js里面的**原型链继承**

当然,通过以上代码,我们的Student只是继承了一个空壳的Person,这样视乎是毫无意义的,我们使用继承的目的,
就是要通过继承获取父类的内容,那我们先给父类加上一点点简单的内容(新增的地方标记 '// 新增的代码'):
```javascript
// 父类
function Person(name,age) {
  this.name = name || 'unknow'     // 新增的代码
  this.age = age || 0              // 新增的代码
}

// 子类
function Student(name){
  this.name = name                 // 新增的代码
  this.score = 80                  // 新增的代码
}

// 继承
Student.prototype = new Person()
```
使用
```javascript
var stu = new Student('lucy')

console.log(stu.name)  // lucy    --子类覆盖父类的属性
console.log(stu.age)   // 0       --父类的属性
console.log(stu.score) // 80      --子类自己的属性
```
这里为了降低复杂度,我们只演示了普通属性的继承,没有演示方法的继承,事实上,方法的继承也很简单,
我们再来稍微修改一下代码,基于上面的代码,给父类和子类分别加一个方法(新增的地方标记 '// 新增的代码')
```javascript
// 父类
function Person(name,age) {
  this.name = name || 'unknow'
  this.age = age || 0
}

// 为父类新曾一个方法
Person.prototype.say = function() {         // 新增的代码
    console.log('I am a person')
}

// 子类
function Student(name){
  this.name = name
  this.score = 80
}

// 继承 注意,继承必须要写在子类方法定义的前面
Student.prototype = new Person()

// 为子类新增一个方法(在继承之后,否则会被覆盖)
Student.prototype.study = function () {     // 新增的代码
    console.log('I am studing')
}
```
使用
```javascript
var stu = new Student('lucy')

console.log(stu.name)  // lucy               --子类覆盖父类的属性
console.log(stu.age)   // 0                  --父类的属性
console.log(stu.score) // 80                 --子类自己的属性
stu.say()              // I am a person      --继承自父类的方法
stu.study()            // I am studing       --子类自己的方法
```
这样,看起来我们好像已经完成了一个完整的继承了,这个就是**原型链继承**,怎么样,是不是很好理解?
但是,原型链继承有一个缺点,就是属性如果是引用类型的话,会共享引用类型,请看以下代码
```javascript
// 父类
function Person() {
  this.hobbies = ['music','reading']
}

// 子类
function Student(){}

// 继承
Student.prototype = new Person()
```
使用
```javascript
var stu1 = new Student()
var stu2 = new Student()

stu1.hobbies.push('basketball')

console.log(stu1.hobbies)   // music,reading,basketball
console.log(stu2.hobbies)   // music,reading,basketball
```
我们可以看到,当我们改变stu1的引用类型的属性时,stu2对应的属性,也会跟着更改,这就是**原型链继承缺点** --引用属性会被所有实例共享,
那我们如何解决这个问题呢? 就是下面我们要提到的**借用构造函数继承**,我们来看一下使用构造函数继承的最简单例子:
```javascript
// 父类
function Person() {
  this.hobbies = ['music','reading']
}

// 子类
function Student(){
    Person.call(this)              // 新增的代码
}
```
使用
```javascript
var stu1 = new Student()
var stu2 = new Student()

stu1.hobbies.push('basketball')
console.log(stu1.hobbies)   // music,reading,basketball
console.log(stu2.hobbies)   // music,reading
```
这样,我们就解决了引用类型被所有实例共享的问题了

> 注意,这里跟 原型链继承 有个比较明显的区别是并没有使用prototype继承,而是在子类里面执行父类的构造函数,
相当于把父类的代码复制到子类里面执行一遍,这样做的另一个好处就是可以给父类传参
```javascript
// 父类
function Person(name) {
  this.name = name              // 新增的代码
}

// 子类
function Student(name){
    Person.call(this,name)      // 改动的代码
}
```
使用
```javascript
var stu1 = new Student('lucy')
var stu2 = new Student('lili')
console.log(stu1.name)   // lucy
console.log(stu2.name)   // lili
```
看起来已经很有Java,C#的味道了有没有?

但是,构造函数解决了引用类型被所有实例共享的问题,但正是因为解决了这个问题,导致一个很矛盾的问题出现了,**--函数也是引用类型**,
也没办法共享了.也就是说,每个实例里面的函数,虽然功能一样,但是却不是同一个函数,就相当于我们每实例化一个子类,就复制了一遍的函数代码
```javascript
// 父类
function Person(name) {
  this.say = function() {}    // 改动的代码
}

// 子类
function Student(name){
    Person.call(this,name)
}
```
使用
```javascript
var stu1 = new Student('lucy')
var stu2 = new Student('lili')
console.log(stu1.say === stu2.say)   // false
```
以上代码可以证明,父类的函数,在子类的实例下是不共享的

#### 总结
|  继承方式   |   继承核心代码   |   优缺点   |
|------------|---------------|----------|
|  原型链继承   |   `Student.prototype = new Person()`   |   实例的引用类型共享   |
|  构造函数继承   |   在子类(Student)里执行 `Person.call(this)`   |   实例的引用类型不共享   |

从上表我们可以看出 `原型链继承` 和 `构造函数继承` 这两种继承方式的优缺点刚好是互相矛盾的,那么我们有没有办法鱼和熊掌兼得呢?
没有的话,我就不会说出来了,^_^,接下来请允许我隆重介绍 **组合继承**

组合继承,就是各取上面2种继承的长处,**普通属性** 使用 `构造函数继承`,**函数** 使用 `原型链继承`,
这个代码稍微复杂一点,不过相信有了上面的基础后,看起来也是很轻松
```javascript
// 父类
function Person() {
  this.hobbies = ['music','reading']
}

// 父类函数
Person.prototype.say = function() {console.log('I am a person')}

// 子类
function Student(){
    Person.call(this)             // 构造函数继承(继承属性)
}
// 继承
Student.prototype = new Person()  // 原型链继承(继承方法)
```
使用
```javascript
// 实例化
var stu1 = new Student()
var stu2 = new Student()

stu1.hobbies.push('basketball')
console.log(stu1.hobbies)           // music,reading,basketball
console.log(stu2.hobbies)           // music,reading

console.log(stu1.say == stu2.say)   // true
```
这样,我们就既能实现属性的独立,又能做到函数的共享,是不是很完美呢?

> 组合继承据说是JavaScript中最常用的继承方式(具体无法考证哈).

至此,我们就把js里面的常用继承了解完了,其实也没有那么难嘛!不过,我们总结一下3种继承

1. 原型链继承,会共享引用属性
2. 构造函数继承,会独享所有属性,包括引用属性(重点是函数)
3. 组合继承,利用原型链继承要共享的属性,利用构造函数继承要独享的属性,实现相对完美的继承

上面为什么要说相对完美呢? 因为本文的标题叫【JS中的继承(上)】,那肯定是还有
[【JS中的继承(下)】](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E7%9A%84%E7%BB%A7%E6%89%BF(%E4%B8%8B).md)咯，
目前为止,我们只讲了3种最基本的继承,事实上,JavaScript还有好多继承方式,为了让你不至于学习疲劳，所以我打算分开来讲，
如果你没有那个耐性继续看下去，那么看完这篇对于理解JavaScript的继承，也是够用的。但是建议多看两遍，加深印象，
我学js继承的时候，那本犀牛书都被我翻烂了，写这篇文字的时候，我还在一遍翻一边写的呢(嘘！)

好了，今天就到这里，感谢收看.