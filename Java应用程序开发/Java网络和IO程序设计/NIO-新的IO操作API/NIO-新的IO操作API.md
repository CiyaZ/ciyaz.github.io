# NIO简介

Java中，最常用的IO方式是基于流的，比如FileInputStream。流式IO被视为单个字节的流动，而NIO（New IO）则是基于块的，每次读写的不是一个字节而是一个缓冲区。因此，通常NIO的读写性能更高，但没有流式IO使用简单。流式IO位于`java.io.*`包中，NIO位于`java.nio.*`中。

注意：实际上`java.io.*`下一些带缓冲区的IO实现也已经是基于NIO的实现的，这样IO性能更好，但是NIO还具有一些`java.io.*`中没有的好处，你可以把NIO看成一种更底层，功能更强大的API。

# 通道和缓冲区

## Channel 通道

Channel相当于流式IO中的流，读写文件到内存缓冲区（Buffer），要经过Channel。

## Buffer 缓冲区

缓冲区就是内存中的一块区域，IO过程中数据会在这块区域停留，最常用的的缓冲区是ByteBuffer（字节缓冲区），实际上对于Java每种基本类型都有对应的缓冲区，例如IntBuffer。

## 例子

复制文件
```java
public static void main(String[] args) throws Exception
{
  File srcFile = new File("src/1.png");
  File targetFile = new File("src/2.png");

  FileInputStream fileInputStream = new FileInputStream(srcFile);
  FileOutputStream fileOutputStream = new FileOutputStream(targetFile);

  //获得Channel
  FileChannel channelIn = fileInputStream.getChannel();
  FileChannel channelOut = fileOutputStream.getChannel();

  //分配缓冲区
  ByteBuffer byteBuffer = ByteBuffer.allocate(1024);

  while(true)
  {
    //清空缓冲区
    byteBuffer.clear();
    int n = channelIn.read(byteBuffer);
    if(n == -1)
    {
      channelIn.close();
      channelOut.close();
      break;
    }
    //将缓冲区从读取模式转换为输出模式
    byteBuffer.flip();
    channelOut.write(byteBuffer);
  }
}
```

上述代码中，首先从流对象中获得Channel对象，然后分配缓冲区，从channelIn读入缓冲区，从channelOut写出，最终完成文件的复制。

将数组中内容写入文件

```java
public static void main(String[] args) throws Exception
{
  File file = new File("src/test.txt");
  FileOutputStream fileOutputStream = new FileOutputStream(file);
  FileChannel fileChannel = fileOutputStream.getChannel();

  String str = "hello, world!";
  byte[] bytes = str.getBytes();

  ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
  byteBuffer.put(bytes);

  byteBuffer.flip();

  fileChannel.write(byteBuffer);
  fileChannel.close();
}
```

上述代码中，使用put()将字节数组的内容存入缓冲区，并写入文件。

## 缓冲区读写原理

上面两个例子中，有几个需要解释的地方：

1. read()/put()和flip()是如何实现的？
2. 为什么例2缓冲区分配了1024B，却只向文件写了bytes大小的数据？

我们需要了解Buffer的内部实现。

Buffer有三个状态变量：

* position 当前位置：buffer读取时，position只向下一个可写入字节，buffer输出时，position只向下一个要输出的字节
* limit 限制：buffer读取时，limit=capacity为缓冲区分配的容量，buffer输出时，为buffer已写入的数据长度
* capacity 容量：buffer分配的容量

这些状态变量都有对应的方法可以获取，如`buffer.limit()`。position和limit还能手动指定。

### read()和put()做了什么？

read()和put()类似，一个是从通道中写入缓冲区，一个是从字节数组中写入缓冲区。

例如调用put()时，buffer是初始的读状态，因此limit和capacity都表示缓冲区容量，随着读操作执行，position向前移动。

### flip()做了什么？

当调用flip()，buffer从读取模式转换为输出模式，limit值变为position，表示已写入数据长度，然后position归为0，随着后续写入操作执行，position重新向前移动。

### clear()做了什么？

上述代码中，还使用了clear()操作，它将缓冲区恢复为最初始的状态，即调用flip()和读取数据之前。

## 访问缓冲区

实际上，缓冲区有一组get()/put()方法，用来读写缓冲区。这些方法中，要注意的是有的方法会改变缓冲区的状态变量，有的则不会。

