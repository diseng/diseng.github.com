---
layout: post
title : SIP测试记录
category : program
tags : [sip]
---
{% include JB/setup %}

随着互联网+在国家层面的战略实施，在不久的将来，越来越多的传统企业将会结合互联网优势(大数据、云计算、物联网等)来升级现有商业模式或者创造新的商业模式，而这其中，通信则是最为基础，也是最为关键的一个环节。

电子邮件、短信等旧有形式的信息通知，在到达率、转化率、安全性等方面已经逐渐不能满足业务方的需求，于是越来越多的业务方转而采用语音通信的方式，也许你们已经或者有可能要进行这方面的转变，那么希望这篇文章能够给你带来一些帮助。

### 1.什么是SIP，为什么要了解SIP测试

一般来说，我们可以把当今的通信网络，大致分为两种，一种是IP网络，另一种是PSTN网络(俗称电话网)，如何把这两种网络互联在一起，有多种解决方案，SIP便是其中的一种。

[SIP(Session Initiation Protocol，会话发起协议)](http://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D%E5%8F%91%E8%B5%B7%E5%8D%8F%E8%AE%AE)是用于[VoIP(Voice over IP，网际协议通话技术)](http://zh.wikipedia.org/wiki/%E7%B6%B2%E9%9A%9B%E5%8D%94%E8%AD%B0%E9%80%9A%E8%A9%B1%E6%8A%80%E8%A1%93)最主要的信令协议之一，承载了[SDP(Session Description Protocol，会话描述协议)](http://zh.wikipedia.org/wiki/%E4%BC%9A%E8%AF%9D%E6%8F%8F%E8%BF%B0%E5%8D%8F%E8%AE%AE)内容，SDP协议描述了会话所使用流媒体细的节，如：使用哪个IP端口，采用哪种音视频编解码等等。

通过SIP发起的通话呼叫，我们可以获取到通话的响铃时长、接听时长、挂断时间点、未接听原因、未接通原因、通话时输入的按键等通信元信息，而这些元信息对于解决业务方日益变化和增长的需求，十分重要。依托开源的SIP服务器，我们不必再使用昂贵的硬件设备，使用软交换的形式也可以获得企业级硬件解决方案的效果，而且更便于需求的定制。

### 2.SIP测试包含哪些内容

SIP测试通常包括覆盖率测试、语音质量测试、安全测试、稳定性测试、性能测试等，由于篇幅问题，本文只介绍覆盖率及语音质量测试。

所谓的覆盖率测试，即需要将通话覆盖到全球或者全国各大主要城市，用来测试SIP请求落地PSTN网络的线路覆盖情况。

而语音质量测试，则是SIP请求落地PSTN网络后，实际的一个通话语音质量的测试。这部分主要包括系统评分以及人为评分。

### 3.如何自动化发起语音呼叫

不管是覆盖率测试还是语音质量测试，我们都需要通过SIP服务器来对全球或者全国各大主要城市的电话发起呼叫。

在本文中，我们使用的是一款开源的SIP服务器-[FreeSWITCH](https://freeswitch.org/)，该服务器内置lua脚本支持，因此我们可以使用lua脚本来自动化发起语音呼叫。

其中具体执行呼叫的方法体如下：

{% highlight lua %}
function call( callee )
    --呼叫字符串
    local callStr = "这里是具体的呼叫字符串"
    --呼叫时长限制 单位秒
    local timeout = 20
    --日志打印呼叫字符串
    freeswitch.consoleLog("err",callStr)
    --建立会话
    local callSession = freeswitch.Session(callStr)
    --支持sched_hangup 在指定的时长之后，终止会话
    callSession:execute("sched_hangup","+"..timeout.." alloted_timeout")
    --会话建立成功或者失败的分支处理
    if( callSession:ready() ) then
            freeswitch.consoleLog("notice", "call success\n");
            callSession:execute("echo");
    else
            local cause = callSession:hangupCause();
            local causeQ850 = callSession:getVariable("hangup_cause_q850") or '';
            freeswitch.consoleLog("notice", "call fail,cause:"..cause..",causeQ850:"..causeQ850.."\n");
    end
end
{% endhighlight %}


### 4. 如何按照会话进行抓包

要对上述的呼叫进行覆盖率以及语音质量方面的分析，我们首先需要获取相关的SIP报文，以及承载音视频内容的[RTP(Real-time Transport Protocol，实时传输协议)](http://zh.wikipedia.org/wiki/%E5%AE%9E%E6%97%B6%E4%BC%A0%E8%BE%93%E5%8D%8F%E8%AE%AE)报文。

使用tcpdump或者wireshark自然是可以进行抓包的，但是将所有的会话报文抓取在一起，并不利于我们对单个会话的分析，而我们的覆盖率以及语音质量都是以会话为基本单位进行的。

我们可以使用[pcapsipdump](http://pcapsipdump.sourceforge.net/)这款开源的抓包软件，此软件以会话为单位进行抓包，单个会话的SIP、RTP报文在一个pcap文件中。

使用如下命令即会监听eth0网卡，并将pcap文件写在/home/admin/pcap目录下

```
sudo pcapsipdump -i eth0 -d /home/admin/pcap
```

### 5.如何统计呼叫结果

有了上述报文，我们就可以获取每一个会话的结束状态，比如正常挂断、未应答、拒接、号码错误、用户繁忙、无权限呼叫、服务端错误等

我们使用[tshark](https://www.wireshark.org/docs/man-pages/tshark.html)(wireshark的命令行版本，安装wireshark后就自带该命令)来分析pcap文件

使用下面的命令，可以查看该会话的所有SIP信息

{% highlight bash %}
tshark -Y sip -r pcapFile
# output
# 1   0.000000 198.xxx.xxx.xxx -> 202.xxx.xxx.xxx SIP/SDP 1137 Request: INVITE sip:390115170171@202.xxx.xxx.xxx:5060 |
# 2   0.159216 202.xxx.xxx.xxx -> 198.xxx.xxx.xxx SIP 351 Status: 100 Trying |
# 3   1.774173 202.xxx.xxx.xxx -> 198.xxx.xxx.xxx SIP/SDP 765 Status: 183 Session Progress |
# 4   2.992899 202.xxx.xxx.xxx -> 198.xxx.xxx.xxx SIP/SDP 765 Status: 183 Session Progress |
# 5   8.581402 202.xxx.xxx.xxx -> 198.xxx.xxx.xxx SIP/SDP 776 Status: 200 OK |
# 6   8.582468 198.xxx.xxx.xxx -> 202.xxx.xxx.xxx SIP 458 Request: ACK sip:390115170171@202.xxx.xxx.xxx:5060 |
# 7  15.702624 202.xxx.xxx.xxx -> 198.xxx.xxx.xxx SIP 417 Request: BYE sip:gw+gwName@198.xxx.xxx.xxx:5080;transport=udp;gw=gwName |
# 8  15.703204 198.xxx.xxx.xxx -> 202.xxx.xxx.xxx SIP 477 Status: 200 OK |
{% endhighlight %}


而一般情况下，我们只需要获取最终状态即可，可以使用如下命令对内容进行筛选

{% highlight bash %}
tshark -Y sip -r pcapFile|grep 'Status'|tail -n 1|awk -F '[:|]' '{print $2}'
# output
# 200 OK
{% endhighlight %}

除了获取该会话的最终状态，该会话的呼叫号码也是很重要的一个信息，我们依然可以从SIP信息中筛选中呼叫号码

{% highlight bash %}
tshark -Y "sip.Method == INVITE" -r pcapFile|head -n 1|awk -F '[:@+]' '{print $3}'
# output
# 390115170171
{% endhighlight %}

有了呼叫号码以及对应的会话最终状态，写一个小程序来分析统计各个国家或者各个省市的呼叫成功率，失败率，失败原因自然不是什么难事。

### 6.如何分析呼叫语音质量

语音质量的测试主要有两种方式进行，一种是通过听取会话的音频流，进行人为主观的一个打分，还有一种就是通过系统数据来进行打分，系统数据包括RTP的丢包率、抖动率、最大时延、平均时延等。

#### 6.1 人为打分

人为打分，要做的一个事情就是从pcap文件中还原出音频流文件，这里我们除了使用tshark还需要一个强大的音频处理软件[sox](http://sox.sourceforge.net/)

有了上述两个软件，使用下面的shell脚本即可从pcap文件中提取出wav音频文件，其原理是用tshark读取出双向的rtp.ssrc，分别处理，并取出rtp.payload的HEX值，生成raw文件，然后用sox转成wav文件

{% highlight bash %}
if [ -z $1 ] ; then
    echo "`basename $0` {pcap-file}"
    exit
fi

for SSRC in `tshark -n -r $1 -Y rtp -T fields -e rtp.ssrc -Eseparator=,|sort -u`
do
    tshark -n -r $1 -Y rtp -Y "rtp.ssrc == $SSRC" -T fields -e rtp.payload | tr : '\n' > $SSRC.payloads
    > $SSRC.raw
    for HEX in `cat $SSRC.payloads`
    do
        printf "\\x$HEX" >> $SSRC.raw
    done
    [ -f $SSRC.wav ] && rm $SSRC.wav
    sox -t raw -r 8000 -c 1 -e mu-law $SSRC.raw $SSRC.wav
    if [ -z $A ] ; then
        A=$SSRC
    else
        B=$SSRC
    fi
done
rm *.payloads *.raw
sox -mM $A.wav $B.wav $A-$B.wav
{% endhighlight %}


不要忘记上一节中提到的获取呼叫号码的方法，使用号码来归类存放音频文件，方便后续人为打分

#### 6.2 系统打分

使用tshark可以对pcap文件中RTP报文进行统计，分析得出丢包率、抖动率、最大时延、平均时延等数据

{% highlight bash %}
tshark -q -z rtp,streams -r pcapFile
# output
# ========================= RTP Streams ========================
#     Src IP addr  Port    Dest IP addr  Port       SSRC          Payload  Pkts         Lost   Max Delta(ms)  Max Jitter(ms) Mean Jitter(ms) Problems?
#  202.xxx.xxx.xxx 28106  198.xxx.xxx.xxx 29728 0x1F62A4A1 ITU-T G.711 PCMU  1055     0 (0.0%)           20.75            0.17            0.05 X
#  198.xxx.xxx.xxx 29728  202.xxx.xxx.xxx 28106 0x99E37E4A ITU-T G.711 PCMU   975     0 (0.0%)           21.05            0.31            0.03 X
# ==============================================================
{% endhighlight %}


我们只需要取出其中的Src IP addr、Dest IP addr、Payload、Pkts、Lost、Max Delta(ms)、Max Jitter(ms)、Mean Jitter(ms)字段即可。

{% highlight bash %}
tshark -q -z rtp,streams -r pcapFile|sed -n '3,4p'|awk '{print $1,$3,$8,$9,$10$11,$12,$13,$14}'
# output
# 202.xxx.xxx.xxx 198.xxx.xxx.xxx PCMU 1055 0(0.0%) 20.75 0.17 0.05
# 198.xxx.xxx.xxx 202.xxx.xxx.xxx PCMU 975 0(0.0%) 21.05 0.31 0.03
{% endhighlight %}

依然不要忘记上一节中提到的获取呼叫号码的方法，将号码与其对应的RTP数据包的丢包率、抖动率、最大时延、平均时延关联起来分析

有了这些数据，然后根据业务方要求或者自定要求，定义一个语音质量的计算公式，写个小程序来对每个国家或者每个省市的通话语言质量进行打分，自然不是什么难事
