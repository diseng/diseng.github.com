---
layout: post
title : 《Thinking in Java》再读笔记
category : read
tags : [read]
---
{% include JB/setup %}

《Thinking in Java》这本书大一的时候就买了,还是做银杏黄项目时报的帐,当时还买了本Android的,好像现在在相垚那,还有本《重构-改善既有代码》,一直在涂少那里,这个学期拿过来,不过还没看.

《Thinking in Java》虽然买的很早,但是没怎么认真看过,都是零零碎碎的看了一点.这学期开始要找实习了,看了下网上流传的笔试面试题,发现基础还是太差,于是又拿起这块砖头,快速的看了一下不是很熟悉的部分.

concurrent这部分以前从来没看过,所以这次重点看了这一部分,对原子类,显式锁,免锁容器,ReadWriteLock等一些内容有了了解和认识.下面这是摘自网上的对concurrent的一个一句话介绍:

  java.util.concurrent 包含许多线程安全、测试良好、高性能的并发构建块。不客气地说，创建 java.util.concurrent 的目的就是要实现 Collection 框架对数据结构所执行的并发操作。通过提供一组可靠的、高性能并发构建块，开发人员可以提高并发类的线程安全、可伸缩性、性能、可读性和可靠性。

下面是这次重读时的一些摘录.

### 1.标签

	在java中，标签起作用的唯一的地方刚好是在迭代语句之前。“刚好之前”的意思表明，在标签和迭代之间置入任何语句都不好。在迭代之前设置标签的唯一理由：我们希望在其中嵌套另一个迭代或者一个开关。这是由于break和continue关键词通常只中断当前循环，但若随同标签一起使用，它们就会中断循环，直到标签所在的地方。

{% highlight java %}
public class LabeledFor {
	public static void main(String[] args) {
		int i = 0;
		outer: // Can't have statements here
		for (; true;) { // infinite loop
			inner: // Can't have statements here
			for (; i < 10; i++) {
				System.out.println("i = " + i);
				if (i == 2) {
					System.out.println("continue");
					continue;
				}
				if (i == 3) {
					System.out.println("break");
					i++; // Otherwise i never
							// gets incremented.
					break;
				}
				if (i == 7) {
					System.out.println("continue outer");
					i++; // Otherwise i never
							// gets incremented.
					continue outer;
				}
				if (i == 8) {
					System.out.println("break outer");
					break outer;
				}
				for (int k = 0; k < 5; k++) {
					if (k == 3) {
						System.out.println("continue inner");
						continue inner;
					}
				}
			}
		}
		// Can't break or continue to labels here
	}
}  /*
 * Output: i = 0 continue inner i = 1 continue inner i = 2 continue i = 3 break
 * i = 4 continue inner i = 5 continue inner i = 6 continue inner i = 7 continue
 * outer i = 8 break outer
 *///:~
 {% endhighlight %}

### 2.后期绑定

	后期绑定，它意味着绑定在运行期间进行，以对象的类型为基础。后期绑定也叫作“动态绑定”或“运行期绑定”。若一种语言实现了后期绑定，同时必须提供一些机制，可在运行期间判断对象的类型，并分别调用适当的方法。也就是说，编译器此时依然不知道对象的类型，但方法调用机制能自己去调查，找到正确的方法主体。不同的语言对后期绑定的实现方法是有所区别的。但我们至少可以这样认为：它们都要在对象中安插某些特殊类型的信息。
	Java中除了static方法和final方法(private方法属于final方法)之外,其他所有的方法都是后期绑定。这意味着我们通常不必决定是否应进行后期绑定——它是自动发生的。

### 3.缺陷:"覆盖"私有方法

{% highlight java %}
public class PrivateOverride {
  private void f() { System.out.println("private f()"); }
  public static void main(String[] args) {
    PrivateOverride po = new Derived();
    po.f();
  }
}

class Derived extends PrivateOverride {
  public void f() { System.out.println("public f()"); }
} /* Output:
private f()
*///:~
{% endhighlight %}

	我们所期望的输出是public f()，但是由于private方法被自动认为是final方法，而且对导出类是屏蔽的。因此，在这种情况下，Derived类中的f()方法就是一个全新的方法；既然基类中的f()方法在子类Derived中不可见，因此甚至也不能被重载。 
	结论就是：只有非private方法才可以被覆盖；但是还需要密切注意覆盖private方法的现象，这时虽然编译器不会报错，但是也不会按照我们所期望的来执行。确切地说，在导出类中，对于基类中的private方法，最好采用不同的名字。