例如：`put(int index, byte b)`是修改缓冲区指定位置的字节，不会改变缓冲区的状态变量，而`put(byte b)`则是向缓冲区写入一个字节，这个操作会改变缓冲区的状态变量，这实际上很好理解，不需要解释。具体的用法，还请参考JDK文档。

## 缓冲区的其他操作

### 分配和包装

上面代码中的`Bytebuffer.allocate(1024)`表示分配一个1024B的缓冲区。实际上，缓冲区还可以从byte[]数组包装而来，使用wrap函数进行包装：`Bytebuffer.wrap(byteArray)`。

### 子缓冲区

我们可以在一个大缓冲区上创建一个子缓冲区窗口，他们共享底层的内存。分配一个子缓冲区通常用于较复杂的缓冲区操作，它比多次指定position更加简洁。

```java
ByteBuffer buffer = ByteBuffer.allocate(10);

buffer.position(3);
buffer.limit(7);
ByteBuffer slice = buffer.slice();
```

### 只读缓冲区

```java
ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
ByteBuffer readOnlyBuffer = byteBuffer.asReadOnlyBuffer();
```

使用`buffer.asReadOnlyBuffer()`可以获得一个只读缓冲区，尝试向只读缓冲区写入会引发运行时异常。

### 分散/聚集IO

填充C语言结构体时，有一种方便的写法：直接将结将构体应用于某一段内存缓冲区，便于格式化读取这段缓冲区的数据。例如接收若干字节的网络协议数据包，然后直接将其应用于该协议结构体，直接读取该结构体字段即可。Java也可以实现这种功能，但是用的不是结构体，而是分散/聚集IO。

```java
long read(ByteBuffer[] dsts);
long read(ByteBuffer[] dsts, int offset, int length);
long write(ByteBuffer[] srcs);
long write(ByteBuffer[] srcs, int offset, int length);
```

* 分散读取：传入的缓冲区数组会依次填满。
* 分散输出：传入的缓冲区数组中内容会依次写出。

实际使用时，将各个缓冲区的引用保存在一个类属性中，通常更加方便。

## 提高缓冲区的性能

### allocateDirect 直接分配缓冲区

上面介绍的代码中都是使用`ByteBuffer.allocate(int opacity)`分配的缓冲区，实际上可以使用`ByteBuffer.allocateDirect(int opacity)`分配一个“直接缓冲区”。

Java抽象了底层实现，“非直接缓冲区”实际上是在JVM堆中分配一个数组，IO时数据从硬件读入内核缓冲区，内核缓冲区中可能拷贝多次才到达我们分配的缓冲区的底层数组，而“直接缓冲区”则是JVM根据具体平台，尽力减少了这种拷贝过程，因此性能更高。但是脱离了JVM，就一定要记得，用完后将缓冲区的数据回收。

```java
ByteBuffer byteBuffer = ByteBuffer.allocateDirect(1024);
((DirectBuffer)byteBuffer).cleaner().clean();
```

### 内存映射文件

内存映射文件是操作系统提供的，它可以将文件映射到一个内存地址，像读写内存一样读写文件。在LinuxC编程相关章节中也有介绍。Java中我们也可以利用这种高级的IO功能实现高性能的文件IO。

```java
MappedByteBuffer mbb = fc.map( FileChannel.MapMode.READ_WRITE, 0, 1024 );
```

上述代码中，将文件的前1024B通过FileChannel对象映射到了缓冲区。

## 关于Unicode的处理

NIO提供了Charset类进行文本的编码处理，使用起来十分简单。

例子
```java
public static void main(String[] args) throws Exception
{
  File file = new File("src/test.txt");
  FileChannel fileChannel = (new FileInputStream(file)).getChannel();
  ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
  fileChannel.read(byteBuffer);
  fileChannel.close();

  Charset charset = Charset.forName("utf-8");
  CharsetDecoder decoder = charset.newDecoder();
  byteBuffer.flip();
  CharBuffer charBuffer = decoder.decode(byteBuffer);

  System.out.println(charBuffer.toString());
}
```

Charset类将ByteBuffer转换为对应编码的CharBuffer。注意：ByteBuffer读取前一定要flip()，而生成的CharBuffer已经处于输出状态。其position已经归零，limit则为数据长度。
