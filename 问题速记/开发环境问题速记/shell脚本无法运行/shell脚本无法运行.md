# 问题

```
sh xxx.sh
```

上述命令运行bash shell脚本无法运行，提示语法错误。

# 解决

```
chmod +x xxx.sh
./xxx.sh
```

# 原因

如果系统使用zsh等其他shell，可能语法不兼容最通用的bash shell，而sh命令默认使用当前shell，而shell脚本第一行都指明了运行的环境，比如`#!/bin/bash`指定了bash shell的解释器，所以添加可执行权限再直接运行就没问题了。
