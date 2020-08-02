# Material Design图标

Google提供了免费的Material Design icon可以使用。[github地址](https://github.com/google/material-design-icons)

## 使用Google API服务加载图标

引入CSS
```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

使用图标
```html
<i class="material-icons">face</i>
```

## 从本地加载图标字体

国内访问Google服务十分缓慢，因此建议从本地加载。

编写如下CSS：
```css
/* fallback */
@font-face {
	font-family: 'Material Icons';
	font-style: normal;
	font-weight: 400;
	src: local('Material Icons'), local('MaterialIcons-Regular'), url(../fonts/md-font.woff2) format('woff2');
}

.material-icons {
	font-family: 'Material Icons';
	font-weight: normal;
	font-style: normal;
	font-size: 24px;
	line-height: 1;
	letter-spacing: normal;
	text-transform: none;
	display: inline-block;
	white-space: nowrap;
	word-wrap: normal;
	direction: ltr;
	-moz-font-feature-settings: 'liga';
	-moz-osx-font-smoothing: grayscale;
}
```

注意`url(../fonts/md-font.woff2)`，这个是字体文件的URL。woff2字体文件可以从Google API返回的CSS中获得，我们可以手动下载下来。

将字体文件加入我们的工程，并正确配置URL后，引入该CSS即可正常使用。