### 4.缺陷:域与静态方法

	一旦您了解了多态机制，可能就会开始认为所有事物都可以多态地发生。然而，只有普通的方法调用可以是多态的。例如，如果您直接访问某个域，这个访问就将在编译期进行解析，就像下面的示例所演示的：

{% highlight java %}
class Super {
  public int field = 0;
  public int getField() { return field; }
}

class Sub extends Super {
  public int field = 1;
  public int getField() { return field; }
  public int getSuperField() { return super.field; }
}

public class FieldAccess {
  public static void main(String[] args) {
    Super sup = new Sub(); // Upcast
    System.out.println("sup.field = " + sup.field +
      ", sup.getField() = " + sup.getField());
    Sub sub = new Sub();
    System.out.println("sub.field = " +
      sub.field + ", sub.getField() = " +
      sub.getField() +
      ", sub.getSuperField() = " +
      sub.getSuperField());
  }
} /* Output:
sup.field = 0, sup.getField() = 1
sub.field = 1, sub.getField() = 1, sub.getSuperField() = 0
*///:~
{% endhighlight %}

	当Sub对象转型为Super引用时，任何域访问操作都将由编译器解析，因此不是多态的。在本例中，为Super.field和Sub.field分配了不同的存储空间。这样，Sub实际上包含两个称为field的域：它自己的和它从Super处得到的。然而，在引用Sub中的field时所产生的默认域并非Super版本的field域。因此，为了得到Super.field，必须显式地指明super.field。
	尽管这看起来好像会成为一个容易令人混淆的问题，但是在实践中，它实际上从来不会发生。首先，您通常会将所有的域都设置成private，因此不能直接访问它们，其副作用是只能调用方法来访问。另外，您可能不会对基类中的域和导出类中的域赋予相同的名字，因为这种做法容易令人混淆。

### 5.构造器与多态

	通常，构造器不同于其他种类的方法。涉及到多态时仍是如此。尽管构造器并不具有多态性(它实际上是static方法，只不过该static声明是隐式的)，但还是非常有必要理解构造器怎么通过多态在复杂的层次结构中运作。

构造器的调用顺序 :

	1)调用基类构造器。这个步骤会不断地反复递归下去，首先是构造这种层次结构的根，然后是下一层导出类，等等，直到最底层的导出类。 
	2)按声明顺序调用成员的初始化。 
	3)调用导出类构造器的主体。

### 6.构造器内部的多态方法的行为

{% highlight java %}
class Glyph {
	void draw() {
		System.out.println("Glyph.draw()");
	}

	Glyph() {
		System.out.println("Glyph() before draw()");
		draw();
		System.out.println("Glyph() after draw()");
	}
}

class RoundGlyph extends Glyph {
	private int radius = 1;

	RoundGlyph(int r) {
		radius = r;
		System.out.println("RoundGlyph.RoundGlyph(), radius = " + radius);
	}

	void draw() {
		System.out.println("RoundGlyph.draw(), radius = " + radius);
	}
}

public class PolyConstructors {
	public static void main(String[] args) {
		new RoundGlyph(5);
	}
} /* Output:
Glyph() before draw()
RoundGlyph.draw(), radius = 0
Glyph() after draw()
RoundGlyph.RoundGlyph(), radius = 5
*///:~
{% endhighlight %}

初始化的实际过程：

	1) 在其他任何事物发生之前，将分配给对象的存储空间初始化成二进制的零。 
	2) 如前所述那样调用基类构造器。此时调用被覆盖后的draw()方法，由于1的缘故此时的radius的值是0。 
	3) 按照声明的顺序调用成员的初始化方法。 
	4) 调用导出类的构造器主体。

	就是当调用到父类的构造器的时候，子类的成员变量还没有被初始化，但是方法什么的都已经加载完毕了，所以调用的是子类的方法，但是子类的成员变量没有初始化，所以为0.
	因此，编写构造器时有一条有效的准则：“用尽可能简单的方法使对象进入正常状态；如果可以的画，避免调用其他方法”。在构造器内唯一能够安全调用的那些方法是基类中的final方法。这些方法不能被覆盖，因此也就不会出现上述令人惊讶的问题。你可能无法总是能够遵循这条准则，但是应该朝着它努力。

