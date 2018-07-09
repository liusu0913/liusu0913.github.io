---
layout: blog
javascript: true
background-image: http://img.hb.aicdn.com/01da1f1e0d9e5cb96dde24fb3d1c5fee6fda858e338f-WKMRmV_/fw/480
category: javascript
title: webpack学习
tags:
- webpack
---

## webpack学习
### 基本认识：
1. entry 配置要打包的入口文件
2. loader 可以将用到的文件转换成webpack打包支持的格式文件（格式工厂）
3. plugin插件部分的配置，比如js压缩插件等
4. ouput 文件打包完毕之后的输出配置

### webpack模拟vue的简易模版
1. 首先生成package.json文件；利用npm init来生成；
2. 下载相应的npm包用来进行打包；

npm包 | 用途 
---|---
webpack | 不解释
webpack-cli | 脚手架
webpack-dev-server | webpack的启动本地服务，需要添加package.json的script；"build": "webpack --mode production", "start": "webpack-dev-server --mode development"
html-webpack-pulgin | webpack插件，可以生成index.html模版，会自动引入打包后的js文件名，也可以指定特定的模版来生成页面
css-loader | css中存在的路径问题，比如url()的路径，webpack无法解析需要利用这个的包来解析；
style-loader | 将css-loader解析的结果转换成js代码，运行时动态插入 style 标签来让 CSS 代码生效。
less-loader | 将less转成css的npm包，引用这个包需要引入less包
babel-loader | 将ES6-7的代码转换成ES5的代码，引用包需要babel-core包；
file-loader | 解析图片文件的路径
vue | 模拟的vue你说引不印vue包
vue-style-loader | 解析vue文件行内的style包
vue-loader| 可以将vue文件转换为JS模块；在引用时候Vue-loader在15.*之后的版本都是 vue-loader的使用都是需要伴生 VueLoaderPlugin的；


```js
// package文件
{
  "name": "test",
  "version": "1.0.0",
  "description": "webpack4",
  "main": "index.js",
  "scripts": {
    "build": "webpack --mode production",
    "start": "webpack-dev-server --mode development"
  },
  "author": "liusu",
  "license": "ISC",
  "dependencies": {
    "express": "^4.16.3"
  },
  "devDependencies": {
    "babel-core": "^6.26.3",
    "babel-loader": "^7.1.4",
    "css-loader": "^0.28.11",
    "extract-text-webpack-plugin": "^4.0.0-beta.0",
    "file-loader": "^1.1.11",
    "html-webpack-plugin": "^3.2.0",
    "less": "^3.0.4",
    "less-loader": "^4.1.0",
    "style-loader": "^0.21.0",
    "vue": "^2.5.16",
    "vue-loader": "^15.2.4",
    "vue-style-loader": "^4.1.0",
    "vue-template-compiler": "^2.5.16",
    "webpack": "^4.12.1",
    "webpack-cli": "^3.0.8",
    "webpack-dev-server": "^3.1.4"
  }
}

```
3. 配置webpack.config.js文件

```javascript
const path = require('path');
// 生成html模板并成功配置打包之后js为文件名和路径
const HtmlWebpackPlugin = require('html-webpack-plugin');
// Vue-loader在15.*之后的版本都是 vue-loader的使用都是需要伴生 VueLoaderPlugin的
const VueLoaderPlugin = require('vue-loader/lib/plugin');

const moduleConfig = {
  // 单独生成index.css文件的配置项
  rules: [{
    test: /\.css$/,
    use: ['vue-style-loader', 'css-loader']
  }, {
    test: /\.vue$/,
    loader: 'vue-loader',
      options: {
        loaders: {}
      }
  },
  {
    test: /\.less$/,
    loader: "style-loader!css-loader!less-loader"
  }, {
    // js文件转换babel模块，将ES6-7的js转换成ES5的文件
    test: /\.jsx?/, // 支持 js 和 jsx
    include: [
      path.resolve(__dirname, 'src'), // src 目录下的才需要经过 babel-loader 处理
    ],
    loader: 'babel-loader'
  }, {
    // 处理图片文件 file-loader处理 css-loader 会解析样式中用 url()
    test: /\.(png|jpg|gif)$/,
    use: [{
      loader: 'file-loader',
      options: {},
    }]
  }]
}

module.exports = {
  entry: './src/main.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].js',
  },
  // 插件配置
  plugins: [
    // 配置打包之后html的js的文件路径的插件
    new HtmlWebpackPlugin({
      // 输出的模板的文件名
      filename: 'index.html',
      // 根据模板的位置来生成模板文件
      template: 'index.html'
    }),
    // 调用vue-loader依赖插件
    new VueLoaderPlugin()
  ],
  // 模块配置
  module: moduleConfig
}
```
4. 文件的位置

![路径](http://img.hb.aicdn.com/d28e0a5bf773ce208ab4b64140192efde1a978088d90-gIM1eG_fw658)；

5. 运行服务器来进行查看本地文件 npm run start；