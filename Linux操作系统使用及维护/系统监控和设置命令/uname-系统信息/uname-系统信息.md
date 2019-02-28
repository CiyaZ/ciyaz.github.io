# uname（Unix Name）

uname命令可以输出系统信息。

```shell
uname -a
```

* -a 选项列出所有可用的系统信息。一般只用这一个选项即可。

结果：
```
Linux Linux 3.16.0-38-generic #52~14.04.1-Ubuntu SMP Fri May 8 09:43:57 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
```

结果说明：

从左到右依次是：

* `Linux` 内核名
* `Linux` 网络主机名（我的主机名叫Linux，不要和内核名弄混）
* `3.16.0-38-generic` 内核release版本
* `#52~14.04.1-Ubuntu SMP Fri May 8 09:43:57 UTC 2015` 内核编译信息
* `x86_64` 操作系统版本
* `x86_64` 处理器类型
* `x86_64` 硬件平台
* `GNU/Linux` 操作系统