### 7.向下转型与运行时类型识别(RTTI)

	在某些程序设计语言(如c++)中，我们必须执行一个特殊的操作来获得安全的向下转型。但是在Java语言中，所有转型都会得到检查!所以即使我们只是进行一次普通的加括弧形式的类型转换，在进入运行期时仍然会对其进行检查，以便保证它的确是我们希望的那种类型。如果不是，就会返回一个ClassCastException(类转型异常)。这种在运行期间对类型进行检查的行为称作“运行时类型识别”(RTTI)。下面的例子说明RTTI的行为：

{% highlight java %}
class Useful {
  public void f() {}
  public void g() {}
}

class MoreUseful extends Useful {
  public void f() {}
  public void g() {}
  public void u() {}
  public void v() {}
  public void w() {}
}	

public class RTTI {
  public static void main(String[] args) {
    Useful[] x = {
      new Useful(),
      new MoreUseful()
    };
    x[0].f();
    x[1].g();
    // Compile time: method not found in Useful:
    //! x[1].u();
    ((MoreUseful)x[1]).u(); // Downcast/RTTI
    ((MoreUseful)x[0]).u(); // Exception thrown
  }
} ///:~
{% endhighlight %}

### 8.创建对象

	所有的类都是在对其第一次使用时，动态加载到jvm中的。当程序创建第一个对类的静态成员的引用时，就会加载这个类。这个证明构造器也是静态方法,即使在构造器之前并没有使用static关键字。因为，通过new创建新对象时，也会被当做对类的静态成员的引用。
	类加载器首先检查这个类的Class对象是否已经加载。如果尚未加载，默认的类加载器就会根据类名查找.class文件。一旦类的class对象被载入内存，它就用来创建这个类的所有对象。 

{% highlight java %}
class Candy {
  static { System.out.println("Loading Candy"); }
}

class Gum {
  static { System.out.println("Loading Gum"); }
}

class Cookie {
  static { System.out.println("Loading Cookie"); }
}

public class SweetShop {
  public static void main(String[] args) {	
	  System.out.println("inside main");
    new Candy();
    System.out.println("After creating Candy");
    try {
      Class.forName("Gum");
    } catch(ClassNotFoundException e) {
    	System.out.println("Couldn't find Gum");
    }
    System.out.println("After Class.forName(\"Gum\")");
    new Cookie();
    System.out.println("After creating Cookie");
  }
} /* Output:
inside main
Loading Candy
After creating Candy
Loading Gum
After Class.forName("Gum")
Loading Cookie
After creating Cookie
*///:~
{% endhighlight %}

### 9.Class.newInstance()

	使用newInstance()来创建的类,必须带有默认的构造器.
	利用Java的反射API,可以用任意的构造器来动态的创建类的对象.

### 10.泛化的Class引用

{% highlight java %}
Class<Number> genericNumberClass = int.class;
{% endhighlight %}

这看起来似乎是起作用的,因为Integer继承自Number.但它无法工作,因为Integer Class对象不是Number Class对象的子类.我们可以这样解决:

{% highlight java %}
Class<?> genericNumberClass = int.class;
genericNumberClass = double.class;
Class<? extends Number> genericNumberClass2 = int.class;
genericNumberClass2 = double.class;
genericNumberClass2 = Number.class;
{% endhighlight %}

    在Java SE5中,Class<?>优于平凡的Class,即便它们是等价的，它表示你不是疏忽才这么做的。为了创建一个Class的引用,它被限定为某种类型,或该类型的任何子类型,你需要将通配符与extends关键字相结合,创建一个范围.

### 11.instanceof与Class的等价性

	instanceof保持了类型的概念,它指的是"你是这个类吗,或者你是这个类的派生类吗?",而如果用==比较实际的Class对象,就没有考虑继承——它或者是这个确切的类,或者不是.

### 12.Thread.yield()

	在run()中对静态方法Thread.yield()的调用时对线程调度器（java线程机制的一部分，可以将CPU从一个线程转移给另一个线程）的一种建议，它在声明：我已经执行完生命周期中最重要的部分了，此刻正是切换给其他任务执行一段时间的大好时机。

