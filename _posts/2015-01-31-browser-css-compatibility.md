---
layout: post
title: "浏览器CSS兼容性案例"
description: ""
category: 
tags: []
---

###1. 标签使用不正确
现象代码如下：  

```html
<ul>
	<li>aaa</li>
	<span>bbb</span>
</ul>
```

Chrome、IE标准模式 展示如下：
![picture in chrome](/img/li-bug-chrome.jpg)

IE兼容模式展示如下：
![picture in ie](/img/li-bug-ie.jpg)

原因：标准中ul的子元素只能是li，将span放于ul内属于非规范的做法  
结论：请严格按照html的规范使用标签，在ol、ul下的子标签只使用li

###2. Zindex兼容性问题
现象代码如下  


```html
<!doctype html>
<html>
	<head>
	   <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
	   <style>
		   #container {
				position: relative;
		   }

		   #box1 {
				position: absolute;
				top: 100px;
				left: 210px;
				width: 200px;
				height: 200px;
				background-color: yellow;
				z-index: 20;
		   }

		   #box2 {
				position: absolute;
				top: 50px;
				left: 160px;
				width: 200px;
				height: 200px;
				background-color: green;
				z-index: 10;
		   }
	   </style>
	</head>
	<body>
	   <div id="container">
		   <div id="box1">这个box应该在上面</div>
	   </div>

	   <div id="box2">
		   这个box应该在下面，IE浏览器会对定位元素产生一个新的stacking context ，甚至当元素 z-index的为“auto”。
	   </div>
	</body>
</html>
```

Chrome、IE标准模式 展示如下：
![picture in chrome](/img/zindex-bug-chrome.jpg)

IE兼容模式展示如下：
![picture in ie](/img/zindex-bug-chrome.jpg)

原因：ie7的z-index有bug！CSS规范 中清楚的规定了除了根元素，只有定位元素的z-index被定义一个非auto的z-index值才能产生新的stacking context。子元素的z-index不会超过父元素的z-index。而IE在定位元素都会产生一个z-index为0的stacking context，导致我们看到的这个问题  
结论：当z-index的元素被包在自定义的某个div中时就得注意一下了，这时候可以在已定位的父节点上也加上相同的z-index来解决

###3. 父子Float问题 
现象代码如下：   


```html
<!doctype html>
<html>
	<head>
	   <meta http-equiv="Content-Type" content="text/html;charset=utf-8" />
	   <style>
		   .iemp_m_btn_a {
				border: 1px solid #4A4A4A;
				float: left;
		   }

		   #confirmBtn {
				float: right;
		   }
	   </style>
	</head>
	<body>
	   <div href="#" class="iemp_m_btn_a">
		   <div id="confirmBtn">过滤</div>
	   </div>
	</body>
</html>
```

Chrome、IE标准模式 展示如下：
![picture in chrome](/img/float-bug-chrome.jpg)

IE兼容模式展示如下：
![picture in ie](/img/float-bug-ie.jpg)

原因：标准中对于脱离文档流的容器，当容器没有设定宽度时，其宽度会尽量小，但是IE兼容模式不遵守这一原则。  
结论：不要滥用float，像本例子中第二个float完全没必要，去掉就正常了  
参考： <http://ipmtea.net/css_ie_firefox/201107/29_519.html>