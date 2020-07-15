# font-awesome 图标库

font-awesome是一个图标字体和相关效果的实现库，由于这个库非常的流行，作者也是不断的对其进行完善，现在这个库里面包含非常大量的图标字体，能覆盖我们大部分的需求。这个库分为免费版和pro版，免费版可以商业使用，我们一般使免费版就行。不过要注意的是这个库体积还是比较大的。

除此之外，Bootstrap3自带的图标字体（图标比较少），还有Google的Material Design图标（需要Google API或者自己手动导出）也是不错的选择。

Github地址：[https://github.com/FortAwesome/Font-Awesome](https://github.com/FortAwesome/Font-Awesome)

项目地址：[https://fontawesome.com](https://fontawesome.com)

注意：用百度搜第一个结果好像是个山寨版的fontawesome，不太清楚是什么东西，使用这些开源库一定要小心，页面不要被注入恶意代码，这非常危险，搜索时直接用Github或Google都非常准。

## 使用npm安装

如果需要将font-awesome整合到我们基于webpack的前端项目中，就需要用包管理工具进行安装，除此之外，我们还需要对应的loader才能正确加载。

```
yarn add @fortawesome/fontawesome-free style-loader css-loader font-awesome-sass-loader url-loader file-loader
```

在webpack.config.js中配置对应的loader：
```javascript
rules: [
    {
        test: /\.css$/,
        use: ['style-loader', 'css-loader'],
    },
    {
        test: /\.js$/,
        exclude: /node_modules/,
        loader: "babel-loader"
    },
    {
        test: /\.woff(2)?(\?v=[0-9]\.[0-9]\.[0-9])?$/,
        loader: "url-loader?limit=10000&mimetype=application/font-woff"
    },
    {
        test: /\.(ttf|eot|svg)(\?v=[0-9]\.[0-9]\.[0-9])?$/,
        loader: "file-loader"
    }
]
```

这样我们就能在`main.js`中引入font-awesome的css了：
```javascript
import '@fortawesome/fontawesome-free/css/all.min.css'
```

## 使用传统方式载入页面

在官方地址下载font-awesome的包，然后在页面中引入相应的CSS即可，目前最新的版本是5.7。

## 例子

font-awesome的CSS都是以`fa`前缀开头的，找到对应的图标名使用`<i>`标签引用即可，非常简单。

```html
<i class="fas fa-图标名"></i>
```

可以在官方网站搜索我们需要的图标：

![](res/1.png)