### 13.三种基本ThreadPool

	Executors.newCachedThreadPool();创建一个可根据需要创建新线程的线程池，但是在以前构造的线程可用时将重用它们。
	Executors.newFixedThreadPool(int nThreads);创建一个可重用固定线程数的线程池，以共享的无界队列方式来运行这些线程。
	Executors.newSingleThreadExecutor();创建一个使用单个 worker 线程的 Executor，以无界队列方式来运行该线程。

### 14.从任务中产生返回值 

	Runnable是执行工作的独立任务，但是它不返回任何值。如果你希望任务在完成时能够返回一个值，那么可以实现Callable接口而不是Runnable接口。在java SE5中引入的Callable是一种具有类型参数的泛型，它的类型参数表示的是从方法call()中返回的值，并且必须使用ExecutorService.submit()方法调用它.

{% highlight java %}
import java.util.concurrent.*;
import java.util.*;

class TaskWithResult implements Callable<String> {
  private int id;
  public TaskWithResult(int id) {
    this.id = id;
  }
  public String call() {
    return "result of TaskWithResult " + id;
  }
}

public class CallableDemo {
  public static void main(String[] args) {
    ExecutorService exec = Executors.newCachedThreadPool();
    ArrayList<Future<String>> results =
      new ArrayList<Future<String>>();
    for(int i = 0; i < 10; i++)
      results.add(exec.submit(new TaskWithResult(i)));
    for(Future<String> fs : results)
      try {
        // get() blocks until completion:
        System.out.println(fs.get());
      } catch(InterruptedException e) {
        System.out.println(e);
        return;
      } catch(ExecutionException e) {
        System.out.println(e);
      } finally {
        exec.shutdown();
      }
  }
} /* Output:
result of TaskWithResult 0
result of TaskWithResult 1
result of TaskWithResult 2
result of TaskWithResult 3
result of TaskWithResult 4
result of TaskWithResult 5
result of TaskWithResult 6
result of TaskWithResult 7
result of TaskWithResult 8
result of TaskWithResult 9
*///:~
{% endhighlight %}

### 15.休眠&优先级

	对sleep()的调用可以抛出InterruptedException异常,并且你可以看到,它在run()中被捕获.因为异常不能跨线程传播会main(),所以你必须在本地处理所有在任务内部产生的异常.

	线程的优先级将该线程的重要性传递给了调度器.尽管CPU处理现有线程集的顺序是不确定的，但是调度器将倾向于让优先级最高的线程先执行.然而，这并不意味着优先级较低的线程将得不到执行，而是优先级较低的线程仅仅是执行的频率较低.

### 16.后台线程

	后台（daemon）线程，是指在程序运行的时候在后台提供一种通用服务的线程，并且这种线程并不属于程序中不可或缺的部分。当所有的非后台线程结束时，程序也就终止了，同时会杀死进程中的所有后台线程。如果是一个后台线程，那么它创建的任何线程将被自动设置成后台线程。

{% highlight java %}
import java.util.concurrent.*;

public class SimpleDaemons implements Runnable {
  public void run() {
    try {
      while(true) {
        TimeUnit.MILLISECONDS.sleep(100);
        System.out.println(Thread.currentThread() + " " + this);
      }
    } catch(InterruptedException e) {
    	System.out.println("sleep() interrupted");
    }
  }
  public static void main(String[] args) throws Exception {
    for(int i = 0; i < 10; i++) {
      Thread daemon = new Thread(new SimpleDaemons());
      daemon.setDaemon(true); // Must call before start()
      daemon.start();
    }
    System.out.println("All daemons started");
    TimeUnit.MILLISECONDS.sleep(175);
  }
} /* Output: (Sample)
All daemons started
Thread[Thread-0,5,main] SimpleDaemons@530daa
Thread[Thread-1,5,main] SimpleDaemons@a62fc3
Thread[Thread-2,5,main] SimpleDaemons@89ae9e
Thread[Thread-3,5,main] SimpleDaemons@1270b73
Thread[Thread-4,5,main] SimpleDaemons@60aeb0
Thread[Thread-5,5,main] SimpleDaemons@16caf43
Thread[Thread-6,5,main] SimpleDaemons@66848c
Thread[Thread-7,5,main] SimpleDaemons@8813f2
Thread[Thread-8,5,main] SimpleDaemons@1d58aae
Thread[Thread-9,5,main] SimpleDaemons@83cc67
...
*///:~
{% endhighlight %}

后台进程在不执行finally子句的情况下就会终止其run()方法:

{% highlight java %}
import java.util.concurrent.*;

