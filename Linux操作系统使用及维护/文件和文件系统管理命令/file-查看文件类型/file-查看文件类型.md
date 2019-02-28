# file命令

`file`命令用于输出一个文件的文件类型，可以用来查看文本文件编码，和二进制文件的MIME类型。

## 查看文本文件编码

`file <filename>`

例子：

![](res/1.png)

## 查看文件MIME类型

`file -i <filename>`

例子:

![](res/2.png)

可以观察到，`blog.mp4`虽然后缀名是mp4，但实际上是avi，`movie.mp4`才是真正的mp4。
