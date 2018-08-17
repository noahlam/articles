js-xlsx + handsontable + echarts + print.js 实现在前端导入excel数据并生成echart报表
### 前言
最近都在做类似 ERP 的项目,所以呢,又碰到一个比较变态的需求(至少对我来说是),在前端导入 excel 文件,
然后在浏览器里面预览和编辑,接下来选择一些数据,用echarts生成报表.最后再把报表打印出来.

### 依赖

[js-xlsx](https://github.com/SheetJS/js-xlsx) 读取excel数据到js

[handsontable](https://github.com/handsontable/handsontable) 类似Excel一样显示和编辑列表数据

[echarts](https://github.com/apache/incubator-echarts) 一个生成各种报表的库

[Print.js](https://github.com/crabbly/Print.js) 在浏览器里打印数据


### 数据导入

数据导入这边需要用到 浏览器的 `FileReader对象` 的 `readAsBinaryString()` 函数, 把选择的文件读取出来,
然后再监听 FileReader 对象的 `onload 事件` , 在 onload 事件 的回调函数中,我们可以获取到 读取的二进制数据.
这里顺便提一下, FileReader 对象提供以下方法,用来读取各种格式的数据(参考自MDN)

FileReader.readAsArrayBuffer()  // 读取文件的 ArrayBuffer 数据对象.

FileReader.readAsBinaryString() // 读取文件的原始二进制数据

FileReader.readAsDataURL()      // 返回一个URL格式的字符串以表示所读取文件的内容

FileReader.readAsText()         // 返回一个字符串以表示所读取的文件内容


> tips: 需要注意的是 readXxxxx() 函数,是不直接返回读取结果的,因为读取这个动作异步的.

readAsBinaryString 读取到的内容应该是一个二进制的字符串,这个时候,需要调用 `js-xlsx`
的 `read` 方法, read 返回的是一个可读性很强的对象了,我看了一下,里面有关于表格的属性很多都有
,如 样式, vsb宏, sheets等等 (反正我对excel也不熟,认识的也就这些哈),

