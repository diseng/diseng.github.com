---
layout: post
title : LinuxMint14启动软盘挂载报错解决方法
category : linux
tags : [linux]
---
{% include JB/setup %}

最近在vmware中虚拟了一个LinuxMint14，其代号为“Nadia”。安装完了后，每次启动，都会弹窗提示以下错误内容：

	Error mounting system-managed device /dev/fd0: Command-line `mount "/media/floppy0"' exited with non-zero exit status 32: mount: /dev/fd0 is not a valid block device

原来是挂载软盘出的错，我就修改了/etc/fstab文件,将软盘相关的那一行注释掉,重启之,结果还是报错.好吧,我就进行了搜索.

出错的原因是"Nadia"启动是将软盘驱动作为模块载入到了内核,我们只需把这个模块拉入黑名单,不让它载入就行了.解决方法记录如下:

```bash
echo "blacklist floppy" | sudo tee /etc/modprobe.d/blacklist-floppy.conf
sudo rmmod floppy
sudo update-initramfs -u
```