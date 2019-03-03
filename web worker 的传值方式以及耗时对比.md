### 背景

前一阵子开发的项目 `pptx 导入`, 由于自己的代码问题,引起了个性能问题,一个 40p 的 pptx 文件,转换成 json 数据,
大概要耗时 60s+ ,虽然后面发现是某个使用频率非常高的函数内部,用了 `new Function 构造函数` 造成的
(所以这里顺便提醒一下,如果你很在乎几毫秒的差距的话,建议谨慎使用哈),
但是在优化的过程中,一度怀疑是性能达到了瓶颈,所以尝试了使用 web worker 去优化,由于是文件,一般内容都比较大,
发现 web worker 在传值这块占用了大部分的时间,所以想开这篇来详细聊聊.

### 两种传值方式

关于 web worker 的基本用于以及传值方式,网上以及有一大堆介绍了,这里就不赘述了,
这里我们重点来看一下同一个文件用两种方式来传值,会有多大的差别,这边随意从电脑里面找了一个 96MB 的 PSD 文件来测试.

主线程
```javascript
    fetch('./case.psd').then(file => {
            return file.blob();
        })

        .then(blob => {
            return new Promise(resolve => {
                let fileReader = new FileReader();
                fileReader.onload = e => {
                    resolve(e.target.result);
                }
                fileReader.readAsArrayBuffer(blob);
            })
        })

        .then(buf => {
            let worker = new Worker('1.js');

            console.time('计算时间');
            worker.postMessage(buf);

            worker.onmessage = e => {
                console.timeEnd('计算时间');
            }


        })
```

worker(子)线程, 这里为了避免不必要的因素干扰,worker 线程里面什么也不做,在收到消息后,直接 post 一个消息回去
```
    self.onmessage = e => {
        postMessage(0);
    }
```

这边我直接用 FileReader 的 readAsArrayBuffer,读出来是一个长度为 96,138,230 的字符串,
长度大概 0.96 亿, 耗时大概 70ms 左右(同一个台电脑取 10 次平均值,下同)


我们稍微改一下上面主线程的代码,改用 `转移数据` 的方式
```
- worker.postMessage(buf);

+ worker.postMessage(buf, [buf]);
```

同样的数据, 耗时大概 17ms 左右,这 17ms 好像是个固定值,我尝试换了个 800MB+ 的文件和一个里面啥都没有的空文本文件,
大概都是这个时间.

### 不同的数据类型,用值传递的耗时也是不一样的

```javascript
    fetch('./case.psd').then(file => {
            return file.blob();
        })

        .then(blob => {
            return new Promise(resolve => {
                let fileReader = new FileReader();
                fileReader.onload = e => {
                    resolve(e.target.result);
                }
                fileReader.readAsText(blob);
            })
        })

        .then(str => {
            console.log(str.length);
            let worker = new Worker('1.js');

            console.time('计算时间');
            worker.postMessage(str);

            worker.onmessage = e => {
                console.timeEnd('计算时间');
            }


        })
```

这里我们改用 FileReader 的 readAsText,读出来是一个长度为 95,855,954 的字符串,长度大概 0.95 亿, 耗时大概 118ms 左右,
同样我换了上面那个里面啥都没有的空文本文件,耗时也是 17ms 左右.

那我们试试用 readAsDataURL 看看读出来的数据要多久

```
    fetch('./case.psd').then(file => {
            return file.blob();
        })

        .then(blob => {
            return new Promise(resolve => {
                let fileReader = new FileReader();
                fileReader.onload = e => {
                    resolve(e.target.result);
                }
                fileReader.readAsDataURL(blob);
            })
        })

        .then(str => {
            console.log(str.length);
            let worker = new Worker('1.js');

            console.time('计算时间');
            worker.postMessage(str);

            worker.onmessage = e => {
                console.timeEnd('计算时间');
            }


        })
```

读出来是一个长度为 128,184,345 的字符串,长度大概 1,28 亿, 耗时大概 85ms 左右(虽然字符串长度更长,但是耗时却更短)

> 以上耗时,均为主线成向 worker 线程单向传递数据的耗时.

### 结论

1. 转移数据几乎是零开销(因为和传递空字符串的耗时是差不多的).
2. 值传递的话,不同的数据类型,耗时也有差别,ArrayBuffer < base64 < 普通字符串.
3. postMessage 传递消息,除了发送数据的耗时外,还有其他开销(就是上面的 17ms). 当然每台电脑性能不一样,耗时也是不一样的,不过按比例来看,这个占比还挺大的.

关于转移的缺点, 网上也是有很多的, 这里也就不啰嗦了, 总结一句就是数据无法同时在2个线程上使用.

另外个人觉得如果是普通的数据，为了转移而去转换成 `Transferable objects` 的话, 大部分情况下是划不来的, 因为你需要在花在编码解码上的时间,会比直接传递花的时间多.

另外, 如果你是要用子线程处理图片的话, `ImageBitmap` 格式 配合最近新鲜出炉的 `OffscreenCanvas` 也许是不错的选择.前提是你不需要考虑兼容性问题.

### 最后是广告时间

我们40人的前端团队常年招兵买马中,在厦门的和想来厦门的童鞋们,不要吝惜你的简历,使劲砸过来 邮箱：`nuoya@gaoding.com`, 期待你一起来`稿事`


原文地址 https://github.com/noahlam/articles
