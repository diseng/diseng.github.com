---
layout: post
title : 修改Fuubo微博尾巴
category : app
tags : [app]
---
{% include JB/setup %}

昨天有朋友在微博上推荐试用Fuubo,今天安装使用的时候,进入画面可以选择不同的微博尾巴,我就想要是能改成自己的尾巴多好啊.于是谷歌搜了一下,还真有人做这事了,[见这里](http://sypopo.com/pcnet/2518.html),但是这个教程把反编译,编译,签名等内容一笔省略了,对于第一次修改APK文件的人来说,还是需要再查些别的内容,我这里把这次涉及到的内容都记录下来.

### 1.所需工具

    1.android-apktool 下载地址:https://code.google.com/p/android-apktool/
    2.Auto-sign 下载地址:http://www.lt163.com/linux/pc/pc/201008/12094.html

### 2.所需环境

    Java

这个就不说了,现在的JDK安装完后都不需要设置环境变量了

### 3.反编译

解压缩android-apktool,在CMD下进入解压缩后的目录,运行:

    apktool d[ecode] [OPTS] <file.apk> [<dir>]

其中d是decode的简写,OPTS是一些反编译选项,file.apk是反编译的apk文件,dir是反编译出的内容的存放路径,参考示例:

    apktool d E:\apk\Fuubo.apk E:\apk\src

### 4.修改文件

进入反编译文件目录(如上面示例中的E:\apk\src),修改\smali\me\imid\fuubo\ui\LoginActivity.smali这个文件中的一组或多组微博尾巴信息(其实就是App Key和App Secret)

一组微博尾巴信息如下:

    invoke-direct {p0}, Lme/imid/fuubo/ui/base/BaseActivity;-><init>()V

    const/4 v0, 0x3

    new-array v0, v0, [Lme/imid/fuubo/types/ApiKey;

    const/4 v1, 0x0

    new-instance v2, Lme/imid/fuubo/types/ApiKey;

    const-string v3, "Fuubo"

    const-string v4, "1182402349"

    const-string v5, "368aa54583a5b37d900ac5f9703df9d1"

只需修改其中的const-string v3,const-string v4和const-string v5为自己的应用名,App Key和App Secret即可.

如果你的应用的授权回调页面不是“https://api.weibo.com/oauth2/default.html”,那么还需修改`\smali\me\imid\fuubo\ui\OauthActivity.smali`和`\smali\*.smali`中的某个文件(我也不清楚是哪个,原文章作者说每次反编译文件名都不同),将其中的“https://api.weibo.com/oauth2/default.html”修改为你的App授权回调页面.

当然你也可以将自己app的授权回调页面设置为“https://api.weibo.com/oauth2/default.html”,就不需要修改那两个文件了.

### 5.构建

修改完了后,我们需要将文件重新编译成apk文件,运行:

    apktool b[uild] [OPTS] [<app_path>] [<out_file>]

其中b是build的简写,OPTS是一些构建选项,`<app_path>`为需要构建的目录,`<out_file>`为构建生成的文件,参考示例:

    apktool b E:\apk\src E:\apk\build.apk

这样就得到了修改后的APK文件

### 6.签名

将上一步得到的APK文件直接放入手机进行安装,会失败,因此我们要需要签名,为什么APK文件需要签名可以[见这里](http://blog.csdn.net/lyq8479/article/details/6401093).

签了名之后,安装成功,但是运行的时候,每次完成授权,程序就自动退出,原文章作者的解决办法是:打开改好的APK包，将包里的classes.dex 替换到原版的包里。然后签名，安装！

下面说一下如何签名:

    1.解压缩Auto-sign,得到文件sign_pack.bat和文件夹_Data
    2.对步骤5中得到的APK文件和原版APK文件分别解压,用修改后的APK解压包中的classes.dex替换原版APK解压包中的classes.dex.
    3.将修改后的原版APK解压包拖放到Auto-sign文件夹中的sign_pack.bat文件上,完成签名(这个操作法大亮啊)
    4.签名后会生成一个XXX_www.ophone8.com的目录,目录中就是签了名的APK文件

将签了名的APK文件放入手机进行安装吧.


ps:我的app是没有读取用户分组的权限的,但是在使用了自带的Fuubo后(有读取用户分组的权限),再切回自己的app,发现可以使用分组功能了(虽然程序会提示读取分组失败,但是Fuubo使用后的缓存依然起作用),不知道这算不算一个bug.