class ADaemon implements Runnable {
  public void run() {
    try {
      System.out.println("Starting ADaemon");
      TimeUnit.SECONDS.sleep(1);
    } catch(InterruptedException e) {
    	System.out.println("Exiting via InterruptedException");
    } finally {
    	System.out.println("This should always run?");
    }
  }
}

public class DaemonsDontRunFinally {
  public static void main(String[] args) throws Exception {
    Thread t = new Thread(new ADaemon());
    t.setDaemon(true);
    t.start();
  }
} /* Output:
Starting ADaemon
*///:~
{% endhighlight %}

### 17.异常捕获清理中断标志

{% highlight java %}
class Sleeper extends Thread {
  private int duration;
  public Sleeper(String name, int sleepTime) {
    super(name);
    duration = sleepTime;
    start();
  }
  public void run() {
    try {
      sleep(duration);
    } catch(InterruptedException e) {
      System.out.println(getName() + " was interrupted. " +
        "isInterrupted(): " + isInterrupted());
      return;
    }
    System.out.println(getName() + " has awakened");
  }
}

class Joiner extends Thread {
  private Sleeper sleeper;
  public Joiner(String name, Sleeper sleeper) {
    super(name);
    this.sleeper = sleeper;
    start();
  }
  public void run() {
   try {
      sleeper.join();
    } catch(InterruptedException e) {
    	System.out.println("Interrupted");
    }
   System.out.println(getName() + " join completed");
  }
}

public class Joining {
  public static void main(String[] args) {
    Sleeper
      sleepy = new Sleeper("Sleepy", 1500),
      grumpy = new Sleeper("Grumpy", 1500);
    Joiner
      dopey = new Joiner("Dopey", sleepy),
      doc = new Joiner("Doc", grumpy);
    grumpy.interrupt();
  }
} /* Output:
Grumpy was interrupted. isInterrupted(): false
Doc join completed
Sleepy has awakened
Dopey join completed
*///:~
{% endhighlight %}

当另一个线程在该线程上调用interrupt()时将给该线程设定一个标志,表明该线程已经被中断.然而,异常被捕获时将清理这个标志,所以在catch子句中,在异常被捕获的时候这个标志总是为假.

### 18.捕获异常

	Thread.UncaughtExceptionHandler是Java SE5中的新接口，它允许你在每个Thread对象上都附着一个异常处理器。Thread.UncaughtExceptionHandler.uncaughtException()会在线程因未捕获的异常而临近死亡时被调用。

{% highlight java %}
import java.util.concurrent.*;

class ExceptionThread2 implements Runnable {
  public void run() {
    Thread t = Thread.currentThread();
    System.out.println("run() by " + t);
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    throw new RuntimeException();
  }
}

class MyUncaughtExceptionHandler implements
Thread.UncaughtExceptionHandler {
  public void uncaughtException(Thread t, Throwable e) {
    System.out.println("caught " + e);
  }
}

class HandlerThreadFactory implements ThreadFactory {
  public Thread newThread(Runnable r) {
    System.out.println(this + " creating new Thread");
    Thread t = new Thread(r);
    System.out.println("created " + t);
    t.setUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    System.out.println(
      "eh = " + t.getUncaughtExceptionHandler());
    return t;
  }
}

public class CaptureUncaughtException {
  public static void main(String[] args) {
    ExecutorService exec = Executors.newCachedThreadPool(
      new HandlerThreadFactory());
    exec.execute(new ExceptionThread2());
  }
} /* Output: (90% match)
HandlerThreadFactory@de6ced creating new Thread
created Thread[Thread-0,5,main]
eh = MyUncaughtExceptionHandler@1fb8ee3
run() by Thread[Thread-0,5,main]
eh = MyUncaughtExceptionHandler@1fb8ee3
caught java.lang.RuntimeException
*///:~
{% endhighlight %}

如果你知道将要在代码中处处使用相同的异常处理器,那么更简单的方式是在Thread类中设置一个静态域,并将这个处理器设置为默认的未捕获异常处理器

{% highlight java %}
import java.util.concurrent.*;

class TestThread implements Runnable{
	public void run() {
		throw new RuntimeException();
	}
}
public class SettingDefaultHandler {
  public static void main(String[] args) {
    Thread.setDefaultUncaughtExceptionHandler(
      new MyUncaughtExceptionHandler());
    ExecutorService exec = Executors.newCachedThreadPool();
    exec.execute(new TestThread());
  }
} /* Output:
caught java.lang.RuntimeException
*///:~
{% endhighlight %}

