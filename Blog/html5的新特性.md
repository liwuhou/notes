---
title: html5的新特性
date: 2018-04-05
tags: Html
summary: 一些html5中的新api和特性
---

> 简述API
> API(Application Programming Interface)
> 是一些预先定义的函数，目的是提供应用程序与开发人员基于某软件或硬件得以访问一组例程的能力，而又无需访问源码，或理解内部工作机制的细节。


#### Video/Audio-API
**video对象的方法**
 
| 方法名         | 效果                                    |
| -------------- | --------------------------------------- |
| load()         | 加载其他视频（如果有的话）              |
| play()         | 视频开始播放                            |
| pause()        | 视频暂停                                |
| addTextTrack() | 向音频/视频添加新的文本轨道             |
| canPlayType()  | 检测浏览器是否能播放指定的音频/视频资源 |

<!-- more -->


**video对象的属性**

| 属性                | 描述                                                       |
| ------------------- | ---------------------------------------------------------- |
| autoplay            | 设置或返回是否在加载完成后马上播放音频/视频                |
| controls            | 设置或返回音频/视频是否显示控件                            |
| loop                | 设置或返回音频/视频是否应在结束时重新播放                  |
| muted               | 设置或返回音频/视频是否静音                                |
| paused              | 设置或返回音频/视频是否暂停                                |
| src                 | 设置或返回音频/视频元素的当前来源                          |
| volume              | 设置或返回音频/视频的音量                                  |
| audioTracks         | 返回表示可用音轨的AudioTrackList对象                       |
| buffered            | 返回表示音频/视频已缓冲部分的TimeRanges对象                |
| controller          | 返回表示音频/视频当前媒体控制器的MediaController对象       |
| corssOrigin         | 设置或返回音频/视频的CORS设置                              |
| currentSrc          | 返回当前音频/视频的URL                                     |
| currentTIme         | 设置或返回音频/视频中当前播放位置（以秒计）                |
| defaultMuted        | 设置或返回音频/视频默认是否静音                            |
| defaultPlaybackRate | 设置或返回音频/视频的默认播放速度                          |
| duration            | 返回当前音频/视频的长度（以秒计）                          |
| ended               | 返回音频/视频的播放是否已结束                              |
| error               | 返回表示音频/视频错误状态的MediaErro让对象                 |
| mediaGroup          | 设置或返回音频/视频所述的组合（用于连接多个音频/视频元素） |
| networkState        | 返回音频/视频的当前网络状态                                |
| playBackRate        | 设置或返回音频/视频播放的速度                              |
| played              | 返回表示音频/视频一播放部分的TimeRanges对象                |
| preload             | 设置或返回音频/视频是否应该在页面加载后进行加载            |
| readyState          | 返回音频/视频当前的就绪状态                                |
| seekable            | 返回表示音频/视频可寻址部分的TimeRabges对象                |
| seeking             | 返回用户是否正在音频/视频中进行查找                        |
| startDate           | 返回表示当前时间偏移的Date对象                             |
| textTracks          | 返回表示可用文本轨道的TextTrackList对象                    |
| VideoTracks         | 返回表示可用视频轨道的videoTrackList对象                   |

**video对象的事件**

