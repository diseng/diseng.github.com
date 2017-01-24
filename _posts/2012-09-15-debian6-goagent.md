---
layout: post
title : debian6配置GoAgent
category : linux
---
{% include JB/setup %}

在Windows上使用goagent基本上波澜不惊，一步一步来就ok，不会遇到什么问题。但是在debian下就没有那么顺利。主要有这么两个问题：

**我们首先看看整体的步骤:**

1.申请Google Appengine并创建appid。

2.下载[goagent稳定版](http://code.google.com/p/goagent/)

3.修改`local\proxy.ini`中的`[gae]`下的`appid`为你的appid(多appid请用|隔开)

4.在server目录下运行:
	
`python uploader.zip`

5.chrome请安装SwitchySharp插件，然后导入这个设置:
	
`http://goagent.googlecode.com/files/SwitchyOptions.bak` 


**然后,自带的那个uploader.zip无法完成上传.**

**对于这个问题，解决办法如下:**

1.下载[Google App Engine SDK for Python](https://code.google.com/appengine/downloads.html)

2.下载一个zip包，将zip包放进goagent路径下，解压得到一个google_appengine目录。

3.修改AppID。修改`goagent/server/python/app.yaml`文件中的appid

4.在goagent路径下，终端内输入:

`python google_appengine/appcfg.py update server/python/`

5.然后按照提示进行下去。

6.chrome访问很多https网站弹出证书警示，需要将local/CA.crt证书导入:

`设置->HTTPS/SSL管理证书->授权中心->导入->选择local下到CA.crt->打上3个勾->重启chrome` 