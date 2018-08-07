### 前言  

最近收到一个这样的需求,要求做一个基于 vue 和 element-ui 的通用后台框架页,具体要求如下:
1. 要求通用性高,需要在后期四十多个子项目中使用，所以大部分地方都做成可配置的.
2. 要求做成脚手架的形式.可以 npm 安装
3. 要求实现多页签,并且可以通过浏览器 url 回显多页签.而且页签内要维护一个历史记录，可以后退
4. 组件要求异步加载,减少首屏加载时间.

很明显,这就是一个 `类 ERP` 的应用. 做过 JSP 等后台的同学,对多页签应该都很熟悉吧.  

那接下来我们就来谈谈实现.


### 通用性高

这点其实没啥难点,无非就是麻烦了点,把所有的数据,都提取出来,放在一个 `config` 文件里面.然后在框架页里面引入,并且绑定到相应的位置上去. 这边有个比较难以取舍的问题，就是如果把一溜的数据全部绑定到 vue 的 data 上面，由于数据量比较多，会导致性能问题，如果分开，又会使配置文件看起来相对复杂，增加后期使用人员的学习成本。这块要看具体的项目需求，由于我这边暂时对前端的性能要求没那么高，所以暂时用全部绑定到 data 的方案

### 做成脚手架形式

起初产品对这个的需求使做成组件的形式，然后发布 npm 包，方便后期更新的时候，只需更新一下 npm 就可以了，无需每个项目去复制粘贴替换，但是基于这是一个框架页，而且可配置项非常多，还要实现 tab 多页签等多方面的考虑，最终选择了脚手架的方案，即便这样后期升级会稍微麻烦一点(起初的方案是框架页放在一个文件夹里，到时候直接替换该文件夹)，但相对于组件来说，还是更好维护的，况且后期可以再写一个更新的脚手架，毕竟现在发布一个 npm 工具的成本实在是太低了。

第一次开发脚手架，看了很多社区的帖子，发现目前大部分脚手架，一般都基于2种形式，一种基于文件复制的形式，另一种基于 git-clone 的形式，经过对比，我觉得文件复制的有点复杂了，我其实只是需要一个能一键安装的工具而已，所以 git-clone 的形式还是比较适合我。

以下就是脚手架的代码，虽然只是简单的五六十行代码，不过查资料＋趟坑，也花了我一个上午的时间。

```javascript
#!/usr/bin/env node
const shell = require('shelljs');
const program = require('commander');
const inquirer = require('inquirer');
const ora = require('ora');
const fs = require('fs');
const path = require('path');
const spinner = ora();
const gitClone = require('git-clone')
const chalk = require('chalk')


program
	.version('1.0.0', '-v, --version')
	.parse(process.argv);

const questions = [{
  type: 'input',
  name: 'name',
  message: '请输入项目名称',
  default: 'my-project',
  validate: (name)=>{
    if(/^[a-z]+/.test(name)){
      return true;
    }else{
      return '项目名称必须以小写字母开头';
    }
  }
}]

inquirer.prompt(questions).then((dir)=>{
  downloadTemplate(dir.name);
})

function downloadTemplate(dir){

  //  判断目录是否已存在
  let isHasDir = fs.existsSync(path.resolve(dir));
  if(isHasDir){
    spinner.fail('当前目录已存在!');
    return false;
  }
  spinner.start(`您选择的目录是: ${chalk.red(dir)}, 数据加载中,请稍后...`);

  // 克隆 模板文件
  gitClone(`https://github.com/noahlam/main-frame.git`, dir , null, function(err) {
    // 移除无用的文件
    shell.rm('-rf', `${dir}/.git`)
	  spinner.succeed('项目初始化成功!')
    // 运行常用命令
    shell.cd(dir)
	  spinner.start(`正在帮您安装依赖...`);
    shell.exec('npm i')
	  spinner.succeed('依赖安装成功!')
    shell.exec('npm run dev')
  })
}
```

如果你这个脚手架有疑问或者兴趣，可以直接访问 github 上的代码 [no-cli](https://github.com/noahlam/no-cli.git)


### 实现多页签

要想实现多页签，那么 vue-router 基本算是废了，为什么？ vue-router 是根据 url 来切换单个组件的，而页签则需要再组件内部同时存在多个子组件的，所以路由无法胜任（至少我是这么认为的，如果你有更好的方案，恳请不吝赐教）。

多个页签的显示，其实不难， element 有现成的 tab 组件，于是老夫写代码就是一把梭，撸起袖子就是干，噼里啪啦一顿写，写完一测，没有任何问题，实在是不要太简单，丢给产品预览：

1. 复制浏览器地址到别的地方粘贴，tab 不能正确回显
2. tab 内需要实现跳转，而且要能返回。

第一个问题比较简单，自己手写一个基于 hash 的 `伪路由`  把当前 tab 的 id 放到 url 上去，然后回显的时候，根据 url 打开对应的 tab.

> tip: 关于如何实现路由，请看我另外一篇博客 [自己动手实现一个前端路由](https://github.com/noahlam/articles/blob/master/%E8%87%AA%E5%B7%B1%E5%8A%A8%E6%89%8B%E5%AE%9E%E7%8E%B0%E4%B8%80%E4%B8%AA%E5%89%8D%E7%AB%AF%E8%B7%AF%E7%94%B1.md)

第二个问题，大概就是本文的重点了，这里详细说明一下需求，每个 tab 都可以在 tab 内部 `跳转` ,这里的跳转，要做的跟 vue-router 的有大体上差不多，要能 push, replace, back,还能带参数。

那么怎么实现呢？ 首先维护一个打开的 tab 列表，然后每个列表里面再维护一个用过的组件列表(包含参数),这样大概就能实现了吗？当然不是，组件的跳转，参数的传递，不可能让使用者自己去实现这些方法吧，我选择把封装一个公共对象，然后挂载在 vue.prototype上。然后类似 vue.$router.xxxx 一样(我的命名是 vue.$tab)可以在页面的任何地方使用，如果你对具体的实现方法有兴趣，欢迎点击本文结尾的链接，去我的Github仓库上查看。

### 组件异步加载

之前只用过基于 vue-router 的异步加载方法，然而这个项目里面并没有使用 vue-router,怎么异步呢？ 翻了一下 vue 的官方文档是这么写的：

```javascript
Vue.component(
  'async-webpack-example',
  // 这个 `import` 函数会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```
然而我试了一下，发现报错了，import 不能在这里使用，换了 require 也不行，不知道上我哪里没弄好，如果你刚好知道又刚好有空，请告诉我，谢谢！后面在 segmentfault 上 看到 [这一篇](https://segmentfault.com/a/1190000011519350), 使用 webpack 的 require.ensure 可以实现

```javascript
// 第一个字符串是 组件名，第二个是 组件路径，第三个是 chunkName (如果不指定则以1.js,2.js....n.js命名)
vue.component('home', (resolve) => {require.ensure([], ()=>resolve(require('@/Views/index.vue')), 'home')})
```
顺便还要在 webpack 里面的 output 下面配置一下 `chunkFilename: '[name].js',`,  当然文件名格式可以按你项目的需求来，我这边就按最简单的

### 结束语

首先，当然上献上该项目的 [github地址](https://github.com/noahlam/main-frame.git) 咯  

其次是本文的的地址 [个人技术帖合集](https://github.com/noahlam/articles)  

以上项目 欢迎随意 `star` 和 `follow`, 和不随意的 `issue`  