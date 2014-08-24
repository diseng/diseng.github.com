---
layout: post
title : 使用Groovy时有哪些必要工具吗
category : software
tags : [software]
---
{% include JB/setup %}

Groovy是什么?我们可以看一下[官方](http://groovy.codehaus.org/Chinese+Home)的介绍:

* 是一个基于Java虚拟机的敏捷动态语言。
* 构建在强大的Java语言之上并添加了从Python，Ruby和Smalltalk等语言中学到的诸多特征。
* 为Java开发者提供了现代最流行的编程语言特性，而且学习成本很低（几乎为零）。
* 支持DSL（Domain Specific Languages领域定义语言）和其它简洁的语法，让你的代码变得易于阅读和维护。
* Goovy拥有处理原生类型，面向对象以及一个Ant DSL，使得创建Shell Scripts变的非常简单。
* 在开发Web，GUI，数据库或控制台程序时 通过减少框架性代码大大提高了开发者的效率。
* 支持单元测试和模拟（对象），可以简化测试。
* 无缝集成所有已经存在的Java对象和类库。
* 直接编译成Java字节码，这样可以在任何使用Java的地方使用Groovy。

在Quora上有一个用户问 [使用Groovy时有哪些必要的工具吗](http://www.quora.com/Groovy-programming-language/What-are-some-essential-developer-tools-to-use-in-groovy)?

Groovy项目管理者和[Gaelyk](http://gaelyk.appspot.com/)创建者 Guillaume Laforge 在回答中说到了几点，我简单整理如下：

**1. IDE**

如果你想要开发复杂和大型的应用，那么一个好用的IDE是需要的。目前来说，Eclipse(最好有Groovy/Grails工具套件)和Intellij IDEA都是很好的选择。NetBeans也在持续不断的更新对Groovy的支持，你可以关注它的进展。

**2. 自动化建构工具**

为了构建你的应用，你需要一个自动化建构工具。在自动构建、测试、打包、发行等方面，[Gradle](http://www.gradle.org/)都是你最好的选择。

**3.  代码静态分析工具**

为了确保代码的质量，跟进各种有用的标准，建议关注一下[CodeNarc](http://codenarc.sourceforge.net/)项目。这个项目提供了Checkstyle/PMD/Findbugs等功能，尤其适用于Groovy代码。

**4. 测试工具**

在测试方面，[Spock](http://docs.spockframework.org/en/latest/)是一个很棒的测试框架，它基于JUnit(因此Spock可以和你的构建无缝集成，在IDE中工作也很稳定)。Spock允许你使用BDD(行为驱动开发)风格来编写你的测试代码，并提供了强大的assertions，mocking等功能。

**5. Groovy环境管理工具**

[GVM](http://gvmtool.net/)也是一个不错的工具，你可以关注一下，它是一个命令行工具，提供了简单易用的Groovy、Grails、Gradle等开发工具的安装、卸载、版本切换的功能。

**6. 有趣的框架或者类库**

除了上述这些开发工具外，还有很多有趣的框架和类库。例如[Grails](http://grails.org/)，一个非常成熟的web开发框架；[Griffon](http://griffon.codehaus.org/)，桌面应用程序开发框架；[GPars](http://gpars.codehaus.org/)，提供给你直观和安全的方式来处理并发任务。
