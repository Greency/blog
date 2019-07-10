- #### 声明
> 你需要对如下知识或库有所了解
1. commander
2. execa
3. webpack输入、输出
4. webpack提供的node API（webpack-dev-server等）
5. fs-extra

- ##### 定义命令
1. 我们知道，如果要执行npm run serve这种命令，我们需要在package.json中新增如下内容：
```
"scripts": {
    "serve": "xxxx"
}
```
> 注意，“scripts”应该添加在config/package.js文件中。因为此条命令针对的是用户自己生成的vue项目，而不是example-cli这项目。

1. lib/config/package.js新增如下内容：
> 为了能正常的解析.vue文件，我们还需要添加以下依赖：
```
let package = {
    version: '0.0.1',
    scripts: {
        "serve": "example-cli serve"
    },
    dependencies: {
        "css-loader": "^3.0.0",  //新增
        "vue": "^2.6.10",
        "vue-loader": "^15.7.0",  //新增
        "vue-template-compiler": "^2.6.10",  //新增
        "webpack": "^4.35.3"  //新增
    },
    devDependencies: {}
};

module.exports = package;
```
> 注意：当我们执行npm run serve的时候，实际上是执行的example-cli serve。所以，我们还要添加serve命令。

2. 在bin/index.js中新增如下内容：
```
program
    .command('serve')
    .action(()=>{
        require('../lib/command/serve')();
    });
```
3. 在lib/command下新增serve.js文件，并新增如下内容：
```
const path = require('path'),
    execa = require('execa');

module.exports = function () {
    //这里必须用绝对路径，不能改变cwd的值
    let execaDir = path.resolve(__dirname, '../scripts/devServer.js');

    execa('node', [execaDir]).stdout.on('data', (buffer) => {
        process.stdout.write(buffer);
    });
};
```
4. 在lib/scripts下新增devServer.js文件，并新增如下内容：
> 请先安装webpack，webpack-dev-server
```
const path = require('path'),
    webpack = require('webpack'),
    WebpackDevServer = require('webpack-dev-server'),
    devServerConfig = require('../config/server');

const compiler = webpack(devServerConfig);

new WebpackDevServer(compiler, devServerConfig.devServer).listen('8080', '127.0.0.1', () => {
    console.log('Listening at http://127.0.0.1:8080');
});
```
5. 在lib/config下新增server.js文件，并新增如下内容：
```
const HtmlWebpackPlugin = require('html-webpack-plugin'),
    VueLoaderPlugin = require('vue-loader/lib/plugin'),
    path = require('path'),
    cwd = process.cwd();

module.exports = {
    mode: 'development',
    devtool: 'cheap-module-eval-source-map',
    entry: path.resolve(cwd, 'src/main.js'),
    devServer: {
        open: false,
        overlay: true,
        stats: "none"
    },
    module: {
        rules: [
            {
                test: /\.vue$/,
                loader: 'vue-loader'
            },
            {
                test: /\.css$/,
                use: [
                    'css-loader'
                ]
            }
        ]
    },
    plugins: [
        new VueLoaderPlugin(),
        new HtmlWebpackPlugin({
            template: path.resolve(__dirname, '../template/index.html')
        })
    ],
    resolve: {
        alias: {
            'vue$': 'vue/dist/vue.esm.js'
        },
        extensions: ['*', '.js', '.vue', '.json']
    }
};
```

- ##### 完毕

> 现在你就可以执行npm run serve来运行项目啦！