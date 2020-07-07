# FormData

在`FormDataAPI`出现之前，我们页面上的表单必须实际存在于HTML中。但有时，我们希望能手动构造一个表单对象，`FormDataAPI`能够解决这个问题。

除此之外，借助于`FormDataAPI`手动构造的表单，我们也可以轻松实现ajax文件上传，而不需要借助`iframe`等方式了。

## 构造表单

下面例子中，我们手动构造了一个表单，包含一个键值对表单项`username`，以及一个文件表单项`userFile`。

```javascript
var userFile = document.getElementById('user-file');

var formData = new FormData();
formData.append("username", "Tom");
formData.append("userFile", userFile.files[0]);
```

实际上代码非常简单，我们使用`new`初始化`FormData()`对象后，然后使用`append()`添加表单项就行了。

## 使用JQuery发送FormData数据

FormData是xhr2引入的一个功能，JQuery对此支持不太友好。要想使用JQuery发送FormData对象，我们需要如下设置：

```javascript
$.ajax({
    type: 'POST',
    async: true,
    url: '/',
    data: formData,
    contentType: false,
    processData: false,
    dataType: 'json',
    success: function (msg)
    {
        console.log(msg);
    }
});
```

注意`contentType`和`processData`我们设置为了`false`，否则JQuery会尝试把发送的数据转为字符串报错。
