---
layout: post
title: "xss防御"
description: "xss的防御"
category: 
tags: ["安全", "WEB", "xss"]
---

由于前台涉及的编程语言繁多，javascript、css、html、jsp、php、flex、java（插件）等，其中很多语言本身又非常灵活，
比如html的属性值既可以用双引号也可以用单引号，甚至直接不用引号，在浏览器都还是可以正确执行~~~另外前端的需求也是多样化的，数据的使用场景也是纷繁多样。
这些都使得xss攻击变得非常多样化。  

基于以上的原因，我们要明白xss的防御没有银弹。需要根据不同的需求，不同的使用场景具体分析如何防御，才能真正避免xss。

#后台通过动态语言输出html到前台

	<div>${nickname}</div>

这时候只要构造payload: `<script>alert(1)</script>`就可以注入成功  

我们可以发现，之所以被注入成功，是因为数据会被浏览器当成标签执行。那么防御的方法就简单了，只要把数据中会被浏览器解析的字符做转码，让它被解析后展示成原字符就可以了。   

通用的htmlEncode，会将字符`<` `>` `"` `'` `&`  ```转码成 `&lt;`  `&gt;`  `&quot;` `&#x27;` `&amp;` `&#x60;`

那么经过转码后，我们返回到前台的html就变成  `<div>&lt;script&gt;alert(1)&lt;/script&gt;</div>`
