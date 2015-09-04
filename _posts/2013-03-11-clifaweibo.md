---
layout: post
title : 为clifaweibo增加长文字微博分片发送功能
category : program
tags : [program,go]
---
{% include JB/setup %}

有时想把kindle上的读书书摘分享到微博上，但是单条微博的长度限制在140个字（所以会有长微博图片发送的应用），加上受到微博名人任志强分享读书书摘采用短信分片的影响，我也无意中倾向于使用纯文字的方式来分享读书书摘。但是目前貌似只有短信支持自动分片，这样使用起来就不是很方便了。于是昨天下午fork了JessonChan的clifaweibo项目，在原功能基础上增加了长文字微博分片发送功能。（PS：clifaweibo是一款终端查看和发送微博的应用）

最新版本的clifaweibo采用GO语言编写（第一次接触GO语言），昨天下午就熟悉了一下GO语言，并简单了解了一下fmt、net/http、strconv、strings、unicode/utf8、time等包。

发现GO没有包自带substring功能，但是对微博分片需要用到这种功能，于是就现学现用了。

```go
func substring(s string, start int, length int) string {
        by := []byte(s)
        i:=0
        for i=0; start >0 && i < len(by); i++ {
                value := by[i]
                if value >= 224 {
                        i+=2;
                        start --
                } else if value >=192 && value <224 {
                        i+=1;
                        start --
                } else {
                        start --
                }   
        }   
        ii := i 
        for ; length>0 && i < len(by); i++ {
                value := by[i]
                if value >= 224 {
                        i+=2;
                        length --
                } else if value >=192 && value <224 {
                        i+=1;
                        length --
                } else {
                        length --
                }   
        }   
        return s[ii: i]
}
```

有了substring，接下去只要对微博计算一下需要分割成几片就行了。

```go
func send_divided_text_weibo(text string) bool {
	divided_number := (utf8.RuneCountInString(text) + weibo_text_length - 1) / weibo_text_length
	for i := 0 ; i < divided_number ; i++ {
		divided_text := "(" + strconv.Itoa(i+1) + "/" + strconv.Itoa(divided_number) + ")"
		divided_text += substring(text, i*weibo_text_length, weibo_text_length)
		fmt.Println(divided_text)	
		if false == send_text_weibo(divided_text) {
			return false
		}	
		fmt.Println("分片发送成功")	
		time.Sleep(5000 * time.Millisecond)		
	}		
	return true
}
```

最初没有`time.Sleep(5000 * time.Millisecond)`这一句，结果测试的时候始终只能发送出去第一条分片，因此我还请教了JessonChan。后面我猜想可能是微博API的限制，不允许短时间内发送多条微博（这个可能性比较小）或者是http.Post()返回结果需要一定的时间，于是加了睡眠5秒。经测试，分片发送成功。

<center><img alt="clifaweibo" src="{{ ASSET_PATH }}hooligan/img/post/clifaweibo.PNG"/></center>


以后也可以学任志强分享读书摘录啦，O(∩_∩)O哈哈~

查看完整源码[点这里](https://github.com/diseng/clifaweibo/blob/master/main.go)
