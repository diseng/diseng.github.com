---
layout: post
title : 通过Guacamole远程web控制你的电脑
category : program
---
{% include JB/setup %}

Guacamole是一个HTML5的远程桌面网关。

Guacamole使用VNC、RDP等远程桌面协议来提供桌面环境的访问。Guacamole服务器既是一个隧道也是一个代理，通过浏览器就可以访问多种桌面。

使用Guacamole不需要安装别的软件或者插件，仅仅需要一个支持HTML5和AJAX的现代浏览器。

更多关于Guacamole的信息，你可以查看[这里](http://guac-dev.org/)

下面简单讲解一些如何快速体验Guacamole。

**os:ubuntu 12.04**

**1.添加guacamole的PPA源并更新**

	sudo add-apt-repository ppa:guacamole/stable
	sudo apt-get update

**2.安装guacamole(会自动安装依赖组件)**

	sudo apt-get install guacamole-tomcat	

**3.部署配置**

	sudo mkdir /usr/share/tomcat6/.guacamole
	sudo ln -s /etc/guacamole/guacamole.properties /usr/share/tomcat6/.guacamole/

**4.启动vnc**

	sudo vnc4server 
	会提示你设置密码
	注意创建出的vnc端口号=5900+vnc号   vnc号一般是1~10   例如New 'ubuntu:1 (root)' desktop is ubuntu:1  表示vnc号为1

<center><img alt="guacamole-usage-3" src="{{ ASSET_PATH }}hooligan/img/post/guacamole-usage-3.jpg"/></center>

**5.修改/etc/guacamole/user-mapping.xml**

```xml
	<authorize username="web登录用户名" password="web登录密码">
            <protocol>vnc</protocol>
            <param name="hostname">localhost</param>
            <param name="port">第4步中的vnc端口号</param>
            <param name="password">第4步中设置的vnc密码</param>
    </authorize> 
```

**6.重启tomcat**

	sudo /etc/init.d/tomcat6 restart

**7.使用浏览器访问http://ip地址:8080/guacamole**

<center><img alt="guacamole-usage-1" src="{{ ASSET_PATH }}hooligan/img/post/guacamole-usage-1.JPG"/></center>

<center><img alt="guacamole-usage-2" src="{{ ASSET_PATH }}hooligan/img/post/guacamole-usage-2.JPG"/></center>

上述步骤可以让你简单快速的体验Guacamole，但是Guacamole本身还提供了其他的认证方式，包括MySQL、LDAP等，官网也有开发者指引手册，如果你要兴趣了解更多请访问[Guacamole官网](http://guac-dev.org/)。
