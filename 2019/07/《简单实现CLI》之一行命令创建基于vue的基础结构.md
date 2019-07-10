- #### 声明
> 你需要对如下知识或库有所了解
1. commander
2. execa
3. webpack输入、输出
4. webpack提供的node API（webpack-dev-server等）
5. fs-extra

- #### 定义命令
- ##### 定义创建vue基础项目的命令：example create
> 当执行example create xxx时，就会创建一个名为xxx的目录。此目录下的结构如下：
```
src
    assets  //放置项目的静态文件
    components  //组件
    pages  //页面
    main.js  //项目入口文件
    App.vue  
    
package.json  //项目的基础信息，依赖等
```

- ##### 编写命令
1. 在bin/index.js中新增如下内容：
```
program
    .command('create <projectName>')
    .action((projectName)=>{
        //调用执行真正的代码
    });
```
2. 在template目录下新建src目录
> 并在src目录下新建如下文件或目录
```
src
    assets  //放置项目的静态文件
    components  //组件
    pages  //页面
    main.js  //项目入口文件
    App.vue 
```
> 在main.js中新增如下内容：
```
import Vue from 'vue';
import App from 'App.vue';

new Vue({
    render: (h) => h(App)
}).$mount('#app');
```
> 在App.vue中新增如下内容：
```
<template>
    <h2>Hello World!</h2>        
</template>

<script>
export default {
    name: 'App'
}
</script>

<style lang="css">
    h2 {
        text-align: center;
    }
</style>
```

3. 将src整个目录移动到指定位置下
> 在用户敲下example-cli create xxx命令的时候，我们需要将src整目录移动到xxx目录下，这样就完成了基本的vue项目结构搭建。

> 开始编写移动目录的代码，在command目录下新建create.js文件，并新增如下内容：
```
const fs = require('fs'),
    path = require('path'),
    cwd = process.cwd(),  //获取执行命令时的路径
    createAndMove = require('../utils/createAndMove'),
    writePackageInfo = require('../utils/writePackageInfo'),
    installDependencies = require('../command/installDependencies');

module.exports = function (projectName) {
    let targetDir = path.join(cwd, projectName);

    //判断目标路径是否已经存在
    if (!fs.existsSync(targetDir)) {
        //创建以projectName为名称的目录，并将template下的src目录移动到projectName目录下
        createAndMove(projectName);
        //在projectName目录下创建package.json文件，并新增基础的内容
        writePackageInfo(projectName);
        //执行npm install安装项目的依赖
        installDependencies(targetDir);
    } else {
        console.log(`${projectName} 目录已存在，请重新填写名称！`);
    }
};
```
> 先安装fs-extra库，方便进行文件的操作。

> 在utils下新增createAndMove.js文件，执行具体的新建文件，并移动文件的操作：
```
const fs = require('fs'),
    fsExtra = require('fs-extra'),
    path = require('path'),
    cwd = process.cwd();

module.exports = function (projectName) {
    let targetDir = path.join(cwd, projectName),
        templateDir = path.resolve(__dirname, '../template/src');
    
    try {
        fs.mkdirSync(targetDir);
        fsExtra.copySync(templateDir, path.join(targetDir, 'src'));
    } catch (err) {
        console.error(err);
    }
};
```
> 在config下新增package.js文件：

> 先只填写如下的信息，后续会再添加。
```
let package = {
    version: '0.0.1',
    scripts: {},
    dependencies: {
        "vue": "^2.6.10"
    },
    devDependencies: {}
};

module.exports = package;
```

> 在utils下新增writePackageInfo.js文件，执行package.json文件的创建：
```
const fs = require('fs'),
    path = require('path'),
    cwd = process.cwd();
let package = require('../config/package');

module.exports = function (projectName) {
    let targetDir = path.join(cwd, `${projectName}/package.json`);

    package = Object.assign(package, {
        name: projectName
    });

    try {
        fs.writeFileSync(targetDir, JSON.stringify(package));
    } catch (err) {
        console.log(err);
    }
}
```
> 先安装execa库，方便对于命令的执行。 

> 再command下新建installDependencies.js文件，用于执行npm install：
```
const execa = require('execa');

module.exports = function (cwd) {
    const childProcess = execa.command('npm install', {
        cwd
    });

    childProcess.stdout.on('data', buffer => {
        process.stdout.write(buffer);
    });
};

```

> 最后，修改bin/index.js文件如下：
```
#! /usr/bin/env node

const program = require('commander');

program.version('版本为：0.0.1');

program
    .command('create <projectName>')
    .action((projectName)=>{
        require('../lib/command/create')(projectName);  //新增的代码
    });

program.parse(process.argv);
```

- #### 完毕

> 完成以上步骤后，你已经知道了如何创建一个命令，如何利用子进程执行一些命令，并了解到如何创建文件，复制文件等操作。后续的内容敬请期待！