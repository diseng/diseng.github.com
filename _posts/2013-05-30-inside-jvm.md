---
layout: post
title : 《深入Java虚拟机》读书摘录
category : book
tags : [book]
---
{% include JB/setup %}

《深入Java虚拟机》一书已经绝版，但是看到很多人推荐，于是下了个电子版，无奈电子版实在太模糊，看着费眼睛，就去淘宝上买了个复印本，书的质量很不错，内容也很清晰。

这本书对Java虚拟机做了一个从大体系到小模块的大致实现介绍，第一章是Java体系的说明，第二章到第四章是对Java体系每个方面的深入介绍，第五章到第九章是本书的一个重点，分别分章讲解了Java虚拟机、Java class文件、类型的生命周期、连接模型和垃圾收集，第十章到第二十章开始讲解各种操作的指令。

读完本书，对Java虚拟机有了初步的认识，对Java内存模型，Java类型生命周期，垃圾收集算法等都有了一定的认识，下面是读书时候的一些摘录：

## Java体系结构介绍

### 1.Java体系结构

    平台无关性、安全性和网络移动性,Java体系的这三个方面共同使得Java和发展中的网络计算环境相得益彰.

### 2.常见Java虚拟机执行引擎

    1)软件实现
    1.1)一次性解释字节码:最简单的执行引擎
    1.2)即时编译(JIT):更快,更耗内存!第一次被执行的字节码编译成本地机器码,并且缓存重用.
    1.3)自适应优化器:监视程序活动,将使用频繁的代码段编译成本地机器码.
    2)硬件实现
    2.1)内嵌在芯片中的执行引擎,用本地方法执行Java字节码

### 3.类装载器

    类装载器大致分为:启动类装载器、系统类装载器（1.2版本引入）和用户自定义类装载器.当被装载的类引用了另一个类类时,虚拟机就会使用装载第一个类的类装载器装载被引用的类.
    1)启动类装载器通常使用某种默认的方式从本地磁盘中装载类,包括Java API类
    2)系统类装载器:
    3)用户自定义类装载器:使得在运行时扩展Java程序成为可能,扩展的类可以通过网络下载、数据库获取或者动态生成.


### 4.安全管理器&访问控制器

    当Java API的方法进行任何有潜在危险的操作(如I/O)之前,都会通过查询安全管理器来验证是否得到了授权.安全管理器是一个为应用程序提供自定义安全策略的特殊对象(1.2版本中被访问控制器所取代,访问控制器是用来执行栈检验以决定是否准许某种操作的类).

### 5.Java对内存操作的约束

    1)在Java中,没有通过使用强制转换指针类型或者通过进行指针运算直接访问内存的方法
    2)Java避免无意间破坏内存的另一个办法是自动垃圾收集
    3)Java在运行时保护内存完整性的第三个方法是数组边界检查
    4)对对象引用的检查:每次使用引用时,Java都会确保引用不为空值

### 6.Java体系结构代价

    1)和其他技术相比,Java程序的执行速度可能比较低,这是Java在面向网络特性上所付出的最主要的代价之一
    2)Java在面向网络特性上所付出的另一个代价是,在内存管理和线程调度上的缺陷
    3)Java为了实现平台无关性,也要付出代价,即最小公分母问题.如某个特性只在一种OS上存在,API设计者可能会决定不支持这个特性;某个特性在绝大多数OS上存在,设计者可能会决定为不具备这项特性的OS用API模型实现它.
    4)Java class文件中包含了自身和其他类引用字段的名字和描述符(字段的类别)的详细说明,引用成员方法的名称和描述符(方法的返回类型,方法参数的数量和类型)的详细说明,这使得Java class文件逆向编译为Java源码文件相当容易.

## 安全

