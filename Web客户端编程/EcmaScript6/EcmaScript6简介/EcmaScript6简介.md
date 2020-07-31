# EcmaScript6

EcmaScript6是JavaScript语言的新标准，6是版本号，于2015年6月发布，也叫作EcmaScript2015。由于新语言特性不停的加入，标准委员会以后决定每年6月发布一个新版本，因此以后按年划分版本比较适合。

## 浏览器支持

目前主流浏览器和node环境对EcmaScript6支持都已经比较完善，但是由于传统的JavaScript标准在浏览器中已经使用了相当长的一段时间，有大量的代码资源可用，因此大多数传统项目仍使用旧语言标准。对于一些新兴前端框架，大多是使用EcmaScript6或TypeScript，项目通过编译成EcmaScript5标准的代码部署。而node是新兴的一个领域，其生态则完全构建在新语言标准上。

## 使用EcmaScript6

在node中可以直接使用大部分语言新特性，如果部署在浏览器中，建议使用转码工具将代码转换到EcmaScript5标准，Babel是最常用的转码工具。

### 基于浏览器搭建开发环境

我们可以使用Webpack将我们编写的代码打包，然后用Babel插件将代码转换为ES5（随着时间的推移，如果你使用最新的浏览器，可能这一步也不是必须的，但生产环境依然还是需要使用Babel），部署到浏览器中运行。

package.json
```json
{
  "name": "myapp",
  "version": "1.0.0",
  "description": "a simple test project",
  "main": "index.js",
  "scripts": {
    "build": "webpack --mode production"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {},
  "devDependencies": {
    "@babel/core": "^7.11.0",
    "@babel/plugin-transform-runtime": "^7.11.0",
    "@babel/preset-env": "^7.11.0",
    "babel-loader": "^8.1.0",
    "webpack": "^4.44.1",
    "webpack-cli": "^3.3.12"
  }
}
```

webpack.config.js
```javascript
const path = require('path');
module.exports = {
    entry: './index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, './dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /node_modules/,
                loader: "babel-loader"
            }
        ]
    },
};
```

.babelrc
```json
{
    "presets": [
        "@babel/preset-env"
    ],
    "plugins": [
        [
            "@babel/plugin-transform-runtime",
            {
                "regenerator": true
            }
        ]
    ]
}
```

编译代码：
```
npm run build
```
