### JS中的继承

> 学过java或者c#之类语言的同学,应该会对js的继承感到很困惑--不要问我怎么知道的,js的继承主要是基于原型(prototype)的,对js的原型感兴趣的同学,
可以了解一下我之前写的[JS中的原型对象](https://github.com/noahlam/articles/blob/master/JS%E4%B8%AD%E7%9A%84%E5%8E%9F%E5%9E%8B%E5%AF%B9%E8%B1%A1.md)

相信很多同学也跟我一样,刚开始接触js的面向对象编程的时候,都抱着一种排斥的心态--为什么js这么麻烦?
其实了解完原型链后,再来看js的继承,你会发现js的继承其实比其他OOP语言更简单,更灵活,我们来看一个基于原型链的继承

    // 父类
    function Person() {}

    // 子类
    function Student(){}

    // 继承
    Student.prototype = new Person()

我们只要`把子类的prototype设置为父类的实例`,就完成了继承,怎么样? 是不是超级简单? 有没有比Java,C#的清晰?
事实上,以上就是js里面的**原型链继承**

当然,通过以上代码,我们的Student只是继承了一个空壳的Person,这样视乎是毫无意义的,我们使用继承的目的,
就是要通过继承获取父类的内容,那我们先给父类加上一点点简单的内容:

    // 父类
    function Person(name,age) {
      this.name = name || 'unknow'     // 这个是新添加的
      this.age = age || 0              // 这个是新添加的
    }

    // 子类
    function Student(name){
      this.name = name                 // 这个是新添加的
      this.score = 80                  // 这个是新添加的
    }

    // 继承
    Student.prototype = new Person()

使用

    var stu = new Student('lucy')

    console.log(stu.name)  // lucy    --子类覆盖父类的属性
    console.log(stu.age)   // 0       --父类的属性
    console.log(stu.score) // 80      --子类自己的属性

这里为了降低复杂度,我们只演示了普通属性的继承,没有演示方法的继承,事实上,方法的继承也很简单,
我们来稍微修改一下代码,给上面的父类和子类分别加一个方法

    // 父类
    function Person(name,age) {
      this.name = name || 'unknow'
      this.age = age || 0
    }

    // 为父类新曾一个方法
    Person.prototype.say = function() {         // 这个是新添加的
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
    Student.prototype.study = function () {     // 这个是新添加的
        console.log('I am studing')
    }

使用

    var stu = new Student('lucy')

    console.log(stu.name)  // lucy               --子类覆盖父类的属性
    console.log(stu.age)   // 0                  --父类的属性
    console.log(stu.score) // 80                 --子类自己的属性
    stu.say()              // I am a person      --继承自父类的方法
    stu.study()            // I am studing       --子类自己的方法

这样,看起来我们好像已经完成了一个完整的继承了,这个就是**原型链继承**,怎么样,是不是很好理解?
但是,原型链继承有一个缺点,就是属性如果是引用类型的话,会共享引用类型,请看以下代码

---未完待续---