### 1.类装载器体系结构

    1)类装载器体系结构可以防止恶意的代码去干涉善意的代码,这是通过为由不同类装载器装入的类提供不同的命名空间来实现的
    2)不同命名空间的类之间可以设置一个防护罩.同一个命名空间内的类可以直接进行交互,不同命名空间的类甚至不能觉察彼此的存在,除非显式地提供了允许它们进行交互的机制
    3)类装载器体系结构守护了被信任的类库边界,这是通过分别使用不同的类装载器装载可靠和不可靠的包来实现的

### 2.类装载器保护可信任类库

    1)类装载器体系结构通过剔除装作被信任的不可靠类,来保护可信任类库的边界.
    2)因为java.lang.Integer的class文件在Java API中已经存在,它将被启动类装载器抢先装载,而网络类装载器将没有机会下载并定义名为java.lang.Integer的类,它只能使用由它的双亲返回的类.
    3)因为java.lang.Virus和来自核心Java API的java.lang的成员不属于同一个运行时包,java.lang.Virus就不能访问Java API的java.lang包中的类型和包内可见成员.
    4)类装载器必须将每一个被装载的类放置在一个保护域中,一个保护域定义了这个代码在运行时将得到怎样的权限.

### 3.class文件检验器

    class文件检验器实现的安全目标之一就是程序的健壮性,它会对class文件进行四趟独立的扫描.
    1)第一趟扫描是在类被装载时进行的,在这次扫描中,class文件检验器检查这个class文件的内部结构,以保证它可以被安全地编译.
    2)第二和第三趟是在连接过程中进行的,在这两次扫描中,class文件检验器确认类型数据遵从Java编程语言的语义,包括检验它所包含的所有字节码的完整性.
    3)第四趟扫描是在进行动态连接的过程中解析符号引用时进行的,在这次扫描中,class文件检验器确认被引用的类,字段以及方法确认存在


### 4.Java虚拟机内置的安全特性

    1)通过保证一个Java程序只能使用类型安全的,结构化的方法去访问内存,Java虚拟机使得Java程序更为健壮,也使得它们的运行更为安全.
    2)内置在Java虚拟机中的另一个安全特性——作为内存的结构化访问的一个后背——就是并未指明运行时数据空间在Java虚拟机内部是怎么分布的.
    3)内置于Java虚拟机的最后一个安全特性就是异常的结构化错误处理.

### 5.代码签名和认证

[这里](http://my.freebsd.org.hk/html/jdk1.2/tooldocs/win32/jarsigner.html)是一个代码签名和认证的实例

### 6.doPrivileged()方法

  	1)属于一个权限较少的保护域的方法无权调用属于权限更高的保护域的方法
    2)当调用doPrivileged()方法时,就像调用其他方法一样,会将一个新的栈帧压入栈.再由AccessController执行的栈检查中,一个doPrivileged()方法调用的栈帧标识了检查过程的提前终止点.如果和调用doPrivileged()方法相关联的保护域拥有执行请求操作的权限,AccessController将立即返回.这样这个操作就被允许,即使在栈下层的代码可能没有执行这个操作的权限.

## 网络移动性

### 1.网络移动性支持&控制网络传送class文件时间

    1)动态连接(类明确的在代码中使用,但是程序需要这个类的时候才去装载)
    2)动态扩展(程序运行时,通过Class类的forname()方法等方式,动态装载额外的类)
    3)紧凑的class文件
    4)JAR文件允许在一次网络传输过程中传送多个文件
    5)不采取按需下载class文件的做法

## Java虚拟机

### 1.Java虚拟机内部两种线程

    守护线程:通常是虚拟机自己使用的,比如执行垃圾收集任务的线程.Java程序也可以把创建的任何线程标记为守护线程
    非守护线程:Java程序的初始线程,就是开始于main()的那个,是非守护线程.
    只有还有任何非守护线程在运行,那么Java程序也在继续运行.当非守护线程都终止,虚拟机实例将自动退出.若安全管理器允许,也可通过Runtime或者System的exit()方法退出.

