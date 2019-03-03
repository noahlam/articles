最近接手了一个项目,接触到一些对文件操作的业务.所以在这边整理一下日常用到的处理方式,当学习笔记吧,有不对的地方,欢迎指正哈

### FileReader

首先我们来看一下 `FileReader` 这个万能的对象, 就如同它的名字一样,就是个文件读取器,
之所以说它是个万能的对象是因为它可以读取任意格式的内容,最近我尝试过用 FileReader 读取过 `psd`, `ppt`, 各种图片等等.
虽然很多情况下,它读出来的是我们完全看不懂的东西.不过通过一定的转换,理论上我们可以在浏览器里面打开任何文件类型.

下面抄一段 MDN 的文档

> FileReader 对象允许Web应用程序异步读取存储在用户计算机上的文件(或原始数据缓冲区)的内容,使用 File 或 Blob 对象指定要读取的文件或数据.

> 其中File对象可以是来自用户在一个 `<input>` 元素上选择文件后返回的 FileList 对象,也可以来自拖放操作生成的 DataTransfer 对象,还可以是来自在一个 HTMLCanvasElement 上执行 mozGetAsFile() 方法后返回结果.

由于是抄的,就不做详细讲解,重点是要知道你要读的文件的编码类型,然后调用对应的方法取读就可以了.这里有[原文](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)

由于 FileReader 可以把文件读取成各种格式,所以这里可以利用这个特性,进行编码的转换,如 ArrayBuffer, Blob对象 和 字符串, base64 之间的相互转换/单向转换, 部分类型只能单向转换,是因为 FileReader 只接受 File 或 Blob 类型的数据(事实上 File 也 Blob 的一种),如果数据无法转换成指定类型,就无法用 FileReader 转换.

```javascript
 const filereader = new FileReader();
 const blob = new Blob(['hello file-reader'], { type: 'text/plain'});

 filereader.onload = e => {

     console.log(e.target.result); // 输出 data:text/plain;base64,aGVsbG8gZmlsZS1yZWFkZXI=

 }

 filereader.readAsDataURL(blob);
```

网上传的用 Uint16Array 进行 String 和 ArrayBufer 互转的其实有编码长度的问题,我亲身体验过,用 FileReader 就可以避开这个问题.可以调用它的 `readAsArrayBuffer()` 和 `readAsText()` 方法,把指定对象读取成 ArrayBuffer 格式 或 纯文本格式的数据.

当然 FileReader 不仅仅能用在编码转换上,读取各种文件才是它主要的能力,配合 \<input type="file" \>、DataTransfer、Blob 等,可以把任意格式的数据读取到浏览器里面.

### Blob 对象 (Binary Large Object)

上面多次提到 Blob 对象,私以为 Blob 也是个非常强大的对象,所以这里我觉得非常有必要介绍一下,先看看 MDN 怎么说的

> Blob 对象表示一个不可变、原始数据的类文件对象.Blob 表示的不一定是JavaScript原生格式的数据.File 接口基于Blob,继承了 blob 的功能并将其扩展使其支持用户系统上的文件.

Blob 对象有个同名构造函数,该构造函数接收 2 个参数,第一个参数必须是个 Array 类型的,哪怕你只有一项,也必须用 `[ ]` 包着,如 `['hello file-reader']`,
第二个参数是个可选的,是个对象,有2个选项,type 和 endings,type 指定第一个参数的 MIME-Type, endings 指定第一个参数的数据格式,其可选值有 transparent(不变,默认) 和 native(随系统转换)

```javascript
const blob = new Blob(['hello file-reader'], { type: 'text/plain'});
```

Blob.size 可以获取对象中所包含数据的大小(字节), Blob.type 可以获取对象所包含数据的MIME类型.如果类型未知,则该值为空字符串.

Blob.slice() 方法可以返回一个新的 Blob对象,包含了源 Blob对象中指定范围内的数据, 共接收3个参数,前两个参数和 Array.slice 的参数类似

参数1：开始索引,默认为0
参数2：截取结束索引(不包括当前值)
参数3：新Blob的MIME类型,默认为空字符串

```javascript
const newBlob = blob.slice(0, 5, 'text/plain');
```

大文件分段上传就靠它了,配合 Blob.size 食用,口感更佳哈

