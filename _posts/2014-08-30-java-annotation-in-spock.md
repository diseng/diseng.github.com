---
layout: post
title : Java Annotation入门及Spock Anntation编写 
category : program
---
{% include JB/setup %}

## 1. 前言

最近接手一位同学的接口自动化测试代码，先是花了几天时间对他的代码进行了——可以说是大刀阔斧的重构，然后按照那位同学原先的excel数据驱动的方式写了几个接口的测试用例，渐渐的感觉不太喜欢在代码和excel文件之间来回切换，于是后面改用一个用例一个测试方法的方式来写。可是，问题又出来了，其实一个接口的测试用例代码，只是入参和返参校验不同而已，其他都是一样的，而一个用例一个方法，其实就是在做无用的copy改而已，先前的excel数据驱动则很好的解决了这个问题，但是鄙人不喜欢excel啊！

那么有没有即不使用excel，也不用重复的将代码进行copy改的解决方式呢？恩，很自然的就去请教谷歌老师了。从谷歌老师这边得到两个答案，一是用JUnitParams，代码是这个样子的：

```java
@RunWith(JUnitParamsRunner.class)
public class JUnitParamsTest {
    @Test
    @Parameters({"17, false",
            "22, true"})
    public void personIsAdult(int age, boolean valid) throws Exception {
        assertThat(new Person(age).isAdult(), is(valid));
    }
}
```

