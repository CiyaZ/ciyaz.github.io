# 获取IE版本

我们实际开发项目中，经常需要考虑IE问题。我们前端实现的某些功能、样式，虽然通常不需要完整支持IE浏览器，但也要保证IE下不出太大的问题。这时，我们经常要对浏览器版本进行判断。

## 例子代码

我们可以根据浏览器的`User-Agent`来判断浏览器的版本。

```javascript
function getIeVersion() {
    var userAgent = navigator.userAgent;
    var isIE = userAgent.indexOf("compatible") > -1 && userAgent.indexOf("MSIE") > -1;
    var isEdge = userAgent.indexOf("Edge") > -1 && !isIE;
    var isIE11 = userAgent.indexOf('Trident') > -1 && userAgent.indexOf("rv:11.0") > -1;
    if (isIE) {
        var reIE = new RegExp("MSIE (\\d+\\.\\d+);");
        reIE.test(userAgent);
        var fIEVersion = parseFloat(RegExp["$1"]);
        if (fIEVersion === 7) {
            return 7;
        } else if (fIEVersion === 8) {
            return 8;
        } else if (fIEVersion === 9) {
            return 9;
        } else if (fIEVersion === 10) {
            return 10;
        } else {
            return 6;
        }
    } else if (isEdge) {
        return 999;
    } else if (isIE11) {
        return 11;
    } else {
        return 999;
    }
}
```

上面函数中，对于新浏览器返回`999`，对于IE则返回`6`、`7`、`8`等值，使用时直接判断返回值的大小就行了。
