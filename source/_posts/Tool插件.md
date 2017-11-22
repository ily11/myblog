---
title: Linux下安装eclipse以及配置spring-tool-suite插件
date: 2017-09-29 11:38:38
tags: eclipse spring
categories: 软件安装
---
### Eclipse安装
首先要安装JDK，网上的教程很多，我这里就不赘述了，大体步骤就是先解压然后配置环境变量、生效。下面详细说明eclipse的安装。

1、首先在[官网](http://www.eclipse.org/downloads/packages/eclipse-ide-java-ee-developers/oxygen1)下载相应版本的eclipse： eclipse-jee-oxygen-1-linux-gtk-x86_64.tar.gz

2、执行命令：
```
sudo tar zxvf eclipse-jee-oxygen-1-linux-gtk-x86_64.tar.gz -C /opt/jvm
```
将文件解压到/opt/jvm/下

3、创建eclipse桌面快捷方式图标。
```
cd 桌面
sudo touch eclipse.desktop
sudo gedit eclipse.desktop
```
然后在文本编辑器里输入以下内容：
```
[Desktop Entry]
Name=Eclipse 4
Type=Application
Exec=/opt/eclipse/eclipse
Terminal=false
Icon=/opt/eclipse/icon.xpm
Comment=Integrated Development Environment
NoDisplay=false
Categories=Development;IDE;
Name[en]=Eclipse
```
保存。

执行:sudo chmod u+x eclipse.desktop 将其变为可执行文件.

4、在桌面打开eclipse，会提示未信任的应用程序启动器。此时的解决方案是：
打开终端，输入`sudo nautilus`，nautilus这个命令是用于以root权限打开文件管理窗口。在打开的文件管理窗口找到桌面的“eclipse.desktop”，然后右击【属性】，勾选“允许作为程序执行文件”，如图所示：
![](/images/eclipse权限.png)
然后再点击文件即可运行。

### Eclipse上安装spring-tool-suite
spring tool suite 是一个基于eclipseIDE开发环境中的用于开发spring应用程序的工具。

1、首先查看自己安装的eclipse版本，打开eclipse->Help->about eclipse，可以看到我的是4.7.1：
![](/images/eclipse版本.png)
2、登录https://spring.io/tools/sts/all，找到相应版本的spring tool suite并复制地址:
![](/images/spring版本.png)
3、Help-->Install New Software-->work with 中输入	http://dist.springsource.com/release/TOOLS/update/e4.7/ ,回车等待片刻，然后按照下图选择选项，去掉自动更新，然后选择next,
![](/images/springAdd.png)
然后一路next，出现接受协议的选择接受。

4、安装完成后会提示重启，重启后，Spring IDE出现在欢迎界面：
![](/images/springSuccess.png)
安装成功。
