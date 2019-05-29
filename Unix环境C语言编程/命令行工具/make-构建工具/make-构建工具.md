# Makefile
Makefile是一个构建工具，一般的程序都是由多个源文件编译链接而成的，这些源文件的处理步骤通常用Makefile来管理。Makefile由一组规则（ Rule） 组成，每条规则的格式是：

```Makefile
target ... : prerequisites ...
    command1
    command2
    ...

例如main是目标，.o文件称为规则（必须存在的依赖文件）：
main: main.o stack.o maze.o
    gcc main.o stack.o maze.o -o main
```

注：
* Makefile语法十分严格，命令必须以tab开头，make会调用shell执行这些命令。
* 终端输入make命令会自动寻找Makefile。
* make会自动选择那些受影响的源文件重新编译，不受影响的源文件则不重新编译。

通常Makefile都会有一个clean规则，用于清除编译过程中产生的二进制文件，保留源文件，例（使用make clean调用）：

```Makefile
clean:
    @echo "cleanning project"
    -rm main *.o
    @echo "clean completed"
```

注：
* 如果make执行的命令前面加了`@`字符，则不显示命令本身而只显示它的结果
* 通常make执行的命令如果出错（该命令的退出状态非0）就立刻终止，不再执行后续命令，但如果命令前面加了`-`号，即使这条命令出错， make也会继续执行后续命令。
* 如果当前目录存在一个叫clean的文件， clean目标又不依赖于任何条件， make就认为它不需要更新了。而我们希望把clean当作一个特殊的名字使用，不管它存在不存在都要更新，可以添一条特殊规则，把clean声明为一个伪目标：`.PHONY: clean`

clean目标是一个约定俗成的名字，在所有软件项目的Makefile中都表示清除编译生成的文件，类似这样的约定俗成的目标名字有：
* `all`，执行主要的编译工作，通常用作缺省目标。
* `install`，执行编译后的安装工作，把可执行文件、配置文件、文档等分别拷到不同的安装目录。
* `clean`，删除编译生成的二进制文件。
* `distclean`，不仅删除编译生成的二进制文件，也删除其它生成的文件，例如配置文件和格式转换后的文档，执行`make distclean`之后应该清除所有这些文件，只留下源文件。

# 隐含规则和模式规则

其实Makefile有很多灵活的写法，可以写得更简洁，同时减少出错的可能。本节我们来看看这样一个例子还有哪些改进的余地。

一个目标依赖的所有条件不一定非得写在一条规则中，也可以拆开写，但命令列表只能有一个，例如：
```Makefile
main.o: main.h stack.h maze.h
main.o: main.c
    gcc -c main.c
```

如果一个目标在Makefile中的所有规则都没有命令列表， make会尝试在内建的隐含规则（ Implicit Rule）数据库中查找适用的规则。make的隐含规则数据库可以用make -p命令打印，打印出来的格式也是Makefile的格式，包括很多变量和规则。下例中有几个目标没有命令列表，但也能正常运行：
```Makefile
main: main.o stack.o maze.o
    gcc main.o stack.o maze.o -o main
main.o: main.h stack.h maze.h
stack.o: stack.h main.h
maze.o: maze.h main.h
clean:
    -rm main *.o
.PHONY: clean
```

一些make的隐含规则：
```Makefile
# default
OUTPUT_OPTION = -o $@
# default
CC = cc
# default
COMPILE.c = $(CC) $(CFLAGS) $(CPPFLAGS) $(TARGET_ARCH) -c
%.o: %.c
# commands to execute (built-in):
    $(COMPILE.c) $(OUTPUT_OPTION) $<
```

`#`号在Makefile中表示单行注释，就像C语言的`//`注释一样。`CC`是一个Makefile变量，用`CC = cc`定义和赋值，用`$(CC)`取它的值，其值应该是`cc`。 Makefile变量像C的宏定义一样，代表一串字符，在取值的地方展开。`cc`是一个符号链接，通常指向gcc。`$@`和`$<`是两个特殊的变量，`$@`的取值为规则中的目标，`$<`的取值为规则中的第一个条件。`%.o:%.c`是一种特殊的规则，称为模式规则（ Pattern Rule）。
