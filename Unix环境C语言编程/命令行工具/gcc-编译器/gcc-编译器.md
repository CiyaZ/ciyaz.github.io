# gcc

## 常用命令参数

* ﻿`-c`：编译生成目标文件
* `-Dmacro[=defn]`：定义宏
* `-E`：只做预处理而不编译
* `-g`：在生成的目标文件中添加调试信息，所谓调试信息就是源代码和指令之间的对应关系，在gdb调试和objdump反汇编时要用到这些信息
* `-Idir`：头文件所在目录
* `-Ldir`：库文件所在的目录
* `-M和-MM`：输出“.o文件: .c文件 .h文件”这种形式的Makefile规则， `-MM`的输出不包括系统头文件
* `-o`：指定文件名
* `-O`：编译优化
* `-print-search-dirs`：打印库文件的默认搜索路径
* `-S`：生成汇编代码
* `-v`：打印详细的编译链接过程
* `-Wall`：打印所有的警告信息
* `-Wl,options`：options是传递给链接器的选项﻿

## 使用实例

### 编译静态库

```shell
#编译
gcc -c stack.c
#打包
ar rs libstack.a stack.o
#...省去目录操作步骤
#编译main.c和静态库链接到一起
gcc main.c -L. -lstack -Istack -o main
```

### 编译共享库

```shell
gcc -c -fPIC stack.c
gcc -shared -o libstack.so stack.o
#...省去目录操作步骤
gcc main.c -g -L. -lstack -Istack -o main
```

### 编译有正式版本的共享库

```shell
gcc -c -fPIC stack.c
#编译时使用的realname
ln -s libstack.so.1.0 libstack.so
gcc -shared -Wl,-soname,libstack.so.1 -o libstack.so.1.0 stack.o push.o pop.o is_empty.o
#执行时加载的soname
ln -s libstack.so.1.0 libstack.so.1
./main
```

### 查看头文件依赖

```shell
gcc -MM *.c
```