通过 URL.createObjectURL(Blob对象), 可以把 Blob对象 转换成一个链接地址,该地址可以直接用在某些 DOM 的 src 或者 href 上, 从而实现前端下载或图片显示.
一个比较w神奇的用法是 阮老师的 [web worker 教程](http://www.ruanyifeng.com/blog/2018/07/web-worker.html) 里的「同页面的 Web Worker」, 这个再配合动态插入 DOM, 是不是就可以绕开 webworker 的同源策略？

> 上面提到 FileReader 只能接受 Blob 格式的数据(其他格式其实也是Blob的子集), 其实 Blob 也只能通过 FileReader 读取.简直就是泡面跟火腿肠,最佳搭档啊哈


### Arraybuffer, 类型数组对象, DataView

先说说 Arraybuffer 之所以要介绍它,是因为 FileReader 有个 readAsArrayBuffer() 方法,如果的被读的文件是二进制数据,那用这个方法去读应该是最合适的,读出来的数据,就是一个 Arraybuffer 对象,老规矩,看看定义：

> ArrayBuffer 对象用来表示通用的、固定长度的原始二进制数据缓冲区.ArrayBuffer 不能直接操作,而是要通过类型数组对象或 DataView 对象来操作,它们会将缓冲区中的数据表示为特定的格式,并通过这些格式来读写缓冲区的内容.

Arraybuffer 也有个同名构造函数,用于创建一个指定长度的,内容全部为 0 的 ArrayBuffer 对象,构造函数接收一个参数,用来指定要创建的内容长度.如：


```javascript
let ab = new ArrayBuffer(8); // 创建一个 8 字节的 ArrayBuffer
```

由于无法对 Arraybuffer 直接进行操作,所以我们需要借助其他对象来操作. 所有就有了 TypedArray(类型数组对象)和 DataView.

1. TypedArray, TypedArray 是一类对象的统称,事实上 JS 里面并没有一个叫 TypedArray 的对象或构造函数.所以你不能直接使用 TypedArray.以下是 9 个 TypedArray 对象/构造函数

```javascript
Int8Array();
Uint8Array();
Uint8ClampedArray();
Int16Array();
Uint16Array();
Int32Array();
Uint32Array();
Float32Array();
Float64Array();
```
具体用法请参考 [MDN: TypedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)

TypedArray 虽然不是真的数组,但是有几乎跟数组一样的 API,我们可以像操作数组一样操作 TypedArray ,所以有了 TypedArray 我们就可以把 ArrayBuffer 转换成 TypedArray,然后在进行读写操作,达到操作二进制的目的,下面是个例子

```javascript

    let arrayBuffer = new ArrayBuffer(8);

    console.log(arrayBuffer[0]);  // undefined

    let uint8Array = new Uint8Array(arrayBuffer);

    console.log(uint8Array);  // [0, 0, 0, 0, 0, 0, 0, 0]

    uint8Array[0] = 1;
    console.log(uint8Array[0]); // 1

    console.log(uint8Array);  // [1, 0, 0, 0, 0, 0, 0, 0]

```

可以看出,用 `arrayBuffer[0]` 的方式直接获取 ArrayBuffer 对象的内容是获取不到的,而 TypedArray 可以.

> 直接 console.log(arrayBuffer) 在控制台是可以看到 `[[Int8Array]] [[Int16Array]] [[Int32Array]] [[Uint8Array]]` 4 种 TypedArray 数据,不过这应该是浏览器为了方便开发者观察数据,而做的转换,而不是 ArrayBuffer 真的拥有这些数据,毕竟对象名称看起来也不是那么正式(用 `[[]]` 包含)

> 用 `arrayBuffer[0] = 1` 给 ArrayBuffer 对象的某个下标赋值是不会报错的,而且稍后你可以用同样的路径取出该值 `console.log(arrayBuffer[0]) // 1`, 但这并不代表你操作了 ArrayBuffer 的数据,道理跟 `给数组设置属性` 相同.


2. DataView, DataView 提供了跟 TypedArray 类似的功能,与 TypedArray 不同的是 DataView 是一个真实存在的对象,通过提供各种方法来操作不通类型的数据,直接看栗子吧.

```javascript
let arrayBuffer = new ArrayBuffer(8);

    let dataView = new DataView(arrayBuffer);

    console.log(dataView.getUint8(1)); // 0

    dataView.setUint8(1, 2);
    console.log(dataView.getUint8(1)); // 2
    console.log(dataView.getUint16(1)); // 512

    dataView.setUint16(1, 255);
    console.log(dataView.getUint16(1)); // 255
    console.log(dataView.getUint8(1)); // 0

```


就像你看到的,我们可以在同一个数据上面,调用不同的方法,读取/写入 不同类型(长度)的数据,但是大部分情况下,这么做会很难得到我们预期的效果.就像上面的输出,看起来好像不是那么的正常,这是因为 一个 16 位的二进制,用 8 位的格式来读,刚好可以读成 2 个 8 位的二进制.举个栗子

```javascript

// 16 位的 1

0000 0000 0000 0001

// 用 8 位的读就变成

0000 0000  // 0

0000 0001  // 1

```

因为前面 8 位刚好都是 0 , 所以结果看起来除了多个 0 似乎没啥区别？ 当数字比较大的时候

```javascript

// 应该是256？ 我也不太会算这个

0000 0001 0000 0001

// 用 8 位的读就变成

0000 0001  // 1

0000 0001  // 1

```

扯远了,我们回头看看 DataView 提供了哪些方法

```
// 读
DataView.prototype.getInt8()
DataView.prototype.getUint8()
DataView.prototype.getInt16()
DataView.prototype.getUint16()
DataView.prototype.getInt32()
DataView.prototype.getUint32()
DataView.prototype.getFloat32()
DataView.prototype.getFloat64()

// 写
DataView.prototype.setInt8()
DataView.prototype.setUint8()
DataView.prototype.setInt16()
DataView.prototype.setUint16()
DataView.prototype.setInt32()
DataView.prototype.setUint32()
DataView.prototype.setFloat32()
DataView.prototype.setFloat64()
```

> 跟 TypedArray 比 少了一个 Uint8ClampedArray() 具体看 [MDN: DataView](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView)


### atob 和 btoa

base64 这个利器,相信前端的你不会陌生吧,最常用的操作可能就是图片转 base64 了吧? 在之前 要在字符串跟 base64 直接互转,我们可能需要去网上拷一个别人的方法,而且大部分情况下,你没有时间去验证这个方法是不是真的可靠,有没有 bug, 现在我们可以直接用内置的方法了

```javascript
    let str = 'I am a string';

    let a = btoa(str);  // a = 'SSBhbSBhIHN0cmluZw=='

    let b = atob(a);  // b = 'I am a string'
```

没错,就是这么简单,而且大部分浏览器都支持 除了 IE9-, 具体可以参考 [CanIUse: atob](https://caniuse.com/#search=atob)

> btoa 方法不支持中文和特殊字符,所以保险起见,在转换之前还是 encodeURIComponent 一下吧, 当然别忘了在 atob 后,再 decodeURIComponent 回来。


### jspack zipjs xml2js

最后再安利 3 个包

jspack https://github.com/pgriess/node-jspack js操作二进制文件, 我们的 psd 文件解析就用到这个包.

jszip https://github.com/Stuk/jszip js操作压缩文件, 我们的 pptx 的压缩比解析成 xml 都靠它.

xml2js https://github.com/Leonidas-from-XIV/node-xml2js 把 xml 文件转换成 json, 我们的 pptx 解析就是用它进行 pptx 的 xml 文件的转换.


### 广告时间

我们40人的前端团队常年招兵买马中,在厦门的和想来厦门的童鞋们,不要吝惜你的简历,使劲砸过来 邮箱：`atob('bnVveWFAZ2FvZGluZy5jb20=')`, 期待你一起来`稿`事



对本文有意见或者建议,请尽量在 [github](https://github.com/noahlam/articles/blob/master/js%E5%AF%B9%E6%96%87%E4%BB%B6%E5%92%8C%E4%BA%8C%E8%BF%9B%E5%88%B6%E6%93%8D%E4%BD%9C%E7%9A%84%E4%B8%80%E4%BA%9B%E6%96%B9%E6%B3%95%E6%B1%87%E6%80%BB.md) 上提 [issue](https://github.com/noahlam/articles/issues), 最近比较忙,比较不怎么逛社区
