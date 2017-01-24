---
layout: post
title : 《Thinking in Java》再读笔记(二)
category : book
---
{% include JB/setup %}

上一次对concurrent部分重点读了一遍,这一次对NIO部分进行了重读.对FileChannel,ByteBuffer(包括其视图缓冲器CharBuffer,ShortBuffer,IntBuffer,LongBuffer,FloatBuffer,DoubleBuffer,MappedByteBuffer)以及文件加锁等内容有了更深的认识.

下面是这次重读时的一些摘录.

### 1.新I/O

	 JDK 1.4的java.nio.*包中引入了新的JavaI/O类库,其目的在于提高速度.实际上，旧的I/O包已经使用nio重新实现过，以便充分利用这种速度，即使我们不显式地用nio编写代码，也能从中受益！nio速度的提高来自于所使用的结构更接近于操作系统执行I/O的方式：通道和缓冲器。

### 2.缓冲器细节

Buffer由数据和可以高效地访问及操纵这些数据的四个索引组成:mark,position,limit和capacity.

    capacity()           返回缓冲区容量
    clear()              清空缓冲区,将position置0,limit置为capacity.我们可以调用此方法复写缓冲区
    flip()               将limit置为position,position置为0.此方法用于准备从缓冲区读取已经写入的数据
    limit()              返回limit值
    limit(int lim)       设置limit值
    mark()               将mark置为position
    position()           返回position值
    position(int pos)    设置position值
    remaining()          返回limit-position
    hasRemaining()       若有介于position和limit之间的元素,则返回true

### 3.读写例子

```java
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class ChannelCopy {
  private static final int BSIZE = 1024;
  public static void main(String[] args) throws Exception {
    if(args.length != 2) {
      System.out.println("arguments: sourcefile destfile");
      System.exit(1);
    }
    FileChannel
      in = new FileInputStream(args[0]).getChannel(),
      out = new FileOutputStream(args[1]).getChannel();
    ByteBuffer buffer = ByteBuffer.allocate(BSIZE);
    while(in.read(buffer) != -1) {
      buffer.flip(); // Prepare for writing
      out.write(buffer);
      buffer.clear();  // Prepare for reading
    }
  }
} ///:~
```

当FileChannel.read()放回-1时,表示达到了输入的末尾.每次read()操作,会将数据输入到缓冲器中,flip()则是准备缓冲器以便它的信息可以由write()提取.clear()操作则对所以内部指针进行重新安排.

上面的操作也可以用特殊方法transferTo()和transferFrom()将两个通道相连:

```java
import java.nio.channels.*;
import java.io.*;

public class TransferTo {
  public static void main(String[] args) throws Exception {
    if(args.length != 2) {
      System.out.println("arguments: sourcefile destfile");
      System.exit(1);
    }
    FileChannel
      in = new FileInputStream(args[0]).getChannel(),
      out = new FileOutputStream(args[1]).getChannel();
    in.transferTo(0, in.size(), out);
    // Or:
    // out.transferFrom(in, 0, in.size());
  }
} ///:~
```

### 4.转换数据

```java
import java.nio.*;
import java.nio.channels.*;
import java.nio.charset.*;
import java.io.*;

public class BufferToText {
  private static final int BSIZE = 1024;
  public static void main(String[] args) throws Exception {
    FileChannel fc =
      new FileOutputStream("data2.txt").getChannel();
    fc.write(ByteBuffer.wrap("Some text".getBytes()));
    fc.close();
    fc = new FileInputStream("data2.txt").getChannel();
    ByteBuffer buff = ByteBuffer.allocate(BSIZE);
    fc.read(buff);
    buff.flip();
    // Doesn't work:
    System.out.println(buff.asCharBuffer());
    // Decode using this system's default Charset:
    buff.rewind();
    String encoding = System.getProperty("file.encoding");
    System.out.println("Decoded using " + encoding + ": "
      + Charset.forName(encoding).decode(buff));
    // Or, we could encode with something that will print:
    fc = new FileOutputStream("data2.txt").getChannel();
    fc.write(ByteBuffer.wrap(
      "Some text".getBytes("UTF-16BE")));
    fc.close();
    // Now try reading again:
    fc = new FileInputStream("data2.txt").getChannel();
    buff.clear();
    fc.read(buff);
    buff.flip();
    System.out.println(buff.asCharBuffer());
    // Use a CharBuffer to write through:
    fc = new FileOutputStream("data2.txt").getChannel();
    buff = ByteBuffer.allocate(24); // More than needed
    buff.asCharBuffer().put("Some text");
    fc.write(buff);
    fc.close();
    // Read and display:
    fc = new FileInputStream("data2.txt").getChannel();
    buff.clear();
    fc.read(buff);
    buff.flip();
    System.out.println(buff.asCharBuffer());
  }
} /* Output:
卯浥?數
Decoded using GBK: Some text
Some text
Some text口口口
*///:~
```

缓冲器容纳的是普通的字节,为了把他们转换成字符,我们要么在输入它们的时候对其进行编码(这样,它们输出时才具有意义),要么在将其从缓冲器输出时对它们进行解码.

### 5.获取基本类型

尽管ByteBuffer只能保存字节类型的数据,但是它具有可以从其所容纳的字节中产生出各种不同基本类型值的方法.下面的例子展示了怎样用这些方法来插入和抽取各种数值:

