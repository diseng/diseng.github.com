---
layout: post
title : 微软笔试小结
category : work
---
{% include JB/setup %}

先上笔试题,后小小总结一下.

### NO.1

    Which of the following calling convention(s) support(s) supportvariable-length parameter(e.g. printf)?(3 Points)

    A. cdecl    

    B. stdcall    

    C. pascal    

    D. fastcall

第一题就难住了,题目都没读太懂,当时理解为支持参数个数可变的函数,看了下4个选项,就知道stdcall和fastcall两个(上个学期软开实验的时候出现过),但是只是单纯的知道名字而已....又看到e.g. printf这句,觉得printf这种应该算stdcall吧,然后就选了个B.

回来后查了下,答案应该是A.

**\_cdecl**

  按从右至左的顺序压参数入栈，由调用者把参数弹出栈。对于“C”函数或者变量，修饰名是在函数名前加下划线。对于“C++”函数，有所不同。
  如函数void test(void)的修饰名是_test；对于不属于一个类的“C++”全局函数，修饰名是?test@@ZAXXZ。
  这是缺省调用约定。由于是调用者负责把参数弹出栈，所以可以给函数定义个数不定的参数，如printf函数。

**\_stdcall**

  按从右至左的顺序压参数入栈，由被调用者把参数弹出栈。对于“C”函数或者变量，修饰名以下划线为前缀，然后是函数名，然后是符号“@”及参数的字节数，如函数int func(int a, double b)的修饰名是_func@12。对于“C++”函数，则有所不同。     
  所有的Win32 API函数都遵循该约定。

**\_pascal**

  头两个DWORD类型或者占更少字节的参数被放入ECX和EDX寄存器，其他剩下的参数按从左到右的顺序压入栈。由被调用者把参数弹出栈，对于“C”函数或者变量，修饰名以“@”为前缀，然后是函数名，接着是符号“@”及参数的字节数，如函数int func(int a, double b)的修饰名是@func@12。对于“C++”函数，有所不同。

**\_fastcall**

  fastcall约定用于对性能要求非常高的场合。fastcall约定将函数的从左边开始的两个大小不大于4个字节（DWORD）的参数分别放在ECX和EDX寄存器，其余的参数仍旧自右向左压栈传送，被调用的函数在返回前清理传送参数的堆栈。

### NO.2

    What's the output of the following code?(3 Points)

    A. B::f()B::f()const    

    B. B::f()A::f()const   

    C. A::f()B::f()const    

    D. A::f()A::f()const

```c
class A
{
  public:
    virtual void f()
    {
        cout<<"A::f()"<<endl;
    }
    void f() const
    {
        cout<<"A::f() const"<<endl;
    }
};

class B: public A
{
  public:
    void f()
    {
        cout<<"B::f()"<<endl;
    }
    void f() const
    {
        cout<<"B::f() const"<<endl;
    }
};

void g(const A* a)
{
    a->f();
}

int main()
{
    A* a = new B();
    a->f();
    g(a);
    delete a ;
}
```

虽然没写过C++,不过这道题比较简单,选的B

### NO.3

    What is the difference between a linked list and an array?(3 Points)

    A. Search complexity when both are sorted

    B. Dynamically add/remove

    C. Random access efficiency

    D. Data storage type

链表和数组的区别,我选的ABC,D我没理解清楚,是说数据存储类型还是数据存储方式,所以没敢选D

### NO.4

    About the Thread and Process in Windows, which description(s) is(are) correct:(3 Points)

    A. One application in OS must have one Process, but not a necessary to have one Thread

    B. The Process could have its own Stack but the thread only could share the Stack of its parent Process

    C. Thread must belongs to a Process

    D. Thread could change its belonging Process

考线程和进程的概念,这学期刚学操作系统.选的C

### NO.5

    What is the output of the following code?(3 Points)

    A. 10, 10

    B. 10, 11

    C. 11, 10

    D. 11, 11

```c
{
  int x = 10 ;
  int y = 10 ;
  x = x++ ;
  y = ++y ;
  printf("%d, %d\n",x,y);
}
```

选的D,但是不确定,回来试了下,gcc下输出的是11,11,我用Java试了下,输出的是10,11,可能是Java中间缓存变量的原因吧,所以还不清楚正确答案应该是什么

### NO.6

    For the following Java or C# code,What will myArray3[2][2] returns?(3 Points)

    A. 9

    B. 2

    C. 6

    D. overflow

