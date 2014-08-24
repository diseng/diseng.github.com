---
layout: post
title : Chrome扩展程序OSChina-Notifier
category : app
tags : [app]
---
{% include JB/setup %}

Chrome 扩展程序 OSChina Notifier 可以显示用户在开源中国的未读消息数.

代码也很简单,我参考的Github Notifier,在这基础上进行了修改.

### api.js

{% highlight javascript %}
(function() {
	'use strict';

	var xhr = function() {
		var xhr = new XMLHttpRequest();
		return function( method, url, callback ) {
			xhr.onreadystatechange = function() {
				if ( xhr.readyState === 4 ) {
					callback( xhr.responseText );
				}
			};
			xhr.open( method, url );
			xhr.send();
		};
	}();

	window.OSChinaNotifCount = function( callback ) {

		var NOTIFICATIONS_URL = 'http://www.oschina.net/';
		xhr( 'GET', NOTIFICATIONS_URL, function( data ) {
			if(data != ""){
				var index = data.indexOf("total_count");
				callback( index == -1 ? "" : data.substring(index+12,index+13));
			}else{
				callback(false);
			}
		});

	};

})();
{% endhighlight %}

### main.js

{% highlight javascript %}
(function() {
	'use strict';

	function render( badge, color, title ) {
		chrome.browserAction.setBadgeText({
			text: badge
		});
		chrome.browserAction.setBadgeBackgroundColor({
			color: color
		});
		chrome.browserAction.setTitle({
			title: title
		});
	}

	function update() {
		OSChinaNotifCount(function( count ) {
			if(count !== false){
				render( count, [65, 131, 196, 255], 'OSChina Notifier' );
			}else{
				render( ':(', [166, 41, 41, 255], 'You have to be connected to the internet and logged into OSChina' );
			}
		});
	}

	var UPDATE_INTERVAL = 1000 * 60;

	setInterval( update, UPDATE_INTERVAL );

	chrome.browserAction.onClicked.addListener(function() {
		window.open('http://www.oschina.net/');
	});

	update();

})();
{% endhighlight %}

原理其实很简单,但是期间还是有些感受.

因为AJAX原生是不支持跨域访问的,但是这个扩展要拿到数据必须得跨域啊,怎么办,我搜了一堆方法,结果试了都不行.郁闷的不行,结果发现在manifest.json文件中可以配置权限,我只需要写上

	"permissions": [
		"http://www.oschina.net/"
	],

就可以访问OSChina了......

现在拿到首页内容了,但是OSChina把未读消息数放在了一个JS里面,额,不能通过id,class等方式来访问了,我对js也不熟,于是问了班上一女生,她说直接正则匹配吧.然后就用正则实现了.

接着是图标的问题,我下载的OSChina图标,用win7自带的画图工具调整图片大小后,就失去透明度了.于是让班上的PS大神荣矮帮我P一下,他说这简单,不过电脑刚重装,没PS了.于是我下了个破解版的PS,他打开图片一看,说这图标本来就周围是透明的,直接调整大小保存就行了.得到了16×16,19×19,48×48,128×128的图标后,发现图标越小,上下左的黑色影越明显,于是又让荣矮PS了一下.秀一下荣矮的作品.

<center><img alt="oschina-notifier" src="{{ ASSET_PATH }}hooligan/img/post/oschinanotifier-1.png"/></center>

接下去把应用提交到chrome应用商店,结果第一次提交应用需要$5开发人员注册费,我了个fuck,但是都做出来,5刀就5刀吧.结果付费的时候不支持大陆的银行卡啊.....我就找了现在在加拿大的高中同学,4年了从来没联系过啊,不知道怎么说好.结果后面我直接说,你知道我找你肯定是有事,不然不会找你的,她也很爽快,哈哈一下,说什么事.然后她说回去帮我付,现在在上课,我说好.第二天(我们晚上睡觉,他们白天上课嘛),垚神说财付通可以海外代付的,于是后面就让垚神帮我代付了这5刀.后面跟加拿大的同学说不用了,已经付了,作品在chrome应用商店了.她说你可以做一个看youku的插件啊,国外youku上很多视频看不了的,有版权保护.然后我说,这个和法律有关的,可能做了会被喝茶.于是,这里问一下,国外如何才能看youku的一些只能在国内播的视频?

<center><img alt="oschina-notifier" src="{{ ASSET_PATH }}hooligan/img/post/oschinanotifier-2.PNG"/></center>

最后要说的是,这个扩展其实问题还是比较多,感谢OSChina网友柯激情提出的意见

	1)直接对oschina抓包并用indexOf的方式来拿消息数量,随着oschina的改版需要被动的升级，不应该这么设计。
	2)文字截断逻辑有问题，未读消息只能取个位数callback( index == -1 ? "" : data.substring(index+12,index+13));

还是要不断修改啊.

最后附上OSChina Notifier相关信息:

chrome应用商店地址:<https://chrome.google.com/webstore/detail/oschina-notifier/okphpdhjakgckekfkhkjkkgddolnmfag>

源码托管在github:<https://github.com/diseng/OSChina-Notifier>