### 19.原子性与易变性

	当一个域的值依赖于它之前的值时（例如递增一个计数器），volatile就无法工作了。如果某个域的值受到其他域的值的限制，那么volatile也无法工作。
	使用volatile而不是synchronized的唯一安全的情况是类中只有一个可变的域。

	基本上，如果一个域可能被多个任务同时访问，或者这些任务中至少有一个是写入任务，那么你就应该将这个域设置为volatile的。如果你将一个域定义为volatile，那么它就会告诉编译器不要执行任何移除读取和写入操作的优化，这些操作的目的是用线程中的局部变量维护对这个域的精确同步。实际上，读取和写入都是直接针对内存的，而却没有被缓存。但volatile并不能对递增不是原子性操作这一事实产生影响。

### 20.原子类

	Java SE5引入了诸如AtomicInteger,AtomicLong,AtomicReference等特殊的原子性变量类,这些类被调整为可以使用在某些现代处理器上的可获得的,并且是在机器级别上的原子性,因此在使用它们时,通常不需要担心.对于常规编程来说,它们很少会派上用处,但是在涉及性能调优时,它们就大有用武之地了.

### 21.线程本地存储

	防止任务在共享资源上产生冲突的第二种方式是根除对变量的共享。线程本地存储是一种自动化机制，可以为使用相同变量的每个不同的线程都创建不同的存储。因此如果你有5个线程都要使用变量x所表示的对象，那线程本地存储就会生成5个用于x的不同的存储块。主要是，它们使得你可以将状态与线程关联起来。

创建和管理本地线程由java.lang.ThreadLocal类实现

{% highlight java %}
import java.util.concurrent.*;
import java.util.*;

class Accessor implements Runnable {
  private final int id;
  public Accessor(int idn) { id = idn; }
  public void run() {
    while(!Thread.currentThread().isInterrupted()) {
      ThreadLocalVariableHolder.increment();
      System.out.println(this);
      Thread.yield();
    }
  }
  public String toString() {
    return "#" + id + ": " +
      ThreadLocalVariableHolder.get();
  }
}

public class ThreadLocalVariableHolder {
  private static ThreadLocal<Integer> value =
    new ThreadLocal<Integer>() {
      private Random rand = new Random(47);
      protected synchronized Integer initialValue() {
        return rand.nextInt(10000);
      }
    };
  public static void increment() {
    value.set(value.get() + 1);
  }
  public static int get() { return value.get(); }
  public static void main(String[] args) throws Exception {
    ExecutorService exec = Executors.newCachedThreadPool();
    for(int i = 0; i < 5; i++)
      exec.execute(new Accessor(i));
    TimeUnit.SECONDS.sleep(3);  // Run for a while
    exec.shutdownNow();         // All Accessors will quit
  }
} /* Output: (Sample)
#0: 9259
#1: 556
#2: 6694
#3: 1862
#4: 962
#0: 9260
#1: 557
#2: 6695
#3: 1863
#4: 963
...
*///:~
{% endhighlight %}

	ThreadLocal对象通常当作静态域存储.在创建ThreadLocal时,你只能通过get()和set()方法来访问该对象的内容,其中,get()方法将返回与其线程相关联的对象的副本,而set()会将参数插入到为其线程存储的对象中,并返回存储中原有的对象.

### 22.中断

{% highlight java %}
import java.util.concurrent.*;
import java.io.*;

class SleepBlocked implements Runnable {
  public void run() {
    try {
      TimeUnit.SECONDS.sleep(100);
    } catch(InterruptedException e) {
      System.out.println("InterruptedException");
    }
    System.out.println("Exiting SleepBlocked.run()");
  }
}

class IOBlocked implements Runnable {
  private InputStream in;
  public IOBlocked(InputStream is) { in = is; }
  public void run() {
    try {
    	System.out.println("Waiting for read():");
      in.read();
    } catch(IOException e) {
      if(Thread.currentThread().isInterrupted()) {
    	  System.out.println("Interrupted from blocked I/O");
      } else {
        throw new RuntimeException(e);
      }
    }
    System.out.println("Exiting IOBlocked.run()");
  }
}

