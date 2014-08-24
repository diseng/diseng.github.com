---
layout: post
title : MySQL优化之hints
category : software
tags : [software]
---
{% include JB/setup %}

大妈让我整理MySQL优化的一些工具和方法，整理到MySQL hints的时候，没有找到满意的中文文章，于是自己翻译了一篇关于MySQL hints的小文章。如下：

每个程序员都喜欢优化，甚至有时我们知道不应该去做。为了满足大家的意愿，MySQL提供了一些关键字，在SQL语句中使用这些关键字，可以使得数据库按照明确的优化指令执行SQL语句。

应当指出的是，不正确的使用hints很有可能使你的查询语句表现的更糟糕，在使用hints之前需要确保这是有意义的。通常你可以使用explain查看执行计划并阅读hint文档来决定是否用它。

使用SQL注释括住hints也是一个好方法，比如

{% highlight sql %}
SELECT /*! SQL_NO_CACHE */ columns FROM table;
{% endhighlight %}

这可以使你的应用得到更大的兼容性。

下面介绍一些常用的MySQL优化hints。

**SQL_NO_CACHE**

SQL_NO_CACHE hint 会使用特殊的查询来关闭MySQL内置的查询缓存机制。在动态性很强或者执行频率很低的查询上使用SQL_NO_CACHE hint，可以帮助MySQL提高缓存的使用效率。不过确保在使用SQL_NO_CACHE hint时，MySQL已经开启了查询缓存，否则没有必要使用。

{% highlight sql %}
SELECT SQL_NO_CACHE field1, field2 FROM TABLE1;
{% endhighlight %}

关于MySQL查询缓存的更多信息可以查看[这里](http://www.petefreitag.com/item/390.cfm)

**SQL_CACHE**

如果你已经配置了query_cache_type = 2(仅在使用SQL_CACHE时进行缓冲)，那么可以使用SQL_CACHE hint来告诉MySQL哪些查询需要进行缓存。

{% highlight sql %}
SELECT SQL_CALHE * FROM TABLE1;
{% endhighlight %}

**HIGH_PRIORITY**

在SELECT 或者 INSERT语句中使用 HIGH_PRIORITY hint可以告诉MySQL这是一个高优先级的查询。这个hint允许此查询跳过优先级较低的语句而得到优先执行。

{% highlight sql %}
SELECT HIGH_PRIORITY * FROM TABLE1;
{% endhighlight %}

**LOW_PRIORITY**

LOW_PRIORITY hint 可以使用在INSERT 和 UPDATE语句中.如果你使用LOW_PRIORITY 关键字，语句的执行会被延迟到没有其他的读请求时才执行。这意味着你可能等待很长一段时间，甚至在读请求为主的服务器上发生永久等待。

{% highlight sql %}
update LOW_PRIORITY table1 set field1=foo where field1=bar;
{% endhighlight %}

**INSERT DELAYED**

一个INSERT LOW_PRIORITY 语句在其得到真正执行之前，不会做任何返回，如果你想立即获得返回值，可以使用INSERT DELAYED 语句。INSERT DELAYED 语句会立即返回，但是依然等到其他客户端关闭了数据表后才执行。

{% highlight sql %}
INSERT DELAYED INTO table1 values(xxx);
{% endhighlight %}

关于MySQL Insert Delayed的更多信息可以查看[这里](http://www.petefreitag.com/item/430.cfm)

注意：INSERT DELAYED 只作用于MyISAM、MEMORY和ARCHIVE表。

**STRAIGHT_JOIN**

STRAIGHT_JOIN hint告诉MySQL使用FROM从句的顺序来进行表连接。

使用EXPLAIN 来明确MySQL没有指出这个最佳的连接顺序。如果你指定了一个不佳的顺序，可能会使MySQL做很多无需做的工作。

{% highlight sql %}
SELECT TABLE1.FIELD1, TABLE2.FIELD2 FROM TABLE1 STRAIGHT_JOIN TABLE2 WHERE xxx;
{% endhighlight %}

**SQL_BUFFER_RESULT**

SQL_BUFFER_RESULT hint告诉MySQL将查询的结果放入到临时表中，这样当查询结果还在输出时,就可以解除锁定的数据表。因此你可能只需要在结果集很大的时候使用SQL_BUFFER_RESULT hint。

{% highlight sql %}
SELECT SQL_BUFFER_RESULT * FROM TABLE1 WHERE;
{% endhighlight %}

**SQL_BIG_RESULT**

SQL_BIG_RESULT hint 能够与DISTINCT 和 GROUP BY SELECT 语句一起使用，它会告诉MySQL结果集可能会很大。根据MySQL文档，如果使用了这个hint，那么在需要时，MySQL会直接使用磁盘临时表，并且使用GROUP BY的元素关键字对临时表进行排序。

{% highlight sql %}
SELECT SQL_BIG_RESULT * FROM TABLE1 WHERE;
{% endhighlight %}

**SQL_SMALL_RESULT**

SQL_SMALL_RESULT基本上与SQL_BIG_RESULT相反。当使用了SQL_SMALL_RESULT时，MySQL使用内存临时表来存储结果而不会进行非索引排序。因此通常这是默认的优化选项，这个hint通常无需使用。

{% highlight sql %}
SELECT SQL_SMALL_RESULT * FROM TABLE1 WHERE;
{% endhighlight %}

**FORCE INDEX**

{% highlight sql %}
SELECT * FROM TABLE1 FORCE INDEX (FIELD1);
{% endhighlight %}

以上的SQL语句只使用建立在FIELD1上的索引，而不使用其它字段上的索引。

**IGNORE INDEX**

{% highlight sql %}
SELECT * FROM TABLE1 IGNORE INDEX (FIELD1, FIELD2);
{% endhighlight %}

在上面的SQL语句中，TABLE1表中FIELD1和FIELD2上的索引不被使用。


英文原文在[这里](http://www.petefreitag.com/item/613.cfm)