### 2.数据类型

                        |浮点数类型:float,double
             |-数值类型-|整数类型:byte,short,int,long,char
    基本类型-|-boolean
             |-returnAddress
    
    引用类型-|引用:类类型,接口类型,数组类型

    当编译器把Java源码编译为字节码时,会用int或者byte来表示boolean.在Java虚拟机中,false由整数0表示,所有非零整数表示true.boolean数组当作byte数值来访问,但是在"堆"区,它也可以被表示为位域.

    Java虚拟机对byte,short,char是直接支持的,这些类型的值可以作为实例变量或者数组元素存储在局部变量区,也可以作为类变量存储在方法区中.但在局部变量区和操作数栈中都会被转换成int类型的值.它们在栈帧中的时候都是当作int来处理的,只有当它被存回堆或者方法区时,才会转换回原来的类型.

### 3.装载、连接、初始化

    1)装载——查找并装载类型的二进制数据
    2)连接——执行验证、准备、以及解析(可选)
      验证  确保被导入类型的正确性
      准备  为类变量分配内存,并将其初始化为默认值
      解析  把类型中的符号引用转换为直接引用
    3)初始化——把类变量初始化为正确初始值

### 4.类型信息

    对每一个装载的类型,虚拟机都会在方法区中存储以下类型信息:
    1)类型全限定名
    2)类型的直接超类的全限定名(除非java.lang.Object,它没有超类)
    3)类型是类类型还是接口类型
    4)类型的修饰符(public,abstract或final的某个子集)
    5)直接超接口的全限定名有序列表
    6)类型常量池
    7)字段信息
    8)方法信息
    9)除了常量以外的所有类(静态)变量
    10)一个到类ClassLoader的引用
    11)一个到Class类的引用

### 5.栈帧
  
    1)实例方法栈帧的局部变量中第一个参数是一个引用类型(隐含加入的this,表示调用该方法的对象本身)
    2)类方法没有隐含this变量,因为在方法调用时没有关联到一个具体的实例

### 6.指令集设计因素

    1)平台无关性是影响指令集设计的最大因素:指令集以栈为中心,而非以寄存器为中心的设计方法,使得那些只有很少寄存器或者寄存器很没有规律的机器上实现Java更便利.
    2)Java以栈为中心设计指令集的另一个动机是,编译器一般采用以栈为基础的结构向连接器或者优化器传递编译的中间结果
    3)指导指令集设计的另一个目标就是进行字节码验证的能力,特别是使用数据流分析器进行的一次性验证.

## Java class文件

### 1.字段

    尽管在Java程序设计语言中不会有两个具有相同名字的字段存在于同一个类或者接口中,但一个class文件中的两个字段可以拥有同一个名字——只有他们的描述符不同.

### 2.方法

    在Java源文件的同一个类里,如果声明了两个具有相同名字和相同参数类型,但返回值不同的方法,这个程序将无法编译通过.在Java程序设计语言中,不能仅仅通过返回值不同来重载方法.但是这样的两个方法可以和谐地在一个class文件中共存.

## 类型的生命周期

### 1.验证

    1)超类需要在子类初始化前被初始化,所以这些类应该已经被装载.
    2)当实现了父接口的类被初始化时,不需要初始化父接口.
    3)当实现了父接口的子类(或者是扩展了父接口的子接口)被装载时,父接口也必须被装载(只是装载,不会被初始化)

### 2.clinit()方法

    所有的类变量初始化语句和类型的静态初始化器都被Java编译器收集在一起,放到一个特殊的方法中.对于类来说,这个方法被称作类初始化方法;对于接口来说,它被称为接口初始化方法.在类和接口的Java class文件中,这个方法被称为<clinit>方法.通常的Java程序方法是无法调用这个<clinit>方法的,它只能被Java虚拟机调用,专门把类型的静态变量设置为正确的初始值.

