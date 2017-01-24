---
layout: post
title : Linux VPS 配置脚本
category : linux
---
{% include JB/setup %}

去年暑假，我使用优惠码花费9.99美刀购买了一个bandwagonhost的vps，配置是这样的：

	5G PROMO V2 
	Unmanaged service
	HDD: 5 GB SSD
	RAM: 512 MB
	CPU: 1x Intel Xeon 
	BW: 500 GB/mo
	Link speed: 1 Gigabit

	VPS technology: OpenVZ/KiwiVM 
	Linux OS: 32-bit and 64-bit Centos, Debian, Ubuntu, Fedora
	Instant OS reload
	1 Dedicated IPv4 address
	Full root access
	PPP and VPN support (tun/tap)
	Instant RDNS update from control panel
	No contract, anytime cancellation 
	Strictly unmanaged, no support
	99% uptime guarantee
	30-day money back guarantee

我没有在上面架站，只是搭了一个pptp、一个openvp、一个shadowsocks。VPS的使用情况，常年如下：

<center><img alt=“linux-vps-config-script” src="{{ ASSET_PATH }}hooligan/img/post/linux-vps-config-script.png" width="50%" /></center>

最初，都是自己手动一步一步的配置pptp、openvpn，有时在vps上折腾小玩意后重启vps，有小概率会重启失败，进入网页控制面板也重启不了，只能重新安装系统，这么几次之后，觉得手动配置太麻烦了，于是就想到了脚本。网上有符合要求的现成脚本(github上一搜一大把，我会乱说?)就直接拿来使用，没有的话就自己写。

现在我主要使用3个脚本，一个是配置pptp的，脚本在[这里](https://github.com/diseng/ezpptp)，一个是配置openvpn的，脚本在[这里](https://github.com/diseng/openvpn-install)，还有个是配置VPS基础安全的，比如禁止root登录，禁用密码登录，更改ssh端口等，脚本在[这里](https://github.com/diseng/shell-scripts/tree/master/vps)

以上3个脚本在Debian/Ubuntu系统上使用正常，其他系统可能会有略微区别，如果你愿意折腾，可自行修改脚本内容。