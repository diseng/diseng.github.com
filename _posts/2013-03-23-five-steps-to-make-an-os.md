---
layout: post
title : 五步创建HelloWorld操作系统
category : software
tags : [software]
---
{% include JB/setup %}

我们学习计算机编程语言的时候,都喜欢先写一个Hello World,那么在学习计算机操作系统的时候,是不是也应该先写一个Hello World的操作系统呢?

这学期教我们计算机操作系统的老师就是这么干的.只需要5步,你就可以制作出一个Hello World操作系统.

### 1 编写汇编程序

        org 07c00h      ; 告诉编译器程序加载到7c00处
        mov ax, cs
        mov ds, ax
        mov es, ax
        call  DispStr     ; 调用显示字符串例程
        jmp $     ; 无限循环
    DispStr:
        mov ax, BootMessage
        mov bp, ax      ; ES:BP = 串地址
        mov cx, 16      ; CX = 串长度
        mov ax, 01301h    ; AH = 13,  AL = 01h
        mov bx, 000ch   ; 页号为0(BH = 0) 黑底红字(BL = 0Ch,高亮)
        mov dl, 0
        int 10h     ; 10h 号中断
        ret
    BootMessage:    
        db  "Hello, OS world!"
        times   510-($-$$)  db  0; 填充剩下的空间，使生成的二进制代码恰好为512字节
        dw  0xaa55        ; 结束标志

### 2 用nasm编译上述汇编程序

    nasm boot.asm –o boot.bin

### 3 将bin文件转换成可启动的映像文件

    #将boot.bin写入第一个扇区
    dd if=boot.bin of=boot.img bs=512 count=1
    #用0填充剩余的扇区
    dd if=/dev/zero of=boot.img skip=1 seek=1 bs=512 count=2879

### 4 新建虚拟机，用img文件启动

  在vmware或者virtualbox中新建虚拟机,设置软驱文件为刚才生成的boot.img(vmware记得选上connect at power on),启动即可

### 5 运行

<center><img alt="os" src="{{ ASSET_PATH }}hooligan/img/post/five-steps-to-make-an-os.PNG"/></center>

后记:如果你对制作操作系统有兴趣,可以在网上搜索《自己动手写操作系统》,也可以购买《30天自制操作系统》一书,进行更加深入的学习.
