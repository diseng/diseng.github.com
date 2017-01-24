---
layout: post
title : windows下使用openvpn
category : tool
---
{% include JB/setup %}

最近常有小伙伴问我怎么翻墙或者知道怎么翻墙，问我要账号来着。

由于现在的电脑操作系统和智能手机都默认支持pptp，出于方便，我一般都是先告诉对方pptp账号，让其尝试是否可以连接成功。但是很多网络都禁止了pptp协议，于是就让openvpn出场了。

由于电脑操作系统和智能手机默认一般都不支持openvpn，所以得安装第三方的openvpn客户端软件。小伙伴就问我要怎么安装，怎么配置了，问了多了，索性写一篇简单的教程吧。

我先在谷歌搜索openvpn，得到的结果是这样的：

<center><img alt=“windows-openvpn-1” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-1.png" width="50%" /></center>

第一个链接打不开，第二个链接继续打不开，这搞个蛋啊，下这个软件就是为了翻墙，结果下载这个软件也得翻墙。。。

<center><img alt=“windows-openvpn-2” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-2.png" width="50%"/></center>

不过上面这个是我小伙伴的情况，我这边有goagent，shadowsocks，pptp，openvpn等等，翻个墙不是问题。

于是我点开第一个链接，下了一个win平台的软件privatetunnel.msi，安装之后是这样的：

<center><img alt=“windows-openvpn-3” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-3.jpg" width="50%"/></center>

我擦，这是什么软件。。。怎么和网上的openvpn长的完全不一样。。

原来应该从第二个链接去下openvpn客户端软件，32位和64位的我都下载了，放着百度云盘上供大家下载，32位在[这里](http://pan.baidu.com/s/1o638U0a)，64位在[这里](http://pan.baidu.com/s/1dDGc8DV)

安装很简单，这里不累赘了

<center><img alt=“windows-openvpn-4” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-4.png" width="50%"/></center>

安装完成后把openvpn的配置文件放到上述软件安装目录的config目录里（这个配置文件问你有openvpn服务的小伙伴要吧）

<center><img alt=“windows-openvpn-5” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-5.jpg"/></center>

记得把文件client.conf改名成client.ovpn

然后可以双击打开软件了，如果你运气好的话，会看到这样：

<center><img alt=“windows-openvpn-6” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-6.jpg" width="50%"/></center>

没事，右键以管理员身份运行即可：

<center><img alt=“windows-openvpn-7” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-7.png" width="30%"/></center>

然后桌面右下角会出现一个图标，右键那个图标，就可以进行连接，断开连接等操作了

<center><img alt=“windows-openvpn-8” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-8.png" width="30%"/></center>

不出意外，等候几秒就可以连接成功了，畅游互联网吧

<center><img alt=“windows-openvpn-9” src="{{ ASSET_PATH }}hooligan/img/post/windows-openvpn-9.png" width="50%"/></center>

以上操作在win7 64位下进行，不保证其他系统情况与这个完全一样。