### 3.主动使用

    Java虚拟机在首次主动使用类型时初始化它们.有6种活动被认为是主动使用:
    1)创建类的新实例
    2)调用类中声明的静态方法
    3)操作类或者接口中声明的非常量静态字段
    4)调用Java API中特定的反射方法
    5)初始化一个类的子类
    6)指定一个类作为Java虚拟机启动时的初始化类

### 4.被动调用

    1)使用一个非常量静态字段只有当类或者接口的确声明了这个字段是才是主动使用.对于子类,子接口和实现了接口的类来说,这就是被动调用

{% highlight java %}
class NewParent {

    static int hoursOfSleep = (int) (Math.random() * 3.0);

    static {
        System.out.println("NewParent was initialized.");
    }
}

class NewbornBaby extends NewParent {

    static int hoursOfCrying = 6 + (int) (Math.random() * 2.0);

    static {
        System.out.println("NewbornBaby was initialized.");
    }
}

class Example2 {

    // Invoking main() is an active use of Example2
    public static void main(String[] args) {

        // Using hoursOfSleep is an active use of NewParent,
        // but a passive use of NewbornBaby
        int hours = NewbornBaby.hoursOfSleep;
        System.out.println(hours);
    }

    static {
        System.out.println("Example2 was initialized.");
    }
}/* Output:
Example2 was initialized.
NewParent was initialized.
0
*///:~
{% endhighlight %}

    2)如果一个字段既是静态的(static)又是最终的(final),并且使用一个编译时常量表达式初始化,使用这样的字段,就不是对声明该字段类的主动使用.Java编译器把这样的字段解析成对常量的本地拷贝.

{% highlight java %}
interface Angry {

    String greeting = "Grrrr!";

    int angerLevel = Dog.getAngerLevel();
}

class Dog {

    static final String greeting = "Woof, woof, world!";

    static {
        System.out.println("Dog was initialized.");
    }

    static int getAngerLevel() {

        System.out.println("Angry was initialized");
        return 1;
    }
}

class Example3 {

    // Invoking main() is an active use of Example3
    public static void main(String[] args) {

        // Using Angry.greeting is a passive use of Angry
        System.out.println(Angry.greeting);
      

        // Using Dog.greeting is a passive use of Dog
        System.out.println(Dog.greeting);
    }

    static {
        System.out.println("Example3 was initialized.");
    }
}/* Output:
Example3 was initialized.
Grrrr!
Woof, woof, world!
*///:~
{% endhighlight %}

### 5.明确类实例化

    实例化一个类有四种途径:
    1)明确地使用new操作符
    2)调用Class或者java.lang.reflect.Constructor对象的newInstance()方法
    3)调用任何现有对象的clone()方法
    4)通过java.io.ObjectInputStream类的getObject()方法反序列化

### 6.隐含类实例化

    1)在任何Java程序中第一个隐含实例化对象可能是保存命令行参数的String对象
    2)对于Java虚拟机装载的每一个类型,它会暗自实例化一个Class对象来代表这个类型.
    3)当Java虚拟机装载了在常量池中包含CONSTANT_String_info入口的类的时候,它会创建新的String对象的实例来表示这些常量字符串
    4)执行包含字符串连接操作符的表达式产生的对象

### 7.init()方法

    Java编译器为它编译的每一个类都至少生成一个实例初始化方法.在Java的class文件中,这个实例初始化方法被称为<init>().针对源代码中每一个类的构造方法,Java编译器都产生一个<init>()方法.如果类没有明确声明构造方法,编译器默认产生一个无参数的构造方法,它仅仅调用超类的无参数构造方法.
    一个<init>()方法可能包含三种代码:
      1)调用另一个<init>()方法
      2)实现对任何实例变量的初始化
      3)构造方法体的代码

### 8.卸载类型

    使用启动类装载器装载的类型永远是可触及的,所以永远不会被卸载.只有使用用户自定义的类装载器的类型才会变成不可触及的,从而被虚拟机回收.

## 连接模型