当然，JUnitParams还提供方法提供参数、类提供参数等，具体的可以看[这里](https://junitparams.googlecode.com/hg/apidocs/junitparams/JUnitParamsRunner.html)，另外一种是使用spock，代码是这个样子的：

```groovy
class Math extends Specification {
    def "maximum of two numbers"() {
        expect:
        Math.max(a, b) == c

        where:
        a | b | c
        1 | 3 | 3
        7 | 4 | 4
        0 | 0 | 0
    }
}
```

恩，首先对spock的代码长这样，很有好感，再加上考虑到入参一多，JUnitParams就显得不那么美观了。于是，我就踏上了spock之路。

## 2. Java Annotation入门

用spock写了几个用例之后，发现数据准备这块不太好，得显示的在方法执行前往数据库插一点内容进去，方法执行之后在把数据从数据库删除。于是我又怀念起原先使用的itest框架提供的ITestDataSet注解了，它能够帮助你在测试方法运行之前向数据库插入或者更新数据，然后在执行之后可以选择删除或保留数据等。于是，很自然的，我想同时使用spock和itest，但是发现使用spock你得继承Specification，使用itest你得继承ITestSpringContextBaseCase，这个好像有点困难啊！那么我又想，其实我只是需要itest的ITestDataSet注解功能而已，我能不能把它移植过来呢！然后我天真的把ITestSpringContextBaseCase的内容复制到了spock的测试类上，试了一下不行。。。报了一堆错，然后我就想，要不我自己来实现一个和ITestDataSet功能一样的注解吧，于是就又去请教谷歌老师了。

谷歌老师给我看了一篇[文章](http://www.journaldev.com/721/java-annotations-tutorial-with-custom-annotation-example-and-parsing-using-reflection)，看了之后我觉得实现一个注解大致是这样的:

首先定义一个注解ShowTime，它的作用是在方法运行之前打印一次时间，在方法运行之后，再打印一次时间，并打印出两次时间的间隔，也就是这个方法运行所消耗的时间。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ShowTime {
}
```

恩，也许你已经发现了，定义注解和定义接口十分相似，只是在interface前加了一个@而已，然后其中的Target指的是注解的作用目标，可以是方法、类、变量、包、参数等，其中的Retention指的是保留时间，比如编译时、运行时、等，上面这个注解就是用于方法，并且在运行时起作用。

接着，我们写一个类来使用ShowTime注解

```java
public class ShowTimeExample {
    @ShowTime
    public void sleep(){
        try {
            System.out.println("I will sleep 10 seconds");
            Thread.sleep(10*1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

然后，需要写一个入口，来执行上面的方法，并让ShowTime注解生效。

```java
public class ShowTimeExecutor {
    public static void main(String[] args) throws InvocationTargetException, IllegalAccessException {
        Method[] methods = ShowTimeExample.class.getMethods();
        for (Method method : methods) {
            if (method.isAnnotationPresent(ShowTime.class)) {
                long startTime = System.currentTimeMillis();
                System.out.println("method " + method.getName() + " start at " + startTime);
                method.invoke(new ShowTimeExample());
                long endTime = System.currentTimeMillis();
                System.out.println("method " + method.getName() + " finished at " + endTime);
                System.out.println("method " + method.getName() + " took " + (endTime - startTime) + " millis");
            }
        }
    }
}
```

可以看到输出：

    method sleep start at 1409408011257
    I will sleep 10 seconds
    method sleep finished at 1409408021258
    method sleep took 10001 millis
    
当然，上面的实现其实是用问题的，并不是严格意义上的动态代理，但是我的目的是了解annotation，这里就不刻意关注这个了。从上面的内容，再结合spring来思考，因为web容器启动的时候，会初始化spring，而那些由spring管理的类，其类、方法、变量等含有的annotation都可以被spring获知，因此就可以执行annotation对应的作用。

好了，下面的任务就是找到spock运行的入口了。

## 3. Spock Anntation编写

在请教了谷歌老师之后，发现原来spock已经给用户预留了annotation的扩展入口。

```java
public interface IAnnotationDrivenExtension<T extends Annotation> {
  //访问类注解，当类含有spock注解时会调用这个方法
  void visitSpecAnnotation(T annotation, SpecInfo spec);
  
  //访问方法注解，当方法含有spock注解时会调用这个方法
  void visitFeatureAnnotation(T annotation, FeatureInfo feature);
  
  //访问固定方法(setup,cleanup等)注解，当固定方法有spock注解时会调用这个方法
  void visitFixtureAnnotation(T annotation, MethodInfo fixtureMethod);
  
  //访问变量注解，当变量有spock注解时会调用这个方法
  void visitFieldAnnotation(T annotation, FieldInfo field);
  
  //方法类，在上面这些方法之后执行这个方法
  void visitSpec(SpecInfo spec);
}
```

我只需要实现上面的IAnnotationDrivenExtension，然后用在自己定义的注解上即可。

spock不仅为用户预留了annotation扩展入口，还定义了一个监听器和拦截器，因此可以在上面5个方法中，加入自己的监听器和拦截器。

spock定义的监听器是这样的：

```java
public interface IRunListener {
  //在类运行之前调用
  void beforeSpec(SpecInfo spec);

  //在方法运行之前调用
  void beforeFeature(FeatureInfo feature);

  //在迭代方法运行之前调用
  void beforeIteration(IterationInfo iteration);

  //在迭代方法运行之后调用
  void afterIteration(IterationInfo iteration);

  //在方法运行之后调用
  void afterFeature(FeatureInfo feature);

  //在类运行之后调用
  void afterSpec(SpecInfo spec);

  //在发生错误时调用
  void error(ErrorInfo error);

  //在类标记了@Ignore时调用
  void specSkipped(SpecInfo spec);

  //在方法标记了@Ignore时调用
  void featureSkipped(FeatureInfo feature);
}
```

spock定义的拦截器是这样的：

```java
public interface IMethodInterceptor {
  void intercept(IMethodInvocation invocation) throws Throwable;
}
```

只需要在实现拦截器时往intercept方法中加入自己的逻辑即可。

在了解了spock的这些特性后，就可以动手写一个自己的annotation，它的作用是在方法或者类运行之前往数据库插入一些数据，在方法或者类运行后将数据从数据库删除(恩，是的，就是ITestDataSet那货)。

首先定义一个annotation：

```java
@Retention(RetentionPolicy.RUNTIME)
@Target([ElementType.TYPE, ElementType.METHOD])
@ExtensionAnnotation(TestDataExtension)
public @interface TestData {
    String value()//用户指定数据文件路径
}
```

用TestDataExtension来实现IAnnotationDrivenExtension：

```groovy
class TestDataExtension extends AbstractAnnotationDrivenExtension<TestData> {

    def listener = new TestDataListener()

    @Override
    void visitSpecAnnotation(TestData annotation, SpecInfo spec) {
        //将类信息和注解指定的数据文件路径传给监听器
        listener.specInfos.put(spec,annotation.value())
    }

    @Override
    void visitFeatureAnnotation(TestData annotation, FeatureInfo feature) {
        //将方法信息和注解指定的数据文件路径传给监听器
        listener.featureInfos.put(feature,annotation.value())
    }

    @Override
    void visitSpec(SpecInfo spec) {
        //为类绑定上监听器
        spec.addListener(listener)
    }
}
```

把具体的操作放在监听器TestDataListener里：

```groovy
class TestDataListener extends AbstractRunListener {
    def specInfos = new HashMap<SpecInfo,String>()
    def featureInfos = new HashMap<FeatureInfo,String>()

    @Override
    void beforeSpec(SpecInfo spec) {
        if(specInfos.containsKey(spec)){
            println("start to insert data use ${specInfos.get(spec)} before Spec $spec.name")
        }
    }

    @Override
    void beforeFeature(FeatureInfo feature) {
        if(featureInfos.containsKey(feature)){
            println("start to insert data use ${featureInfos.get(feature)} before feature $feature.name")
        }
    }

    @Override
    void afterFeature(FeatureInfo feature) {
        if(featureInfos.containsKey(feature)){
            println("start to delete data use ${featureInfos.get(feature)} before feature $feature.name")
        }
    }

    @Override
    void afterSpec(SpecInfo spec) {
        if(specInfos.containsKey(spec)){
            println("start to insert data use ${specInfos.get(spec)} before Spec $spec.name")
        }
    }
}
```

下面写一个测试类：

```groovy
@TestData("directory1/file1")
class TestDataSpec extends Specification{

    @TestData("directory2/file2")
    def "testData"(){
        expect:
        println("something expected")
    }
}
```

根据注解TestData的作用，我希望在类运行之前/后，往数据库中插入/删除directory1/file1中的内容，在方法testData运行之前/后往数据库中插入/删除directory2/file2中的内容(这里使用打印语句代替了数据库操作)。实际运行结果：

    start to insert data use directory1/file1 before Spec TestDataSpec
    start to insert data use directory2/file2 before feature testData
    something expected
    start to delete data use directory2/file2 before feature testData
    start to insert data use directory1/file1 before Spec TestDataSpec
    
这与我期望的结果是一样的，说明TestData注解工作正常，确实达到了我想要的效果。

## 4. 后记

其实，纵观上述内容，我就是在重复造轮子，但是我觉得重要的不是造不造轮子，而是有没有学到东西，有没有对自己的工作有所帮助。我觉得了解了这个之后，感觉，哦，原来annotation也不是一个非常复杂深奥的东西(当然，这得归功了spock为用户预留了注解扩展入口)，那么以后我有什么新的需求，就明白是否可以通过annotation来实现以及怎么实现。

ps：通过上面的一系列事情，也勾起了我对Junit实现原理的好奇，日后要好好学习学习。

pps：ITestDataSet使用的数据格式是excel或者xml，我个人对这两种文件格式都不怎么有好感，因此这次我自己实现的数据库操作使用的文件格式是json。

ppps：对于groovy这种类似动态语言的JVM语言，确实感觉使用起来很灵活，但是又因为是动态的，有些时候处理Map，List等，我期望put的是一个类，结果put进去的是一个String，然后就导致我非常别扭的混用java语法和groovy语法。。。不过我觉得是我没有处理好的缘故，而不是groovy语法的问题，毕竟groovy用的还不熟











