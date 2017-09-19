---
title: Canvas 学习之绘制图形
date: 2017-03-22 11:19:31
tags: "canvas"
categories: "HTML"
---
最近在学习《HTML5 Canvas游戏开发实战》这本书，现整理一下，以便后面翻出来复习。
### 基本图形
#### 圆角矩形
先贴出代码和运行效果
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.beginPath();
    ctx.moveTo(40,20);
    ctx.lineTo(100,20);
    ctx.arcTo(120,20,120,40,20);
    ctx.lineTo(120,70);
    ctx.arcTo(120,90,100,90,20);
    ctx.lineTo(40,90);
    ctx.arcTo(20,90,20,70,20);
    ctx.lineTo(20,40);
    ctx.arcTo(20,20,40,20,20);
    ctx.stroke();
</script>
```
运行效果如下图：
![](/images/CirRec.png)
下面来解释一下代码：Canvas里没有直接画圆角矩形的API，但是可以用arcTo函数来完成圆角的绘制，然后结合直线绘制，就可以完成圆角矩形的绘制了。arcTo函数是用来为当前的子路径添加一条圆弧的，它需要5个参数，分别是：点p1的坐标x和y、点p2的坐标x和y、圆弧的半径。该圆弧有一点与当前位置到p1的线段相切，还有一点和从p1到p2的线段相切。这两个切点就是圆弧的起点和终点，圆弧绘制的方向就是连接这两个点的最短圆弧方向。
#### 擦除Canvas画板
擦除Canvas画板上的内容需要用到clearRect函数，此函数可以擦除一个矩形区域。它需要四个参数：起点的坐标x和y，擦除区域的长和宽。代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.fillStyle = "red";
    ctx.beginPath();
    ctx.fillRect(10,10,200,100);
    ctx.clearRect(30,30,50,50);
</script>
```
上述代码是先绘制了一个红色的实心矩形，然后在红色矩形内擦除了一个50*50的小正方形，运行效果如下图所示：
![](/images/clearRect.png)
### 绘制复杂图形
#### 画曲线
#####二次贝塞尔曲线
二次贝塞尔曲线有一个控制点。在Canvas中用quadraticCurveTo(cpx,cpy,x,y)函数来绘制二次贝塞尔曲线，cpx、cpy表示控制点的坐标；x、y表示终点坐标。其代码如下：
```
var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.beginPath();
    ctx.moveTo(100,100);
    //绘制二次贝塞尔曲线
    ctx.quadraticCurveTo(20,50,200,20);
    ctx.stroke();
```
效果图如下：
![](/images/二次贝塞尔曲线.png)
##### 利用clip在指定区域绘图
clip函数使用当前路径作为连续绘制操作的剪切区域。我们可以把它看作一扇窗户，无论在画板上绘制了多大的图形，最后看到的图形都只能由clip这个窗户来决定。
代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    //绘制半圆
    ctx.arc(100,100,40,0,180 * Math.PI/180,true);
    ctx.clip();
    ctx.beginPath();
    //设置颜色
    ctx.fillStyle = "lightblue";
    //绘制矩形
    ctx.fillRect(0,0,300,150);
</script>
```
效果图如下：
![](/images/clip.png)
可以看到，虽然我们画了一个矩形，但是最后显示的却是一个半圆。这是因为我们我们首先画了一个半圆，然后clip函数将当前的这个半圈作为绘制操作的区域，所以之后画的图形只能显示在这个区域内。
### 绘制文本
#### 绘制文字
绘制文字有fillText和strokeText两种方法。
1、使用fillText绘制文字
fillText(text,x,y,maxWidth)函数很简单，它有四个参数：文本字符串、坐标x和y，文本宽度，其中第四个参数可以略去。实例代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    //设置文字大小和字体
    ctx.font = "30px Arial";
    //设置文字内容
    ctx.fillText("Hello World",100,50);
</script>
```
运行效果如下图所示：
![](/images/fillText.png)
2、使用strokeText绘制文字
使用strokeText(text,x,y,maxWidth)函数同样需要4个参数，它的用法和fillText函数用法完全相同，具体代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    //设置文字大小和字体
    ctx.font = "30px Arial";
    //设置文字内容
    ctx.strokeText("Hello World",100,50);
