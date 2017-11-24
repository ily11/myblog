---
title: IDEA 文件格式问题
date: 2017-11-24 12:09:58
tags: IntelliJ IDEA
categories: 学习
---

今天用IDEA新建文件的时候踩了一个很诡异的坑，师兄给调了好久没有调成功，最后还是百度，感谢百度这个强大的工具。不说废话了,上图：
![](/images/error.png)

我是右键new->file，然后没写后缀，接着IDEA会提示我选择文件类型，
![](/images/createFile.png)

结果我没选择，默认是text格式，然后我直接在text加了个java后缀，然后问题就出现了，我要实现的是接口文件，结果我写完代码后，不能自动识别成接口文件
![](/images/error2.png)

然后各种调、各种删、新建都不行，然后在百度贴吧里找到了解决方案：在settings里找到File Types然后找到Text，选中，下面会出现注册的text模式里就有text.java，
![](/images/fileTypes.png)

就因为IDEA记住了这个，所以每次创建text.java文件的时候就是text格式，现在把它删了就好了，就会自动识别成接口文件。
![](/images/success.png)