class SynchronizedBlocked implements Runnable {
  public synchronized void f() {
    while(true) // Never releases lock
      Thread.yield();
  }
  public SynchronizedBlocked() {
    new Thread() {
      public void run() {
        f(); // Lock acquired by this thread
      }
    }.start();
  }
  public void run() {
	  System.out.println("Trying to call f()");
    f();
    System.out.println("Exiting SynchronizedBlocked.run()");
  }
}

public class Interrupting {
  private static ExecutorService exec =
    Executors.newCachedThreadPool();
  static void test(Runnable r) throws InterruptedException{
    Future<?> f = exec.submit(r);
    TimeUnit.MILLISECONDS.sleep(100);
    System.out.println("Interrupting " + r.getClass().getName());
    f.cancel(true); // Interrupts if running
    System.out.println("Interrupt sent to " + r.getClass().getName());
  }
  public static void main(String[] args) throws Exception {
    test(new SleepBlocked());
    test(new IOBlocked(System.in));
    test(new SynchronizedBlocked());
    TimeUnit.SECONDS.sleep(3);
    System.out.println("Aborting with System.exit(0)");
    System.exit(0); // ... since last 2 interrupts failed
  }
} /* Output: (95% match)
Interrupting SleepBlocked
InterruptedException
Exiting SleepBlocked.run()
Interrupt sent to SleepBlocked
Waiting for read():
Interrupting IOBlocked
Interrupt sent to IOBlocked
Trying to call f()
Interrupting SynchronizedBlocked
Interrupt sent to SynchronizedBlocked
Aborting with System.exit(0)
*///:~
{% endhighlight %}

SleepBlock是可中断的阻塞示例,而IOBlocked和SynchronizedBlocked是不可中断的阻塞示例.

### 23.同一个互斥可以被同一个任务多次获得

{% highlight java %}
public class MultiLock {
  public synchronized void f1(int count) {
    if(count-- > 0) {
      System.out.println("f1() calling f2() with count " + count);
      f2(count);
    }
  }
  public synchronized void f2(int count) {
    if(count-- > 0) {
      System.out.println("f2() calling f1() with count " + count);
      f1(count);
    }
  }
  public static void main(String[] args) throws Exception {
    final MultiLock multiLock = new MultiLock();
    new Thread() {
      public void run() {
        multiLock.f1(10);
      }
    }.start();
  }
} /* Output:
f1() calling f2() with count 9
f2() calling f1() with count 8
f1() calling f2() with count 7
f2() calling f1() with count 6
f1() calling f2() with count 5
f2() calling f1() with count 4
f1() calling f2() with count 3
f2() calling f1() with count 2
f1() calling f2() with count 1
f2() calling f1() with count 0
*///:~
{% endhighlight %}

    在main()中创建了一个调用f1()的Thread,然后f1()和f2()互相调用直至count变为0.由于这个任务已经在第一个对f1()的调用中获得了multiLock对象的锁,因此同一个任务将在对f2()的调用中再次获取这个锁,依此类推.这么做是有意义的,因为一个任务应该能够调用在同一个对象中的其他synchronized方法,而这个任务已经持有锁了.

### 24.wait()与notifyAll()

    调用sleep()的时候锁并没有被释放,调用yield()也属于这种情况,理解这一点很重要.另一方面,当一个任务在方法里遇到了对wait()的调用的时候,线程的执行被挂起,对象上的锁被释放.因为wait()将释放锁,这就意味着另一个任务可以获得这个锁,因此在该对象中的其他synchronized方法可以在wait()期间被调用.这一点至关重要,因为这些其他的方法通常将会发生改变,而这种改变正是使被挂起的任务重新唤醒所感兴趣的变化.因此,当你调用wait()是,就是在声明:"我已经刚刚做完能做的所有事情,因此我要在这里等待,但是我希望其他的synchronized操作在条件适合的情况下能够执行."

    wait(),notify()以及notifyAll()有一个比较特殊的方面,那就是这些方法是基类Object的一部分,而不是属于Thread的一部分.尽管开始看起来有点奇怪——仅仅针对线程的功能却作为通用类的一部分实现,不过这是有道理的,因为这些方法操作的锁也是所有对象的一部分.

### 25.类库的线程安全

    Random.nextInt()是线程安全的,虽然JDKw文档并没有指明.对于Java标准类库中的方法来说,也大都存在:哪些是线程安全的?哪些不是? 
