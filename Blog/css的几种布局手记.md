---
title: css的几种布局手记
date: 2017-10-01
tags: Css
summary: 几种开发中，常见的css布局的笔记
---

总结了几种css布局的方法

<!-- more -->

## 三列布局
中间一列永恒居中且高度自适应效果，如图
![三列布局](https://blogs-1257826393.cos.ap-shenzhen-fsi.myqcloud.com/15281227413351.jpg)

总共有三种操作方式

#### 利用子元素缺省宽度自适应的特性
由于块元素在默认情况下宽度会跟随父元素，也就是100%，然而如果该块元素设置了一个左右的margin的话，他的宽度是会受到影响的，这样就可以来实现自适应宽度的居中布局了

```html
<!-- css：-->
.parent{
		width:100%;
		height:40px;
}
.middle{
		/*width:100%;*/
		height:100%;
		margin:0 100px;/*神来之笔*/
}
<!--html：-->
<div class="parent">
		<div class="middle"></div>
	</div>
```

#### 利用父元素的padding
利用父元素的padding，将子元素“挤”向中间

```
<!--css：-->
.parent{
    width:100%;
    height:40px;
}
.form{
    width:100%;
    height:100%;
    box-sizing:border;/*重要*/
    padding-left:50px;
    padding-right:50px;
}
.search-box{
    width:100%;
}
<!--html：-->
<div class="parent">
    <form class="form">
        <input type="search" class="search-box" />
    </form>
</div>
```
#### flex伸缩盒模型
简单粗暴，不过有兼容性问题

## 识别IE浏览器
在开发中经常要对ie浏览器单独做一些兼容，特别是在css这块。
通过给属性值加上`\9`，可以让该属性值只作用于IE浏览器

```css
.body{
    height:500px\9;
    height:600px;
}
/* ie浏览器body为500px */
```

## 在高度百分比失效的情况下，设置出百分比长宽的四方形
根据父盒子的大小/设备屏幕的宽高，设置出一个长宽相等的四边形，在一些具体的界面的圆形按钮制作中很常见，特别是移动设备；
但是css在父盒子高度值不确定的条件下（具体就是父盒子没有设置确定的高），子盒子的高度设置百分比会失效，也就是恢复默认值auto；这显然不是我们想要的，这里有两种方式，让子盒子找回失去的高度

#### 给父对象一个确定高度
有时候，我们想让一个盒子的高度跟随浏览器页面的高度，所以设置了100%，但是这是无效的；因为他的父元素（body也好html也好）没有设置确定的高度，所以你设置了height:100%是没有用的

```html
<!-- 这里省略了一些标签 -->
<html>
<body>
    <div style="height:100%"></div>
</body>
</html>
```

解决的方案就是，给它的父盒子都添加上100%，如下：

```html
<html style="height:100%">
<body style="height:100%">
    <div style="height:100%"></div>
</body>
</html>
```
#### 给子盒子设置padding-top值
这个就是一个很巧妙的方法，不想上面的方法一样需要改父盒子(们)的高度，把padding-top设置为跟width一样的百分比宽度，将高度挤出来

```
<!-- css -->
.parent{
    width:21%;
    padding-top:21%;
    border-radius:
}
<!-- html -->
<div class="parent"></div>
```
如果，我想要在这个四方形中居中显示内容呢，他的子元素不就受到padding-top的影响了嘛，怎么办呢；

不要慌，可以使用定位结合transform属性将子元素脱离文档流，不受父元素padding的影响，在结合top和translateY将内容垂直居中，再按需设置text-align属性即可

```html
.parent{
    position:relative
    width:21%;
    padding-top:21%;
}
.son{
    position:absolute;
    top:50%;left:0;
    transform:translateY(-50%);
    text-align:center;
}
<!-- html -->
<div class="parent">
    <div class="son">我是要居中显示的内容</div>
</div>
```

关注本人公众号

![](http://cdn.liwuhou.cn/blog/20200306223709.png)
