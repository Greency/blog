- #### 声明
> 你需要对如下知识或库有所了解
1. commander
2. execa
3. webpack输入、输出
4. webpack提供的node API（webpack-dev-server等）

- #### 开始
- ##### 规划项目目录结构
> 我们给项目命名为example-cli，所以根目录为example-cli，里面的内容如下：
```
bin  //放置可在node环境中运行的脚本文件
lib  //项目主目录
    command  //放置命令具体执行的逻辑
    config  //放置配置文件
    scripts  //放置脚本文件
    template  //放置模板文件
    utils  //放置工具函数
package.json  //配置项目基础的信息以及依赖等
```

- ##### package.json基础信息填写
```
{
    "name": "example-cli",
    "version": "0.0.1",
    "description": "A simple CLI for Vue!",
    "license": "MIT"
}
```
- ##### 用如下命令初始化项目
```
npm i
```
> 完成上面的步骤后，我们的项目基础的架子已经搭建好。那么我们接下该做什么呢？

- #### 让系统能够识别我们的命令
> 在vue官方的cli中，我们可以通过vue create xxx 的方式新建一个基于vue的基础项目。那么系统是如何识别这个vue-create的命令的呢？这时候commander这个库就派上用场了！

- ##### 命令的编写
- ###### 安装commander
```
npm i --save commander
```
- ###### 在bin目录下新建index.js文件，并编写如下内容
> 先简单编写一个查看版本号的命令，具体的其他的API请直接去commander对应的GITHUB上查看即可，这里不再详细的解答API的用途。
```
#! /usr/bin/env node

const program = require('commander');

program.version('版本为：0.0.1');

program.parse(process.argv);
```
- ###### 添加脚本命令的名称
> 在package.json中新增内容

>当我们在命令行窗口使用example-cli的时候，实际是执行的bin下的index.js文件
```
{
    "bin": {
        "example-cli": "bin/index.js"
    }
}
```
> 在example-cli目录下执行如下命令后，就可以在任何地方使用example-cli命令

```
npm link
```

- ###### 执行如下命令看看效果
```
example-cli -V
```
> 结果如下：

![image](https://greency.github.io/images-site/youdaoyunbiji/4.png)
> 根据结果来看，执行example-cli命令的时候就是执行的bin/index.js中的内容。

- #### 完毕

> 到此，我们已经搭建好了基础的项目结构，以及知道了如何向vue-cli一样使用自己定义的命令。后续的内容敬请期待！