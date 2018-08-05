### 前言  

最近收到一个这样的需求,要求做一个基于 vue 和 element-ui 的通用后台框架页,具体要求如下:
1. 要求通用性高,需要在后期四十多个子项目中使用，所以大部分地方都做成可配置的.
2. 要求做成脚手架的形式.可以 npm 安装
3. 要求实现多页签,并且可以通过浏览器 url 回显多页签.而且页签内要维护一个历史记录，可以后退
4. 组件要求异步加载,减少首屏加载时间.

很明显,这就是一个 `类 ERP` 的应用. 做过 JSP 等后台的同学,对多页签应该都很熟悉吧.  

那接下来我们就来谈谈实现.


### 通用性高

这点其实没啥难点,无非就是麻烦了点,把所有的数据,都提取出来,放在一个 `config` 文件里面.
然后在框架页里面引入,并且绑定到相应的位置上去. 这边有个比较难以取舍的问题，就是如果把一溜的数据全部绑定到 vue 的 data 上面，由于数据量比较多，
会导致性能问题，如果分开，又会使配置文件看起来相对复杂，增加后期使用人员的学习成本。这块要看具体的项目需求，由于我这边暂时对前端的性能要求没那么高，
所以暂时用全部绑定到 data 的方案

### 做成脚手架形式

起初产品对这个的需求使做成组件的形式，然后发布 npm 包，方便后期更新的时候，只需更新一下 npm 就可以了，无需每个项目去复制粘贴替换，
但是基于这是一个框架页，而且可配置项非常多，还要实现 tab 多页签等多方面的考虑，最终选择了脚手架的方案，即便这样后期升级会稍微麻烦一点
(起初的方案是框架页放在一个文件夹里，到时候直接替换该文件夹)，但相对于组件来说，还是更好维护的，况且后期可以再写一个更新的脚手架，
毕竟现在发布一个 npm 工具的成本实在是太低了。

第一次开发脚手架，看了很多社区的帖子，发现目前大部分脚手架，一般都基于2种形式，一种基于文件复制的形式，另一种基于 git-clone 的形式，
经过对比，我觉得文件复制的有点复杂了，我其实只是需要一个能一键安装的工具而已，所以 git-clone 的形式还是毕竟适合我。

以下就是脚手架的代码，虽然只是简单的五六十行代码，不过查资料＋趟坑，也花了我一个上午的时间。

｀｀｀javascript
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
｀｀｀

如果你这个脚手架有疑问或者兴趣，可以直接访问 github 上的代码[no-cli](https://github.com/noahlam/no-cli.git)


### 实现多页签

要想实现多页签，那么 vue-router 基本算是废了，为什么？ vue-router 是根据 url 来切换单个组件的，
而页签则需要再组件内部同时存在多个子组件的，所以路由无法胜任（至少我是这么认为的，如果你有更好的方案，恳请不吝赐教）。

多个页签的显示，其实不难， element 有现成的 tab 组件，于是老夫写代码就是一把梭，撸起袖子就是干，噼里啪啦一顿写，


起初我天真的以为，多页签无非就是 element 的 tab 嘛，于是老夫写代码就是一把梭，撸起袖子就是干，噼里啪啦一顿写


https://github.com/noahlam/main-frame.git