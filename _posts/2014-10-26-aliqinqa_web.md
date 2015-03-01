---
layout: post
title : aliqinqa_web应用开发小记
category : program
tags : [aprogram,java]
---
{% include JB/setup %}

国庆节在家的时候，没啥事做，就动手写了一个简单的web应用，封装了一下CRM测试过程中常用到的一些可以自动化的操作，国庆回来后对新计费系统也越来越熟悉，后续也会把计费系统中一些常用到的可以自动化的操作封装进来。

下面小记一下在编写这个web应用过程中遇到的一些问题


### 1. 多数据源

以前自己写写小应用，都是本地搭个数据库，然后建一个库，程序里面配置一个数据源就行了。但是CRM有十几个库，不同的操作在不同的库里面完成，或者有些操作需要同时操作好几个库。这时就需要配置多个数据源了。

配多个数据源到不难，关键是配了之后怎么用呢？

Spring为用户提供了一个叫作AbstractRoutingDataSource的类，看名字就可以知道是干嘛用的了，该类的部分内容如下：

{% highlight java %}
public abstract class AbstractRoutingDataSource extends AbstractDataSource implements InitializingBean {

    private Map<Object, Object> targetDataSources;

	private Object defaultTargetDataSource;
	
	protected abstract Object determineCurrentLookupKey();
}
{% endhighlight %}

简单使用的话，只需要关注上述3点就行了，其中targetDataSources是一个Map，Value是我们自己配置的数据源Bean，Key的话可以是枚举类，String或其他任意你喜欢的类型，defaultTargetDataSource是默认的数据源Bean，determineCurrentLookupKey则是需要我们自己实现的一个方法，该方法的作用就是判断当前正在使用的数据源Key，这个Key需要在上面的那个Map中存在，不然通过这个Key拿不到对应的数据源Bean，那就。。。

下面是我对该类的一个继承

{% highlight java %}
public class DynamicDataSource extends AbstractRoutingDataSource {

    private static final ThreadLocal<DataSoureType> dataSourceKey = new ThreadLocal<>();

    @Override
    protected Object determineCurrentLookupKey() {
        return dataSourceKey.get();
    }

    public static DataSoureType getDataSourceKey() {
        return dataSourceKey.get();
    }

    public static void setDataSourceKey(DataSoureType dataSoureType) {
        dataSourceKey.set(dataSoureType);
    }

    public static void clearDataSourceKey() {
        dataSourceKey.remove();
    }

}
{% endhighlight %}

以及DynamicDataSource的使用

{% highlight java %}
    @Bean
    public DynamicDataSource dynamicDataSource() {
        DynamicDataSource dataSource = new DynamicDataSource();
        Map<Object, Object> targetDataSources = new HashMap<>();
        targetDataSources.put(DataSoureType.CRM_RES, dataSourceCRMRead());
        targetDataSources.put(DataSoureType.ECS, dataSourceECS());
        dataSource.setTargetDataSources(targetDataSources);
        dataSource.setDefaultTargetDataSource(dataSourceCRMRead());
        return dataSource;
    }
{% endhighlight %}

上述使用ThreadLocal来存储当前的数据源Key，确保线程安全，DataSoureType是自己定义的一个枚举类，在DynamicDataSource Bean的声明中，加入了2个数据源CRM_RES和ECS，其对应的数据源Bean通过dataSourceCRMRead和dataSourceECS方法获得，并设置dataSourceCRMRead为默认的数据源。

在使用过程中，当我们需要切换数据源的时候，可以通过显式的执行DynamicDataSource.setDataSourceKey方法，也可以进行约定，通过Spring的AOP来自动切换。

### 2. RestController

在使用Spring RestController的时候，我希望返回的对象在浏览器中直接以json串的形式显式，但是一直报错，网上查了很多资料，试了很多方法也一直不行，后来只能去看Spring的官方文档，看了之后发现Spring 4.0中RestController解析json用的是jackson 2，而之前版本用的是jackson 1，前面试的那些方法原来都是针对Spring 4.0以下版本的，难怪不成功。

解决办法就是在gradle中引入以下依赖即可

    compile 'com.fasterxml.jackson.core:jackson-core:2.4.3'
    compile 'com.fasterxml.jackson.core:jackson-databind:2.4.3'
    compile 'com.fasterxml.jackson.core:jackson-annotations:2.4.3'
    