```java
import java.nio.*;

public class GetData {
  private static final int BSIZE = 1024;
  public static void main(String[] args) {
    ByteBuffer bb = ByteBuffer.allocate(BSIZE);
    // Allocation automatically zeroes the ByteBuffer:
    int i = 0;
    while(i++ < bb.limit())
      if(bb.get() != 0)
        System.out.println("nonzero");
    System.out.println("i = " + i);
    bb.rewind();
    // Store and read a char array:
    bb.asCharBuffer().put("Howdy!");
    char c;
    while((c = bb.getChar()) != 0)
      System.out.print(c + " ");
    System.out.println();
    bb.rewind();
    // Store and read a short:
    bb.asShortBuffer().put((short)471142);
    System.out.println(bb.getShort());
    bb.rewind();
    // Store and read an int:
    bb.asIntBuffer().put(99471142);
    System.out.println(bb.getInt());
    bb.rewind();
    // Store and read a long:
    bb.asLongBuffer().put(99471142);
    System.out.println(bb.getLong());
    bb.rewind();
    // Store and read a float:
    bb.asFloatBuffer().put(99471142);
    System.out.println(bb.getFloat());
    bb.rewind();
    // Store and read a double:
    bb.asDoubleBuffer().put(99471142);
    System.out.println(bb.getDouble());
    bb.rewind();
  }
} /* Output:
i = 1025
H o w d y !
12390
99471142
99471142
9.9471144E7
9.9471142E7
*///:~
```

### 6.视图缓冲器

视图缓冲器(view buffer)可以让我们通过某个特定的基本数据类型的视窗查看其底层的ByteBuffer.ByteBuffer依然是实际存储数据的地方,"支持"着前面的视图,因此,对视图的任何修改都会映射成为对ByteBuffer中数据的修改.

### 7.性能

尽管"旧"的I/O流在用nio实现后性能有所提高,但是"映射文件访问"往往可以更加显著地加快速度.下面的程序进行了简单的性能比较:

```java
import java.nio.*;
import java.nio.channels.*;
import java.io.*;

public class MappedIO {
  private static int numOfInts = 4000000;
  private static int numOfUbuffInts = 200000;
  private abstract static class Tester {
    private String name;
    public Tester(String name) { this.name = name; }
    public void runTest() {
      System.out.print(name + ": ");
      try {
        long start = System.nanoTime();
        test();
        double duration = System.nanoTime() - start;
        System.out.format("%.2f\n", duration/1.0e9);
      } catch(IOException e) {
        throw new RuntimeException(e);
      }
    }
    public abstract void test() throws IOException;
  }
  private static Tester[] tests = {
    new Tester("Stream Write") {
      public void test() throws IOException {
        DataOutputStream dos = new DataOutputStream(
          new BufferedOutputStream(
            new FileOutputStream(new File("temp.tmp"))));
        for(int i = 0; i < numOfInts; i++)
          dos.writeInt(i);
        dos.close();
      }
    },
    new Tester("Mapped Write") {
      public void test() throws IOException {
        FileChannel fc =
          new RandomAccessFile("temp.tmp", "rw")
          .getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_WRITE, 0, fc.size())
          .asIntBuffer();
        for(int i = 0; i < numOfInts; i++)
          ib.put(i);
        fc.close();
      }
    },
    new Tester("Stream Read") {
      public void test() throws IOException {
        DataInputStream dis = new DataInputStream(
          new BufferedInputStream(
            new FileInputStream("temp.tmp")));
        for(int i = 0; i < numOfInts; i++)
          dis.readInt();
        dis.close();
      }
    },
    new Tester("Mapped Read") {
      public void test() throws IOException {
        FileChannel fc = new FileInputStream(
          new File("temp.tmp")).getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_ONLY, 0, fc.size())
          .asIntBuffer();
        while(ib.hasRemaining())
          ib.get();
        fc.close();
      }
    },
    new Tester("Stream Read/Write") {
      public void test() throws IOException {
        RandomAccessFile raf = new RandomAccessFile(
          new File("temp.tmp"), "rw");
        raf.writeInt(1);
        for(int i = 0; i < numOfUbuffInts; i++) {
          raf.seek(raf.length() - 4);
          raf.writeInt(raf.readInt());
        }
        raf.close();
      }
    },
    new Tester("Mapped Read/Write") {
      public void test() throws IOException {
        FileChannel fc = new RandomAccessFile(
          new File("temp.tmp"), "rw").getChannel();
        IntBuffer ib = fc.map(
          FileChannel.MapMode.READ_WRITE, 0, fc.size())
          .asIntBuffer();
        ib.put(0);
        for(int i = 1; i < numOfUbuffInts; i++)
          ib.put(ib.get(i - 1));
        fc.close();
      }
    }
  };
  public static void main(String[] args) {
    for(Tester test : tests)
      test.runTest();
  }
} /* Output: (90% match)
Stream Write: 0.67
Mapped Write: 0.10
Stream Read: 0.52
Mapped Read: 0.04
Stream Read/Write: 7.32
Mapped Read/Write: 0.01
*///:~
```

### 8.Arrays实用功能

    1)复制数组                   System.arraycopy()或Arrays.copyOf()或Arrays.copyOfRange()
    2)数组比较                   Arrays.equal()
    3)数组元素比较或排序           Arrays.sort()或指定比较器Arrays.sort(T[] a, Comparator<? super T> c) 
    4)在已排序的数组中查找         Arrays.binarySearch()
    5)数组填充                   Arrays.fill()
    6)转换成List                 Arrays.asList