# Linq查询

我们知道Java有Hibernate这种框架，它能够基于ORM映射，让使用者直接编写基于Java对象的HQL查询，然后由框架底层根据数据库自动映射为特定的SQL进行持久层访问。更进一步，能不能在集合（List等）上实现HQL查询呢？Java中没有这种功能（据说Scala有），而C#发明了Linq。

Linq是个相当好用的特性，它可以在集合中用类似SQL的方式进行查询，用法和Hibernate的HQL比较神似，但其实两者解决的问题还是不同的，这篇笔记我们简单介绍一下Linq查询。

## 例子

下面例子实现了在一个`List<string>`中，查询长度为4的所有字符串：

```csharp
using System;
using System.Linq;
using System.Collections.Generic;

namespace demo01
{
    class Program
    {
        static void Main(string[] args)
        {
            List<string> strings = new List<string>();
            strings.Add("Tom");
            strings.Add("Jerry");
            strings.Add("Kate");
            strings.Add("Lucy");
            strings.Add("Bob");

            // 查询长度为4的字符串
            IEnumerable<string> linqQuery = from s in strings where s.Length == 4 select s;
            // 输出结果
            foreach (string result in linqQuery)
            {
                Console.WriteLine(result);
            }

            Console.ReadLine();
        }
    }
}
```

有人可能觉得上面这里例子其实用不用Linq没什么区别，迭代判断实现也不复杂，实际上，Linq其实就是体现了一种结构化查询的思想，这和在关系型数据库中，我们使用的SQL语言的理由是一样的。如果集合元素是个复杂的对象，而且有多个集合「关联查询」的情况，Linq的优势就体现出来了。
