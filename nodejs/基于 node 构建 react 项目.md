# 基于 node 构建 react 项目

本文只是介绍一个基于 nodejs 的前端 react 项目的搭建，不会深入介绍具体的技术。

## 创建

首先我们先去 github 或者创建一个项目

![node version](http://p2u26co2w.bkt.clouddn.com/github-create-repository.jpeg)

* 简述一下上述圈起来的部分都是做什么的。
	1. 新建项目名
	2. 选择是否公开（因为我们这个是模版、教学，所以选择 public，不过 github 上 private 的项目是需要收费的，大家可以选择 osc 的 [码云](https://gitee.com/)）
	3. 是问你是否需要使用 readme 初始化项目，可以勾选上，也可以以后再补。
	4. 这就比较关键了，选择 gitignore 文件类型，该文件的作用是 git 忽略某些文件，也就是说在 gitignore 内的文件是不会触发 git 的 hash 检查的，也不会被提交（前提是文件不能被先提交再加入 gitignore 文件中）
	5. 添加开源协议（感兴趣的同学可以自行查阅关于开源协议的文章）
	
此时点击 crate repository 就创建出一个自己的 git 项目了。

## 安装

创建完成后，使用 git clone 命令将项目安装到本地。

```@javascript
git clone https://github.com/zhangporco/webpack-react-templete.git
```

## node 初始化

首先你的电脑得全局安装了 [node](https://nodejs.org/en/)。

```@javascript
node -v 
```

![node version](http://p2u26co2w.bkt.clouddn.com/node-npm-version.jpeg)

然后进入项目根目录，输入

```@javascript
npm init
```

接下来 npm 会向你提一系列问题，如果没有特殊要求，一路回车就行，最后 yes 确定就完成了初始化（后面也可以在 package.json 文件中修改）。

此时我们会在项目目录中看见多了一个 package.json（该文件是 node 模块的描述文件），该文件还是红色的，说明它不在 git 目录中，需要手动 git add 把它加入git 目录中，这样他才能被提交。

![node version](http://p2u26co2w.bkt.clouddn.com/project-git-init.jpeg)

因为我使用的编辑器是 Intellij idea ，所以目录中会多一个 iml 文件（项目描述文件），一个隐藏文件夹 .idea（编译器环境配置文件，这两个文件都不需要被提交，所以在 gitignore 文件最后需要加上两行）。

```@javascript
.idea
*.iml
```

#### 注意：以上步骤对于任何 git、node 项目都是通用的。

### 安装依赖

你可以将以下代码复制进 package.json 文件中，然后在根目录下执行 **npm i**

```@javascript
  "dependencies": {
    "babel-core": "^6.26.0",
    "babel-loader": "^7.1.2",
    "babel-plugin-transform-object-rest-spread": "^6.26.0",
    "babel-plugin-transform-runtime": "^6.23.0",
    "babel-preset-env": "^1.6.1",
    "babel-preset-react": "^6.24.1",
    "copy-webpack-plugin": "^4.3.1",
    "css-hot-loader": "^1.3.6",
    "css-loader": "^0.28.9",
    "extract-text-webpack-plugin": "^3.0.2",
    "file-loader": "^1.1.6",
    "html-loader": "^0.5.5",
    "html-webpack-plugin": "^2.30.1",
    "react": "^16.2.0",
    "react-dom": "^16.2.0",
    "react-router-dom": "^4.2.2",
    "sass-loader": "^6.0.6",
    "style-loader": "^0.20.1",
    "url-loader": "^0.6.2",
    "webpack": "^3.10.0",
    "webpack-dev-server": "^2.11.1"
  }
```

或者执行以下安装命令（需要在项目根目录下执行）

```@javascript
npm install --save react react-dom react-router-dom 
npm install --save webpack webpack-dev-server extract-text-webpack-plugin copy-webpack-plugin html-webpack-plugin 
npm install --save babel-loader babel-core babel-plugin-transform-object-rest-spread babel-preset-env babel-preset-react babel-plugin-transform-runtime
npm install --save file-loader url-loader css-loader style-loader sass-loader css-hot-loader html-loader
```

此时我们可以在根目录下发现多了一个 node_modules 文件夹，这里就是项目所依赖第三方模块的目录。

## 代码目录

### 1. webpack

```@javascript
// 根目录下执行
mkdir webpack

// ./webpack 目录下新建 webpack.config.dev.js，并写入以下内容

module.exports = require('./webpack.config')(false);
```

```@javascript
// webpack 目录下新建 webpack.config.prod.js，并写入以下内容

module.exports = require('./webpack.config')(true);
```

```@javascript
// 新建 webpack.config.js，并写入以下内容

const webpack = require('webpack');
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
const HtmlWebpackPlugin = require('html-webpack-plugin');
const CopyWebpackPlugin = require('copy-webpack-plugin');

const extractCSS = new ExtractTextPlugin('[name].fonts.css');
const extractSCSS = new ExtractTextPlugin('[name].styles.css');

const BUILD_DIR = path.resolve(__dirname, '../build');
const SRC_DIR = path.resolve(__dirname, '../src');

console.log('BUILD_DIR', BUILD_DIR);
console.log('SRC_DIR', SRC_DIR);

module.exports = function(NODE_ENV) {
    return {
        entry: {
            index: [SRC_DIR + '/index.js']
        },
        output: {
            path: BUILD_DIR,
            filename: '[name].bundle.js'
        },
        watch: true,
        // devtool: env.prod ? 'source-map' : 'cheap-module-eval-source-map',
        devServer: {
            contentBase: BUILD_DIR,
            port: 8080, // 启动端口
	        host: '0.0.0.0',
	        historyApiFallback: true,
	        disableHostCheck: true,
            compress: true,
            hot: true,
            open: true
        },
        module: {
            rules: [
                {
                    test: /\.(js|jsx)$/,
                    exclude: /node_modules/,
                    use: {
                        loader: 'babel-loader',
                        options: {
                            cacheDirectory: true,
                            presets: ['react', 'env']
                        }
                    }
                },
                {
                    test: /\.html$/,
                    loader: 'html-loader'
                },
                {
                    test: /\.(scss)$/,
                    use: ['css-hot-loader'].concat(extractSCSS.extract({
                        fallback: 'style-loader',
                        use: [
                            {
                                loader: 'css-loader',
                                options: {alias: {'../img': '../public/img'}}
                            },
                            {
                                loader: 'sass-loader'
                            }
                        ]
                    }))
                },
                {
                    test: /\.css$/,
                    use: extractCSS.extract({
                        fallback: 'style-loader',
                        use: 'css-loader'
                    })
                },
                {
                    test: /\.(png|jpg|jpeg|gif|ico)$/,
                    use: [
                        {
                            loader: 'url-loader',
                            // loader: 'file-loader',
                            options: {
                                name: './img/[name].[hash].[ext]'
                            }
                        }
                    ]
                },
                {
                    test: /\.(woff(2)?|ttf|eot|svg)(\?v=\d+\.\d+\.\d+)?$/,
                    loader: 'file-loader',
                    options: {
                        name: './fonts/[name].[hash].[ext]'
                    }
                }]
        },
        plugins: [
            new webpack.HotModuleReplacementPlugin(),
            // new webpack.optimize.UglifyJsPlugin({sourceMap: true}),
            new webpack.NamedModulesPlugin(),
            new webpack.DefinePlugin({
                'process.env': {
                    NODE_ENV: JSON.stringify(NODE_ENV ? 'production' : 'development')
                }
            }),
            extractCSS,
            extractSCSS,
            new HtmlWebpackPlugin(
                {
                    inject: true,
                    template: './public/index.html'
                }
            ),
            new CopyWebpackPlugin([
                    {from: './public/img', to: 'img'}
                ],
                {copyUnmodified: false}
            )
        ]
    }
};
```

### 2. 根目录下新建 public 目录，并在 public 下新建 img 目录存放图片。

```@javascript
// ./public 目录下新建 index.html，并写入以下内容

<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <!--<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">-->
        <meta name="author" content="Porco">
        <link rel="shortcut icon" href="./img/favicon.png" />
        <title>Node React</title>
    </head>
    <body class="app header-fixed sidebar-fixed aside-menu-fixed aside-menu-hidden">
        <div id="root"></div>
    </body>
</html>
```

### 3. config 

```@javascript
// 根目录下执行
mkdir config

// ./config 目录下新建 config.js，并写入以下内容


/**
 * 开发各自维护
 */
const development = {
    SERVER: {
        DOMAIN: 'http://127.0.0.1:3334',
        PORT: 3334,
    },
    CLIENT: {
        DOMAIN: 'http://127.0.0.1:3333',
    }
};

const test = {
    SERVER: {
        DOMAIN: 'http://127.0.0.1:3334',
        PORT: 3334,
    },
    CLIENT: {
        DOMAIN: 'http://127.0.0.1:3333',
    }
};
/**
 * 禁止修改
 */
const production = {
    SERVER: {
        DOMAIN: 'http://127.0.0.1:3334',
        PORT: 3334,
    },
    CLIENT: {
        DOMAIN: 'http://127.0.0.1:3333',
    }
};

let CONFIG;
switch (process.env.NODE_ENV) {
    case 'development':
        CONFIG = development;
        break;
    case 'test':
        CONFIG = test;
        break;
    case 'production':
        CONFIG = production;
        break;
}

export default CONFIG;
```

此处将根据环境变量返回对应配置项，可以通过启动命令却分开发、生产、测试环境。

### 4. 业务代码

```@javascript
// 根目录下执行
mkdir src

// ./src 目录下新建 index.js，并写入以下内容

import React from 'react';
import ReactDOM from 'react-dom';
import {HashRouter as Router, Route, Switch, Redirect} from 'react-router-dom';

// Import style
import './style/layout.css';

// Views
import Hello from './views/Hello';

ReactDOM.render((
	<Router>
		<Switch>
			<Route exact path="/hello" name="Hello Page" component={Hello}/>
			<Redirect from="/" to="/hello" />
		</Switch>
	</Router>
), document.getElementById('root'));
```

```@javascript
// src 目录下执行
mkdir view

// ./src 目录下新建 Hello.js，并写入以下内容

import React, {Component} from "react";

class Hello extends Component {
    render() {
        return (
            <div>
                Hello world
            </div>
        );
    }
}
export default Hello;
```

### 5. 编辑 package.json 文件

使用以下代码替换 scripts 部分。

```@javascript
"scripts": {
    "dev-client": "webpack-dev-server --config webpack/webpack.config.dev.js --progress --colors --inline",
    "prod-client": "webpack-dev-server --config webpack/webpack.config.prod.js --progress --colors --inline"
  },
```

## 运行

根目录下运行

```@javascript
npm run dev-client
```

因为我们在 webpack 里设置了自动打开默认浏览器，所以它会自动启动浏览器并打开 http://0.0.0.0:8080/#/hello 地址，如果没有启动浏览器，当看到控制台输出

```@javascript
webpack: Compiled successfully.
```

表示编译成功，可以手动在浏览器打开 http://0.0.0.0:8080/#/hello 地址，看到页面显示 **Hello world** 说明你成功了。

## 总结

你可以直接从 github 安装我的 [项目](https://github.com/zhangporco/webpack-react-templete)。该项目的架构经过我的团队的多个前端项目总结出来的，定位就是轻量级、便捷、对拓展友好的项目框架。
