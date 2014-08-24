---
layout: post
title : Xen虚拟化技术中PV和HVM的区别
category : translate
tags : [translate]
---
{% include JB/setup %}

Xen是一个开源的type-1或者裸机管理程序，它使得一个物理主机能够同时并行运行多个相同的或者不同的操作系统实例。Xen是目前唯一的开源可得的type-1管理程序。Xen被应用于许多商业和开源的应用程序中，比如：服务器虚拟化(server virtualization)、基础设施即服务(Infrastructure as a Service)、桌面虚拟化(desktop virtualization)、安全应用程序(security applications)、嵌入式和硬件设备(embedded and hardware appliances)。毫无疑问，Xen驱动着当今大部分的云计算市场。

Xen支持运行两种不同类型的虚拟机:半虚拟化(PV)和全虚拟化(HVM)。在一个单一的Xen系统中可以同时运行这两种不同类型的虚拟机。另外，在全虚拟化(HVM)虚拟机中也能够使用半虚拟化(PV)技术：实质上是创建一个半虚拟化(PV)和全虚拟化(HVM)的连续体。这种方式被称为PV on HVM。想要获取更多关于虚拟化的知识可以看[这里](http://wiki.xenproject.org/wiki/Virtualization_Spectrum)

那么Xen虚拟化技术中的半虚拟化(PV)和全虚拟化(HVM)有什么区别呢?

**Xen Paravirtualization (PV)**

半虚拟化是由Xen引入的高效和轻量的虚拟化技术，随后被其他虚拟化平台采用。半虚拟化技术不需要物理机CPU含有虚拟化扩展。但是，要使虚拟机能够高效的运行在没有仿真或者虚拟仿真的硬件上，半虚拟化技术需要一个Xen-PV-enabled内核和PV驱动。可喜的是，Linux、NetBSD、FreeBSD和OpenSolaris都提供了Xen-PV-enabled内核。Linux内核从2.6.24版本起就使用了[Linux pvops框架](http://wiki.xenproject.org/wiki/XenParavirtOps)来支持Xen。这意味着半虚拟化技术可以在绝大多数的Liunx发行版上工作(除了那么内核很古老的发行版)。关于半虚拟化技术的更多信息可以看[这里](http://wiki.xenproject.org/wiki/Paravirtualization_(PV))

**Xen Full Virtualization (HVM)**

全虚拟化或者叫硬件协助的虚拟化技术使用物理机CPU的虚拟化扩展来虚拟出虚拟机。全虚拟化技术需要Intel VT或者AMD-V硬件扩展。Xen使用[Qemu](http://wiki.qemu.org/Main_Page)来仿真PC硬件，包括BIOS、IDE硬盘控制器、VGA图形适配器(显卡)、USB控制器、网络适配器(网卡)等。虚拟机硬件扩展被用来提高仿真的性能。全虚拟化虚拟机不需要任何的内核支持。这意味着，Windows操作系统可以作为Xen的全虚拟化虚拟机使用(众所周知，除了微软没有谁可以修改Windows内核)。由于使用了仿真技术，通常来说全虚拟化虚拟机运行效果要逊于半虚拟化虚拟机。

**PV on HVM**

为了提高性能，全虚拟化虚拟机也可以使用一些特殊的半虚拟化设备驱动(PVHVM 或者 PV-on-HVM驱动)。这些半虚拟化驱动针对全虚拟化环境进行了优化并对磁盘和网络IO仿真进行分流，从而得到一个类似于或优于半虚拟化虚拟机性能的全虚拟化虚拟机。这意味着，你可以对只支持全虚拟化技术的操作系统进行优化，比如Windows。

Xen半虚拟化虚拟机自动使用PV驱动-因此不需要提供这些驱动，你已经在使用这些优化过的驱动了。另外，只有Xen全虚拟化虚拟机才需要PVHVM驱动。关于PV on HVM的更多信息可以看[这里](http://wiki.xenproject.org/wiki/PV_on_HVM)

**PV in an HVM Container (PVH) - New in Xen 4.4**

Xen 4.4会带来一个被称作PVH的新的虚拟化模式。实质上，它是一个使用了针对启动和I/O的半虚拟化驱动的半虚拟化模式。与全虚拟化不同的是，它使用了硬件虚拟化扩展，但是不需要进行仿真。在Xen 4.3发布后，xen-unstable会加入对此模式的补丁，Xen 4.4中将可以预览到这个功能。PVH拥有结合和权衡所以虚拟化模式优点的潜力，与此同时简化Xen的架构。