```java
  int [][] myArray3 =
  new int[3][]{
  new int[3]{5,6,2},
  new int[5]{6,9,7,8,3},
  new int[2]{3,2}};
```

我选的D,回来一试,Eclipse提示错误Cannot define dimension expressions when an array initializer is provided.然后看了下Java中数组声明的规范,发现很多细节,以前从来没注意过.如下

```java
//如要一个{1,2,3}的数组
int[3] array = {1,2,3};//错误
int[] array = {1,2,3};//可以
int[] array = new int[3]{1,2,3};//错误
int[] array = new int[]{1,2,3};//可以
int[] array;array = {1,2,3};//错误
int[3] array;array = new int[]{1,2,3};//错误
int[] array;array = new int[3]{1,2,3};//错误
int[] array;array = new int[]{1,2,3};//可以
```

从上面几个例子很容易看出来,在Java中声明或者初始化一个数组时,不能显式的指定数组大小.当声明和初始化语句分开时,只能通过new语句来初始化.

所以我把题目中的数字全去掉,运行时报ArrayIndexOutOfBoundsException错误.不知道C#下是什么情况.

### NO.7

    Please choose the right statement about const usage:(3 Points)

    A. const int a; //const integer

    B. int const a; //const integer

    C. int const *a; //a pointer which point to const integer

    D. const int *a; //a const pointer which point to integer

    E. int const *a; // a const pointer which point to integer

这道题,主要是考指针常量和常量指针,但是忘记指针常量和常量指针具体的区别了,但是看题目,如果C正确,那D肯定正确了,否则就是E正确,CD错误.后来想了想,算了,保险点选了个A,回来看了下正确答案应该是ABE.另外注意常量指针英文说法是Pointer to Constant,而指针常量是Const Pointer,还有一个是Constant Pointer to a Constant,中文该叫指向常量的指针常量么?或者直接叫常量指针常量吧:)

### NO.8

    Given the following code,What is the correct result?(3 Points)

    A. 11111111

    B. 12121212

    C. 11112222

    D. 21212121

```c
  #include <iostream>

  class A{
  public:
      long a;
  };

  class B : public A
  {
  public:
      long b;
  };

  void seta(A* data, int idx)
  {
      data[idx].a = 2;
  }

  int _tmain(int argc, _TCHAR *argv[])
  {
      B data[4];

      for(int i=0; i<4; ++i)
      {
          data[i].a = 1;
          data[i].b = 1;
          seta(data, i);
      }

      for(int i=0; i<4; ++i)
      {
          std::cout<<data[i].a<<data[i].b;
      }

      return 0;
  }
```

感觉有陷阱,但是觉得还是D比较靠谱,就选了D,回来一试,结果是22221111,真心不懂,求大神解释了.

### NO.9

    1 of 1000 bottles of water is poisoned which will kill a rat in 1 week if the rat drunk any amout of the water. Given the bottles of water have no visual difference, how many rats are needed at least to find the poisoned one in 1 week?(5 Points)

    A. 9

    B. 10

    C. 32

    D. None of the above

这道题,以前看到过类似的,所以很快就选出答案了,没见过类似题目的可能会花点时间.很简单,瓶子编号用二进制表示,让二进制中为1的位置对应的老鼠喝水.比如5号瓶子,二进制表示为101,所以让1号和3号老鼠喝这瓶水即可.一周后,查看死掉的老鼠编号,即可知道哪瓶水是有毒的.2的10次方可达1024,所以应付1000也是没问题的,选B.

### NO.10

    Which of the following statement(s) equal(s) value 1 in C programming language?(5 Points)

    A. the return value of main function if program ends normally

    B. return (7&1)

    C. char *str="microsoft"; return str=="microsoft"

    D. return "microsoft"=="microsoft"

    E. None of the above

B肯定正确,C的话str是个地址,执行str=="microsoft"的时候不知道是str和字符串的首地址比较还是和字符串内容比较,所以没敢选,D的话也是不清楚是比较内容还是字符串首地址,也没选,就选了个B.回来一试,BCD都可以的.其实D应该不管是比较地址还是内容都返回1的.

### NO.11

    If you computed 32 bit signed integers F and G from 32 bit signed X using F = X / 2 and G = (X>>1), and you found F!=G, this implies that(5 Points)

    A. There is a compiler error

    B. X is odd

    C. X is negative

    D. F - G = 1

    E. G - F = 1

首先C肯定是对的,B中的odd不知道是什么意思啊,汗,D不清楚所有可能的情况,所以也没敢选.后来一起的几个人有人选BCD的,odd是奇数的意思,汗.不过我还是不清楚为什么奇数会出现不等的情况,求大神解释了.