<!-- 来自http://www.w3school.com.cn/tags/html_ref_audio_video_dom.asp -->
<table><thead><tr><th style="width:30%;">事件</th><th>描述</th></tr></thead><tbody><tr><td>abort</td><td>当音频/视频的加载已放弃时</td></tr><tr><td>canplay</td><td>当浏览器可以播放音频/视频时</td></tr><tr><td>canplaythrough</td><td>当浏览器可在不因缓冲而停顿的情况下进行播放时</td></tr><tr><td>durationchange</td><td>当音频/视频的时长已更改时</td></tr><tr><td>emptied</td><td>当目前的播放列表为空时</td></tr><tr><td>ended</td><td>当目前的播放列表已结束时</td></tr><tr><td>error</td><td>当在音频/视频加载期间发生错误时</td></tr><tr><td>loadeddata</td><td>当浏览器已加载音频/视频的当前帧时</td></tr><tr><td>loadedmetadata</td><td>当浏览器已加载音频/视频的元数据时</td></tr><tr><td>loadstart</td><td>当浏览器开始查找音频/视频时</td></tr><tr><td>pause</td><td>当音频/视频已暂停时</td></tr><tr><td>play</td><td>当音频/视频已开始或不再暂停时</td></tr><tr><td>playing</td><td>当音频/视频在已因缓冲而暂停或停止后已就绪时</td></tr><tr><td>progress</td><td>当浏览器正在下载音频/视频时</td></tr><tr><td>ratechange</td><td>当音频/视频的播放速度已更改时</td></tr><tr><td>seeked</td><td>当用户已移动/跳跃到音频/视频中的新位置时</td></tr><tr><td>seeking</td><td>当用户开始移动/跳跃到音频/视频中的新位置时</td></tr><tr><td>stalled</td><td>当浏览器尝试获取媒体数据，但数据不可用时</td></tr><tr><td>suspend</td><td>当浏览器刻意不获取媒体数据时</td></tr><tr><td>timeupdate</td><td>当目前的播放位置已更改时</td></tr><tr><td>volumechange</td><td>当音量已更改时</td></tr><tr><td>waiting</td><td>当视频由于需要缓冲下一帧而停止</td></tr><tbody></table>


#### Canvas-API
- canvas元素用于在网页上绘制图形
- canvas元素使用JavaScript在网页上绘制图像
- 画布是一个矩形区域，你可以控制其每一像素

#### canvas绘图之线条及线条属性
> 创建canvas

首先创建一个canvas元素，只需要在html文件中加入一句代码

```html
<canvas id="canvas"></canvas>
```
可通过canvas的标签属性width和height设置canvas画布的大小：
```html
<canvas id="canvas" width="800" height-"600">
```

> 获取环境

通过js中获取到该canvas元素，然后设置它的宽高，并获取到上下文的环境：

```JavaScript
var canvas = document.getElementById("canvas");
var context = canvas.getContext("2d");//获取上下文环境
```
接下来我们的所有操作都是基于这个context的上下文环境。

> 试着画一条简单的直线

```JavaScript
context.moveTo(100,100);
comtext.lineTo(500,500);
```

- moveTo()方法表示一次绘制的起点坐标
- lineTo()表示基于上一个坐标点到这个坐标点之间的直线连接

**注意：canvas是基于状态的绘制，而不是基于对象的绘制。所以上面的代码都是状态的设置，还需要使用`stroke()`方法进行绘制：

```JavaScript
context.stroke();//绘制
```
除此之外，我们还可以设置线条的一些基本属性：

```JavaScript
context.lineWidth = 8; //线条的宽度
context.strokeStyle = "#333"; //线条的颜色
```
一个简单简单的绘制一条直线的完整例子：

```JavaScript
var canvas = document.getElementById("canvas");
var context = canvas.getContext("2d");

context.moveTo(100,100);
context.lineTo(200,200);

context.lineWidth = 5;
context.strokeStyle = "red";

context.stroke();
```
> 结果如下：

![](http://cdn.liwuhou.cn/blog/20200306224154.png)

> 绘制一个连续的折线

```JavaScript
context.moveTo(100,100);
context.lineTo(600,100);
context.lineTo(600,600);
context.lineTo(100,600);

context.stroke();
```
> 结果如下：

![](http://cdn.liwuhou.cn/blog/20200306224214.png)

上面简单的例我们使连续绘制的折线，也就是说可以一笔连续画完的折线。如果是条多间断的折线，那么就需要多次使用context.moveTo()来重新绘制一条折线的起点坐标：

```JavaScript
context.moveTo(100,200);
context.lineTo(300,400);
context.lineTo(100,600);

context.moveTo(300,200);
context.lineTo(500,400);
context.lineTo(300,600);

context.moveTo(500,200);
context.lineTo(700,400);
context.lineTo(500,600);

context.stroke();
```
> 结果如下：

![](http://cdn.liwuhou.cn/blog/20200306224234.png)


关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
