# 文件系统

Laravel中，处理文件上传、保存、下载非常简单优雅，因为Laravel默认整合了一套虚拟文件系统，能够将指定的本地目录或是云存储系统，映射到统一的路径，它提供了一层合理的抽象，使用起来非常方便。我们使用Laravel，甚至不超过10行代码就能实现一个简单的OSS系统。

flysystem模块参考：[https://github.com/thephpleague/flysystem](https://github.com/thephpleague/flysystem)

## 配置

文件存储系统的配置文件为`config/filesystems.php`，其中定义了虚拟磁盘、符号链接等相关内容。

下面是默认的配置内容：

```php
<?php

return [

    /*
    |--------------------------------------------------------------------------
    | 默认磁盘
    |--------------------------------------------------------------------------
    */

    'default' => env('FILESYSTEM_DRIVER', 'local'),

    /*
    |--------------------------------------------------------------------------
    | 默认云存储磁盘
    |--------------------------------------------------------------------------
    */

    'cloud' => env('FILESYSTEM_CLOUD', 's3'),

    /*
    |--------------------------------------------------------------------------
    | 定义磁盘
    |--------------------------------------------------------------------------
    */

    'disks' => [

        'local' => [
            'driver' => 'local',
            'root' => storage_path('app'),
        ],

        'public' => [
            'driver' => 'local',
            'root' => storage_path('app/public'),
            'url' => env('APP_URL').'/storage',
            'visibility' => 'public',
        ],

        's3' => [
            'driver' => 's3',
            'key' => env('AWS_ACCESS_KEY_ID'),
            'secret' => env('AWS_SECRET_ACCESS_KEY'),
            'region' => env('AWS_DEFAULT_REGION'),
            'bucket' => env('AWS_BUCKET'),
            'url' => env('AWS_URL'),
            'endpoint' => env('AWS_ENDPOINT'),
        ],

    ],

    /*
    |--------------------------------------------------------------------------
    | 定义符号链接
    |--------------------------------------------------------------------------
    */

    'links' => [
        public_path('storage') => storage_path('app/public'),
    ],

];
```

配置信息中，`driver`指定驱动类型，`local`就是使用本地文件，除此之外，还支持云存储、FTP等。具体配置跟驱动类型有关，这里参考相关文档即可。

注：其中`s3`指`Amazon S3 OSS`，国内一般使用阿里云、七牛云等，不需要直接去掉就行了。

## 符号链接

Laravel框架中，允许直接访问的文件都位于项目根目录的`public`下。如果需要访问其他位置的文件，最佳解决方案是创建符号链接。

符号链接用于浏览器直接通过URL来访问虚拟文件系统中公开的文件，基于不同的操作系统（真实文件系统），实现上有区别。Linux下是`ln -s`创建的软链接，Windows下是`mklink /J`创建的`Junction Point`。

```php
'links' => [
    public_path('storage') => storage_path('app/public'),
],
```

在配置文件`config/filesystems.php`中定义好如上符号链接后，我们需要执行如下命令在真实的文件系统中创建符号链接：

```
php artisan storage:link
```

执行该命令后，符号链接的文件描述符会创建在工程根目录的`public`文件夹下，并指向`storage/app/public`。

## 文件上传

下面我们使用Laravel的文件系统，实现一个文件上传下载功能。

upload.blade.php
```html
<!doctype html>
<html lang="zh-cmn-Hans">
<head>
    <meta charset="UTF-8">
    <meta name="viewport"
          content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>上传</title>
</head>
<body>
    <form action="{{ url('/upload')  }}" enctype="multipart/form-data" method="post">
        @csrf
        <input type="file" name="file" />
        <input type="submit" value="提交" />
    </form>
</body>
</html>
```

UploadController.php
```php
public function doUploadLocal(Request $request)
{
    if ($request->hasFile('file') && $request->file('file')->isValid()) {
        $file = $request->file('file');
        $path = $file->store('avatars', 'public');
        return '上传成功 路径：' . $path;
    } else {
        abort(400);
    }
}
```

```
上传成功 路径：avatars/BOLDZHHmBnV2KrfYX1Kjzj0RH1JLnO03paG8R3rR.jpeg
```

其实文件上传代码非常简单，Laravel能够自动识别文件上传的表单类型，只需要用`$request->file()`取出就可以了。

取出文件对象后，直接调用`$file->store()`，第一个参数是相对路径，这里我们将其存储到`avatars`文件夹，第二个参数指定磁盘（虚拟磁盘），这里我们使用的是`public`，它实际对应的路径是`storage_path('app/public')`。

## 文件下载URL

`Storage::url()`能够根据文件系统的路径，生成用于客户端访问的URL。这样，我们就可以直接根据生成的文件路径，在浏览器中访问该文件了。

```php
public function doUploadLocal(Request $request)
{
    if ($request->hasFile('file') && $request->file('file')->isValid()) {
        $file = $request->file('file');
        $path = $file->store('avatars', 'public');
        return Storage::url($path);
    } else {
        abort(400);
    }
}
```

URL访问例子：
```
http://localhost:8000/storage/avatars/YOubreVOy2zbaZEn9Ux94b15Dm51PyBFwrRWVl6g.jpeg
```

上面URL中，`storage`是我们定义的符号链接，指向文件系统中的`app/public`，`avatars`和后面得文件名，则是我们上传时在文件系统中创建的。

注意：要想浏览器访问到生成的文件URL，必须执行`php artisan storage:link`为所选的公共磁盘创建符号链接。

## 文件下载响应

上面我们通过URL来访问上传的文件，但有时候我们不希望上传的内容都是公开的，我们需要手动进行一些判断，然后动态读取文件返回给客户端，这时可以使用Laravel框架提供的`response()->download()`来生成文件下载响应：

```php
public function downloadLocal(Request $request)
{
    $file_id = $request->input('file_id');
    try {
        return response()->download(storage_path('app/public/avatars/' . $file_id));
    } catch (FileNotFoundException $e) {
        return response('出错了', 500);
    }
}
```

上面代码中，`$file_id`其实是上传后自动生成的文件名，这里我们调用`response()->download()`，参数是下载文件的路径，Laravel能够自动生成文件下载响应。

## 文件操作

### 读取文件

`get()`可以返回指定文件的字符串内容：

```php
$contents = Storage::disk('public')->get('data.txt');
```

`exists()`可以判断指定文件是否存在：

```php
$is_exist = Storage::disk('public')->exists('data.txt');
```

### 写入文件

`prepend()`和`append()`方法能在文件的开头或结尾写入数据：

```php
Storage::disk('public')->append('data.txt', '你好');
```

### 复制和移动文件

`copy()`能够复制文件到新位置，`move`能够将文件移动到新位置，也可以用来实现文件重命名：

```php
Storage::disk('public')->copy('data.txt', 'data2.txt');
Storage::disk('public')->move('data.txt', 'data3.txt');
```

### 删除文件

`delete()`能够删除文件：

```php
Storage::disk('public')->delete('data.txt', 'data3.txt');
```

## 目录操作

### 获取目录下所有文件

`files()`能够返回指定目录下的所有文件，`allfiles()`能够返回指定目录下，包括子目录的所有文件。

```php
$files = Storage::disk('public')->files();
$all_files = Storage::disk('public')->allFiles();
```

返回值为数组类型。

### 获取目录下所有目录

`directories()`能够返回指定目录下的所有文件夹，`allDirectories()`能够返回指定目录下，包括子目录的所有文件夹。

```
$dirs = Storage::disk('public')->directories();
$all_dirs = Storage::disk('public')->allDirectories();
```

### 创建目录

`makeDirectory()`能够创建一个目录。

```php
Storage::disk('public')->makeDirectory('testdir');
```

### 删除目录

`deleteDirectory()`能够删除一个目录。

```php
Storage::disk('public')->deleteDirectory('testdir');
```
