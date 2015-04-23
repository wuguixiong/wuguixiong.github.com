---
layout: post
title: "xss防御"
description: "xss的防御"
category: 
tags: ["安全", "WEB", "xss"]
---

##1. HttpOnly
前面讲到攻击者可以使用xss获取用户的cookie，从而劫持用户权限。那么有没有办法做到，即使攻击者做到了xss攻击，也不让他拿到我们的cookie呢？答案就是HttpOnly!
服务器通过http请求头往前台写cookie时，可以带上HttpOnly的属性，如

![google cookie](/img/2015-04-23_220811.png)

访问http://www.google.com 可以看到google往客户端写了两个cookie，一个使用了HttpOnly。
HttpOnly表示该cookie只能随着http请求提交到后台，不能通过js获取到。我们还是用js获取刚才访问的google页面的cookie

可以看到，我们只拿到了没有带HttpOnly的那个cookie。
所以，往前台写session信息的cookie一定要带上HttpOnly，这样即使页面上有xss漏洞也可以避免cookie被轻易拿到。

##2. 输入检查
对于大多数的业务输入，都是可以找出一定的规则限制。比如：用户名只允许使用字母、数字和下划线；不允许使用`< > “ ‘ &` 作为业务数据；字符长度限制等。
这些检查能从输入入口限制可输入的字符，可以提高xss的攻击难度，对于大多数场景可以解决业务数据造成的xss攻击。
但是，还是像前面说的，xss防御没有银弹，只能根据代码场景具体分析才能真正杜绝。不能指望仅仅通过输入检查就解决了xss攻击，有一堆xss攻击技巧可以绕过输入检查，实现攻击效果。

##3. 转码
对于xss攻击，大多数人知道可以用htmlEncode来做数据转码，防止html注入。但是单单靠htmlEncode是远远不够的~~

###3.1 htmlEncode

	<div>${untrusted}</div>

对于html内容，只要保证数据不被当成html标签解析就可以了，所以使用htmlEncode可以轻松搞定。
ESAPI的htmlEncode会对以下字符做转码

	& --> &amp;
	< --> &lt;
	> --> &gt;
	" --> &quot;
	' --> &#x27;     //不推荐用'因为它不是HTML的规范而是XML和XHTML的规范
	/ --> &#x2F;     //会帮助关闭标签，因此也做转码


###3.2 attributeEncode
	
	//注意：attr 不能是onclick等事件，否则用attributeEncode也无法防御

	<div attr="${untrusted}"></div> 
	<div attr='${untrusted}'></div>
	<div attr=${untrusted}></div>   

可以发现对于前两种使用htmlEncode可以保证安全，因为转码后的数据无法突破引号的限制，只能成为属性的值。
但是第三种写法就奇葩了，htmlEncode后只要构造payload: xxx onmouseover=alert(1)
就可以攻击成功了。该payload不包含htmlEncode转码中的任何字符，但是利用空格和等号就可以成功注入新属性，从而达成攻击。
ESAPI对这种场景做了非常严格的转码，除了字母数字外的其他字符统统转码成&#xHH格式，可见最后一种写法是非常危险的，应该在编程规范中约束以直接杜绝！！

###3.3 jsEncode
	<script>
		var data0 = "${untrusted}"; // 场景1
		var data1 = '${untrusted}'; // 场景2
		var data2 = ${untrusted};   // 场景3
	</script>
	<div onclick="fn('${untrusted}')"></div>  // 场景4
	<div onclick="fn(${untrusted})"></div>  // 场景5

经常可以看到，很多人对于这种场景也是简单通过htmlEncode来转码，这时会带来几个问题。
对于第1、2、4中场景，因为`" '`都在htmlEncode转码范围内，所以不会出现xss攻击，但是可能导致业务数据错误，比如，数据中包含`<`，被转码后变成`&amps;`，数据已经发生了变化，可能会导致数据搜索、更新等功能出问题~     

对于第3、5场景，实际上还是有xss攻击的，随意上payload: alert(1) 就攻击成功了~~     

所以这种场景是不应该用htmlEncode的。    

ESAPI对这种场景除了字母和数字以外的字符统统转码成\xHH格式。
需要注意的是，对于4、5场景，即使ESAPI的jsEncode也无法真正解决问题。因为如果数据包含特殊字符被转码后是没办法在script中正常执行的。But，场景4、5完全是没必要的，如果出现了这种情况，建议还是审视一下系统前后台的数据交互设计是否合理了。     

*注意：* 以下的场景无法防御
	
	eval("${untrusted}");   
	eval('${untrusted}');  
	eval(${untrusted});  
	new Function("${untrusted}");
	new Function('${untrusted}');
	new Function(${untrusted});

###3.4 cssEncode

	selector {property: "${untrusted}"} 
	selector {property: '${untrusted}'} 
	selector {property: ${untrusted}}

很多css属性可以造成xss注入，如：
	
	{ background-url : "javascript:alert(1)"; } // 所有包含url的都可以这样注入
	{ text-size: "expression(alert('XSS'))"; } // 只支持IE

对于这种场景，ESAPI会将字母数字以外的字符全部转码为\HH的格式   

参考资料：  

[https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet](https://www.owasp.org/index.php/XSS_(Cross_Site_Scripting)_Prevention_Cheat_Sheet)