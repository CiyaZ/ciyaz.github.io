# crossenv-设置环境变量

实际开发的软件项目中，我们经常需要针对不同的环境加载不同的配置文件。比如某前端工程中，开发环境、测试环境、线上环境的某个接口URL各不相同，这就需要我们针对不同的环境加载不同的配置，指示Webpack分环境打包。

Node本身支持读取一个环境变量叫做`NODE_ENV`，在程序运行时可以通过`process.env.NODE_ENV`来进行访问，然而Linux、Windows下设置环境变量的命令是不同的，有没有一个跨平台的解决方法呢？

`cross-env`可以实现这个功能。

## 安装

```
yarn add cross-env --dev
```

## 使用例子

对于前端工程，我们直接在`package.json`的命令脚本中指定`cross-env`和环境变量即可。

```json
{
  "scripts": {
    "build": "cross-env NODE_ENV=production webpack --config build/webpack.config.js"
  }
}
```
