---
layout: post
title : debian6安装笔记
category : linux
tags : [linux]
---
{% include JB/setup %}

## 准备:

	1.debian-6.0.5-dvd1.iso 
	2.超过4G的U盘 
	3.安装时候需要用到的额外固件(firmware)

## 步骤:

	1.用unetbootin把dvd刻录到U盘 
	2.U盘启动,进行安装

## 基本配置:

### 1.把home下的文件名改成英文

	export LANG=en_US  
	xdg-user-dirs-gtk-update  
	export LANG=zh_CN.UTF-8 

### 2.字体美化

打LCD补丁

方法一：添加http://mozilla.debian.net 源，升级到新版的cairo（这个源为各个Debian发行版提供新版的iceweasel）

安装方法请移步到：[http://mozilla.debian.net/](http://mozilla.debian.net/)

方法二：手动打LCD补丁

代码:

	mkdir build_cairo && cd build_cairo
	sudo apt-get source cairo
	sudo aptitude build-dep cairo
	sudo aptitude install  devscripts
	wget http://archive.ubuntu.com/ubuntu/pool/main/c/cairo/cairo_1.8.10-2ubuntu1.debian.tar.gz
	tar xvf cairo_1.8.10-2ubuntu1.debian.tar.gz
	cd cairo-1.8.10 && patch -Np1 <../debian/patches/04_lcd_filter.patch
	dch -l local "LCD Patch"
	fakeroot debian/rules binary
	cd .. && sudo dpkg -i *.deb 


字体配置设置

~/.fonts.conf'里添加下面内容（'~/.fonts.conf'默认没有需要自己新建） 代码:

{% highlight xml %}
<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>
	<match target="font">
		<edit mode="assign" name="rgba">
			<const>rgb</const>
		</edit>
	</match>
	<match target="font">
		<edit mode="assign" name="hinting">
			<bool>true</bool>
		</edit>
	</match>
	<match target="font">
		<edit mode="assign" name="hintstyle">
			<const>hintslight</const>
		</edit>
	</match>
	<match target="font">
		<edit mode="assign" name="antialias">
			<bool>true</bool>
		</edit>
	</match>
	<match target="font">
		<edit mode="assign" name="lcdfilter">
			<const>lcddefault</const>
		</edit>
	</match>
</fontconfig> 
{% endhighlight %}

字体细节设置

	次像素--轻微--RGB

安装文泉驿微米黑:

	sudo apt-get install ttf-wqy-microhei 

### 3.解决声音问题

在声音管理里把speaker打开

### 4.安装无线网卡驱动

Realtek官网下载驱动（64位需使用2.6.24那个版本,我的无线网卡是Realtek 8191sevb）

安装内核头文件:

	sudo apt-get install linux-headers-XXX 

安装编译开发基本组件:

	sudo apt-get install build-essential 

按照readme.txt进行make,make install,reboot

### 5.安装ATI的non-free驱动

详见[debian wiki](http://wiki.debian.org/ATIProprietary)

### 6.安装fazen图标

### 7.安装笔记本触摸板管理软件TrackPoint and TouchPad Tweeks

	sudo apt-get install gpointing-device-settings 

### 8.安装火狐

编辑/etc/apt/source.list并添加

	#Mint packages
	deb http://packages.linuxmint.com debian import

执行:

	sudo apt-get update
	sudo apt-get install firefox

### 9. 删除桌面图标

	1.应用程序-附件-终端-输入gconf-editor
	2.打开后,在窗口左侧依次点开:apps->nautilus->desktop
	3.在右边的窗口中找到“volumes_visible”选项,去掉后面的勾

### 10.基本完成,其他的看个人需要了

	sudo apt-get install vim preload smplayer
