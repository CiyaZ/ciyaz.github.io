# C和C++混合编程

我们知道，很多系统底层的库都是由C语言写的，我们开发软件项目时，为了模块化的开发，也可能使用C语言写一些基础的函数库作为动态链接库（windows对应.dll，linux下对应.so）加载到我们的程序中。这篇笔记介绍如何使用C语言和C++混合编程，尤其是如何使用C++调用C写的函数库。

## 在C++中调用C写的函数

func.c
```c
int max(int a, int b)
{
	if(a > b)
		return a;
	else
		return b;
}
```

func.h
```c
#ifndef FUNC_H
#define FUNC_H

#ifdef __cplusplus
extern "C"
{
#endif

	int max(int a, int b);

#ifdef __cplusplus
}
#endif

#endif
```

main.c
```cpp
#include <iostream>
#include "func.h"

int main()
{
	int x = 1;
	int y = 2;

	std::cout << max(x, y) << std::endl;

	return 0;
}
```

编译和链接
```
gcc -c func.c
g++ -c main.cpp
g++ func.o main.o -o main
```

注意头文件中我们使用了`extern "C"`这样的写法，这是因为在C++中，为了支持函数重载，默认编译后的函数名和代码中定义的函数名肯定是不一样的，这和C语言不同。如果不使用`extern "C"`，就找不到函数定义的位置了。加上这个关键字，C++编译器会按照C语言的方式，不改变原来的函数名，这样就能正常调用C语言中写的函数了。

注意`__cplusplus`这个宏，通常C++编译器会默认定义这个宏，因此我们可以通过它来识别引用这个头文件的源代码，它使用的编译器是C编译器还是C++编译器。

```
#ifdef __cplusplus
extern "C"
{
#endif
```

这种写法保证了C++可以调用头文件中的函数，C调用也不会因为编译器不认识`extern "C"`而出错。

## 在C++中调用C编写的动态链接库

调用动态链接库的道理和上面是完全一样的，代码无需更改，这次我们先把`func.c`编译成`.so`库，然后编译`main.cpp`时，链接这个库。

编译和链接
```
gcc -shared -o libfunc.so func.c
g++ main.cpp -L. -lfunc -I. -o main
```

执行：注意这里要先指定下我们编写的共享库的路径
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:.
./main
```

编译相关的内容可以参考`C++/C语言基础/静态库和共享库`相关章节。
