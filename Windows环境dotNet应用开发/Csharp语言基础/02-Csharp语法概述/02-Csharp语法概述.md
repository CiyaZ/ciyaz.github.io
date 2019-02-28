# C#语法概述

这篇笔记记录C#的语法，假定读者已经扎实掌握了Java语言，在此基础上，说明一些C#和Java不同，以求快速掌握C#语言的基本使用。本文没提到的语言特性，基本就是和Java一样，放心用就行了！

## 代码规范上的差异

C#一般建议对于局部变量和私有的成员变量使用`lowerCamleCase`（也有人建议私有成员变量使用下划线`_`或是小写字母`m`开头，Java中也是一样的），其余（公开的成员变量、方法名、类名、命名空间）使用`PascalCase`，这和Java不同。

Java建议变量、方法名使用`lowerCamleCase`，类名使用`PascalCase`，包名使用全小写。

## 命名空间 namespace

C#中，命名空间相当于Java的包（package）。

上一篇笔记中，Hello，world程序的例子，有这样一句：
```csharp
using System;
```

因为我们要使用`System.Console`这个类的`WriteLine()`函数，因此引入了`System`这个命名空间。如果不引入，我们使用`Console`时，就必须写它的全限定类名`System.Console.WriteLine()`了。

创建namespace需要使用`namespace`关键字：

```csharp
namespace Company
{
  namespace Project
  {
  }
}
```

namespace的命名规则：
```
公司名.项目名.子系统名
```

其中是可以包含点号的，例如：
```
CiyaZ.DemoProject.FirstDemo
```

命名空间遵循首字母大写的规范。

和Java的区别：Java的包结构和源代码结构都统一使用文件路径与文件名组织，简单粗暴而且高效；C#使用namespace关键字组织命名空间结构，而源代码路径则由构建工具（Visual Studio）单独组织，类名和`.cs`文件名可以不同，更加灵活。

## 变量的自动类型推断

C#声明变量时直接初始化，能够使用自动类型推断功能，示例代码如下：

```csharp
using System;

namespace CiyaZ
{
    public class Demo
    {
        static void Main(string[] args)
        {
            var i = 1;
            var j = 1.2;
            var str = "hello";
            Console.WriteLine("type i : " + i.GetType());
            Console.WriteLine("type j : " + j.GetType());
            Console.WriteLine("type str : " + str.GetType());
        }
    }
}
```

不在声明时直接初始化就不能使用自动类型推断了，编译是无法通过的。

其实这个功能除了降低代码可读性外，并没有什么卵用。

## 常量

C#的关键字`const`可以定义常量，编译时常量的具体值会替换掉引用常量位置的代码。常量声明在类内部或是方法内部都是可以的。

```csharp
const int i = 100;
```

和Java的区别：Java没有直接实现“常量”这个功能，而是使用`static final`声明一个不可改变的静态变量，其实就是常量，效果是一样的。Java中，`static`是不能用于方法内部的，`final`可以用于方法内部。

## 基本类型和引用类型

基本类型：

* 整数：8位有符号整数sbyte 8位无符号整数byte 16位有符号整数short 16位无符号整数ushort 32位有符号整数int 32位无符号整数uint 64位有符号整数long 64位无符号整数ulong
* 浮点数：32位单精度浮点数float 64位双精度浮点数double
* 高精度浮点数：decimal
* 布尔类型：bool
* 字符类型：char

引用类型：

* 对象：object
* 字符串：string

值传递、引用传递规则和Java相同，string同样为不可变对象（看起来像是值传递）。

## foreach循环

C#的foreach循环和Java写法不太一样。

Java循环一个数组：
```java
int[] arr = {1, 2, 3};
for(int i : arr)
{
  System.out.println(i);
}
```

C#循环一个数组：
```csharp
int[] arr = {1, 2, 3};
foreach (int i in arr)
{
    Console.WriteLine(i);
}
```

C#的foreach也可以使用自动类型推断。