</script>
```
效果图如下：
![](/images/strokeText.png)
从效果图上我们可以看出strokeText与fillText的区别是，strokeText相当于是线，而fillText相当于实心图形。
#### 文字设置
1、文字大小与字体
上面的代码中用到了`ctx.font = "30px Arial";`它设置了字体大小为“30px”，字体为“Arial”。
我们可以设定多种不同的字体、大小，代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.beginPath();
    //设置文字大小为30px
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,50);
    ctx.beginPath();
    //设置文字大小为50px
    ctx.font = "50px Arial";
    ctx.fillText("Hello World",50,150);
    ctx.beginPath();
    //设置文字大小为70px
    ctx.font = "70px Arial";
    ctx.fillText("Hello World",50,250);
    ctx.beginPath();
    //设置文字字体为Verdana
    ctx.font = "30px Verdana";
    ctx.fillText("Hello World (Verdana)",50,300);
    ctx.beginPath();
    //设置文字字体为Times New Roman
    ctx.font = "30px Times New Roman";
    ctx.fillText("Hello World (Times New Roman)",50,350);
    ctx.beginPath();
    //设置文字字体为Courier New
    ctx.font = "30px Courier New";
    ctx.fillText("Hello World (Courier New)",50,400);
</script>
```
效果图如下：
![](/images/font.png)
2、文字粗体与斜体
同样可以通过font来设置文字粗体和斜体，如粗体：`ctx.font = 'normal 30px Arial';`font-weight可以是normal、bold、bolder、lighter，还可以通过数字直接设置，如`ctx.font = '300 30px Arial';`。斜体可以这样设置：`ctx.font = 'italic 30px Arial';`。效果图就不一一展示了。
3、文字的对齐方式
Canvas中的文字通过textAlign和textBaseline来实现文字的对齐。textAlign是水平方向的文字对齐，它的值包括center、end、left、right、start。textBaseline是竖直方向的文字对齐，它的值包括alphabetic、bottom、ideographic、middle、top。
首先看水平方向的对齐。为了看出对齐方式之间的区别，我们在文字坐标位置画一条竖线，代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.moveTo(160,0);
    ctx.lineTo(160,300);
    ctx.stroke();
    ctx.beginPath();
    ctx.textAlign = "start";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",160,50);

    ctx.beginPath();
    ctx.textAlign = "end";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",160,100);

    ctx.beginPath();
    ctx.textAlign = "center";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",160,150);

    ctx.beginPath();
    ctx.textAlign = "left";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",160,200);

    ctx.beginPath();
    ctx.textAlign = "right";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",160,250);
</script>
```
效果图如下：
![](/images/textAlign.png)
从效果图上可以看出，start与left相同，表示文字从左侧开始对齐；end与right相同，表示文字从右侧开始对齐，center表示文字居中。
下面看一下竖直方向的对齐，代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    ctx.textBaseline = "alphabetic";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,50);
    ctx.moveTo(0,50);
    ctx.lineTo(250,50);
    ctx.stroke();

    ctx.textBaseline = "bottom";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,100);
    ctx.moveTo(0,100);
    ctx.lineTo(250,100);
    ctx.stroke();

    ctx.textBaseline = "hanging";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,150);
    ctx.moveTo(0,150);
    ctx.lineTo(250,150);
    ctx.stroke();

    ctx.textBaseline = "ideographic";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,200);
    ctx.moveTo(0,200);
    ctx.lineTo(250,200);
    ctx.stroke();

    ctx.textBaseline = "middle";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,250);
    ctx.moveTo(0,250);
    ctx.lineTo(250,250);
    ctx.stroke();

    ctx.textBaseline = "top";
    ctx.font = "30px Arial";
    ctx.fillText("Hello World",50,300);
    ctx.moveTo(0,300);
    ctx.lineTo(250,300);
    ctx.stroke();
</script>
```
效果图如下：
![](/images/textBaseline.png)
### 图片操作
Canvas中提供了drawImage函数和putImageData函数来绘制图片。
#### 利用drawImage绘制图片
drawImage函数有3种函数原型，其语法如下：
drawImage(image,dx,dy);
drawImage(image,dx,dy,dw,dh);
drawImage(image,sx,sy,sw,sh,dx,dy,dw,dh);
第一个参数image是要绘制的对象，这个参数可以是HTMLImageElement、HTMLCanvasElement或者HTMLVideoElement，dx、dy是image在Canvas中定位的坐标值，dw、dh表示image在Canvas中即将绘制的区域（相对dx、dy坐标的偏移量）的宽度和高度值，sx、sy是image所要绘制的起始位置，sw、sh表示image所要绘制区域（相对于image的sx、sy坐标的偏移量）的宽度和高度值。
代码如下：
```
<body>
img标签<br>
<img id="home" src="image/22.png" width="240" height="240"><br>
Canvas画板<br>
<canvas id="myCanvas" width="600" style="border: 1px" height="350">
    您的浏览器不支持Canvas
</canvas>
</body>
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    var image = new Image();
    image.src = "image/22.png";
    image.onload = function () {
        ctx.drawImage(image,10,10);
        ctx.drawImage(image,410,10,100,100);
        ctx.drawImage(image,50,50,100,100,410,130,100,100);
    };
</script>
```
效果图展示：
![](/images/drawImage.png)
从图中可以看出Canvas画板里的右上小图是原图的缩小，而右下小图是左边大图截取的一部分（图中标注部分）。
#### 利用getImageData和putImageData绘制图片
putImageData（imgdata,dx,dy,sx,sy,sw,sh）函数需要7个参数，其中imgdata为像素数据，dx、dy是绘制图片的定位坐标值，sx、sy是imgdata所要绘制图片的起始位置，sw、sh是imgdata所要绘制区域（相对于imgdata的sx和sy坐标的偏移量）的宽度和高度值。这里面第4个参数以及其后的所有参数都可以省略，如果这些参数都省略了，则表示绘制整个imgdata。
在使用putImageData之前，需要先用getImageData(x,y,w,h)函数得到像素数据，这里只得是从Canvas画板上取得所选区域的像素数据，它的四个参数分别是选择区域起点坐标x、y，选择区域的长和宽。
代码如下：
```
<script>
    var c = document.getElementById("myCanvas");
    var ctx = c.getContext("2d");
    var image = new Image();
    image.src = "image/22.png";
    image.onload = function () {
        ctx.drawImage(image,10,10);
        var imgData = ctx.getImageData(50,50,200,200);
        ctx.putImageData(imgData,10,260);
        ctx.putImageData(imgData,200,260,50,50,100,100);
    };
</script>
```
效果图如下：
![](/images/putImage.png)
