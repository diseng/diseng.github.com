---
layout: post
title : 写给服务端同学看的“不负责任”前端入门文档
category : program
tags : [font-end]
---
{% include JB/setup %}

## 0. 序

最近两周，我开始接触前端工作。两周之前，我的前端姿势还停留在jsp+bootstrap这一层面，随着这两周的入门学习，我愈发的觉得，不管是否从事前端开发工作，作为一名技术人员，都应该掌握一些前端的入门姿势。

虽然前后经历的时间跨度有两周，但是真实用来学习的时间远远没有这么多，因此此文更多的是我个人的一个学习记录，如果没有帮助到各位看官，那么请各位看官帮助一下我，给我指出一条明路。。。

## 1. 基础知识

基础知识这一块，我买了几本书做入门，然而并没有怎么看，就已经开始接手工作了，有同学可能就会问，书都没看完就上手工作了，会不会有很多问题啊？是有很多问题，但是有谷歌，谷歌不了还可以问周围的同学，所以没有看过书并不是不能上手工作的借口。但是如果你问我，那还要不要看书，我肯定明确的回答，要看，而且要仔细的看。

看书可以较为系统的学习一门姿势，使用谷歌则是快速的解决问题。系统的学习一门姿势，可以做到知其然知其所以然，掌握了这点，很多问题就引刃而解。谷歌帮助我们快速的解决问题，但是往往不能做到详细的解释，因为从一个问题点，很难引申到整个姿势体系，而有些问题不站在姿势体系上是很难从原理上解释清楚的。当然那些善于搜索，喜欢庖丁解牛，又乐于总结分享的人，不管什么方式，都能成为姿势达人，然而我并不能做到这样子。。。

废话不多说，上硬菜：

### 1.1 HTML & CSS: 

[Learn to Code HTML & CSS](http://learn.shayhowe.com/html-css/)

[CSS3 教程](http://waylau.gitbooks.io/css3-tutorial/content/docs/Introduction.html)

[学习CSS布局](http://zh.learnlayout.com/)

书籍：我买了一本《CSS权威指南》，但是只看了一点点，这里不好做推荐，想要买书籍的同学自己网上搜一下，看别人的推荐吧

### 1.2 JavaScript:

[JavaScript 标准参考教程（alpha）](http://javascript.ruanyifeng.com/)

[ECMAScript 6入门](http://es6.ruanyifeng.com/)

书籍：我买了一本《JavaScript高级程序设计》，同样还没看完，这里也不好做推荐，但是一般网上推荐入门的也都是这本红宝书或者OReilly的犀牛书《JavaScript权威指南》

另外说一下，上面两本电子书都是阮一峰老师写的，不得不说，我对前端的兴趣很大一部分是来自阮一峰老师的感化，如果还不认识阮一峰的同学可以关注一下他的[博客](http://www.ruanyifeng.com/blog/)，涵盖了技术、文学、社会、哲理都多方面的姿势。

### 1.3 其他

本节只涉及最最基础的知识，关于sass、less、CoffeeScript、TypeScript等不做介绍，有兴趣的同学请自行谷歌

## 2. 工具篇

掌握了基础知识，就可以上手做一些网页了，但是想要更加专业，效率更高，就需要再学习了解一些工具。

以下几个工具我只做最简单的介绍，具体的操作，请查阅官方文档。关于这些工具的定位，在我目前简陋的认识下，可能会有错误，欢迎指正。

### 2.1 Yeoman

如果有一个工具，能够像maven的archetype一样，帮助我们快速生成一个项目骨架就太好了，是的，它就叫[Yeoman](http://yeoman.io/)

### 2.2 Bower

如果有一个工具，能够像maven一样帮助我们管理各种依赖，并且自动从远程仓库下载下来那就太好了，是的，它就叫[Bower](http://bower.io/)

### 2.3 Grunt

如果有一个工具，能够像maven一样帮助我们把一些负责繁琐又重复的事情，比如构建、打包、测试、分发、部署等做到自动化就好了，是的，它就叫[Grunt](http://gruntjs.com/)

### 2.4 其他

前端的工具当然不仅仅是上面这几个，但是这些是最基本最常用最重要的工具，我们可以根据项目的实际需要，再添加使用别的工具。另外，前端的工具变化非常快，有时候，我们没有必要掌握每一个工具，而是在每个领域掌握并最好做到精通至少一个工具就可以了，这样即使出现了新工具，了解切换也是很快的。

## 3. 框架篇

近年来为了适应多平台的发展趋势，后端只负责返回数据，前端则开始涉及数据展现逻辑及与其相关的部分业务逻辑。随着前端代码量和逻辑的日益复杂，当年在后端发生的一些列开发模式也渐渐的发生在了前端，为了提高代码的复用性、维护性、测试性等等方面的表现，前端也开始出现了MVC的开发模式。所以历史往往都是相似的，对java感兴趣的同学，可以看一下去年我整理的这篇文章[《【整理】写给java web一年左右工作经验的人》](http://diseng.github.io/2014/08/10/to-java-web-one-year-coder/)

### 3.1 模块化编程

对JavaScript模块化编程感兴趣的同学可以阅读一下这三篇文章：

[Javascript模块化编程（一）：模块的写法](http://www.ruanyifeng.com/blog/2012/10/javascript_module.html)

[Javascript模块化编程（二）：AMD规范](http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html)

[Javascript模块化编程（三）：require.js的用法](http://www.ruanyifeng.com/blog/2012/11/require_js.html)

### 3.2 AngularJS

JavaScript的MVC框架，或者叫MV*框架，有很多，比如Ember.js、Backbone.js、AngularJS、React等，本文不对这些框架做介绍和对比(不是我不想，而是我确实还没有这个能力。。。)，由于姿势有限，这里我只对AngularJS做一个简单介绍(估计有些认识还是错的。。。)

AngularJS主要有module、controller、provider、directive等几个概念，对应到Java，可以简单的理解成下面这样：

AngularJS|Java
---------|----
module|package
controller|controller
provider|spring component
directive|模板引擎

其实directive我还不是太理解，对于AngularJS原生的一些directive有点类似于模板引擎，而对于自定义的directive可以是一些功能上独立的dom(这句话纯属我瞎扯。。。)

关于AngularJS的入门可以查看官方的入门教程[AngularJS Tutorial](https://docs.angularjs.org/tutorial)，另外推荐一下慕课网大漠穷秋老师的[AngularJS实战](http://www.imooc.com/learn/156)

## 4. 其他

推荐一下知乎的这个问题：[大公司里怎样开发和部署前端代码?](http://www.zhihu.com/question/20790576)，其中张云龙老师的回答写的非常好，给我解决了很多疑惑