### 3. 网页格式化输出JSON

上述已经成功的把json返回到浏览器了，但是显式出来的是长长的一整条json串，不便于阅读。如何能够用缩进的方式展现json呢？

解决方法有很多，你可以使用网上别人开源的类库，除了能支持缩进，还可以语法高亮等等，但是我比较喜欢简单，能够不引入类库就尽量不引入。

我的做法是，将json串中的每个对象缩进4个字符的长度，然后在css中指定显式样式为white-space: pre

    $('#response').html(JSON.stringify(json, null, 4))

    #response {
        white-space: pre;
    }

### 4. 动态显式HTML

整个web应用只有一个页面，不同的操作，需要输入的内容不同，因此需要根据不同的操作动态的显式HTML内容，如何做到？这个还是比较简单的，可以通过js直接在DOM树中加元素，也可以直接在某个HTML标签里面通过InnerHTML插内容，但是这两种形式，写出来的HTML代码不直观，不易读，怎么办？可以通过angular.js来解决这个问题，但是有没有更加轻量的解决方法？在网上找到一个很绝妙的方法，做法是定义一个js函数，将HTML代码作为这个函数的注释，然后通过toString方法获得这段HTML代码

{% highlight javascript %}
function phone() {/*
 <div class="bs-example bs-example-form">
 <div class="input-group">
 <span class="input-group-addon">手机号码</span>
 <input id="phoneNum" type="text" class="form-control">
 </div>
 <button class="btn btn-primary" onclick="byPhoneMethod(action,phoneNum.value)">提交</button>
 </div>
 */}
 
function heredoc(fn) {
    return fn.toString().split('\n').slice(1, -1).join('\n') + '\n';
}
{% endhighlight %}

### 5. event.preventDefault()
	
最初我使用的是form来提交数据，后来改成了Ajax，但是每次Ajax执行完了，都会莫名其妙的跳转到一个当前地址+?的一个页面，导致显示失败，后来发现是form中的submit最终都会执行action，如果没有配置action则默认为？，因此如果不想执行form提交，可以在Ajax中使用event.preventDefault()来阻止事件的默认行为。但是嫌麻烦，我后来就不使用form了。

### 6. Ajax与Spring MVC交互

在Spring MVC中使用@RequestParam注解获取客户端入参时，如果客户端的Ajax请求指定了dataType和contentType为json，那么是会报400错误的，为什么？因为客户端的json是作为request body发送的，而不是request param，解决方法有二种，要么客户端Ajax请求去掉json指定，要么服务端把@RequestParam改成@RequestBody

### 7. Mybatis 

CRM中有些逻辑库是分8个物理库的，每个物理库中又分8个物理表，因此需要根据不同的用户，通过CRM的分库分表规则来确定具体的库表信息，所以需要在Mybatis中动态指定库名和表名，其中参数是用于库表名的使用${param}，参数是用于where语句中的条件值的使用#{param}，另外当Mybatis的接口方法有多个入参时，需要明确的指定入参与SQL语句中参数的对应关系，例如下面这样：

{% highlight java %}
@Select("SELECT * FROM crm_${center}.ins_user_os_state_${regionId} WHERE USER_ID = #{userId}")
Map<String, Object> getOSStateByUserId(@Param("center") String center, @Param("regionId") String regionId, @Param("userId") String userId);
{% endhighlight %}

### 8. 文件下载

计费有个功能是造话单文件，这个造出来的话单文件有两种展现方式，一种是明文显示，一种是文件下载，那么需要做的是，Spring MVC方法中返回的是一个文件，这个简单，有很多种方式，简单点的可以直接返回一个FileSystemResource，但是有个问题，这个返回的FileSystemResource的文件名都是请求的URL，而不是真实的文件名，其实这个问题比较常见，有时我们在网上下载东西时，文件的网页显示名和打开这个链接让我们保存时的名字是不一样的，而有些网站则是一样的，那么怎么做到一样呢？其实很简单，只需要在响应头中指定Content-Disposition的值即可

{% highlight java %}
@RequestMapping(value = "getFile", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
public FileSystemResource getFile(HttpServletResponse response, @RequestParam String path) {
    response.setHeader("Content-Disposition", "attachment; filename=\"" + path + "\"");
    return new FileSystemResource(path);
}
{% endhighlight %}








