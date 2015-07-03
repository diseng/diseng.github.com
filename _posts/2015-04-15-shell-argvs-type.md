---
layout: post
title : shell脚本的几种传参方式
category : program
tags : [program]
---
{% include JB/setup %}

### 0. 序

由于应用的特殊性以及七网隔离, 某些功能只能在预发或者线上机器执行JMX来进行测试, JMX方法的入参是一个String,传入的值为json格式的文件路径.

这样使用了一段时间之后,发现当需要频繁修改json内容时,就显得太繁琐了.我需要vim打开文件,修改json串,然后保存退出,再使用curl来调用JMX.

于是打算写个支持长短选项的shell脚本来提高一下效率

### 1. shell脚本的传参方式

我们写shell脚本,一般的传参方式有3种

* 手工处理方式:example.sh argv1 argv2
* getopts(不支持长选项):example.sh -a  -b value2 
* getopt(支持长选项):example.sh -a -b value2 --argv3 value3 

下面分别对这3种方式进行一个使用介绍

### 2. 手工处理方式

这种方式,一般也是简单脚本里面最常用的,例如

    example.sh argv1 argv2 argv3

一般取这种方式的入参,有下面几个点需要注意一下

* $0:指命令本身,如上面的example.sh
* $1:第一个入参,后续入参数字依次递增
* $#:入参总的个数,不包括命令本身
* $@:入参列表,不包括命令本身
* $* :和$@相同，但"$*" 和 "$@"(加引号)并不同,"$*"将所有的参数解释成一个字符串,而"$@"是一个参数数组

这种方式的入参,所有的入参解析工作都交由脚本编写人员来处理,可以起到灵活多变的高度,但是处理起来比较累,所以这种方式一般用在比较简单的脚本上.

### 3. getopts

处理命令行参数是一个相似而又复杂的事情,而在shell中,我们可以使用getopts和getopt来简化这件事.

getopts是内置在bash中的,它不支持长选项,getopt是独立的可执行文件,它支持长选项.

下面介绍一下getopts的用法.

    example.sh -a -b value2

比如上面的命令,我们指定了两个短选项,分别是-a和-b,其中-a是不带参数值的,而-b是带参数值的,那么如何定义短选项是否需要参数呢

向上面这样的脚本,我们一般是这么定义入参的:

    getopts "ab:"

我们看到b后面跟了一个冒号,而a没有,是的,shell里面用来区分短选项是否需要参数,就是通过冒号来进行的

* 后面没有冒号,表示选项不接参数
* 后面有一个冒号,表示选项必须接参数

下面是一个完整的示例脚本:

{% highlight shell %}
#!/bin/bash

while getopts "ab:" arg #选项后面的冒号表示该选项需要参数
do
        case $arg in
             a)
                echo "i am a"
                ;;
             b)
                echo "i am b, my value is $OPTARG" #参数存在$OPTARG中
                ;;
        esac
done
{% endhighlight %}

### 4. getopt

绝大多数脚本使用getopts应该就可以满足需求了，如果需要支持长选项以及可选参数，那么就需要使用getopt了.

getopt和getopts类似,也是通过冒号来区分选项是否接受参数值,其定义如下:

* 后面没有冒号,表示选项不接参数
* 后面有一个冒号,表示选项必须接参数
* 后面有两个冒号,表示选项参数可选

我们来看一下这个命令

    example.sh -a -b value2 --argv3 value3

我们看到这个命令有两个短选项和一个长选项,分别是-a,-b和--argv3,其中a不接收参数,b和argv3接收参数,我们看一下如何在脚本里面进行定义

    ARGS=`getopt -o ab: -l "argv3:,help"  -- "$@"`
    eval set -- "${ARGS}"

因为getopt是一个独立的程序,所以我们使用``来进行getopt的执行和结果获取,并通过eval set将规范化后的命令行参数分配至位置参数（$1,$2,...)

其中-o或--options选项后面接可接受的短选项;-l或--long选项后面接可接受的长选项,用逗号分开

一个完整的示例脚本如下:

{% highlight shell %}
#!/bin/bash

ARGS=`getopt -o ab: -l "argv3:,help" -- "$@"`
eval set -- "${ARGS}"

while true;
do
    case "$1" in
        -a) 
            echo "i am a"
            shift
            ;;
        -b) 
            echo "i am b, my value is $2" 
            shift 2
            ;;
        --argv3)
            echo "i am argv3, my value is $2"
            shift 2
            ;;
        --help)
            echo "i am help info"
            exit 0
            ;;
        --)
            shift
            break
            ;;
        *)
            echo "Internal error!"
            exit 1
            ;;
    esac
done
{% endhighlight %}