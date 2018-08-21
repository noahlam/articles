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

```
npm i xlsx
```

```
import XLSX from 'xlsx'
...
let res = XLSX.read(data, {type: 'binary'})
let sheetName = res.Sheets[res.SheetNames[0]]
let table = XLSX.utils.sheet_to_json(sheetName, {header: 'A', raw: true, defval: ' '})
```

这里的 res 得到的我猜是 excel 表格原有的数据,里面包含上面说的很多种数据,而正常情况下,
我们需要的往往只是第一个 sheet 里面的 `纯数据`, 所以调用 `XLSX.utils.sheet_to_json`
获取第一个 sheet 的数据, table 拿到的应该是一个我们熟悉的数组了.这个时候如果你只是单纯的渲染的话,
你甚至可以就此打住,自己写一个渲染方法(比如字符串拼接哈)把数据渲染出来即可.

如果单纯的显示无法满足你的需求,那么你可能需要 handsontable 了.

### 数据展示

首先当然是安装,我的项目是基于 vue 的,所以要安装 vue 版本的,其他框架的,只要安装响应的版本即可.
```
npm i @handsontable/vue
```
然后就可以直接这么用
```
<template>
 <HotTable :settings="settings"></HotTable>
</template>

<script>
import HotTable from '@handsontable/vue'
export default {
  ...
  components: {HotTable}
  ...
}
</script>
```

模板里面的 settings 是 handsontable 的一些配置, 每个项目的需求不同,配置也不同,这里就不列举出来了, 上面获取到的 table 在这里要赋值给 settings.data

### 生成报表

生成完
