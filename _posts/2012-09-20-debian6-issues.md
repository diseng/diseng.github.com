---
layout: post
title : debian6日常问题解决方法
category : linux
tags : [linux]
---
{% include JB/setup %}

### 1.解决debian6挂起后无法唤醒:

	添加文件:
	vim /etc/pm/config.d/10no-vt-switch
	文件内容:
	ADD_PARAMETERS="--quirk-no-chvt"

### 2.debian下禁用adsl的ifupdown:

检查一下/etc/network/interfaces文件，除以下两行外，其他全部注释掉:

	auto lo
	iface lo inet loopback

### 3.解决debian下看acfun评论乱码:

	播放界面->设置->播放器设置->评论字体->选一个不产生乱码的字体

### 4.设置debian下默认程序:

	1)主菜单中添加应用(会在'~/.local/share/application/'下生成'*.desktop'文件)
	2)复制'/etc/gnome/defaults.list'中对应的配置条目到'~/.local/share/application/mimeapps.list'
	3)在'~/.local/share/application/mimeapps.list'中将刚才复制条目中的'*.desktop'改成上面生成的那个'*.desktop'

### 5.解决debian下eclipse部署tomcat发生404错误:

	1)打开eclipse的server视图，双击已配置的那个tomcat，打开编辑窗口，查看server locations
	2)默认是第一个选项(use workspace metadata(does not modify tomcat instation))
	3)应该选择第二个选项，use tomcat installation.然后再启动服务就可以了
	4)如果server location中的三个单选框呈灰色不可编辑，那么你就要把先前导入的删除掉，重新导入一次，修改server locatioins.

### 6.解决debian与win7双系统时间错误

	修改文件:
	sudo gedit /etc/default/rcS
	将UTC=yes改成UTC=no

### 7.修改debian开机grub分辨率

	修改文件:
	sudo vim /etc/default/grub
	在#GRUB_GFXMODE=640x480下添加GRUB_GFXMODE=1366x768
	sudo update-grub

另外:debian的默认grub背景,登录界面背景,桌面背景都在/usr/share/images/desktop-base/目录下