### NO.12

    How many rectangles you can find from 3*4 grid?(5 Points)

    A. 18

    B. 20

    C. 40

    D. 60

    E. None of above is correct

这道题,我觉得正方形应该也算矩形的,所以选了E,也有人选D的,还有人理解成三角形的.....

### NO.13

    One line can split a surface to 2 part, 2 line can split a surface to 4 part. Given 100 lines, no two parallel lines, no tree lines join at same point, how many parts can 100 line split?(5 Points)

    A. 5051

    B. 5053

    C. 5510

    D. 5511

这个应该是个递增队列,2+2+3+4+....+100,应该是5051.

### NO.14

    Which of the following sorting algorithm(s) is(are) stable sorting?(5 Points)

    A. bubble sort

    B. quick sort

    C. heap sort

    D. merge sort

    E. Selection sort

这个一看就选了A,D,E,回来网上一查选择排序,其实应该是不稳定的,比如{5,5,2},排完就不稳定了,但是很多书上都是说稳定的,哎,怪自己太信教科书了.

### NO.15

    Model-View-Controller(MVC) is an architectural pattern that frequently used in web applications. Which of the following statement(s) is(are) correct:(5 Points)

    A. Models often represent data and the business logics needed to manipulate the data in the application

    B. A view is a (visual) representation of its model. It renders the model into a form suitable for interaction, typically a user interface element

    C. A controller is the link between a user and the system. It accepts input from the user and instructs the model and a view to perform actions based on that input

    D. The common practice of MVC in web applications is, the model receives GET or POST input from user and decides what to do with it, handing over to controller and which hand control to views(HTML-generating components)

    E. None of the above

说实话,虽然写过Web,上过软件工程,但是碰到这种题,还是招架不住,掂量半天选了个C,不知道对不对.回来看了下维基百科,感觉A是对的,C不确定.求大神解释.

### NO.16

    we can recover the binary tree if given the output of(5 Points)

    A. Preorder traversal and inorder traversal

    B. Preorder traversal and postorder traversal

    C. Inorder traversal and postorder traversal

    D. Postorder traversal

这个学过数据结构的应该都知道,AC

### NO.17

    Given a string with n characters, suppose all the characters are different from each other, how many different substrings do we have?(5 Points)

    A. n+1
      
    B. n^2

    C. n(n+1)/2

    D. 2^n-1

    E. n!

这个随便找几个字符串带进去检验一下就行,如果要推导也行,但是这个选择题,没有必要.

### NO.18

    Given the following database table, how many rows will the following SQL statement update?(5 Points)

    A. 1

    B. 2

    C. 3

    D. 4

    E. 5

<center><img alt="microsoft-written-examination" src="{{ ASSET_PATH }}hooligan/img/post/microsoft-written-examination.jpg"/></center>

这个也很简单,第2和第3行被更新.

### NO.19

    What is the shortest path between node S and node T, given the graph below? Note: the numbers represent the lengths of the connected nodes.(13 Points)

    A. 17

    B. 18

    C. 19

    D. 20

    E. 21

<center><img alt="microsoft-written-examination-1" src="{{ ASSET_PATH }}hooligan/img/post/microsoft-written-examination-1.jpg"/></center>

这个......选D

### NO.20

    Given a set of N balls and one of which is defective (weighs less than others), you are allowed to weigh with a balance 3 times to find the defective. Which of the following are possible N?(13 Points)

    A. 12

    B. 16

    C. 20

    D. 24

    E. 28

这道题也不难,最大值应该是3^TIME,这里TIME是3,所以最大应该是27,然后我就选了个D....我理解成选以下选项中最大值.....可是题目要求是possible.....半对只给7分.....6分就这么飞了....


## 总结

其实题目总体来说,不算难.但是不会的就应该不选,可惜高中做英语,不会就乱猜一个的良好习惯还是改不了啊.还有就是题目一定要看仔细,毕竟英语没中文熟,中间有个翻译过程(至少我目前是这样的),万一翻译错了,就遭殃了,比如最后一题,我就理解成了找选项中的最大值,而实际是找选项中可能的值.

最后,反正是打个酱油而已,增加一点自己的经历嘛,没坏处.

ps:当初我没报微软的实习生,是去年认识的一个朋友说她想报,叫我也试试看,然后我进了笔试,她没进.不过她现在在知乎实习,混的比我好啊:(
