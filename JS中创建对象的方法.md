### JS中创建对象的方法

> 最近手头一个项目刚完成，下一个显目还在准备中，趁这个空档期，拿起尘封多年的JS书，
重温一遍JS面向对象程序设计，然后就得出下文，算是一个总结吧。

也许，你会说 “创建对象不就是一对花括号的事吗？”，是的，目前我们最常用，
也是最便捷的方式就是所谓的一对花括号的事，也就是我们常说的JSON对象（严格意义上，这其实不算JSON对象，具体我们这里不做深入），如下：

    let obj = {
        name:'xiaohong',
        age: 17,
        gender: 'female'
    }
这是就是我们常说的**对象字面量**(Literal)的方法创建对象，应该是目前使用最广泛的方法了。
这总方法基本上等同与下面（创建Object实例）的方法

    let obj = new Object()
    obj.name = 'xiaohong'
    obj.age = 17
    obj.gender: 'female'
但是由于 对象字面量的方法，比较简洁直观，足以满足大部分场景下的需求，所以被开发着们广泛采用，
而Object实例的方式就很少有人问津了。
不过字面量方法也是有缺点的，缺点就是完全没有复用性可言，如果需要创建多个对象，
就会有很多重复的代码，比如：

    var studentA = {
        name: 'xiaohong'
        age: 17,
        gender: 'female'
    }
    var studentB = {
        name: 'xiaoming'
        age: 18,
        gender: 'male'
    }
    var studentC = {
        name: 'lili'
        age: 17,
        gender: 'female'
    }
不难看出，三个对象只有冒号后面的内容不一样，其他的代码都是冗余的，那有什么办法可以避免这个冗余呢？
平常开发中，如果我们碰到一个要重复用到的功能的时候，我们会怎么做？对的，
就是把这个功能封装成一个函数，然后重复调用：

    function Student(name,age,gender){
        // 以下代码还可以用es6这样写 return {name,age,gender}
        // 详情请参考es6 属性简写
        return {
            name:name,
            age:age,
            gender:gender
        }
    }
    // 然后在需要的时候，调用一下这个函数，传进一些参数即可
    var studentA = Student('xiaohong',17,'f')
    var studentB = Student('xiaoming',18,'m')
    var studentC = Student('lili',17,'f')

这样是不是简洁很多，每次只要调用一下Student这个函数，然后传进名字，年龄，性别，
就能得到一个你想要的对象了。
这种方法叫 **工厂模式** ，是不是真的很像一个加工厂呢？ 这种方法对创建多个对象的时候很有用，
不过这种方法也是有缺点的，就是无法知道对象的类型，比如，
我想通过条件语句判断studentA是不是一个Student对象，就做不到

    typeof studentA === 'Student'   //false
    studentA instanceof Student     // false

由于工厂模式在**对象识别**的问题上不堪重任，所以我们通常用 **构造函数** 模式来解决对象识别的问题

    function Student(name,age,gender){
        this.name = name
        this.age = age
        this.gender = gender
    }
    // 通过调用构造函数，new一个对象（这个估计是有其他面向对象语言基础的童鞋对容易接受的一种方式）
    var studentA = new Student('xiaohong',17,'f')
    var studentB = new Student('xiaoming',18,'m')
    var studentC = new Student('lili',17,'f')

构造函数跟工厂模式的很相似，区别主要在以下2点：
1. 没有返回对象，而是直接把参数赋值给this作用域下的同名变量，因为new的时候，
会把this指向调用new出来的那个实例对象，所以就完成了赋值操作
2. 调用构造函数的时候，在构造函数前面加一个new（
如果没加new,就当做普通函数使用，作用域会在当前代码块的环境下面，函数里面的值会赋给当前作用域）

通过构造函数new出来的对象，我们是能检测到它的类型的

    studentA instanceof Student // true
    studentA instanceof Object  // true

事实上，当我们使用 new 实例化一个构造函数的时候，js其实偷偷的在背后做了4件事，这个也是个比较经典的面试题：
1. 创建一个新对象(prototype 指向构造函数的prototype)
2. 把作用域（this）指给这个对象
3. 执行构造函数的代码
4. 返回这个对象

然而，构造函数也不是没有缺点，使用构造函数创建的对象里面都是数据是没啥问题，
但是如果对象里面有函数（方法）呢？
还是上面那个代码，我们拿来稍微修改一下，需要给学生增加一个学习的技能：

    function Student(name,age,gender){
        this.name = name
        this.age = age
        this.gender = gender
        this.study = fucntion() { console.log('我在学习...')}
     }

这样咋看起来，也没啥毛病，调用一些实例的study,也可以打印出“我在学习...”

    var studentA = new Student('xiaohong',17,'f')
    studentA.study()   // 我在学习...

但是，如果我们这样

    var studentA = new Student('xiaohong',17,'f')
    var studentB = new Student('xiaoming',18,'m')
    studentA.study == studentB.study  // false

我们发现，2个实例的study不是指向同一个函数，而是2个不同的函数，但是他们的功能一模一样
相当于这样

    studentA.study = fucntion() { console.log('我在学习...')}
    studentB.study = fucntion() { console.log('我在学习...')}
    studentC.study = fucntion() { console.log('我在学习...')}

**这让强迫症怎么接受？**
事实证明，写代码的，大部分都有强迫症，于是，就有了**原型模式**
原型模式的原理，就是我不把方法和属性添加到构造函数里面去，我直接添加到构造函数的原型里面去，
由于原型的共享的，所以我们在这边就解决了冗余：

    function Student(){}   // 声明一个空函数
    Student.prototype.name = 'xiaohong'
    Student.prototype.age = 17
    Student.prototype.gender = 'f'
    Student.prototype.study = fucntion() { console.log('我在学习...')}

    var studentA = new Student()
    var studentB = new Student()
    studentA.study == studentA.study  // true

这样，就能解决函数重复声明的问题，所有的实例，都共享原型上的函数study.然而，函数是共享了没错，
不过其他数据也共享了，所有实例上都会有相同的name,age,gender，这也不是我们想要的效，这时，
聪明的你肯定会想，把数据放在构造函数里面，只把方法放在原型里面:

    function Student(name,age,gender){
        this.name = name
        this.age = age
        this.gender = gender
     }
     Student.prototype.study = fucntion() { console.log('我在学习...')}
这样把普通的数据放在构造函数里面声明，把函数（方法）放在原型上声明的模式，
我们称之为**组合模式**（即组合使用构造函数模式和原型模式），组合模式，既有数据的独立，又有方法的共享
可以说是比较完美的一种对象的创建方式了。ES6的class语法糖实现的原理大体上也是利用组合模式。

以上就是ES5里面创建对象的一些常用模式，当然还有一些不常用的奇葩的模式，比如动态原型模式，
寄生构造函数模式，稳妥构造函数模式...等等，，这里就不一一列举了，感兴趣的童鞋自行百度一下

好了，关于创建对象的话题，就到这里了，感谢收看！

如果觉得对您有用，请给本文的[github](https://github.com/noahlam/articles)加个star,万分感谢，另外，github上还有其他一些关于前端的教程和组件，有兴趣的童鞋可以看看，你们的支持就是我最大的动力。