### 1.初始类装载器&定义类装载器

    在Java术语中,要求某个类装载器(A)去装载一个类型,但是却返回了其他类型装载器(B)装载的类型,这种装载器(A)被称为是那个类型的初始化类装载器,而实际定义那个类型的类装载器(B)被称为该类型的定义类装载器.任何被要求装载类型,并且能够返回Class实例的引用代表这个类型的类装载器,都是这个类型的初始类装载器.

### 2.字符串拘留

    在Java程序中,可以调用String类的intern()方法来拘留一个字符串.如果具有相同序列的Unicode字符串已经被拘留过,intern()方法返回一个指向相符的已经被拘留的字符串对象的应用.如果字符串对象的intern()方法被调用(该字符串对象包含的字符串序列还没有被拘留),那么这个对象本身就被拘留.

{% highlight java %}
class Example1 {

    // Assume this application is invoked with one command-line
    // argument, the string "Hi!".
    public static void main(String[] args) {

        // argZero, because it is assigned a String from the command
        // line, does not reference a string literal. This string
        // is not interned.
        String argZero = args[0];

        // literalString, however, does reference a string literal.
        // It will be assigned a reference to a String with the value
        // "Hi!" by an instruction that references a
        // CONSTANT_String_info entry in the constant pool. The
        // "Hi!" string will be interned by this process.
        String literalString = "Hi!";

        // At this point, there are two String objects on the heap
        // that have the value "Hi!". The one from arg[0], which
        // isn't interned, and the one from the literal, which
        // is interned.
        System.out.print("Before interning argZero: ");
        if (argZero == literalString) {
            System.out.println("they're the same string object!");
        }
        else {
            System.out.println("they're different string objects.");
        }

        // argZero.intern() returns the reference to the literal
        // string "Hi!" that is already interned. Now both argZero
        // and literalString have the same value. The non-interned
        // version of "Hi!" is now available for garbage collection.
        argZero = argZero.intern();
        System.out.print("After interning argZero: ");
        if (argZero == literalString) {
            System.out.println("they're the same string object!");
        }
        else {
            System.out.println("they're different string objects.");
        }
    }
}/* Output:
Before interning argZero: they're different string objects.
After interning argZero: they're the same string object!
*///:~
{% endhighlight %}

### 3.接口方法和实例方法

    不管何时Java虚拟机从接口引用调用一个方法,它必须搜索对象的类的方法表来找到一个合适的方法(实现某个接口的类并不能保证都是从同一个超类继承,因此接口中声明的方法并不能保证处于方法表的同一个位置).这种调用接口引用的实例方法的途径会比在类引用上调用实例方法慢很多.

## 垃圾收集

### 1.对象状态

    在垃圾收集器看来,堆中每一个对象都有三种状态之一:
    1)可触及的
    2)可复活的
    3)不可触及的
    在1.2版本中,对原来的三个状态扩充了三个新状态:
    1)软可触及
    2)弱可触及
    3)影子可触及
    "可触及"状态被称作"强可触及",比如从根节点开始的任何直接引用,任何从强可触及对象的实例变量引用的对象也是强可触及的.

### 2.软引用

    软引用使你可以创建内存中的缓存,它与程序的整体内存需求有关.如果内存变得紧张,垃圾收集器会决定清除软引用,回收被软引用的数据所占用的空间.

### 3.弱引用

    弱引用使你可以创建规范映射,比如哈希表,它的关键字和值在没有其他程序部分的引用时可以从映射中清除.

### 4.影子引用

    影子引用使你可以实现除终结方法以外的更加复杂的临终清理政策

## 类型转换

### 1.int转byte

    从int类型值转换到byte类型值的时候,第7位的值将会被拷贝到第8位到第31位.

## 整数运算

### 1.整数运算溢出

    Java虚拟机中出现的整数运算的溢出并不会导致抛出异常,其结果只被简单地截短以符合数据类型.

