---
layout: post
title : 新浪微博应用PlayOnConsole
category : app
tags : [app]
---
{% include JB/setup %}

PlayOnConsole是我于2013年3月17日开发而来。起因是我最初使用califaweibo（一款Linux终端发微博的应用）感觉非常好玩，并且后来还为其增加了长文字微博分片发送功能([见这里](http://diseng.github.com/work/2013/03/11/clifaweibo/))，但是这款应用只能发送微博和查看未读消息通知，功能比较单一。于是我就自己用Java写了个功能更加丰富一点的，包括了常用的发送微博，读取微博，评论微博，回复评论，查看@我的微博，查看对我的评论，查看我发出的评论等。目前的功能还比较粗糙，有些细节方面的东西还需要不断的修改完善。由于Java应用运行依赖于JRE，所以当初用Java来写，现在看来并不是一个好决定，所以，以后也有可能会进行重写。

下面说一下整个新浪微博应用从开发到审核的一些感受.

最开始我选的是站内应用(那时候都不知道站内应用和网页应用的区别),开发起来倒不难,现成的新浪微博SDK for Java,已经把基础性的工作全封装好了,我只需要去实现我自己的程序逻辑就行.大家都知道要玩一个应用,需要给这个应用授权,但是那个授权页面得是自己的,我当时就在github blog上传了一个页面做授权回调用,写了个js函数,把授权回调URL中的code参数拿出来.

{% highlight javascript %}
function GetParam()
{	
	var url = document.location.href;	
	var code=""	
	if (url.indexOf("=")>0)	
	{	
		code = url.substring(url.indexOf("=")+1,url.length) 
	}
	document.write(code);
}
{% endhighlight %}

到这里整个应用就完成的差不多了,接下去需要提交审核应用了.由于选的是站内应用,新浪微博官方会为这个应用建立一个介绍页面,但是这个介绍页面其实也是需要你自己做的,新浪只负责把你的页面嵌入到官方的介绍页面.同样,我又往github blog上传了个应用的介绍页面.然后就开始第一次提交审核了.

提交后,会有个提示,说通常会在5个工作日内完成审核,我真想说新浪的效率真差,结果不到2个小时,我的审核就被驳回了,理由是介绍页面错误.我打开介绍页面一看,405 not allowed,纳尼,github blog的页面不允许外嵌使用,额,咋办.

网上找了下,发现bae目前还是免费的,嗯,那就申请个bae吧,在bae上做了应用的介绍页面,授权回调页面,顺便把新浪应用删了重新建了个网页应用,就不需要提供内嵌的介绍页面了.继续提交审核,由于是周五晚上提交的,一直到周一才有结果,期间我还觉得肯定过审核了,不然怎么审核这么久,后来才发现那是周末,新浪肯定放假了.周一又收到了审核驳回的通知,理由是应用名称不符合规范,好吧,我的应用叫"卷毛Console",卷毛和这应用当然没有半毛钱的关系,这只是我高中时候的一个外号而已,我是想让应用的名字有个性一点,可惜不行啊.

受到前两次的教训,这次我变乖了,就叫"PlayOnConsole"的,意思是在字符终端玩微博.提交审核,结果一天时间,就审核通过了.

<center><img alt="playonconsole" src="{{ ASSET_PATH }}hooligan/img/post/playonconsole.PNG"/></center>

虽然不会有很多人用这个应用,不过这个过程还是挺有收获的,至少知道,哦,原来开发一个新浪微博应用,也不是很难的事嘛.:)

应用介绍页面:<http://0.juanmao.duapp.com/>

应用源码托管在github:<https://github.com/diseng/PlayOnConsole-Weibo>